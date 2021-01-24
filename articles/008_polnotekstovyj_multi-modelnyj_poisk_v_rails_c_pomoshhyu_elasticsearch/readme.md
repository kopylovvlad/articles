# Полнотекстовый мульти-модельный поиск в Rails c помощью ElasticSearch

![image01](image01.png)

В этой статьe я хочу поделится опытом реализации поискового скрипта. Передо мной стояла задача реализовать не просто поисковик по нескольким текстовым полям, а сделать поиск по нескольким моделям с учетом морфологии языка и префиксного анализа. Старшие товарищи порекомендовали использовать для этой задачи ElasticSearch. Такая реализации не будет нагружать основное приложение, а сам ElasticSearch имеет хороший API на все возможные случаи использования и легок в настройке.

Для данной статьи достаточно будет использовать три модели Article, News и BlogPost. Поиск в них будет проходить по двум атрибутам title и description. При этом, важно чтобы в поисковую выдачу попадали только те записи у которых атрибут searching имеет значение true.

## Начинаем работу

Сперва устанавливаем ElasticSearch на свою машину. Если вы используете MacOS то это можно сделать одной командой в терминале.

```bash
brew install elasticsearch
```

Создаем необходимые модели по которым будет проходить поиск. Создаем для каждой из них миграции и запускаем их.

```ruby
class CreateArticles < ActiveRecord::Migration
  def change
    create_table :articles do |t|
      t.string :title
      t.string :description
      t.boolean :searching, default: false
      t.timestamps null: false
    end
  end
end
class CreateNews < ActiveRecord::Migration
  def change
    create_table :news do |t|
      t.string :title
      t.string :description
      t.boolean :searching, default: false
      t.timestamps null: false
    end
  end
end
class CreateBlogPosts < ActiveRecord::Migration
  def change
    create_table :blog_posts do |t|
      t.string :title
      t.string :description
      t.boolean :searching, default: false
      t.timestamps null: false
    end
  end
end
```

