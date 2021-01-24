# Паттерны на ruby: Фабрика

**Описание:**

Данный паттерн определяет интерфейс для создания объектов в классе-родителя. Наследуемые от него классы изменяют тип создаваемых объектов, каждый под конкретную ситуацию. Грубо говоря, каждый "наследник" отвечает на один вопрос "Какой класс использовать?".
Данный паттерн применяется когда заранее неизвестны типы и зависимости объектов, с которыми должен работать ваш код.
Также этот паттерн хорош для того чтобы заложить функционал расширения в какую либо библиотеку. Что бы предоставить другим разработчика возможность создавать новые классы вместо стандартных.

**Пример задачи:**

Отправка логов в Slack, по Email или в какое то другое хранилище логов.

**Реализация:**

У нас есть три класса для функционала логирования <SlackLogger>, <EmailLogger>, <StorageLogger>. У них функционал отправки логов реализован через метод .sending.
Создадим абстрактную фабрику <AbstractLogging>. У нее есть два метода: .new_log_object который еще не определен и .action который берет результат метода .new_log_object и вызывает у него метод .sending.
От класса <AbstractLogging> создадим классы <SlackLogging>, <EmailLogging>, <StorageLogging> и переопределим у них метод .new_log_object.

```ruby
# some classes
class SlackLogger
  def initialize(message)
    @message = message
  end
  def sending
    puts 'we are sending message to Slack'
  end
end
class EmailLogger
  def initialize(message)
    @message = message
  end
  def sending
    puts 'we are sending message to Email'
  end
end
class StorageLogger
  def initialize(message)
    @message = message
  end
  def sending
    puts 'we are sending message to some storage'
  end
end

# abstract factory
class AbstractLogging
  attr_reader :log_string
  def initialize(log_string)
    @log_string = log_string
  end

  def action
    new_log_object.sending
  end

  private

  def new_log_object()
    raise 'not implemented error'
  end
end

class SlackLogging < AbstractLogging
  private

  def new_log_object()
    SlackLogger.new(@log_string)
  end
end

class EmailLogging < AbstractLogging
  private

  def new_log_object()
    EmailLogger.new(@log_string)
  end
end

class StorageLogging < AbstractLogging
  private

  def new_log_object()
    StorageLogger.new(@log_string)
  end
end

SlackLogging.new('some error').action
EmailLogging.new('some error').action
StorageLogging.new('some error').action
```

Также этот паттерн имеет вариацию как Абстрактная фабрика

**Описание:**

Если ранее для выбора нужного класса мы наследовали новый классы от абстрактной фабрики, то в данном варианте достаточно дать абстрактной фабрике интерфейс инъекции зависимости.
Главное — тут нет наследования, а просто инъекция зависимости других классов которые отвечают на вопрос “Какой класс использовать?”

**Пример задачи:**

Отправка логов в Slack, по Email или в какое то другое хранилище логов.

**Реализация:**

Основа функционала логирования остается в классе <AbstractLogging>, с тем отличием что на вход, кроме строки, дается класс который реализует функционал отправки сообщения.
Публичный метод .action создает экземпляр класса, данного при инициализации объекта, и вызывает у него метод .sending

```ruby
class SlackLogger
  def initialize(message)
    @message = message
  end
  def sending
    puts 'we sending message to Slack'
  end
end
class EmailLogger
  def initialize(message)
    @message = message
  end
  def sending
    puts 'we sending message to Email'
  end
end
class StorageLogger
  def initialize(message)
    @message = message
  end
  def sending
    puts 'we sending message to some storage'
  end
end

class AbstractLogging
  attr_reader :log_string, :logger_factory
  def initialize(log_string, logger_factory = nil)
    @log_string = log_string
    @logger_factory = logger_factory
  end

  def action
    raise 'logger_factory is not implemented' if logger_factory.nil?
    logger_factory.new(log_string).sending
  end
end

AbstractLogging.new('some error', SlackLogger).action
AbstractLogging.new('some error', EmailLogger).action
AbstractLogging.new('some error', StorageLogger).action
```

[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D1%8B-%D0%BD%D0%B0-ruby-%D1%84%D0%B0%D0%B1%D1%80%D0%B8%D0%BA%D0%B0-7c776a291b8a)
