# Параллелизм в ruby 1: создаем потоки

![image](./001.jpeg)

После 9 лет работы на позиции Ruby-разработчика, я заметил, что нам редко попадаются задачи на параллелизм. К тому же книги на тему "параллелизм в языке N" есть почти про любой язык, но касательно Ruby это ограничивается следующим:

![image](./002.png)

Исторически мы привыкли решать такие задачи на уровне архитектуры. Когда рубист попадает в другую экосистему, то испытывает культурный шок. В связи с этим я хотел бы поставить точки над i в тему тредов и ракторов, как минимум сам для себя; посмотреть какие задачи на параллелизм решаются в других языках, что можно оттуда подчеркнуть и перенести в Ruby. Это будет серия статей, так что наслаждайтесь

## Первый эксперимент

Наша первая тема предсказуема, но это база. Представьте, что мы в мире микросервисов и межсервисной коммуникации. У нас есть 3 сторонних сервиса: first, second и third. Каждый из них с какими-то своими данными и своим временем ответа.

В рамках эксперимента мы может симулировать это при помощи одного контроллера на ruby. И мы будем использовать функцию sleep чтобы симулировать нужное нам время ответа:

```ruby
module Api
  module V1
    module Services
      class ServicesController < ::ApplicationController
        SLEEP_MIN = 0.1
        SLEEP_MAX = 0.2

        def first
          sleep(SLEEP_MIN)
          render json: { success: true, name: __method__ }, status: :ok
        end

        def second
          sleep(SLEEP_MAX)
          render json: { success: true, name: __method__ }, status: :ok
        end

        def third
          sleep(SLEEP_MIN)
          render json: { success: true, name: __method__ }, status: :ok
        end
      end
    end
  end
end
```

Мы можем поднять приложении на 4000 порту с конфигурацией пумы в 1 воркер и 5 тредов. Запустим 1 воркер чтобы процесс использовал всего одно ядро CPU и 5 тредов чтобы ограничить его мощность

```bash
RAILS_ENV=production RAILS_LOG_LEVEL=debug RAILS_MAX_THREADS=5 rails s -p 4000
```

Наши ручки:

- http://127.0.0.1:4000/api/v1/services/first отвечает от 100ms
- http://127.0.0.1:4000/api/v1/services/second отвечает от 200ms
- http://127.0.0.1:4000/api/v1/services/third отвечает от 100ms

```bash
curl http://127.0.0.1:4000/api/v1/services/first
{"success":true,"name":"first"}

curl http://127.0.0.1:4000/api/v1/services/second
{"success":true,"name":"second"}

curl http://127.0.0.1:4000/api/v1/services/third
{"success":true,"name":"third"}
```

Получается что `first` и `third` могут держать 10 запросов на секунду * 5 тредов = до 50 rps

```bash
wrk -t1 -c5 -d30s --latency http://127.0.0.1:4000/api/v1/services/first
Running 30s test @ http://127.0.0.1:4000/api/v1/services/first
  1 threads and 5 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   109.54ms    2.82ms 119.14ms   68.27%
    Req/Sec    46.00      9.36    50.00     90.75%
  Latency Distribution
     50%  109.44ms
     75%  111.35ms
     90%  113.28ms
     99%  116.45ms
  1371 requests in 30.06s, 634.62KB read
Requests/sec:     45.62
Transfer/sec:     21.12KB

$ wrk -t1 -c5 -d30s --latency http://127.0.0.1:4000/api/v1/services/third
Running 30s test @ http://127.0.0.1:4000/api/v1/services/third
  1 threads and 5 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   109.63ms    3.58ms 125.36ms   69.28%
    Req/Sec    45.82      9.03    50.00     88.74%
  Latency Distribution
     50%  109.01ms
     75%  111.87ms
     90%  114.48ms
     99%  120.07ms
  1367 requests in 30.06s, 632.77KB read
Requests/sec:     45.48
Transfer/sec:     21.05KB
```

`second` может держать 5 запросов на секунду * 5 тредов = до 25 rps

```bash
wrk -t1 -c5 -d30s --latency http://127.0.0.1:4000/api/v1/services/second
Running 30s test @ http://127.0.0.1:4000/api/v1/services/second
  1 threads and 5 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   210.98ms    3.92ms 235.80ms   75.49%
    Req/Sec    23.17      5.44    40.00     78.70%
  Latency Distribution
     50%  210.54ms
     75%  212.86ms
     90%  215.47ms
     99%  226.10ms
  710 requests in 30.08s, 329.35KB read
Requests/sec:     23.60
Transfer/sec:     10.95KB
```

## Приложение агрегатор

Теперь задача следующая: нам нужно написать сервис-агрегатор, которое обращается ко всем трем ручкам и собирает результат. Так как у нас мир микровервисов, то это нормальная ситуация что мы стучимся в 3 разные ручки


```ruby
module Api
  module V1
    class BaseController < ::ApplicationController
      def test1
        first = JSON.parse(Faraday.get('http://localhost:4000/api/v1/services/first').body)
        second = JSON.parse(Faraday.get('http://localhost:4000/api/v1/services/second').body)
        third = JSON.parse(Faraday.get('http://localhost:4000/api/v1/services/third').body)

        render json: { success: true, first:, second:, third: }, status: :ok
      end
    end
  end
end
```

Запускаем агрегатор с одним тредом чтобы показать нагрузку с одного потока

```bash
RAILS_ENV=production RAILS_LOG_LEVEL=debug RAILS_MAX_THREADS=1 bin/rails s -P aggregator
```

Видим, что ручка агрегирует результат

