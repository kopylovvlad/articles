# Recursion exercises in Ruby

![Photo by Mahkeo on Unsplash](image01.jpeg)

Every skilled programmer knows about recursion. It can be a powerful tool for writing algorithms to solve problems. In plain English, recursion is the calling of a function from within that same function. You divide one task into many little steps and perform it again and again.

Every recursion function should consist of:

1. Base recursion step
1. Exit condition
1. Limit for input value

For example, the task is: *You were given a natural number 'number'. Return a string with all numbers from 1 to 'number' using recursion separating them by spaces*. The function is:

```ruby
def foo(number, index = 1)
  # limit
  raise ArgumentError, 'number must be <= 10 000' if number > 10_000
  # exit
  return number.to_s if number == index
  # step
  index.to_s + ' ' + foo(number, index + 1)
end
```

The limit is critically important if you write code for production! Because, if you perform the function with a number is 500 000 you will get an error: '*Stack level too deep (SystemStackError)*'. You should avoid it in production.

Recursion is not a silver bullet. Every algorithm can be implemented without recursion. It may be hard to implement, but it has not SystemStackError and sometimes it is faster. Here is Fibonacci sequence example with and without recursion:

```ruby
# Fibonacci sequence with recursion
def fib(number)
  if number < 0
    raise ArgumentError, 'fib(n) defined for number>=0'
  end

  return fib(number - 1) + fib(number - 2) if number > 1
  number
end

# Fibonacci sequence without recursion
def fib2(number)
  raise ArgumentError, 'fib(n) defined for number>=0' if number < 0
  n0, n1 = 0, 1
  (0...number).each do
    n0, n1 = n1, n0 + n1
  end
  n0
end
```

If you want to try solve tasks with recursion, here is GitHub repository with exercises and answers: [https://github.com/kopylovvlad/ruby_recursion_exercises](https://github.com/kopylovvlad/ruby_recursion_exercises)
Happy coding 😀

[Medium](https://kopilov-vlad.medium.com/recursion-exercises-in-ruby-c5d3189c5a80)
