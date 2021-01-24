# Как создать gem и опубликовать его

![image01](image01.jpeg)

## Введение

В процессе работы каждый ruby-разработчик пользуется сторонними библиотеками (так называемые гемы). Не всегда получается найти гем с нужным функционалом, в таких ситуациях приходится писать собственное решение. Чаще всего такие решения не входят за рамки проекта, но иногда разработчики задумываются о том чтобы обернуть свое решение в гем и поделится им с сообществом.

У меня возникла потребность парсить мета-теги на веб-странице. Найденные готовые решения не удовлетворяли меня, потому я написал свой модуль. Для полного счастья не хватало обернуть решение в гем, чем я и занялся. Результаты работы можно посмотреть по ссылке:

[meta_information](https://github.com/kopylovvlad/meta_information)

Данное руководство написано с целью помочь начинающим разработчикам. Оно является пересказом собственного опыта. Более подробную информацию можно получить на сайте официального руководства [http://guides.rubygems.org](http://guides.rubygems.org)

## Начало работы

Для начала нужно создать новый каталог который будет содержать код с нужным функционалом. Так как решение состоит из одного модуля, его следует положить в папку lib/

```ruby
# lib/meta_information.rb
require 'nokogiri'
require 'open-uri'

module MetaInformation
  class << self
    def get_meta(input_url)
      return not_valid_url_error unless valid_url?(input_url)

      document = create_document(input_url)
      return nokogiri_error if document == false

      meta_hash = create_meta_array(document)
      success_hash.merge(all_meta: meta_hash)
    end

    private

    def create_meta_array(document)
      array = []
      document.css('meta').each do |node|
        if !node['name'].nil?
          array.push(
            type: 'name',
            name: node['name'],
            content: node['content']
          )
        elsif !node['property'].nil?
          array.push(
            type: 'property',
            property: node['property'],
            content: node['content']
          )
        end
      end
      array
    end

    def valid_url?(uri)
      !!uri.match(/^(https?:\/\/)?([\w\.]+)\.([a-z]{2,6}\.?)(\/[\w\.]*)*\/?$/)
    end

    def create_document(input_url)
      Nokogiri::HTML(open(input_url))
    rescue
      false
    end

    def not_valid_url_error
      {
        success: false,
        error: 'url is not valid'
      }
    end

    def nokogiri_error
      {
        success: false,
        error: 'error with parsing a document'
      }
    end

    def success_hash
      {
        succes: 'true',
        error: ''
      }
    end
  end
end
```

В корне проекта создаем Gemfile и прописываем там версию ruby и необходимые зависимости. После этого ставим их через bundle install.

```ruby
# Gemfile
source 'https://rubygems.org'
ruby '2.3.3'

gem 'nokogiri'
gem 'rspec'
```

Очень рекомендую написать юнит тесты. Они должны быть в папке test/ или spec/

```ruby
# spec/lib/meta_information_spec.rb
require './lib/meta_information'

RSpec.describe 'MetaInformation' do
  
  describe 'create_meta_array' do
    describe 'we have meta' do
      it 'must create array' do
        document = Nokogiri::HTML('
          <html>
            <meta name="description" content="" />
            <meta name="title" content="some title" />
            <meta property="author" content="Bob" />
            <meta property="og:title" content="og_title" />
            <meta property="twitter:image" content="http://some_host.com/some_path" />
            <meta property="og:locale" content="ru_RU" />
            <meta property="al:ios:app_store_id" content="12345678900" />
            <body>
              <h1>Mr. Belvedere Fan Club</h1>
            </body>
          </html>
        ')
        expect(MetaInformation.send(:create_meta_array, document)).to eq([
          { type: 'name', name: 'description', content: '' },
          { type: 'name', name: 'title', content: 'some title' },
          { type: 'property', property: 'author', content: 'Bob' },
          { type: 'property', property: 'og:title', content: 'og_title' },
          { type: 'property', property: 'twitter:image', content: 'http://some_host.com/some_path' },
          { type: 'property', property: 'og:locale', content: 'ru_RU' },
          { type: 'property', property: 'al:ios:app_store_id', content: '12345678900' }
        ])
      end
    end
    
    describe 'without meta' do
      it 'has empty array' do
        first_document = Nokogiri::HTML('
          <html>
            <body>
              <h1>Mr. Belvedere Fan Club</h1>
            </body>
          </html>
        ')
        second_document = Nokogiri::HTML('')
        expect(MetaInformation.send(:create_meta_array, first_document)).to eq([])
        expect(MetaInformation.send(:create_meta_array, second_document)).to eq([])
      end
    end
  end

  describe 'valid_url?' do
    it 'must be valid' do
      expect(MetaInformation.send(:valid_url?, 'http://www.somesite.com')).to be_truthy
      expect(MetaInformation.send(:valid_url?, 'https://www.somesite.com')).to be_truthy
      expect(MetaInformation.send(:valid_url?, 'https://somesite.com')).to be_truthy
      expect(MetaInformation.send(:valid_url?, 'wwwsome_site.ru')).to be_truthy
      expect(MetaInformation.send(:valid_url?, 'https://somesite.com/some_page')).to be_truthy
      expect(MetaInformation.send(:valid_url?, 'https://somesite.com/some_page/page')).to be_truthy
      expect(MetaInformation.send(:valid_url?, 'https://somesite.com.uk/some_page')).to be_truthy
    end
    
    it 'does not valid' do
      expect(MetaInformation.send(:valid_url?, 'some_site')).to be_falsey
      expect(MetaInformation.send(:valid_url?, 'http\\:wwwsome_site.ru')).to be_falsey
      expect(MetaInformation.send(:valid_url?, 'com.some_site')).to be_falsey
    end
  end
  
  describe 'private hash equal' do
    it 'not_valid_url_error hash' do
      expect(MetaInformation.send(:not_valid_url_error)).to eq({
        success: false,
        error: 'url is not valid'
      })
    end

    it 'nokogiri_error hash' do
      expect(MetaInformation.send(:nokogiri_error)).to eq({
        success: false,
        error: 'error with parsing a document'
      })
    end

    it 'success_hash hash' do
      expect(MetaInformation.send(:success_hash)).to eq({
        succes: 'true',
        error: ''
      })
    end
  end
end
```

После того как функционал написан, покрыт тестами и протестирован, начинаем создавать гем. Прописываем стандартный .gitignore и коммитим наш код.

```ruby
*.gem
*.rbc
.bundle
.config
.yardoc
usage.rb
Gemfile.lock
InstalledFiles
_yardoc
coverage
doc/
lib/bundler/man
pkg
rdoc
spec/reports
test/tmp
test/version_tmp
tmp
```

Выкладываем его на github

```bash
git remote add origin git@github.com:USER_NAME/REP_NAME.git
git push -u origin master
```

Создаем файл с описанием гема GEM_NAME.gemspec куда прописываем всю необходимую информацию включая название, описание, версию, ссылки, зависимости, и файлы.

```ruby
$:.push File.expand_path("../lib", __FILE__)

Gem::Specification.new do |s|
  s.name        = 'meta_information'
  s.version     = '1.0.1'
  s.date        = '2017-02-26'
  s.summary     = 'MetaInformation - Simple gem for parsing meta information'
  s.description = 'Simple gem for parsing meta information from websites. It scan all meta-tags by name or property attributes.'
  s.author      = 'Vladislav Kopylov'
  s.email       = 'kopylov.vlad@gmail.com'
  s.homepage    = 'https://github.com/kopylovvlad/meta_information'
  s.license     = 'MIT'
  s.files       = `git ls-files`.split("\n")
  s.add_dependency('nokogiri', '~> 1.7', '>= 1.7.0')
end
```

Пробуем собрать свой гем.

```bash
$ gem build meta_information.gemspec
Successfully built RubyGem
  Name: meta_information
  Version: 1.0.1
  File: meta_information-1.0.1.gem
```

Проверим установку только что собранного гема.

```bash
$ gem install ./meta_information-1.0.1.gem
Successfully installed meta_information-1.0.1
Parsing documentation for meta_information-1.0.1
Done installing documentation for meta_information after 0 seconds
1 gem installed
```

Обязательно проверим работоспособность в консоли.

```bash
$ irb
2.3.3 :001 > require 'meta_information'
=> true
2.3.3 :002 > MetaInformation::VERSION
=> "1.0.1"
2.3.3 :006 > MetaInformation.get_meta('https://www.yandex.ru’)
```

Если все работает, то начинаем заливать гем в публичный доступ. Первым делом авторизовываемся в [rubygems.org](rubygems.org)

```bash
$ curl -u USER_NAME https://rubygems.org/api/v1/api_key.yaml > ~/.gem/credentials; chmod 0600 ~/.gem/credentials
```

Обновляем репозиторий на github

```bash
$ git add —all
$ git commit
$ git push origin master
```

Публикуем гем и проверяем что он доступен

```bash
$ gem push meta_information-1.0.1.gem
$ gem list -r meta_information
```

Если все прошло удачно, то он доступен по прямой ссылке (rubygems.orb/gems/GEM_NAME).

Пример: [https://rubygems.org/gems/meta_information](https://rubygems.org/gems/meta_information)

Ура, гем удачно опубликован 🎉 , теперь его можно смело установить следующей командой.

```bash
$ gem install meta_information
```

## Дальнейшие обновления гема

![image02](image02.jpeg)

Со временем каждый гем развивается, к нему добавляется новый функционал, убираются ошибки, на github появляются issues которые надо решить и pull-requests которые надо мерджит. Все это являются предпосылками к новому релизу. Для того чтобы выложить новый релиз на rubygems, нужно повторять следующие действия.

Обновить .gemspecfile. В нем нужно как минимум обновить версию. После этого снова собрать обновленный гем, проверить установку и работу.

```bash
$ gem build meta_information.gemspec
$ gem install ./meta_information-1.0.2.gem
```

Если все работает то опубликовать новую версию.

```bash
$ gem push meta_information-1.0.2.gem
```

Вот, собственно все. Ниже публикую полезные ссылки с подробной информацией.

**Publishing your gem**: [http://guides.rubygems.org/publishing/](http://guides.rubygems.org/publishing/)
**Specification reference**: [http://guides.rubygems.org/specification-reference/](http://guides.rubygems.org/specification-reference/)
