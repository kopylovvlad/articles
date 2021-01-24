# Паттерны на ruby: Строитель

**Описание:**

Задача данного паттерна состоит в создании/производстве других экземпляров классов.
К нему обращаются если у создаваемого объекта есть сложные связи с другими объектами, особые условия для валидации, и/или мы должны их абстрагировать с точки зрения интерфейса.

**Пример задачи:**

Необходимо разработать функционал создания статей. Статью можно создать только если она соответствует следующим условиям:
1)У нее заполнен заголовок и описание
2)К каждой статье нужно заполнить от 1 до 3 тегов

**Реализация:**

Теги и статьи представлены в виде классов <Tag>, <Article>. Функционал создание статей и тегов к ним представлен в классе <ArticleBuilder>. Манипуляция данными представлен в методах .add_description, .add_tag, .add_title. А для проверка валидности данных и сохранения есть методы .valid? и .save

```ruby
require 'json'

module JSONable
  def to_json(*options)
    as_json(*options).to_json(*options)
  end
end

class Tag
  attr_reader :title

  include JSONable
  def initialize(title)
    @title = title
  end

  def as_json(options={})
    { title: title }
  end
end


class Article
  attr_accessor :title, :description, :tags

  include JSONable
  def initialize(title, description)
    @title = title
    @description = description
    @tags = []
  end

  def save
    puts 'we have done it!!!'
  end

  def as_json(options={})
    {
      title: title,
      description: description,
      tags: tags
    }
  end
end

class BuilderError
  attr_reader :field, :msg
  include JSONable

  def initialize(field, msg)
    @field = field
    @msg = msg
  end

  def as_json(options={})
    {
      field: field,
      msg: msg
    }
  end
end

# make Article only through ArticleBuilder
class ArticleBuilder
  attr_reader :errors, :article

  def initialize(title = '', description = '')
    @article = Article.new(title, description)
    @errors = []
    @tags = []
  end

  def valid?
    clear_errors!
    valid_title!
    valid_description!
    valid_tags!
    errors.size == 0
  end

  def add_description(val)
    @article.description = val
  end

  def add_tag(val)
    @tags << val
  end

  def add_title(val)
    @article.title = val
  end

  def save
    return false unless valid?
    @tags.each do |tag_string|
      @article.tags << Tag.new(tag_string)
    end
    @article.save
    true
  end

  private

  def clear_errors!
    @errors = []
  end

  def valid_title!
    if @article.title.nil? or !@article.title.is_a?(String) or @article.title.size == 0
      @errors.push(BuilderError.new(:title, 'Обязательно заполните title'))
    end
  end

  def valid_description!
    if @article.description.nil? or !@article.description.is_a?(String) or @article.description.size == 0
      @errors.push(BuilderError.new(:description, 'Обязательно заполните description'))
    end
  end

  def valid_tags!
    check_tags_size and check_tags_content
  end

  def check_tags_size
    if @tags.size == 0 or @tags.size > 3
      @errors.push(BuilderError.new(:tags, 'Количество тего должно быть от 1 до 3'))
      return false
    end
    true
  end

  def check_tags_content
    @tags.each do |tag_string|
      unless tag_string.is_a?(String)
        @errors.push(BuilderError.new(:tags, 'Тег должен быть строкой'))
        return false
      end
      if tag_string.nil? or tag_string.size == 0
        @errors.push(BuilderError.new(:tags, 'Обязательно заполните заголовки для всех тегов'))
        return false
      end
    end
    true
  end
end

# empty article
article_builder = ArticleBuilder.new
article_builder.valid?
# false
article_builder.save
# false
article_builder.errors.to_json
# [{"field"=>"title", "msg"=>"Обязательно заполните title"}, {"field"=>"description", "msg"=>"Обязательно заполните description"}, {"field"=>"tags", "msg"=>"Количество тего должно быть от 1 до 3"}]


# article without tags
article_builder = ArticleBuilder.new
article_builder.add_title('article title')
article_builder.add_description('article description')
article_builder.valid?
# false
article_builder.save
# false
article_builder.errors.to_json
# [{"field"=>"tags", "msg"=>"Количество тего должно быть от 1 до 3"}]


# article with empty tags
article_builder = ArticleBuilder.new('article title', 'article description')
article_builder.add_tag('')
article_builder.valid?
# fasle
article_builder.save
# false
article_builder.errors.to_json
# [{"field"=>"tags", "msg"=>"Обязательно заполните заголовки для всех тегов"}]


# valid article
article_builder = ArticleBuilder.new('article title', 'article description')
article_builder.add_tag('tag title')
article_builder.add_tag('12343')
article_builder.valid?
# true
article_builder.save
# we have done it!!!
# true
```

[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D1%8B-%D0%BD%D0%B0-ruby-%D1%81%D1%82%D1%80%D0%BE%D0%B8%D1%82%D0%B5%D0%BB%D1%8C-7d6d9e19816b)
