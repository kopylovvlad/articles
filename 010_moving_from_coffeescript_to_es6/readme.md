# Moving from Coffeescript to ES6

![image01](image01.jpeg)

After one year of using coffeescript, I started using ES6. Coffeescript is language that transcompiles to JavaScript. It has own syntactic sugar inspired by Ruby and Python.

ES6 is more powerful language standard than coffee. It has the same functional and some new features like constants, promises, proxying and others. Now all browser support es6 natively and you don’t need using a pre-processor like Babel.

Below, I wrote some examples of using es6-features and some alternative in coffeescript. I hope, this article will be useful for all developer that still use coffee and want to moving to ES6.

## The same features

### Function definition

ES6 has default values for function parameters like coffeescript. We can rewrite value of any variable if no value or undefined is passed. Also we can use spread for places any variables into a array.

coffee:

```coffee
square = (x = 0) ->
  x * x
sum = (x = 0, y = 0) ->
  x + y
console.log(square(2))
console.log(square())
console.log(sum(10, 5))
console.log(sum(3, 7))

name = 'John'
lastName = 'Smith'
makeUser = ({ name, lastName })->
  user = { name, lastName }
console.log(makeUser({ name, lastName }))

adding = (numbers...)->
  sum = 0
  for number in numbers
    sum += number
  sum

console.log(adding(1,2,3,4,5))
console.log(adding(14,16,9))
```

ES6:

```js
'use strict'
const square = (x = 0) => {
  return x * x
}
const sum = (x = 0, y = 0) => {
  return x + y
}
console.log(square(2))
console.log(square())
console.log(sum(10, 5))
console.log(sum(3, 7))

let name = 'John'
let lastName = 'Smith'
const makeUser = ({ name, lastName }) => {
  let user = { name, lastName }
  return user
}
console.log(makeUser({ name, lastName }))

const adding = (...numbers) => {
  let sum = 0
  for (let number of numbers) {
    sum += number
  }
  return sum
}

console.log(adding(1,2,3,4,5))
console.log(adding(14,16,9))
```

### String Interpolation

It is feature for inserting value of any variable into string.

coffee:

```coffee
say_hello = (name, job_position) ->
  "Hello. My name is #{name}. And I am a #{job_position}"
console.log(say_hello('John', 'developer'))
console.log(say_hello('Eva', 'designer'))

cat =
  name: 'cat'
  say: 'meow'
say = (animal) ->
  "#{animal.name} says #{animal.say}"
console.log(say(cat))
```

ES6:

```js
'use strict'
const say_hello = (name, jobPosition) => {
  return `Hello. My name is ${name}. And I am a ${jobPosition}`
}
console.log(say_hello('John', 'developer'))
console.log(say_hello('Eva', 'designer'))

let cat = {
  name: 'cat',
  say: 'meow'
}
const say = (animal) => {
  return `${animal.name} says ${animal.say}`
}
console.log(say(cat))
```

### Classes

ES6 has useful class definition more intuitive than ES5

coffee:

```coffee
class Animal
  constructor: (name)->
    @name = name

  walk: ()->
    console.log("#{@name} is walking")

class Rabbit extends Animal
  walk: ()->
    super()
    console.log("...and jumping!")

new Animal('John').walk()
new Rabbit('Bob').walk()
```

ES6:

```js
'use strict'
class Animal {
  constructor (name) {
    this.name = name
  }

  walk () {
    console.log(`${this.name} is walking`)
  }
}

class Rabbit extends Animal {
  walk () {
    super.walk()
    console.log("...and jumping!")
  }
}

new Animal('John').walk()
new Rabbit('Bob').walk()
```

## New features

### let and cost

Let and const are new variable types. Const is constants immutable variables, you can not change it value. Let is standard variable, but it is not hoisted.

Hoisted bug in coffee:

```coffee
_loadDataForUser = (url, callback) ->
  setTimeout(callback, 2 * 1000)

loadDataForUsers = (userNames) ->
  for i in userNames
    _loadDataForUser "users/#{userNames[i]}", ()->
      console.log('Fetching for ', i)

loadDataForUsers(['John', 'Eve', 'Oleg'])
# Fetching for  Oleg
# Fetching for  Oleg
# Fetching for  Oleg
```