```bash
$ curl  http://localhost:3000/api/v1/test1
{"success":true,"first":{"success":true,"name":"first"},"second":{"success":true,"name":"second"},"third":{"success":true,"name":"third"}}%
```

Рассчитаем кол-во rps - 100+200+100 = 400ms * 1 тред = до 2.5 rps

```bash
$ wrk -t1 -c1 -d20s --latency http://localhost:3000/api/v1/test1
Running 20s test @ http://localhost:3000/api/v1/test1
  1 threads and 1 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   441.46ms    4.11ms 450.74ms   68.89%
    Req/Sec     1.62      0.49     2.00     62.22%
  Latency Distribution
     50%  441.04ms
     75%  444.15ms
     90%  447.51ms
     99%  450.74ms
  45 requests in 20.09s, 25.58KB read
Requests/sec:      2.24
Transfer/sec:      1.27KB
```

## Добавляем thread

Отправлять так запросы в один поток - плохая идея. Агрегатор тратит время на I/O. Лучше распараллелить. В Ruby для этого есть треды, будем их использовать

```ruby
URLS = {
  a1:  'http://localhost:4000/api/v1/services/first',
  a2: 'http://localhost:4000/api/v1/services/second',
  a3:  'http://localhost:4000/api/v1/services/third',
}

def test2
  threads = URLS.transform_values do |url|
    Thread.new { JSON.parse(Faraday.get(url).body) }
  end
  results = threads.transform_values(&:value)

  render json: { success: true, **results }, status: :ok
end
```

Проверяем, что ручка работает и дает тот же результат

```bash
$ curl  http://localhost:3000/api/v1/test2
{"success":true,"a1":{"success":true,"name":"first"},"a2":{"success":true,"name":"second"},"a3":{"success":true,"name":"third"}}
```

Рассчитаем кол-во rps. Можно сказать, что мы упираемся в самую долгую ручку second, которая отвечает по 200 ms. Значит, у нас будет до 5 rps

```bash
$ wrk -t1 -c1 -d20s --latency http://localhost:3000/api/v1/test2
Running 20s test @ http://localhost:3000/api/v1/test2
  1 threads and 1 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   222.53ms    5.31ms 244.38ms   77.78%
    Req/Sec     3.84      0.50     5.00     73.33%
  Latency Distribution
     50%  221.15ms
     75%  224.82ms
     90%  231.25ms
     99%  244.38ms
  90 requests in 20.08s, 50.27KB read
Requests/sec:      4.48
Transfer/sec:      2.50KB
```

Дальше мы можем масштабироватся по кол-ву воркеров. Поставим 3 воркера

```bash
RAILS_ENV=production RAILS_LOG_LEVEL=debug RAILS_MAX_THREADS=3 bin/rails s -P aggregator
```

Рассчитаем кол-во rps. 200 ms (самая долгая ручка) *3 = в теории до 15 rps

```bash
# ничего не даст тк один поток. 2 коннекшена просто ждут
$ wrk -t1 -c1 -d20s --latency http://localhost:3000/api/v1/test2
Running 20s test @ http://localhost:3000/api/v1/test2
  1 threads and 1 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   220.56ms    7.56ms 248.08ms   73.63%
    Req/Sec     3.88      0.49     5.00     74.73%
  Latency Distribution
     50%  219.16ms
     75%  224.84ms
     90%  230.87ms
     99%  248.08ms
  91 requests in 20.08s, 50.83KB read
Requests/sec:      4.53
Transfer/sec:      2.53KB

# увеличиваем кол-во коннекшенов по кол-ву пумовских тредов
$ wrk -t1 -c3 -d20s --latency http://localhost:3000/api/v1/test2
Running 20s test @ http://localhost:3000/api/v1/test2
  1 threads and 3 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   261.16ms   49.19ms 335.77ms   63.32%
    Req/Sec    11.49      4.42    20.00     73.96%
  Latency Distribution
     50%  223.98ms
     75%  317.08ms
     90%  323.51ms
     99%  329.81ms
  229 requests in 20.08s, 127.92KB read
Requests/sec:     11.41
Transfer/sec:      6.37KB
```

Данных механизм позволяет не тратить время на ожидание I/O и, пока один thread что-то ждет, ruby переключается на другой thread. Они хорошо подходят под кейсы, когда у вас есть независимые друг от друга запросы.

Thread можно использовать и в запросах в базу. Например, вы пишите CMS и у вас есть ручка, которая отдает статьи по фильтру и выводит кол-во всего статей по данному фильтру, чтобы фронт отрисовал постраничную навигацию. В контроллере обычно это выглядит так

```ruby
def index
  articles = Article.where(filtered_params).limit(limit).offset(offset)
  count = Article.where(filtered_params).count

  render json: { articles:, count: }
end
```

Так как это два независимых друг от друга запроса, то при условии что они медленные, нам может помочь thread:

```ruby
def index
  results = {}
  threads = []
  threads << Thread.new do
    results[:articles] = Article.where(filtered_params).limit(limit).offset(offset)
  end
  threads << Thread.new do
    results[:count] = Article.where(filtered_params).count
  end
  threads.map(&:value)

  render json: results
end
```

## Итоги

Итак, мы научились:

- Высчитывать rps. Поняли что это упирается в математику latency × concurrency
- Отправлять запросы параллельно


И мы поняли, что для параллелизации в Ruby есть thread. Они являются хорошим инструментов если ваш код где-то ждет и тратит время на I/O. Распараллелить http-запросы, запросы в базу или запросы к OS - типичные примеры где могут пригодится threads


[DEV.TO](https://dev.to/kopylov_vlad/parallielizm-v-ruby-1-sozdaiem-potoki-2i83)
