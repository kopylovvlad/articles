# One more article about ruby best practice

![https://dribbble.com/Broncos](image01.png)

Everyone engineer should read tech literature in free time. When you are reading you learn a lot of new things. You read about someone"s experience, best practice and fails. With that useful information you can see new opportunity, can make your code better and more stable, can create new features better and faster.

In this article I want to share some good practice that I have read in tech literature.

## Write readable code and think about responsibility

Ruby is oop language, do not forget it. And endeavor to write readable code. Many developers forget about responsibility principle when they are writing code. Many times I have seen additional condition where they should not be. It makes negative consequences in your code. It is very hard to support that code and create new features in it. Sometimes if you need to realize new feature, the better way is delete all code and rewrite it in scratch.

**First example**. Everyday one information system parses subscribers list. Subscribers list is a csv-file with next columns:

* "number" — index row number.
* "email" — user"s email.
* "first_name" — user"s first name.
* "last_name" — user"s last name.
* "action" — user action. It may contains true or false. True means user want to subscribe. False means user want to unsubscribe.
* "subscription_ids" — list subscription's ids.

Like this:

```
number|email|first_name|last_name|action|subscription_ids
1|some_email.ru|James|Smith|true|1,3
2|some_email2.ru|Barbara|O'connor|true|1,2,3,5
3|some_email3.ru|||false|3,6
```

Below is example of bad code. It has many conditions and it hard to read.

```ruby
subscribers.each do |subscriber_row|
  user = User.find_by(email: subscriber_row[1])
  if user.nil?
    user = User.create_by_subscribe(
      email: subscriber_row[1],
      first_name: subscriber_row[2],
      last_name: subscriber_row[3]
    )
  end

  subscriptions = Subscription.find(subscriber_row[5])
  if subscriber_row[4] == true
    subscriptions.each do |subscr|
      subscr.subscribe_user(user)
    end
    MailSender.greeting_email(user.id, subscriptions).deliver_now
  else
    subscriptions.each do |subscr|
      subscr.unsubscribe_user(user)
    end
    MailSender.unsubscribe_email(user.id, subscriptions).deliver_now
  end
end
```

Let’s make code more readable. We need add some method into User class and create SubscriberRow class. This is better.

```ruby
subscribers.each do |row|
  subscriber_row = SubscriberRow.new(row)
  user = User.find_by_or_create_by_subscribe(subscriber_row.user_hash)
  subscriptions = Subscription.find(subscriber_row.subsc_ids)

  if subscriber_row.subscribe?
    user.subscribe(subscriptions)
    user.send_greeting_email(subscriptions)
  end
  if subscriber_row.unsubscribe?
    user.unsubscribe(subscriptions)
    user.send_unsubscribe_email(subscriptions)
  end
end
```

**Second example**. We write software (rails app) for a restaurant company. The company makes and sells pizza everyday. We need to write module for showing weekly statistics. Company need to know data for everyday in current week:

* day of the week
* number of pizza sold
* sum of orders
* TOP 3 buyers (users who have largest amount of orders)
* TOP 3 pizza sold

For this task, we can create controller with one public method and any private method like this:

```ruby
# app/controllers/statistics_controller.rb
class StatisticsController < ApplicationController
  # ...
  def weekly
    @weekly = []

    get_date.each do |date_obj|
      day = date_obj[:start_day].strftime('%A %d-%m-%Y')
      orders = get_orders(date_obj)
      orders_sum = orders.map(&:price).reduce(:+)
      pizza_count = orders
                    .map { |order| order.items.where(type: :pizza).count }
                    .reduce(:+)

      pizza_arr = get_pizza_arr(orders)
      users_arr = get_users_arr(orders)

      @weekly.push(
        day: day,
        pizza_count: pizza_count,
        orders_sum: orders_sum,
        users: users_arr,
        pizza: pizza_arr
      )
    end
  end

  private

  ##
  # returns array
  # like this
  # [ { pizza: <Item>, count: 10},
  #   { pizza: <Item>, count: 8},
  #   { pizza: <Item>, count: 7},]

  def get_pizza_arr(orders, pizza_count)
    pizza_hash = Hash.new(0)
    orders
      .map { |order| order.items.where(type: :pizza) }
      .reduce(:+).each { |item| pizza_hash[item.id] += 1 }
    pizza_hash = pizza_hash.sort_by { |k, v| v }.reverse.to_h
    Hash[pizza_hash.sort_by { |k,v| -v }[0..2]].map do |k,v|
      {
        pizza: Item.find(k),
        count: v
      }
    end
  end

  ##
  # return array like this
  # [ {user: <User>, sum: 1000},
  #   {user: <User>, sum: 900},
  #   {user: <User>, sum: 850}]

  def get_users_arr(orders)
    users_hash = Hash.new(0)
    orders.each { |order| users_hash[order.user_id] += order.price }
    users_hash = users_hash.sort_by { |k, v| v }.reverse.to_h
    Hash[users_hash.sort_by { |k,v| -v }[0..2]].map do |k,v|
      {
        user: User.find(k),
        sum: v
      }
    end
  end

  def get_orders(date_obj)
    Order.preload(:user, :items).where(
      'created_at >= :start_day AND created_at < :end_day',
      start_day: date_obj[:start_day],
      end_day: date_obj[:end_day]
    )
  end

  ##
  # it returns something like this
  # [ {start_day: Mon, 08 May 2017 00:00:00 +0300,
  #     end_day:  Tue, 09 May 2017 00:00:00 +0300},
  #   {start_day: Tue, 09 May 2017 00:00:00 +0300,
  #     end_day:  Wed, 10 May 2017 00:00:00 +0300},
  #   {start_day: Wed, 10 May 2017 00:00:00 +0300,
  #     end_day:  Thu, 11 May 2017 00:00:00 +0300}]

  def get_date
    arr = []
    start_day = nil
    loop do
      unless start_day.present?
        start_day = DateTime.current.beginning_of_week
      else
        start_day += 1.day
      end
      end_day = start_day + 1.day

      arr.push(
        start_day: start_day,
        end_day: end_day
      )
      break if end_day > DateTime.current.end_of_week
    end
    arr
  end
# ...
end
```

```ruby
# app/statistics/weekly.html.slim
table.table
  thead
    tr
      td Day
      td Pizza count
      td Orders sum
      td Users
      td Pizza
  tbody
    - @weekly.each do |weekly_item|
      tr
        td= weekly_item[:day]
        td= weekly_item[:pizza_count]
        td= "$#{weekly_item[:orders_sum]}"
        td
          - weekly_item[:user_item].each do |user_item|
            = link_to "#{user_item[:user].email} (#{user_item[:sum]})", admin_user_path(user_item[:user])
            br
        td
          - weekly_item[:pizza_item].each do |pizza_item|
            = link_to "#{pizza_item[:pizza].title} (#{pizza_item[:count]})", admin_tem_path(pizza_item[:pizza])
            br
```

We finish our task, but code is bad. If we need to add some big features in this controller we will have problems. Or if we need to duplicate functional for another part of system (another controller, api), or send data by email, we will have problems.

What should we do? We need to define a responsibility and use DRY. Need create some classes to aggregate data and collect everyday data to week.

```ruby
# app/model/weekly_day.rb
WeeklyDay = Struct.new(:start_day, :end_day) do
  def start_day_string
    start_day.strftime('%A %d-%m-%Y')
  end
end

# app/model/daily_statistic.rb
DailyStatistic = Struct.new(:day_string, :orders) do
  def day
    day_string
  end

  def orders_sum
    @orders_sum ||= orders.map(&:price).reduce(:+)
  end

  def pizza_count
    @pizza_count ||=  orders
                      .map { |order| order.items.where(type: :pizza).count }
                      .reduce(:+)
  end

  def users
    @users ||= get_users_arr
  end

  def pizza
    @pizza ||= get_pizza_arr
  end

  private

  ##
  # returns array
  # like this
  # [ { pizza: <Item>, count: 10},
  #   { pizza: <Item>, count: 8},
  #   { pizza: <Item>, count: 7},]

  def get_pizza_arr
    pizza_hash = Hash.new(0)
    orders
      .map { |order| order.items.where(type: :pizza) }
      .reduce(:+).each { |item| pizza_hash[item.id] += 1 }
    pizza_hash = pizza_hash.sort_by { |k, v| v }.reverse.to_h
    Hash[pizza_hash.sort_by { |k,v| -v }[0..2]].map do |k,v|
      {
        pizza: Item.find(k),
        count: v
      }
    end
  end

  ##
  # return array like this
  # [ {user: <User>, sum: 1000},
  #   {user: <User>, sum: 900},
  #   {user: <User>, sum: 850}]

  def get_users_arr
    users_hash = Hash.new(0)
    orders.each { |order| users_hash[order.user_id] += order.price }
    users_hash = users_hash.sort_by { |k, v| v }.reverse.to_h
    Hash[users_hash.sort_by { |k,v| -v }[0..2]].map do |k,v|
      {
        user: User.find(k),
        sum: v
      }
    end
  end
end

# app/models/weekly_statistic.rb
class WeeklyStatistic
  def perform
    date.map do |weekly_day|
      DailyStatistic.new(
        weekly_day.start_day_string,
        get_orders(weekly_day)
      )
    end
  end

  def date
    @date ||= get_date
  end

  private

  ##
  # it returns something like this
  # [ <WeeklyDay>, <WeeklyDay>, <WeeklyDay> ]

  def get_date
    arr = []
    start_day = nil
    loop do
      unless start_day.present?
        start_day = DateTime.current.beginning_of_week
      else
        start_day += 1.day
      end
      end_day = start_day + 1.day

      arr.push(WeeklyDay.new(start_day, end_day))
      break if end_day > DateTime.current.end_of_week
    end
    arr
  end

  def get_orders(date_obj)
    Order.preload(:user, :items).where(
      'created_at >= :start_day AND created_at < :end_day',
      start_day: date_obj.start_day,
      end_day: date_obj.end_day
    )
  end
end

# app/controllers/statistics_controller.rb
class StatisticsController < ApplicationController
  # ...
  def weekly
    @weekly = WeeklyStatistic.new.perform
  end
  # ...
end
```

```ruby
# app/views/statistics/weekly.html.slim
table.table
  thead
    tr
      td Day
      td Pizza count
      td Orders sum
      td Users
      td Pizza
  tbody
    - @weekly.each do |daily_statistic|
      tr
        td= daily_statistic.day
        td= daily_statistic.pizza_count
        td= "$#{daily_statistic.orders_sum}"
        td
          - daily_statistic.user_item.each do |user_item|
            = link_to "#{user_item[:user].email} (#{user_item[:sum]})", admin_user_path(user_item[:user])
            br
        td
          - daily_statistic.pizza_item.each do |pizza_item|
            = link_to "#{pizza_item[:pizza].title} (#{pizza_item[:count]})", admin_tem_path(pizza_item[:pizza])
            br
```

It looks better. In StatisticsController we have only one line that we can easily duplicate to another part of system.

## Do not afraid use .fetch for hash

Sometimes we need to work with complicated, nested hash. When we are trying to access value in deeply depth, code can be like this:

```ruby
var = nil
if(some_hash.present? and some_hash[:key_1].present? and some_hash[:key_1][:key_2].present?)

  var = some_hash[:key_1][:key_2]
end
```

There is a long condition for accessing value. It is hard to read. We can rewrite it using .fetch() method. This hash method takes two arguments key and placeholder. If hash have not key, method will returns placeholder.

```ruby
var = nil
if some_hash.present?
  var = some_hash.fetch(:key_1, {}).fetch(:key_2, nil)
end
```

Fetch has one feature: if hash key exists and it returns nil or false method will return nil or false. But if you use activesupport you can rewrite code on rails-style.

```ruby
var = some_hash.try(:fetch, :key_1, {}).try(:fetch, :key_2, nil)
```

## Check passed arguments

Ruby does not have static typing. It means any passed arguments and returned values can be instance of any class. It can make any confuse in code.

Look at example. We created class GeoCoordinate for creating geo-positions in interactive map.

```ruby
class GeoCoordinate
  def initialize(latitude, longitude)
    @latitude = latitude
    @longitude = longitude
  end
  # and more code
end
```

When are you looking at code you see that initialize method takes two arguments "latitude" and "longitude". But you do not see which instance should latitude and longitude be. Should latitude be instance of Number or instance of String? Different developers can write different code.

```ruby
GeoCoordinate.new(55.45, 37.37)
GeoCoordinate.new('55.45', '37.37')
GeoCoordinate.new(['55.45'], ['37.37'])
GeoCoordinate.new(magicCoordinate.new)
GeoCoordinate.new(Latitude.new('55.45'), Longitude.new('37.37'))
GeoCoordinate.new(latitude: '55.45', longitude: '37.37’)
```

In additions there are many others classes in different parts on system.

```ruby
class Latitude
  def initialize(value)
    @value = value
  end
  
  def to_i
    @value.to_i
  end
end

class Longitude
  def initialize(value)
    @value = value
  end
  def to_i
    @value.to_i
  end
end

class MagicCoordinate
  def initialize; end
  def to_geo_coordinate
    GeoCoordinate.new(magic_latitude, magic_longitude)
  end
end
```

In this case, we can write function with the same name for checking passed arguments. The function gives us assurance that passed arguments are valid. If arguments are invalid it will raise TypeError.

```ruby
def GeoCoordinate(*args)
  case args.first
  when String
    GeoCoordinate.new(*args[0].to_i, *args[1].to_i)
  when Array
    GeoCoordinate.new(*args[0][0], *args[1][0])
  when Hash
    GeoCoordinate.new(*args[0][:latitude], *args[0][:longitude])
  when ->(arg) { %w(Float Fixnum).include?(arg.class.to_s) }
    GeoCoordinate.new(*args)
  when ->(arg) { arg.respond_to?(:to_geo_coordinate) }
    args.first.to_geo_coordinate
  when ->(arg) { arg.is_a?(Latitude) }
    GeoCoordinate.new(*args[0].to_i, *args[1].to_i)
  else
    raise TypeError, "Cannot convert #{args.inspect} to GeoCoordinate"
  end
end

GeoCoordinate(55.45, 37.37)
GeoCoordinate('55.45', '37.37')
GeoCoordinate(['55.45'], ['37.37'])
GeoCoordinate(MagicCoordinate.new)
GeoCoordinate(Latitude.new('55.45'), Longitude.new('37.37'))
GeoCoordinate(latitude: '55.45', longitude: '37.37')
```

## Checking returned values

In static typing programing languages if some method/function returns array it will return array everytime. If some another method returns string it will return string everytime. In not static typing programing languages every method/function can return different instance every time. It can makes any troubles in our code.

For example, takes GeoCoordinate class. It has three_nearest_coordinates method that should returns maximum three instance of GeoCoordinate class. But in current realization it can returns array, an instance, false or nil. It is very hard for debugging.

```ruby
class GeoCoordinate
  def initialize(latitude, longitude)
    @latitude = latitude
    @longitude = longitude
  end

  ##
  # returns array

  def nearest_coordinates
    # magic
  end

  ##
  # returns integer

  def nearest_coordinates_count
    # magic
  end
   
  ##
  # may returns false, nil, <GeoCoordinate>, or array :)

  def three_nearest_coordinates
    return false unless @latitude.present? and @longitude.present?

    case nearest_coordinates_count
    when 0
      return nil
    when 1
      nearest_coordinates.limit(1).first
    when 2
      nearest_coordinates.limit(2)
    when 3
      nearest_coordinates.limit(3)
    else
      nearest_coordinates.limit(3)
    end
  end
end
```

We need rewrite three_nearest_coordinates. It must return array every time.

```ruby
class GeoCoordinate
  # some code
  
  ##
  # returns array

  def three_nearest_coordinates
    return [] unless @latitude.present? and @longitude.present?

    coords_count = nearest_coordinates_count
    case coords_count
    when 0
      return []
    when ->(integer) { [1, 2, 3].include?(integer) }
      nearest_coordinates.limit(coords_count)
    else
      nearest_coordinates.limit(3)
    end
  end
end
```

## Long hash in passed attributes is bad

Any method/function can takes hash in passed attributes with 2–3 keys. It does not sound bad. After some time, system become bigger, has more features and the hash has 8–10 keys instead 2–3. It can create problems. Developer can make a spelling mistake in keys (misspelling is hard to debug), or choosing right parameter takes many times. Look at example.

```ruby
class CoffeeMaker
  def self.make_coffee(params)
    ingredients = {
      espresso: params.fetch(:espresso, false),
      milk_foam: params.fetch(:milk_foam, false),
      whipped_cream: params.fetch(:whipped_cream, false),
      steamed_milk: params.fetch(:steamed_milk, false),
      chokolate_syrup: params.fetch(:chokolate_syrup, false),
      hot_water: params.fetch(:hot_water, false)
    }

    #
    # and some lines of code
    #

    return cup_of_coffee(ingredients)
  end
end

espresso = CoffeeMaker.make_coffee(espresso: true)
espresso_macchiato = CoffeeMaker.make_coffee(espresso: true, milk_foam: true)
espresso_con_panna = CoffeeMaker.make_coffee(espresso: true, whipped_cream: true)
caffe_latte = CoffeeMaker.make_coffee(espresso: true, milk_foam: true, steamed_milk: true)
flat_white = CoffeeMaker.make_coffee(espresso: true, steamed_milk: true)
caffee_mocha = CoffeeMaker.make_coffee(espresso: true, whipped_cream: true, steamed_milk: true, chokolate_syrup: true)
americano = CoffeeMaker.make_coffee(espresso: true, hot_water: true)
```

The better way is rewrite method/function, it should take any instance instead hash. It really make code easily to read and debug.

```ruby
module Coffee
  class Espresso
    def initialize
      @ingredients = {
        espresso: true
      }
    end
  end

  class EspressoMacchiato
    def initialize
      @ingredients = {
        espresso: true,
        milk_foam: true
      }
    end
  end

  class EspressoConPanna
    def initialize
      @ingredients = {
        espresso: true,
        whipped_cream: true
      }
    end
  end

  class CaffeLatte
    def initialize
      @ingredients = {
        espresso: true,
        milk_foam: true,
        steamed_milk: true
      }
    end
  end

  class FlatWhite
    def initialize
      @ingredients = {
        espresso: true,
        steamed_milk: true
      }
    end
  end

  class CaffeeMocha
    def initialize
      @ingredients = {
        espresso: true,
        whipped_cream: true,
        steamed_milk: true,
        chokolate_syrup: true
      }
    end
  end

  class Americano
    def initialize
      @ingredients = {
        espresso: true,
        hot_water: true
      }
    end
  end
end

module CoffeeMaker
  def self.make_coffee(coffee)
    ingredients = coffee.ingredients
    #
    # and some lines of code
    #
    return cup_of_coffee(ingredients)
  end
end

CoffeeMaker.make_coffee Coffee::Espresso.new
CoffeeMaker.make_coffee Coffee::EspressoMacchiato.new
CoffeeMaker.make_coffee Coffee::EspressoConPanna.new
CoffeeMaker.make_coffee Coffee::CaffeLatte.new
CoffeeMaker.make_coffee Coffee::FlatWhite.new
CoffeeMaker.make_coffee Coffee::CaffeeMocha.new
CoffeeMaker.make_coffee Coffee::Americano.new
```

That is all. I hope these advices and practice will be very useful for you.

[Medium](https://kopilov-vlad.medium.com/one-more-article-about-ruby-best-practice-dbce5ea4a7dd)
