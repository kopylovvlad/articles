# Паттерны на ruby: Цепочка обязанностей

**Описание:** Данный паттерн представляет собой реализацию последовательной цепочки обработчиков для какого либо события/сообщения. Каждый обработчиков решает одну задачу: обработать запрос самостоятельно или передать его дальше по цепочке.


**Пример задачи:** У нас есть веб-приложение. На каждый ответ от сервера должен генерироваться небольшой лог в виде json и создаваться запись в лог-файле. Есть определенная корпоративная политика по обработке таких логов:

* Логи c 2xx статусами храняться в файла production.log;
* Логи c 2xx статусами содержащую информацию об оплате храняться в production_payments.log;
* Логи с 4хх статусами храняться в файле production4xx.log и падают в канал в telegram;
* Логи с 5хх статусами храняться в файле production5xx.log и падают в канал в slack

**Реализация:** Основная логика объекта, для цепочки, будет описана в абстрактном классе <LogHandler>. От него будем наследовать классы-обработчики для каждого условия хранения логов. Далее, созданные классы необходимо объединить в цепочку. Цепочку будем формировать от частного к общему.

```ruby
example_200 = {
  path: '/admin/rubrics',
  time: '2018-03-02 12:10:34 +0300',
  http_status: '200',
  http_method: 'POST',
  parameters: {"rubric"=>{"title"=>"111", "locator"=>"111"}},
  processing: {
    by: 'Admin::RubricsController#create',
    as: 'JS'
  },
  current_user_id: '123',
  app_environment: 'production'
}

example_payment = {
  path: '/users/23/orders/65/pay',
  time: '2018-03-02 12:10:34 +0300',
  http_status: '200',
  http_method: 'POST',
  parameters: {},
  processing: {
    by: 'PayController#create',
    as: 'HTML'
  },
  current_user_id: '23',
  app_environment: 'production'
}

example_404 = {
  path: '/rubrics/9934',
  time: '2018-03-02 13:10:34 +0300',
  http_status: '404',
  http_method: 'GET',
  parameters: {},
  processing: {
    by: 'RubricsController#show',
    as: 'HTML'
  },
  current_user_id: nil,
  app_environment: 'production'
}

example_500 = {
  path: '/admin/rubrics/9934',
  time: '2018-03-02 13:10:34 +0300',
  http_status: '500',
  http_method: 'PATCH',
  parameters: {"rubric"=>{"title"=>"", "locator"=>""}},
  processing: {
    by: 'Admin::RubricsController#update',
    as: 'HTML'
  },
  current_user_id: '101',
  error: {
    message: 'Neo4j::ActiveNode::Persistence::RecordInvalidError: Title can\'t be blank, Locator can\'t be blank',
    backtrace: "from ~/.rvm/gems/ruby-2.4.0@my_app/gems/neo4j-9.0.7/lib/neo4j/active_node/persistence.rb:50:in `save!'...."
  },
  app_environment: 'production'
}
```

```ruby
class LogHandler
  attr_reader :successor

  def initialize(successor = nil)
    @successor = successor
  end

  def process(log_item)
    if accept(log_item)
      return true
    elsif @successor
      @successor.process(log_item)
    else
      fail(log_item)
    end
  end

  private

  def fail(log_item)
    msg = "The log-item '#{log_item}' could not be handled."
    save_to_file(msg, 'LogHandler_erros.log')
  end

  def accept(log_item)
    raise '#accept_request method must be implemented.'
  end

  def save_to_file(log_item, file_name)
    File.open(file_name, 'a') do |f|
      f.write(log_item)
      f.write("\n")
    end
  end
end

class StandartLogHandler < LogHandler
  def accept(log_item)

    if log_item[:http_status] =~ /^2\d\d$/
      puts 'StandartLogHandler\'s accept'
      save_to_file(log_item, 'production.log')
      return true
    else
      return false
    end
  end
end

class PayLogHandler < LogHandler
  def accept(log_item)
    if valid_item?(log_item)
      puts 'PayLogHandler\'s accept'
      save_to_file(log_item, 'production_payments.log')
      return true
    else
      return false
    end
  end

  private

  def valid_item?(log_item)
    (log_item[:http_status] =~ /^2\d\d$/ &&
      log_item[:http_method]=='POST' &&
      log_item[:processing][:by]=='PayController#create')
  end
end

class ClientErrorLogHandler < LogHandler
  def accept(log_item)
    if log_item[:http_status] =~ /^4\d\d$/
      puts 'ClientErrorLogHandler\'s accept'
      save_to_file(log_item, 'production4xx.log')
      send_to_telegram(log_item)
      return true
    else
      return false
    end
  end

  private

  def send_to_telegram(log_item)
    # ::TelegramSender.notify_about4xx(log_item)
  end
end

class ServerErrorLogHandler < LogHandler
  def accept(log_item)
    if log_item[:http_status] =~ /^5\d\d$/
      puts 'ServerErrorLogHandler\'s accept'
      save_to_file(log_item, 'production5xx.log')
      send_to_slack(log_item)
      return true
    else
      return false
    end
  end

  private

  def send_to_slack(log_item)
    # ::SlackSender.notify_about5xx(log_item)
  end
end


chain_of_responsibility = ServerErrorLogHandler.new(
  ClientErrorLogHandler.new(
    PayLogHandler.new(
      StandartLogHandler.new
    )
  )
)


chain_of_responsibility.process(example_200)
# StandartLogHandler's accept

chain_of_responsibility.process(example_payment)
# PayLogHandler's accept

chain_of_responsibility.process(example_404)
# ClientErrorLogHandler's accept

chain_of_responsibility.process(example_500)
# ServerErrorLogHandler's accept
```

[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D1%8B-%D0%BD%D0%B0-ruby-%D1%86%D0%B5%D0%BF%D0%BE%D1%87%D0%BA%D0%B0-%D0%BE%D0%B1%D1%8F%D0%B7%D0%B0%D0%BD%D0%BD%D0%BE%D1%81%D1%82%D0%B5%D0%B9-5ede1b9dc5b3)