Hoisted bug in ES6 with using var:

```js
'use strict'
const _loadDataForUser = (url, callback) => {
  setTimeout(callback, 2 * 1000)
}
const loadDataForUsers = (userNames) => {
  for (var i in userNames) {
    _loadDataForUser(`users/${userNames[i]}`, function () {
      console.log('Fetching for ', userNames[i])
    })
  }
}
loadDataForUsers(['John', 'Eve', 'Oleg'])
// Fetching for  Oleg
// Fetching for  Oleg
// Fetching for  Oleg
```

Let and const in ES6:

```js
'use strict'
const _loadDataForUser = (url, callback) => {
  setTimeout(callback, 2 * 1000)
}
const loadDataForUsers = (userNames) => {
  for (let i in userNames) {
    _loadDataForUser(`users/${userNames[i]}`, function () {
      console.log('Fetching for ', userNames[i])
    })
  }
}

loadDataForUsers(['John', 'Eve', 'Oleg'])
// Fetching for  John
// Fetching for  Eve
// Fetching for  Oleg

const COUNT = 4
// const COUNT = 2 raise SyntaxError: Identifier 'COUNT' has already been declared
```

### Set()

Sets are new data structure. It similar to arrays (lists of values), but contain only unique values.

ES6:

```js
'use strict'
let groceryList = new Set()
groceryList
  .add('Bananas')
  .add('Apples')
  .add('Grapes')
  .add('Kiwis')
  .add('Pears')
  .add('Apples') // dublicates will be ignored

console.log(groceryList.size === 5)
console.log(groceryList.has('Kiwis') === true)

for (let item of groceryList) {
  console.log(item)
}

console.log(groceryList.size === 5)
groceryList.delete('Kiwis')
console.log(groceryList.size === 4)

groceryList.clear()
console.log(groceryList.size === 0)
```

coffee:

```coffee
groceryList = {}
groceryList['Bananas'] = true
groceryList['Apples'] = true
groceryList['Grapes'] = true
groceryList['Kiwis'] = true
groceryList['Pears'] = true
groceryList['Apples'] = true # dublicates will be ignored

console.log(Object.keys(groceryList).length == 5)
console.log(groceryList['Kiwis'] == true)

for key of groceryList
  console.log(key) if groceryList.hasOwnProperty(key)

console.log(Object.keys(groceryList).length == 5)
delete groceryList['Kiwis']
console.log(Object.keys(groceryList).length == 4)

for key of groceryList
  delete groceryList[key]
console.log(Object.keys(groceryList).length == 0)
```

### Map()

Maps are like objects. It contains key/value pairs. But it can store not only string for key.

ES6:

```js
'use strict'
let groceryList = new Map()
groceryList.set('Bananas', 6)
groceryList.set('Apples', 12)
groceryList.set('Grapes', 2)
groceryList.set('Kiwis', 10)
groceryList.set('Pears', 8)

console.log(groceryList.get('Kiwis') === 10)
console.log(groceryList.size === 5)

for (let [ key, val ] of groceryList.entries())
  console.log(`${key} = ${val}`)

groceryList.delete('Apples')
console.log(groceryList.size === 4)

console.log(...groceryList.values())
console.log(...groceryList.keys())

groceryList.clear()
console.log(groceryList.size === 0)
```

coffee:

```coffee
groceryList = {}
groceryList['Bananas'] = 6
groceryList['Apples'] = 12
groceryList['Grapes'] = 2
groceryList['Kiwis'] = 10
groceryList['Pears'] = 8

console.log(groceryList['Kiwis'] == 10)
console.log(Object.keys(groceryList).length == 5)

for key, val of groceryList
  console.log("#{key} = #{val}")

delete groceryList['Apples']
console.log(Object.keys(groceryList).length == 4)

values = []
for key, val of groceryList
  values.push(val)
console.log(values)
console.log(Object.keys(groceryList))

for key of groceryList
  delete groceryList[key]
console.log(Object.keys(groceryList).length == 0)
```

That is all. If you are inspired by it and want to read more information, I recommend these addition links:

* [ES6: New Features](http://es6-features.org/)
* [You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS)

[Medium](https://kopilov-vlad.medium.com/moving-from-coffeescript-to-es6-b46222ab479a)
