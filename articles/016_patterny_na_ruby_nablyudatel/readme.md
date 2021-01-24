# Паттерны на ruby: Наблюдатель

**Описание:**

Данный паттерн создает механизм подписки одних объектов на изменение состояний других объектов.

**Пример задачи:**

Представим книжный магазин. Нужно реализовать функционал подписывания на оповещения о состоянии книг. Покупатель может подписаться на сообщение о том что книга появилась в продаже. А кладовщик получает сообщение о том что каких то книжек уже нет на складе.

**Реализация:**

Для описания книжек создадим класс <Book>. За книги отвечает кладовщик (Storekeeper). Каждый день книги покупают покупатели (Customer). Для реализации подписки создадим абстрактный модуль Subject который можно будет заинклудить в класс <Book>. Благодаря чему экземпляры классов <Customer> и <Storekeeper> смогу подписываться на сообщения о состояниях книг.

```ruby
module Subject
  def initialize
    @observers = []
  end

  def add_observer(event_type, observer)
    @observers << { event_type: event_type, observer: observer }
  end

  def delete_observer(event_type, observer)
    @observers.delete(event_type: event_type, observer: observer)
  end

  def notify_observers(event_type)
    selected_observers = @observers.select do |observer|
      observer[:event_type] == event_type
    end
    selected_observers.each do |observer|
      observer[:observer].update(self)
    end
  end
end

class Customer
  attr_reader :name

  def initialize(name)
    @name = name
    puts ''
    puts "Hello, my name is #{@name}"
  end

  def update(changed_book)
    puts "#{self.class} #{name}: Yeah, I can buy a book '#{changed_book.title}'"
  end

  def buy_a_book(book)
    if book.sell_one_book!
      puts "#{name} bought a book"
    else
      puts "#{name} did not buy a book"
    end
  end
end

class Storekeeper
  attr_reader :name

  def initialize(name)
    @name = name
  end

  def update(changed_book)
    puts "#{self.class} #{name}: Oh, I need to purchase books '#{changed_book.title}'"
  end

  def add_book_count(book, new_count)
    puts "#{name} added new books"
    book.count = new_count
  end
end

class Book
  include Subject
  attr_reader :title, :description, :price, :count

  def initialize(title, description, price, count)
    super()
    @title = title
    @description = description
    @price = price
    @count = count
  end

  def sell_one_book!
    return false if @count.zero?
    @count -= 1
    notify_observers(:not_enough_books) if @count.zero?
    true
  end

  def count=(new_count)
    @count = new_count
    notify_observers(:new_books) if new_count != 0
  end

  def show_count
    puts "book count is #{@count}"
  end
end

alexander = Storekeeper.new('Alexander')

a_game_of_thrones = Book.new(
  'A Song of Ice and Fire: A Game of Thrones',
  'Great book',
  2000,
  2
)

a_game_of_thrones.add_observer(:not_enough_books, alexander)

# main perform
a_game_of_thrones.show_count
# book count is 2

jack = Customer.new('Jack')
# Hello, my name is Jack
jack.buy_a_book(a_game_of_thrones)
# Jack bought a book
a_game_of_thrones.show_count
# book count is 1

nadya = Customer.new('Nadya')
# Hello, my name is Nadya
nadya.buy_a_book(a_game_of_thrones)
# Storekeeper Alexander: Oh, I need to purchase books 'A Song of Ice and Fire: A Game of Thrones'
# Nadya bought a book
a_game_of_thrones.show_count
# book count is 0

alf = Customer.new('Alf')
# Hello, my name is Alf
alf.buy_a_book(a_game_of_thrones)
# Alf did not buy a book
a_game_of_thrones.count
# 0
a_game_of_thrones.add_observer(:new_books, alf)

alice = Customer.new('Alice')
# Hello, my name is Alice
alice.buy_a_book(a_game_of_thrones)
# Alice did not buy a book
a_game_of_thrones.show_count
# book count is 0
a_game_of_thrones.add_observer(:new_books, alice)

john = Customer.new('John')
# Hello, my name is John
a_game_of_thrones.add_observer(:new_books, john)
a_game_of_thrones.delete_observer(:new_books, alf)

alexander.add_book_count(a_game_of_thrones, 10)
# Alexander added new books
# Customer Alice: Yeah, I can buy a book 'A Song of Ice and Fire: A Game of Thrones'
# Customer John: Yeah, I can buy a book 'A Song of Ice and Fire: A Game of Thrones'

alice.buy_a_book(a_game_of_thrones)
# Alice bought a book
john.buy_a_book(a_game_of_thrones)
# John bought a book
```

[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D1%8B-%D0%BD%D0%B0-ruby-%D0%BD%D0%B0%D0%B1%D0%BB%D1%8E%D0%B4%D0%B0%D1%82%D0%B5%D0%BB%D1%8C-92803c04a1c4)
