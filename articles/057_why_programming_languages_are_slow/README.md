# Why programming languages are slow

![Photo by Pascal van de Vendel on Unsplash](image01.jpg)

## Introduction

[Grigory Petrov](https://twitter.com/grigoryvp) from [Evrone](https://evrone.com/) has a couple of fascinating talks on YouTube about programming languages. He had learned a lot about programming languages design and how compilers work under the microscope and he has found answers to questions about programming languages performance.

Here is my overview of his lectures ([1](https://www.youtube.com/watch?v=m73VjmjAnuw), [2](https://www.youtube.com/watch?v=39XNklRQJI4)) in English. If you try to find an answer to the question of why Python or Ruby is slow, the article is for you.

## The main question

Some people don’t want to learn and use Ruby, Python, etc because they have heard that these languages are slow. “_Why do I need to use slow language_” - they think. “_It’s better to learn and use golang or javascript because I have heard they are fast._” Also, some junior programmers who have learned Ruby, Python worry about their choice. Did they make the right choice, because they have heard that the language is slow? Let’s find an answer to why some languages are slow.

## CPU

![Photo by Jeremy Zero on Unsplash](image02.jpg)

In order to describe how programming languages work we must begin with the CPU on a machine. Modern CPU has plenty amount of cores for its performance. Modern CPU architecture is challenging to learn and describe. In simple terms, we can say that the code execution speed is the amount of machine code instructions that one CPU core can exec by one moment.

CPU reads each instruction from memory. Read operation is always slow operation. Therefore each modern CPU hash multi-level cache (L1, L2, Ln cache) and processor registers. They help not to read data from memory.

In simple terms, code speed performance is equal to how effective our code (machine code) works with memory. Can we store data in CPU cache or do we have to read data from memory?

## Take a look at languages

What programming languages as C, C++, Rust, Objective-C and Golang have in common? When a programmer writes code he always thinks about memory: we must specify the data type of each variable; we have to allocate memory in heap; always think about pointers, blocks; etc.

As an advantage, the source code will be compiled to machine code and it’s executed so fast. Nevertheless, everyone knows that write code in C, C++, Rust is challenging. It happens because the syntax is tricky and you have to always care about memory.

If a programmer/developer doesn’t want to worry about the memory he can delegate the routine to a compiler. A compiler is a tool that transforms your source code into machine code. it tries to do it effectively. Source code converted to machine code will be easily handled by the CPU and data will be stored in cache and registers

Programming languages as Java, C#, Javascript follow the way. When you write code in Java you don’t have to worry about memory because you delegate chores to a compiler.

Java, C#, Javascript have high-level and good-looking syntax. The code is still executed fast. Unfortunately, they have problems with extensibility. The compiler will convert source code and isolate the memory layer. It’s hard to write an extension for Java code or use third-party code written on C/C++. In order to write an extension designer had to implement an interface as Java Native Interface (JNI), but using it will decrease your code performance.

The third way is to delegate routine to runtime, virtual machine (VM). It means that the language will not compile source code to machine code, it will compile the source code to bytecode and execute the bytecode in VM. Programming languages as Python, Ruby, PHP follow the way. These languages have awesome high-level and sweet syntax and the ability to write an extension easy and fast. But the price of using a virtual machine is speed - code performance is slow.

## Three skills

![Photo by Lucas Santos on Unsplash](image03.jpg)

We can imagine each computer language as a character in a video role-playing game (RPG). In a typical RPG when we create a character we have a limited amount of points and few skills to put points into (strength, defence, intelligence, agility etc). Each language has three skills to put points into:
* Speed - speed of execution
* Syntax - enjoyable and elegant high-level syntax
* Extensibility - memory compatibility in order to easy write and use third-party code or an extension.

When a person, group of people start to design a new programming language they can choose only two skills as the base. A third one will always be hard to achieve. For example, C, C++, Rust, Objective-C, Go are fast and have extensibility, but they have low-level inelegant syntax.

If a language is quick and has nice syntax - it will be hard to write an extension or use third-party code. Java, C#, Javascript are fast and have good high-level syntax. But, it’s hard to write an extension for Java code, or use third-party code written on C/C++.

Ruby, Python and PHP have chosen the third way. They have awesome sweet syntax, the code is easy to write. When you are writing the code you think only about business logic and don’t worry about memory. Also, it’s easy to use third-party code or write an extension as NumPy, SciPy in Python; or Nokogiri, Ruby-openCV, Redcarpet in Ruby. The price is performance these languages are slower than previous languages.

## Conclusion

![Photo by David Heslop on Unsplash](image04.jpg)

Whenever we say or hear that N language is slow remember that it was a meaningful decision in language design. You can ask a question: “_How to became N language fast? Is it possible?_”. Yes, it is possible but not easy to achieve.

Otherwise slow languages are flexible. It’s easy to write an extension in C++ and make some type of operation faster. For example NumPy and SciPy in Python. When we do ML in Python or use Ruby in order to write business login we use advantages of Python/Ruby in a high-level language and write the code fast. Also, we can delegate slow operation to swift extensions in third-party libraries in order to work with data, do math operations, parse text, etc.

Frankly speaking, nowadays Javascript is the only one language that tries to have all three skills. But it’s hard. It tries to spend all resources to be harmonious. I believe that in near future all modern languages will be in harmony will all three skills.

[dev.to](https://dev.to/kopylov_vlad/why-programming-languages-are-slow-1b2d)
