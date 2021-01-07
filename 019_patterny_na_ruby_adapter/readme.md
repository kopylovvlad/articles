# Паттерны на ruby: Адаптер

**Описание:**

Данный паттерн обеспечивает совместную работу классов с несовместимым интерфейсом путем добавления недостающей логики

**Пример задачи:**

К готовому функционалу рендеринга html-тегов, добавить возможность совместимости с другими классами

**Реализация:**

У нас есть модуль TagCreator задача которого — рендерить html теги. На вход он ждет экземпляр класса <Tag>.
Также у нас есть класс <DivTag> в котором есть необходимая информация для рендеринга, но он не является наследником класса <Tag>, а следовательно у него другой интерфейс.
Для решения этой задачи достаточно создать класс <DivTagAdapter> который реализует недостающий интерфейс и берет на вход экземпляр класса <DivTag>.

```ruby
module TagCreator
  def self.render(tag_object)
    return '' if tag_object.nil?
    return tag_object if tag_object.is_a?(::String) or tag_object.is_a?(::Integer)

    str = "<#{tag_object.tag_name}>"
    str += TagCreator.render(tag_object.content)
    str += "</#{tag_object.tag_name}>"
    str
  end
end

class Tag
  attr_reader :tag_name, :content
  def initialize(tag_name, content)
    @tag_name = tag_name
    @content = content
  end
end

class DivTag
  attr_reader :content
  def initialize(content)
    @content = content
  end
end

class DivTagAdapter
  attr_reader :div_tag
  def initialize(div_tag)
    @div_tag = div_tag
  end

  def tag_name
    'div'
  end

  def content
    div_tag.content
  end
end

p_tag = Tag.new('p', 'it is just paragraph')
puts TagCreator.render(p_tag)
# <p>it is just paragraph</p>

p_tag2 = Tag.new(
  'div',
  Tag.new('small', 'it is small text')
)
puts TagCreator.render(p_tag2)
# <div><small>it is small text</small></div>

div_tag = DivTag.new('hello from div')
puts TagCreator.render(DivTagAdapter.new(div_tag))
# <div>hello from div</div>

div_tag = DivTag.new(Tag.new('small', 'hello'))
puts TagCreator.render(DivTagAdapter.new(div_tag))
# <div><small>hello</small></div>
```

[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D1%8B-%D0%BD%D0%B0-ruby-%D0%B0%D0%B4%D0%B0%D0%BF%D1%82%D0%B5%D1%80-60784a4a81ab)
