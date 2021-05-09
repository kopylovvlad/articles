# Algorithms for Ruby developer: Binary search

<!--- Photo by <a href="https://unsplash.com/@jankolar?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Jan Antonin Kolar</a> on <a href="https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
   --->

![image01](image01.jpg)

Every software developer has heard about classic old-school algorithms. If you ask the question ”_Do we really need to study algorithms nowadays?_” the article is for you.

When we discuss algorithms, the binary search algorithms immediately comes to my mind. If you didn’t hear about it or want to refresh your knowledge, I suggest you to watch the [video CS 50](https://www.youtube.com/watch?v=YzT8zDPihmc).

So, do we really need to know and remember how does the binary search work and how to implement it, if we have already had `Array#find_index` method in Ruby?

## Experiment

Let’s write a little experiment just for fun. My plan is to implement the binary search algorithm in plain Ruby. There is our `Main` module and a custom function `#b_index`

```ruby
# script.rb
module Main
  # @param array [Array<Object>]
  # @param value [Object]
  # @return [Integer] index of element or -1
  def self.b_index(array, value)
    low = 0
    high = array.size

    while low < high
      mid = ((low+high) / 2).floor
      midval = array[mid]
      if midval < value
        low = mid+1
      elsif midval > value
        high = mid
      else
        return mid
      end
    end
    -1
  end
end
```

Let’s write some tests in order to check different cases.

```ruby
# spec/test1_spec.rb
require 'rspec'
require_relative '../script'

RSpec.describe 'Script' do
  context 'array with integers' do
    it { expect(Main.b_index([], 1)).to eq(-1) }
    it { expect(Main.b_index([1,2,3,4], 1)).to eq(0) }
    it { expect(Main.b_index([1,2,3,4], 2)).to eq(1) }
    it { expect(Main.b_index([1,2,3,4], 3)).to eq(2) }
    it { expect(Main.b_index([1,2,3,4], 4)).to eq(3) }
  end

  context 'array with similar values' do
    it { expect(Main.b_index([2,2,2], 2)).to eq(1) }
    it { expect(Main.b_index([1,2,2,2,13], 2)).to eq(2) }
  end

  context 'array with string values' do
    it do
      expect(Main.b_index(['a', 'b', 'c'], 'z')).to eq(-1)
      expect(Main.b_index(['a', 'b', 'c'], 'a')).to eq(0)
      expect(Main.b_index(['a', 'b', 'c'], 'b')).to eq(1)
      expect(Main.b_index(['a', 'b', 'c'], 'c')).to eq(2)
      expect(Main.b_index(['a', 'b', 'c'], 'z')).to eq(-1)
    end
  end
end
```

So, it works! All tests are passed. Let’s write another test case and compare our algorithm and `Array#find_index`

```ruby
# spec/test2_spec.rb
require 'rspec'
require_relative '../script'

RSpec.describe 'Match with Array#find_index' do
  # it runs Array#find_index and match result with value
  def test_helper(array, value)
    answer = array.find_index(value)
    answer.nil? ? -1 : answer
  end

  (1..100).to_a.each do |i|
    context "array from 0 to #{i}" do
      arr = (0..i).to_a
      arr.each do |j|
        number = j
        it "can find #{number}" do
          expect(Main.b_index(arr, number)).to eq(test_helper(arr, number))
        end
      end
    end
  end
end
```

So, it works without bugs! 🐞

## Benchmark and result

The most interesting is to write a benchmark test between our function and `Array#find_index` method.

```ruby
# race.rb
require_relative './script'
require 'benchmark'

# we will create an array with 50_000 numbers
i = 50_000
puts "test an array from 0 to #{i}"
arr = (0..i).to_a

Benchmark.bm do |x|
  # benchmark for `Array#find_index`
  x.report('standard find_index') do
    arr.each { |number| arr.find_index(number) }
  end

  # benchmark for our custom function
  x.report('my b_index') do
    arr.each { |number| Main.b_index(arr, number) }
  end
end
```

Let’s run it and watch the result.

```bash
$ bundle exec ruby race.rb
test an array from 0 to 50000
       user     system      total        real
standard find_index  7.970743   0.003301   7.974044 (  7.976267)
my b_index  0.061279   0.000458   0.061737 (  0.062213)
```

It’s fantastic that our algorithm is faster than `Array#find_index` method. The reason is `Array#find_index` uses [linear search algorithm](https://en.wikipedia.org/wiki/Linear_search) and it has the bit O notation `O(n)`. The binary search has performance `O(log n)`.

## Not only numbers

Numbers and strings are boring. Can it match custom objects? Of course, yes!

```ruby
# spec/test3_spec.rb
require 'rspec'
require_relative '../script'

# our custom Object
class MyObject
  include Comparable

  # @param number [Integer]
  def initialize(number)
    @number = number
  end

  attr_reader :number

  def <=>(object)
    if object.is_a?(self.class)
      number <=> object.number
    elsif object.is_a?(Integer)
      number <=> object
    else
      raise "Can_not_compare_the_type"
    end
  end
end

# and tests
RSpec.describe 'Custom objects test' do
  # it runs Array#find_index and match result with value
  def test_helper(array, value)
    answer = array.find_index(value)
    answer.nil? ? -1 : answer
  end

  (1..100).to_a.each do |i|
    context "array from 0 to #{i}" do
      arr = (0..i).map { |i| MyObject.new(i + 1) }
      arr.each do |obj|
        number = obj.number
        it "can find MyObject with #{number}" do
          expect(Main.b_index(arr, number)).to eq(test_helper(arr, number))
        end
      end
    end
  end
end
```

Great, it works! And you see that the binary search can improve performance in your application when you try to find data in a huge array.

## Conclusion

Knowing algorithms isn’t a dummy check and it can be very useful especially for optimisation. I hope the article will help you to improve performance in your application. I wish the article inspired you and you will keep on studying algorithms ✨

Link to source code is [here](https://github.com/kopylovvlad/b_search_ruby)

[dev.to](https://dev.to/kopylov_vlad/algorithms-for-ruby-developer-binary-search-1e16)
