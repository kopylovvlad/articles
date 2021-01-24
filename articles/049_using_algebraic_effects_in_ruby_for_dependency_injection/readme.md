# Using algebraic effects in Ruby for Dependency Injection

![image01](image01.webp)

Everyone has heard about React hooks. The feature is based on algebraic effects and it’s great way to manipulate the state. Now algebraic effects are available in Ruby via [dry-effects gem](https://dry-rb.org/gems/dry-effects/master/). I have found that it’s great library for dependency injection and I show you how to use it.

## Making tiny application

Let’s implement one tiny application based on MVC pattern such a RoR. It will be a currency converter. I will not use any frameworks, only show the main concept.

```ruby
class Application
  # model layer
  class Converter
    def call(currency)
      currency * rate
    end
  end

  # controller layer
  class MainController
    def call(params)
      Converter.new.call(params[:currency])
    end
  end

  # middlewares
  class Middleware1
    # code
  end
  class Middleware2
    # code
  end

  def initialize
    @app = MainController.new
  end
  def call(env)
    middlewares = [Middleware2, Middleware1]
    middlewares.reduce(@app) do |app, middleware|
      middleware.new(app)
    end.call(env.freeze)
  end
end
```

`Application` class is our application. In quick look there is few layers: middleware layer, controller layer and model layer. `MainController` is a controller in our MVC pattern. And `Converter` is our main model.

```ruby
class ApplicationRunner
  def initialize(dependency)
    @dependency = dependency
  end

  def call(currency)
    @dependency
    Application.new.call(currency: currency)
  end
end
```

The class `ApplicationRunner` is our interface to run `Application` and configure it. Looking on the code you see that model `Converter` has two dependencies:

* `currency` that we want to convert
* and `rate` of our convertion

Of course, we imagine that our silly application is a real production app, therefore it should have middleware `Middleware1`, `Middleware2`. It can be data validator, logger, and other stuff.

We would like to create an instance of Application in order to convert roubles to US dollars.

* `currency` should be passed through the app in user input and method `Application#call` handles it.
* `rate` is a dependency and it must be passed in the moment of creating an instance.

Nowadays, the rate to convert 1 rouble to USD is 0.013. It means that 1000 rouble can be converted into 13 USD. `1000` rouble is the user input. We have to pass `0.013` to our application as params.

How to run the application

```ruby
RubToUsdConverter = ApplicationRunner.new(0.013)
puts RubToUsdConverter.call(1_000)

# undefined local variable or method `rate' for #<Application::Converter:0x00007fb3440669f0> (NameError)
```

But how to pass `rate` value through the stack to `Converter`? 🤔

You can see the code in [02_0_oop_way.rb](https://github.com/kopylovvlad/dry_effect_reader/blob/main/02_0_oop_way.rb) file

## Way 1

The most obvious way is to change user input via merging it and all our dependencies.

```ruby
class ApplicationRunner
  def initialize(dependency)
    @dependency = dependency
  end

  def call(currency)
    # NOTE: 1) we change user input
    Application.new.call(currency: currency, rate: @dependency)
  end
end
```

```ruby
class MainController
    def call(params)
      # NOTE: 2) MainController knows about Converter implementation and its dependencies
      Converter.new(params[:rate]).call(params[:currency])
    end
  end
```

The way has disadvantages:

* We change original user input and pass all dependencies through middleware.
* In our controller layer `MainController` knows about `Converter` implementation and its dependencies

Of course, the code works perfectly but I would like to reduce relations between classes. You can see the code in [02_1_oop_way.rb](https://github.com/kopylovvlad/dry_effect_reader/blob/main/02_1_oop_way.rb) file.

## Way 2

Next way is passing dependency through whole code stack except middleware. The approach is better than previous: we don’t change user input and middleware doesn’t know about any application dependencies.

```ruby
class ApplicationRunner
  def call(currency)
    # NOTE: 1) Application receives dependency explicitly
    Application.new(@dependency).call(currency: currency)
  end
end
```

```ruby
# NOTE: 2) Application receives dependency explicitly
def initialize(dependency)
  @app = MainController.new(dependency)
end
```

```ruby
class MainController
  # NOTE: 3) we pass dependency through a lot of classes
  def initialize(dependency)
    @dependency = dependency
  end
  def call(params)
    # NOTE: 3) MainController still knows about Converter implementation and its dependencies
    Converter.new(@dependency).call(params[:currency])
  end
end
```

But there are some disadvantages:

* `Application` receives dependency explicitly
* `MainController` still knows about `Converter` implementation and its dependencies
* We pass dependency through a lot of classes. In our example we have only one class between `Converter` and `Application`. But in real applications there will be a lot of classes. Passing all dependencies through it isn’t a good idea.

You can see the code in [02_2_oop_way.rb](https://github.com/kopylovvlad/dry_effect_reader/blob/main/02_2_oop_way.rb) file.

## Way 3

Next way is saving dependencies into one global object.

```ruby
class Application
  # NOTE: 1) adding value object. singleton
  class ApplicationContainer
    def self.hash
      @@hash ||= {}
    end
    def self.set(key, value)
      hash[key] = value
    end
    def self.[](value)
      hash[value]
    end
  end
end
```

```ruby
def initialize(dependency)
  # NOTE: 2) Application knows about all dependencies
  ApplicationContainer.set(:rate, dependency)
  @app = MainController.new
end
```

```ruby
class Converter
  def call(currency)
    currency * rate
  end

  # NOTE: 3) Converter has explicit dependency to external environment
  def rate
    ApplicationContainer[:rate]
  end
end
```

Ok, we don’t pass dependencies through whole stack of classes. But `Application` still knows about all dependencies. And `Converter` has explicit dependency on external environment.

You can see the code in [02_3_oop_way.rb](https://github.com/kopylovvlad/dry_effect_reader/blob/main/02_3_oop_way.rb) file.

## Way 3.2

Let's change the code little bit.

```ruby
class MainController
  def call(params)
    # NOTE: 3) MainController knows about relation between Converter and ApplicationContainer
    Converter.new(ApplicationContainer[:rate])
             .call(params[:currency])
  end
end
```

Of course, we can change our code and rewrite MainController instead of Converter. But now MainController knows about relation
between Converter and ApplicationContainer.

See [02_4_oop_way.rb](https://github.com/kopylovvlad/dry_effect_reader/blob/main/02_4_oop_way.rb) file.

## Using dry-effects

All previous examples are not bad. All cases have advantages and disadvantages. But how we can change implementation and add dependency implicitly. We can do it by dry-effects. The library has `Reader` effect that allows us to pass a value down to the stack.

```ruby
class Application
  # model
  class Converter
    include Dry::Effects.Reader(:rate)
    def call(currency)
      currency * rate
    end
  end
end
```

```ruby
# we run our application with some dependencies
class ApplicationRunner
  include Dry::Effects::Handler.Reader(:rate)
  def initialize(dependency)
    @dependency = dependency
  end

  def call(currency)
    with_rate(@dependency) do
      Application.new.call(currency: currency)
    end
  end
end
```

We can simply add effect in `Converter` adding one code line include `Dry::Effects.Reader(:rate)`. Next step is adding handle in `ApplicationRunner` class `include Dry::Effects::Handler.Reader(:rate)`. In order to set value for `rate` and run the application we call `with_rate()` function and pass a block.

Now `Application` has minimal relations between it parts and the code works perfectly.

You can see the code in [01_functional_way.rb](https://github.com/kopylovvlad/dry_effect_reader/blob/main/01_functional_way.rb) file.

As conclusion, there are advantages of using dependency injection via effects:

* We don’t change user input
* We don’t use one global object in order to have relations between many parts of our application.
* Our tiny class `Converter` in model layer doesn’t have external relations

In the library you can find many effects for manipulation with state, DI, caching, etc ([dry-rb - dry-effects master - Effects](https://dry-rb.org/gems/dry-effects/master/effects/)). Also effects are thread-safe.

## How to test the code

If you have questions about testing, the effects allow you to write testable code and it has good [documentation](https://dry-rb.org/gems/dry-effects/0.1/effects/reader/#testing-with-reader)

Example of unit test

```ruby
RSpec.describe Application::Converter do
  include Dry::Effects::Handler.Reader(:rate)

  subject { described_class.new.call(value) }

  context 'value = 10' do
    let(:dependency) { 20 }
    let(:value) { 10 }
    it 'multiplies 10 by 20' do
      with_rate(dependency) do
        expect(subject).to eql(10 * 20)
      end
    end
  end
end
```

Example of integration test

```ruby
RSpec.describe Application do
  include Dry::Effects::Handler.Reader(:rate)

  subject { Application.new.call(currency: value) }

  context 'value = 10' do
    let(:dependency) { 20 }
    let(:value) { 10 }
    it 'multiplies 10 by 20' do
      with_rate(dependency) do
        expect(subject).to eql(10 * 20)
      end
    end
  end
end
```

The tests you can see in [03_tests.rb](https://github.com/kopylovvlad/dry_effect_reader/blob/main/03_tests.rb) file.

So, my opinion that using the library is fascinating and thought-provoking experience. If it’s interesting to you, you can read more information in official [documentation](https://dry-rb.org/gems/dry-effects/master/)

[dev.to](https://dev.to/kopylov_vlad/using-algebraic-effects-in-ruby-for-dependency-injection-43j4)
