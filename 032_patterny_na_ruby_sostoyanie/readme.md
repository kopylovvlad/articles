# Паттерны на ruby: Состояние

**Описание:** Данный паттерн позволяет менять поведение объекта в зависимости от его состояния.

**Пример задачи:** Мы пишем сервис документооборота. У каждого документа есть минимум три состояния "черновик", "на модерации", "опубликован". В каждом из этих состояний, у документа должно быть свое поведение плюс необходимо заложить функционал перехода между состояниями.

**Реализация:** За работу документа отвечает класс <Document>. У него есть скрытый атрибут @state отвечающий за состояние. Многие методы класса <Document> делегируют свою работу атрибуту @state . Для состоянии необходимо написать классы <DraftState>, <ModerateState>, <PublishedState> подробно описывающие поведение.

```ruby
class AbstractState
  def initialize(content)
    @content = content
  end

  def status
    raise 'does not implement error'
  end

  def publish
    raise 'does not implement error'
  end

  def go_back
    raise 'does not implement error'
  end

  def render
    raise 'does not implement error'
  end
end

class Document
  attr_accessor :content
  def initialize(content)
    @content = content
    @state = DraftState.new(self)
  end

  def status
    @state.status
  end

  def publish
    @state.publish
  end

  def render
    @state.render
  end

  def change_state(state)
    @state = state
  end
end

class DraftState < AbstractState
  def publish
    @content.change_state(ModerateState.new(@content))
  end

  def status
    :draft
  end

  def render
    ''
  end

  def go_back
    self
  end
end

class ModerateState < AbstractState
  def publish
    @content.change_state(PublishedState.new(@content))
  end

  def status
    :moderate
  end

  def render
    ''
  end

  def go_back
    @content.change_state(DraftState.new(@content))
  end
end

class PublishedState < AbstractState
  def publish
    self
  end

  def status
    :published
  end

  def render
    @content.content
  end

  def go_back
    @content.change_state(ModerateState.new(@content))
  end
end



d = Document.new('<p>Hello</p>')
# document with DraftState
d.status
# :draft
d.render
# ''
d.publish

# document with ModerateState
d.status
# :moderate
d.render
# ''
d.publish

# document with PublishedState
d.status
# :published
d.render
# '<p>Hello</p>'
d.publish
```

[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D1%8B-%D0%BD%D0%B0-ruby-%D1%81%D0%BE%D1%81%D1%82%D0%BE%D1%8F%D0%BD%D0%B8%D0%B5-b92ccb3a02ad)