Для морфологического анализа слов, есть одна очень хорошая библиотека. Она поддерживает много различных версий ElasticSearch и работает с русским и английским языком. [elasticsearch-analysis-morphology](https://github.com/imotov/elasticsearch-analysis-morphology)

Для того чтобы поставить это плагин на ElasticSearch версия 5.2.0 на macOS нужно выполнить следующую команду в терминале


```bash
/usr/local/opt/elasticsearch/libexec/bin/elasticsearch-plugin install http://dl.bintray.com/content/imotov/elasticsearch-plugins/org/elasticsearch/elasticsearch-analysis-morphology/5.2.2/elasticsearch-analysis-morphology-5.2.2.zip
```

Теперь запускаем ElasticSearch и проверяем что он работает. Для его запуска на macOS достаточно ввести.

```bash
brew services start elasticsearch
```

А для проверки работы выполним get-запрос на 9200 порт (порт на котором запускается ElasticSearch по-умолчанию) и получим от него стандартный ответ:

```bash
curl http://127.0.0.1:9200/
{
  "name" : "YtpgR_F",
  "cluster_name" : "elasticsearch_vladislavkopylov",
  "cluster_uuid" : "groLpJgrR5iayFPDtRUzzz",
  "version" : {
    "number" : "5.2.1",
    "build_hash" : "db0d481",
    "build_date" : "2017-02-09T22:05:32.386Z",
    "build_snapshot" : false,
    "lucene_version" : "6.4.1"
  },
"tagline" : "You Know, for Search"
}
```

Ура, он работает 😀.

## Настройка Rails

![image02](image02.jpeg)

Поставим необходимые gem’ы. В gem elasticsearch-rails встроена поддержка работы постраничной навигации с помощью библиотек Kaminari или WillPaginate, чтобы включить этот функционал достаточно объявить один из этих gem’ов перед elasticsearch-rails.

```ruby
# Gemfile
gem 'will_paginate'
gem 'elasticsearch-rails'
gem 'elasticsearch-model'
```

Включим функционал в нужные модели. Для этого в модели Article, News и BlogPost нужно вставить две строчки.

```ruby
include Elasticsearch::Model
include Elasticsearch::Model::Callbacks
```

Для каждой из этих моделей ElasticSearch создаст индексы в своем пространстве имен /articles для модели Article, /news для модели News и т.д. По-умолчанию название индекса находится в методе index_name самой модели. Пример:

```ruby
Article.index_name
```

Перед созданием индексов удостоверимся что их нет. Для этого выполним get-запрос и получим сообщение об ошибке. Пример.

```bash
curl http://127.0.0.1:9200/news
{"error":{"root_cause":[{"type":"index_not_found_exception","reason":"no such index","resource.type":"index_or_alias","resource.id":"news","index_uuid":"_na_","index":"news"}],"type":"index_not_found_exception","reason":"no such index","resource.type":"index_or_alias","resource.id":"news","index_uuid":"_na_","index":"news"},"status":404}
```

Для добавления индекса, воспользуемся rake командой из гема. Для этого создаем файл lib/tasks/elasticsearch.rake и пропишем туда

```ruby
require 'elasticsearch/rails/tasks/import'
```

Теперь нам доступны команды:

```bash
rake elasticsearch:import:all
rake elasticsearch:import:model
```

Но не будем спешить. В данном примере модели имеют всего три атрибута title, description, searching которые мы все хотим добавить в ElasticSearch. В настоящем проекте каждая модель может содержать дополнительные поля/информацию которую не нужно дублировать в ElasticSearch. Благо, мы может контролировать какие атрибуты будут переданы в из нашего Rails приложения. Создадим файл lib/es_helper.rb

```ruby
module EsHelper
  def as_indexed_json(params={})
    {
      title: title,
      description: description,
      searching: searching
    }
  end

  def preview
    if description.size > 25
      description[0, 25] + '...'
    else
      description
    end
  end
end
```

И включим его в каждую нужную модель.

```ruby
include EsHelper
```

Добавляем анализатор для наших атрибутов. Это нам нужно чтобы в поисковике заработал префикс и морфология.

Создадим еще один файл lib/elastic_my_analyzer.rb который будет содержать хэш с настройками. За основу я взял настройки из elasticsearch-analysis-morphology (demo.sh) и добавил ngram.

```ruby
module ElasticMyAnalyzer
  ES_SETTING = {
    index: {
      number_of_shards: 1
    },
    analysis: {
      filter: {
        my_stopwords: {
          type: 'stop',
          stopwords: 'а,без,более,бы,был,была,были,было,быть,в,вам,вас,весь,во,вот,все,всего,всех,вы,где,да,даже,для,до,его,ее,если,есть,еще,же,за,здесь,и,из,или,им,их,к,как,ко,когда,кто,ли,либо,мне,может,мы,на,надо,наш,не,него,нее,нет,ни,них,но,ну,о,об,однако,он,она,они,оно,от,очень,по,под,при,с,со,так,также,такой,там,те,тем,то,того,тоже,той,только,том,ты,у,уже,хотя,чего,чей,чем,что,чтобы,чье,чья,эта,эти,это,я'
        },
        mynGram: {
          type: 'ngram',
          min_gram: 4,
          max_gram: 8
        }
      },
      analyzer: {
        my_analyzer: {
          type: 'custom',
          tokenizer: 'standard',
          filter: [
            'lowercase', 'russian_morphology', 'my_stopwords', 'mynGram'
          ]
        }
      }
    }
  }
end
```

Немного о том что такое ngram и как он работает. Это называется префиксный поиск или поиск с ошибками. Сначала нужно указать конфигурацию, минимальный и максимальный грамм. Например, если указываем от 1 до 4, то при индексации каждого элемента, анализатор будет разбивать слово на под-слова, каждый длинной от 1 до 4 символов.

Например слово ‘Африка’ будет разбито на: a, аф, афр, афри, ф, фр, фри, фрик, р, ри, рик, рика, и, ик, ика, к, ка, а. При самом поиске будет проходит анализ по каждому под-слову. Чем больше совпадений найдет поиск, тем более выше этот элемент будет в поисковой выдаче.

Ссылки на подробную информацию:

* [analysis-ngram-tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-ngram-tokenizer.html)
* [ngrams_for_partial_matching](https://www.elastic.co/guide/en/elasticsearch/guide/current/_ngrams_for_partial_matching.html)

Не все любят использовать морфология и ngram одновременно 😬. Если что-то из этого не нужно для вашей реализации, то смело удаляйте это из значения filter и заново проиндексируйте ваши записи.

Теперь добавить в каждую интересующую нас модель настройки и установим анализатор на некоторые аттрибуты.

```ruby
include ElasticMyAnalyzer
settings ES_SETTING do
  mappings dynamic: 'true' do
    indexes :title, type: 'string', analyzer: 'my_analyzer'
    indexes :description, type: 'string', analyzer: 'my_analyzer'
    indexes :searching, type: 'boolean'
  end
end
```

В итоге каждая модель должна выглядеть примерно так.

```ruby
class Article < ActiveRecord::Base
  include Elasticsearch::Model
  include Elasticsearch::Model::Callbacks
  include ElasticMyAnalyzer
  include EsHelper

  settings ES_SETTING do
    mappings dynamic: 'true' do
      indexes :title, type: 'string', analyzer: 'my_analyzer' # or type: 'text' for version 6.1
      indexes :description, type: 'string', analyzer: 'my_analyzer' # or type: 'text' for version 6.1
      indexes :searching, type: 'boolean'
    end
  end
end
```

Давайте добавим автозагрузку всех файлов что находятся в каталоге lib/. Для этого в файле application.rb добавим две строчки.

```ruby
config.autoload_paths << Rails.root.join('lib')
config.autoload_paths += Dir["#{config.root}/lib/**/"]
```

Теперь можно добавить индексы в ElasticSearch. Для этого воспользуемся rake таском.

```bash
bundle exec rake environment elasticsearch:import:model CLASS='Article' FORCE=true
bundle exec rake environment elasticsearch:import:model CLASS='BlogPost' FORCE=true
bundle exec rake environment elasticsearch:import:model CLASS='News' FORCE=true
```

Чтобы проверить правильность достаточно сделать curl запрос на 9200 порт. Например curl http://127.0.0.1:9200/news. Вместо сообщения об ошибке, вы увидите конфиги 🎉.

Если по какой то причине созданные индексы нужно удалить то это можно сделать через рельсовую консоль.

```bash
rails s
>  Article.__elasticsearch__.client.indices.delete index: Article.index_name rescue nil
>  BlogPost.__elasticsearch__.client.indices.delete index: BlogPost.index_name rescue nil
>  News.__elasticsearch__.client.indices.delete index: News.index_name rescue nil
```

## Еще немного кода =)

![image03](image03.jpeg)

Уже почти все готово. Создаем класс который будет производить поиск по нескольким моделям. Создадим файл app/models/multy_search.rb и в нем пропишем следующее.

```ruby
class MultySearch
  MODELS_TO_SEARCH = [Article, BlogPost, News].freeze

  def initialize
    @raw_data = nil
    @results = nil
  end

  def search(search_word, page = nil, per_page = nil)
    page ||= 1
    per_page ||= 10
    save_data run_elastic(search_word, page, per_page)
    self
  end

  def raw_data
    @raw_data
  end
  
  def results
    @results
  end

  private

  def run_elastic(search_word, page, per_page)
    Elasticsearch::Model
      .search(search_query(search_word), MODELS_TO_SEARCH)
      .paginate(page: page, per_page: per_page)
  end

  def save_data(data)
    @raw_data = data
    @results = create_answers(data)
  end

  def create_answers(data)
    data.records.map do |result|
      {
        hint: build_hint(result),
        record_type: result.class.name,
        record_id: result.id
      }
    end
  end

  def search_query(query)
    {
      query: {
        bool: {
          must: {
            multi_match: {
              query: query,
              fields: %w(title description),
              operator: 'and'
            }
          },
          filter: [
            {
              term: { searching: true }
            }
          ]
        }
      }
    }
  end

  def build_hint(record)
    {
      title: record.title,
      preview: record.preview,
      type: hint_type(record)
    }
  end

  def hint_type(record)
    case record.class.to_s
    when 'BlogPost' then 'Пост'
    when 'Article' then 'Статья'
    when 'News' then 'Новость'
    end
  end
end
```

Пример использования описан ниже. Метод results хранит в себе результат поиска, а метод raw_data сырые данные из ElasticSearch.

```ruby
MultySearch.new.search('Япония').results
MultySearch.new.search('Япония').raw_data
```

## Заполняем БД информацией из википедии


Для этого воспользуемся файлом db/seeds.rb

```ruby
Article.create!(
  title: 'Шерлок Холмс цитата 1',
  description: 'Человек, который любит искусство ради искусства, самое большое удовольствие зачастую черпает из наименее значительных и ярких его проявлений.',
  searching: true
)
Article.create!(
  title: 'Шерлок Холмс цитата 2',
  description: 'Самая смелая фантазия не в силах представить себе тех необычайных и диковинных случаев, какие встречаются в обыденной жизни.',
  searching: true
)
Article.create!(
  title: 'Шерлок Холмс цитата 3',
  description: 'В этом мире неважно, сколько вы сделали. Самое главное — суметь убедить людей, что вы сделали много.',
  searching: true
)

BlogPost.create!(
  title: 'Факты про японию часть 1',
  description: ' Япония занимает третье место в мире по номинальному ВВП и четвёртое по ВВП, рассчитанному по паритету покупательной способности. Япония является четвёртым по величине экспортёром и шестым по величине импортёром.',
  searching: true
)
BlogPost.create!(
  title: 'Факты про японию часть 2',
  description: 'В 1947 году Япония приняла новую пацифистскую конституцию, в которой делается акцент на либеральную демократию. Оккупация Японии союзными войсками закончилась с принятием Сан-Францисского мирного договора, который вступил в силу в 1952 году, а в 1956 году Япония вступила в ООН. Позже Япония добилась рекордного экономического роста, который продолжался четыре десятилетия и составлял в среднем 10 % ежегодно.',
  searching: true
)
BlogPost.create!(
  title: 'Факты про японию часть 3',
  description: 'Крайней северной точкой Японии является остров Бэнтэндзима, крайней южной — остров Окинотори, крайней западной — мыс Иридзаки, расположенный на острове Йонагуни, крайней восточной — остров Минамитори.',
  searching: true
)

News.create!(
  title: 'Вершина Маттерхорн',
  description: 'Маттерхорн (нем. Matterhorn, итал. Monte Cervino, фр. Mont Cervin) — вершина в Пеннинских Альпах на границе Швейцарии в кантоне Вале и Италии в провинции Валле-д’Аоста. Высота вершины составляет 4478 метров над уровнем моря. Маттерхорн имеет примечательную четырёхгранную пирамидальную форму со стенами, обращёнными по сторонам света.',
  searching: true
)
News.create!(
  title: 'Комета Галлея',
  description: 'Во время появления 1986 года комета Галлея стала первой кометой, исследованной с помощью космических аппаратов, в том числе советскими аппаратами «Вега‑1» и «Вега‑2», которые предоставили данные о структуре кометного ядра и механизмах образования комы и хвоста кометы.',
  searching: true
)
News.create!(
  title: 'Японская Иена',
  description: 'Иена (яп. 円 эн) — денежная единица Японии, одна из основных резервных валют мира. Делится на 100 сен (счётная денежная единица, в 1954 году изъята из обращения). Международный код: JPY. Символом иены является знак ¥. В виде серебряных и золотых монет стала чеканиться в 1869—1871 годах.',
  searching: true
)

# не попадет в поисковую выдачу
News.create!(title: 'About Elm', description: 'I really like it =)')
News.create!(title: 'Французские булочки', description: 'съешь ещё этих мягких французских булок, да выпей чаю')
```

И загрузим это все простой командой.

```bash
rake db:seed
```

Перед этим этапом нужно обязательно создать индексы в ElasticSearch иначе колбеки из Elasticsearch::Model::Callbacks будут запускаться с ошибкой.

Произведем небольшой тест в консоли. Сделаем несколько поисковых запросов и выведем только title.

```bash
rails c
>  MultySearch.new.search('Япония').results.map{|r| r[:hint][:title]}
["Японская Иена", "Факты про японию часть 2", "Факты про японию часть 1", "Факты про японию часть 3"]
>  MultySearch.new.search('Холмс').results.map{|r| r[:hint][:title]}["Шерлок Холмс цитата 1", "Шерлок Холмс цитата 2", "Шерлок Холмс цитата 3”]
>  MultySearch.new.search('япониа').results.map{|r| r[:hint][:title]}
["Японская Иена", "Факты про японию часть 2", "Факты про японию часть 1", "Факты про японию часть 3”]
> MultySearch.new.search('Самое').results.map{|r| r[:hint][:title]}
["Шерлок Холмс цитата 1", "Шерлок Холмс цитата 2", "Шерлок Холмс цитата 3"]
> MultySearch.new.search('Самая').results.map{|r| r[:hint][:title]}["Шерлок Холмс цитата 1", "Шерлок Холмс цитата 2", "Шерлок Холмс цитата 3"]
> MultySearch.new.search('альпы').results.map{|r| r[:hint][:title]}["Вершина Маттерхорн"]
```

В целом все хорошо. Конечно, для ElasticSearch, мы имеем очень мало данных, поэтому могут появляться всякие артефакты в поисковой выдаче. Если вы недовольны результатами поиска, то можете изменить конфиги в файле lib/elastic_my_analyzer.rb. Отключить ngram, добавить свой фильтр и прочее.

## Работаем со вью

То что поиск работает в консоли — это очень хорошо. Для полного счастья не хватает сделать отдельную страничку с результатами поисковой выдаче в нашем rails приложении. И удостоверимся что у нас работает постраничная навигации. Прописываем в путь роутере.

```ruby
get '/search' => 'home#search'
```

Создаем файл app/controllers/home_controller.rb и сделаем простеньких экшн.

```ruby
class HomeController < ApplicationController
  def search
    if params[:query].present?
      page = params[:page] || 1
      @searching = MultySearch.new.search(params[:query], page)
    else
      @searching = nil
    end
  end
end
```

Сделаем вью app/view/home/search.html.slim

```ruby
= form_tag '/search', method: 'GET' do
  = text_field_tag :query, params[:query]
  = submit_tag 'Искать'

- if @searching.present?
  .search_resulting
    p 
      | Были найдены следующие результаты
      - @searching.results.each do |result|
        .search_result
          h3= result[:hint][:title]
          p= result[:hint][:preview]
          br
          small= result[:hint][:type]
          br
          = link_to 'Подробнее', search_result_link(result)

  .search_paging
    = will_paginate @searching.raw_data

- unless @searching.present?
  p К сожалению, ничего не найдено
```

И небольшой метод в хелпере app/helper/application_helper.rb

```ruby
module ApplicationHelper
  def search_result_link(result)
    case result[:record_type]
    when 'BlogPost'
      blog_post_path(result[:record_id])
    when 'Article'
      article_path(result[:record_id])
    when 'News'
      news_path(result[:record_id])
    end
  end
end
```

Теперь по url http://localhost:3000/search мы видим результаты нашей поисковой выдаче. Надеюсь что данная статья была очень полезна.

[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%BE%D0%BB%D0%BD%D0%BE%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%BE%D0%B2%D1%8B%D0%B9-%D0%BC%D1%83%D0%BB%D1%8C%D1%82%D0%B8-%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D1%8C%D0%BD%D1%8B%D0%B9-%D0%BF%D0%BE%D0%B8%D1%81%D0%BA-%D0%B2-rails-c-%D0%BF%D0%BE%D0%BC%D0%BE%D1%89%D1%8C%D1%8E-elasticsearch-5e150a55bd06)
