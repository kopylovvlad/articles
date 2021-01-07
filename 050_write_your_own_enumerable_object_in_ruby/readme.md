# Write your own enumerable object in Ruby

![image01](image01.webp)

Now, I show you the best practices how to write an enumerable object in Ruby. The information can be useful if you want to write an object that will contain other objects. Examples: a garage for your cars; a shopping cart with groceries; a chat that contains a lot of messages, etc.

Of course, you can use `Array` for the task, but if you want to add some methods for uniq behaviour, you have to write your own class.

Let's start with one simple example. We write one small garage-class for storing cars. Here is simply Struct to describe information about our cars.

```ruby
Car = Struct.new(:name, :color)
```

And here is our class for describing garage.

```ruby
class MyGarage
  include Enumerable

  def initialize(cars)
    @cars = cars
  end

  def each(&block)
    @cars.each(&block)
  end

  # some methods for uniq behaviour
end
```

It pretty easy to include `Enumerable`-module and write `each`-method to describe how to iterate our cars. Let's create an instance of `MyGarage` with two cars.

```ruby
garage = MyGarage.new(
  [Car.new('Nissan', 'red'),
   Car.new('Mazda', 'blue')]
)
```

Our `garage`-object has all behaviour from `Enumerable`-[module](https://ruby-doc.org/core-2.7.0/Enumerable.html) (map/filter/reduce and others).

```ruby
garage.map { |car| puts car.name }
puts garage.any? { |i| i.name == 'Nissan' }
puts garage.filter { |i| i == Car.new('Nissan', 'red') }
puts garage.count
puts garage.first
```

But, I don't feel satisfied with it because the instance doesn't have necessary behaviour as an instance of `Array`-[class](https://ruby-doc.org/core-2.7.0/Array.html). We don't have methods such as `.last`, `.size` or `.length`. Also, we can't get an element by its index via `garage[1]`.

The most obvious way is to rewrite the class and inherit it from `Array`-class.

```ruby
class MyGarage < Array
  def initialize(arg)
    super(arg)
  end
end
```

Now, each instance of `MyGarage` has the save behaviour as `Enumerable` and `Array`. Unfortunately, inheriting from core classes is a bad practice. Suddenly, your object will be [converted into an Array](https://gist.github.com/steveklabnik/6071687) or you will find another case that hard to debug. Example:

```ruby
puts garage.is_a?(MyGarage)
# true
second_garage = MyGarage.new([Car.new('Nissan', 'blue')])
puts second_garage.is_a?(MyGarage)
# true
new_garage = garage + second_garage
puts new_garage.class
# Array
```

How to solve the problem and what is the best practice to write your own enumerable object? It's a combination of using `Enumerable` and delegating necessary methods from `Array` using `Forwardable`-[module](https://ruby-doc.org/stdlib-2.7.0/libdoc/forwardable/rdoc/Forwardable.html). The final code is here:

```ruby
require 'forwardable'

Car = Struct.new(:name, :color)

class MyGarage
  include Enumerable
  extend Forwardable
  def_delegators :@garage, :size, :length, :[], :empty?, :last, :index

  def initialize(garage)
    @garage = garage
  end

  def each(&block)
    @garage.each(&block)
  end
end

garage = MyGarage.new(
  [Car.new('Nissan', 'red'),
   Car.new('Mazda', 'blue')]
)
```

As a result, `MyGarage`-class has all methods from `Enumerable`-module and delegates required methods to `Array` (`.size`, `.length`, `[]`, `.empty?`, `.last` and `.index`).

[dev.to](https://dev.to/kopylov_vlad/write-your-own-enumerable-object-in-ruby-572i)
