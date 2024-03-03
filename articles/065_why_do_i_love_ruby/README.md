# Why I love Ruby

<!-- Photo by <a href="https://unsplash.com/@ashlynnephotos?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Ashley Levinson</a> on <a href="https://unsplash.com/photos/a-red-heart-shaped-object-sitting-on-top-of-a-table-_x9IVPFH4VI?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a> -->

![image_01](./063_image01.jpg)

I'm a backend developer with more then 8 years of programming experience. As a backend developer, I can use different languages to solve my tasks, therefore I've used many of them. I've explored various options and decided to focus on Ruby. In this essay, I want to explain why I prefer using Ruby for my work tasks, discussing its pros, cons, features, and highlighting why it remains a relevant language in 2023.

## 1 The Reason for the creation

![image02](./063_image02.jpg)

I start the discussion about a programming language with its history. The development of programming languages often serves a utilitarian purpose. For example, C was created to provide a tool at a higher level of abstraction than assembly language. PHP was developed as macros over C to quickly create personal web pages. Python was designed as a simple tool for system administration, aiming to replace bash scripts and C for the Amoeba OS. Some languages were created specifically for some runtime environment.

In Japan, Yukihiro Matsumoto, a programmer with lots of experience in different languages, was looking for a language that suited well. He wanted to spend 8-10 hours a day for programming with comfort. Not finding the perfect tool among existing languages, he decided to create his own - a language that he would find comfortable and enjoyable to work with. His goal was to make a 'true object-oriented language'. The first version of Ruby was born in 1995. Now, let's explore the strengths and weaknesses of this technology.

## 2 Strengths of the technology

![image03](./063_image03.jpeg)

### 2.1 Syntax

![image04](./063_image04.jpeg)

In pursuit of his goal, Matz has paid close attention to the syntax and object structure. Ruby inherited the object model from Smalltalk 80. It gained scripting capabilities and advanced regex support from Perl. The result was a strict object-oriented language where everything is an object and a method. Ruby's syntax gives you many possibilities, emphasizing syntactic sugar. There are many features such as function calls without parentheses, "code blocks" (similar to anonymous functions that can be passed to any method), and metaprogramming (code that writes code). The code in Ruby language can be read like a text in English.

Ruby has a lot of abilities with its syntax, it can be a bit harder for newcomers, unlike languages with simpler syntax such as Python. Ruby doesn't have only simple and straightforward setups like Python or Golang (in those languages, if you want to do something, there's usually just one way to do it). Someone who has just started to use Ruby might find its syntax a bit complicated. For instance, there are many methods that basically do the same thing, just with different names—called 'aliases' (`filter` and `select`, `exit` and `quit` do the same. They are here only for convenience.). There are also methods that do similar things but with slight different behavior (`each` and `each_with_index`). Despite this, Ruby lets you overload operators; it allows you to declare how unary operators behave for your classes (the feature has not found in many languages). Also, Matz have did a lot to make error messages easy to understand for developers. If you got an exception, you won't get a super long backtrace like in Erlang or Java, nor a cut-off one like in NodeJS ([default is 10]((https://nodejs.org/api/errors.html#errorstacktracelimit))).

Thanks to these cool features, the new language has attracted the interest of people all around the world.

### 2.2 Domain-Specific Language (DSL)

![image05](./063_image05.jpeg)

The emphasis on syntactic sugar, function calls without parentheses, and "code blocks" have given Ruby the capability to create its own little language for a specific domain (DSL). While building a modern application in Ruby, a programmer doesn't write in pure Ruby but uses small task-specific dialects for things like testing, routes declaration or writing database migration scripts. It simplifies the code, provides a high level of abstraction and the code reads like English text. Therefore, creating a program in Ruby, you typically need [about 3-4 times less code](https://syndicode.com/blog/why-is-ruby-still-our-choice-in-2020-2/) than in Java or Python.

When I saw a Ruby code for the fist time, I was amazed by its conciseness. Having experience in C, C++ and PHP, I was considering Ruby like LEGO bricks. The fact that you could achieve a lot with just a little code reminded me of jQuery's motto, 'write less, do more,' and it has inspired me to learn Ruby deeper.

### 2.3 Web development

![image06](./063_image06.png)

Ruby is a general-purpose language. You can use it to write system administration scripts, program electronics and more, but Ruby has evolved the most as a web technology thanks to the RubyOnRails framework.

In 2005, David Heinemeier Hansson (DHH), an experienced web developer, uploaded a video on YouTube showing how he creates a blog in 15 minutes ([video](https://www.youtube.com/watch?v=Gzj723LkRJY)). This presentation of the RoR framework was a revolution for its time. Rails quickly gained popularity in contrast to the development speed of other stacks. In the same year, Java ex-developer Bruce Tate released the book [Beyond Java](https://www.oreilly.com/library/view/beyond-java/0596100949/), discussing the drawbacks of Java and highlighting RoR as a big thing in IT. The peak of Ruby's popularity in the web domain lasted from 2007 to 2013. This peak gave rise to an extensive ecosystem of production-ready solutions and libraries for web programming and beyond.

The RoR is an  example in the tech world when the tool is highly opinionated. It means that using this technology, it dictates you the project structure, where different parts of the application should be located, how components should be named and more. The framework has its own [doctrine](https://rubyonrails.org/doctrine). This approach allows to think less about application configurations, but it comes with both pros and cons, which I'll explain further.

RoR is a still full-stack framework. It allows you to quickly set up a web service from scratch, providing default tools for building the user interface. People say that programming in Ruby is on average 30-40% faster than in other languages. Moreover, many Ruby developers are full-stack developers, it reduces costs for businesses. The development speed doesn't compromise quality. The time saved can be invested in refactoring (a desire for developers but not always aligned with business priorities), or in updating libraries, contemplating architecture, or seeking better abstractions. In reality, developers often have lack of time to write code with high quality.

For testing business ideas Ruby has became to be the ideal tool. Despite the status quo that Ruby is slow, people forget its strong suit — the speed of development. Considering that for a business needs to quickly adjust to external changes to survive, using Ruby allows you to quickly adapt your application to new tasks or changes. That's why a significant portion of Y Combinator startups is launched using RoR.

### 2.4 Standardization

![image07](./063_image07.jpeg)

The core of the Ruby ecosystem focuses on the single framework (RoR). It gives us a big amount of tools for it. For example: test writing is standardized, the library for web interfaces is standardized too (Rack). In the infrastructure level, the application server doesn't care whether it's a Rails application or another Ruby web-framework. Code style is also standardized by rubocop. Therefore, a new developer on a new project won't be confused of the code style (a similar situation occurs in Python and Go, while in some languages, you need to [choose from N options](https://betterprogramming.pub/comparing-the-top-three-style-guides-and-setting-them-up-with-eslint-98ea0d2fc5b7)). Many third-party libraries are already adapted for Rails, it reduces the time for code integration into the application. There are no situations where adding an external library requires resolving dependency conflicts or spending time configurating the libraries. This situation gives us the ability to hire a new Ruby developer who can quickly understand the application. Developers already know the folder structure, namespaces and so on. On the negative side, some people get bored, that all applications look the same, but there are other frameworks available (but less popular).


### 2.5 Knowledge Base on Application Development

![image08](./063_image08.jpeg)

Over the years of its popularity and usage, the community has built an large knowledge base on how to write web applications. Because massive applications like Cookpad, GitHub, GitLab, Shopify, BaseCamp and Stripe are written in Ruby. The community has gained expertise in developing monoliths that can scale in the future and how to maintain them long-term.

The Rails community promotes the concept of the [majestic monolith](https://m.signalvnoise.com/the-majestic-monolith/). To avoid the challenges of a extremely big application, a methodology has been crafted on how to write monoliths, build architectural namespaces and prevent the need for later rewriting into microservices. This prevents potential issues like ending up with a distributed monolith or and have to write a lot of schemas describing how microservices communicate and monitor the paths of requests. Additional solutions can be explored by attending community meetings, conferences, or reading blogs from experienced developers.

The technology also follows all security concerns. Ruby is well-suited for applications that handle personal data, such as financial platforms and marketplaces. The ecosystem follows Security Development Lifecycle to control vulnerabilities also there are solutions for the license checking. Rails 7 has out of the box support for encrypting database data. For this reason, fintech products are also launched on Ruby. In this case, the situation is close to Java and the C languages.


### 2.6 A tool to solve business problems

![image09](./063_image09.jpeg)

When you write code in Ruby, you don't often worry about things that colleagues using other languages keep in mind, such as type declarations, memory usage and etc. Typically, you don't have to worry about memory and other low-level details, spending time explaining to the computer what to do with memory. Plus, routine tasks (routing, database structure changes, tests etc.) aren't programmed but they are described by DSL. That's why Ruby has become a convenient language for writing business logic and implementing new features. This suits well with what most developers are currently engaged in. 8 out of 10 developers are involved in business automation or solving business tasks by the code. RoR guides the developers on how to do it and dictates the structure, allowing the developers to focus on solving the actual business problem, writing code specifically for it, rather than "struggling" with the framework.

However, I want to add that if you're working on atypical tasks, you might find yourself "struggling" with the framework or dealing with legacy issues, like on other languages. Fortunately, in RoR, these limits are broad enough, so you rarely step outside them, especially with considerable programming experience.

## 3 Technology Weaknesses

![image10](./063_image10.jpeg)

Now, let's discuss the weaknesses of the technology or the aspects it's criticized for.

### 3.1 Application Performance

Every developer has heard that Ruby is slow, but in reality, many languages are considered as "slow". The execution speed of a computer program depends on memory management features. It's a feature of all technologies where memory management is delegated to a virtual machine (VM). For more details, you can watch Grigory Petrov's presentation [Why is Ruby slow? (rus)](https://www.youtube.com/watch?v=m73VjmjAnuw), based on which I wrote a [short article](https://dev.to/kopylov_vlad/why-programming-languages-are-slow-1b2d).

For many developers, all languages which labeled as "interpreted" (or scripting) are considered as slow because the interpreter reads the program's source code line by line and executes it. However, in 2023, modern "interpreted" languages are no longer strictly interpreted. Languages like Ruby and Python are compiled into bytecode and executed in a VM. In Ruby since version 1.9 (2011), the interpreter precompiles Ruby into bytecode before execution. With Ruby 2.6 (2018), Just-In-Time compiler (JIT) was introduced, it compiles frequently used instructions into binary code (similar to nodeJS). This approach significantly speed up mathematical calculations by tens of times, bit it struggles to speed up complex projects. Matz paid the price for it because he prioritized on large and diverse syntax feature not speed.

Also in the core of Ruby there is Global Interpreter Lock mechanism (GIL). It slows down program execution as it needs to pause the execution to clean up memory. This is inherited from the time when computers had single-core processors. Although the language has evolved, now Ruby has the Global Virtual Machine Lock (Global VM Lock) inside. Fortunately, technology continues to evolve. In places where the Global VM Lock causes a problem, developers can use features like Fiber, Ractor and others. If your goal is to write a speedy "number calculator" (a script for matrix multiplication, machine learning (ML) or rendering calculations), it's better to replace Ruby with languages where memory management management is delegated to a developer. That's why Ruby isn't the best choice for ML. However, in web development significant portions of time is spent on I/O (waiting for responses from the database or HTTP connections). For these cases your can use Threads where it's critical. For example, Rails 7 introduces parallel execution of database queries with [load_async](https://blog.skylight.io/rails-7-load_async/).

In 2019 Basecamp (the main developer of RoR) [published an article](https://m.signalvnoise.com/only-15-of-the-basecamp-operations-budget-is-spent-on-ruby/) which stating that only 15% of the operational budget is spent on executing Ruby code. If you write and configure your application well, you don't notice the slowdowns. Even if they spent a lof of resources to make the language twice as fast, it wouldn't significantly reduce the company costs.

The code execution speed isn't critical for every application. You notice slowdowns if your application processes hundred requests per second. Rare developer face the same load in the initial stages of creating an application/product. If your MVP starts with less than hundred requests that's quite good. If more, RoR can handle it because the application is scalable by increasing amount of processes and threads.

In computer science, there are areas where execution speed matters: parsing XML, working with encryption and more. For this tasks, Ruby is suitable because the language allows to write native extensions in C++. Popular libraries for these tasks are written in C++ and Ruby provides only a wrapper/interface. The similar situation exists in Python in the ML area. Python code doesn't handle numbers or matrix multiplication, it only manages the data flow. All mathematical operations are performed by C++ extensions. The Python community spent a lot of time to make this real.

### 3.2 Other Usage Areas

Historically, Ruby's main usage area is web applications. If you need ML or machine vision, Python's ecosystem is much richer. For data analysis R/Python are best suited. However, it doesn't mean that you can't do simple data calculations with Ruby. Also there are domains where one technology dominates by historical factors: frontend, mobile development, etc. In reality, Ruby is not limited to the web. If you're curious, check out how it's used in its home country, Japan. There's a huge [Ruby community](https://www.jetbrains.com/lp/devecosystem-2021/ruby/) in Japan and historically, Ruby is more than just Rails. They use it to programming controllers (with mruby), as a scripting language for other systems, in search engines and so on. If you wish talks about Ruby, not just Rails, feel free to explore [RubyKaigi](https://www.youtube.com/@rubykaigi4884/videos).

Ruby is actively used in infrastructure tasks. The popular utility for installing applications and packages on macOS [HomeBrew](https://brew.sh/) is written in Ruby, which is why Ruby comes by default on macOS. Additionally, if you look at sysadmin job listings, experience with Ruby is often among the requirements. There are also solutions/libraries for other platforms, such as gamedev and frontend. While they might not be very popular, you can explore and play with them if you're interested.

### 3.3 Type Annotations

Languages like Javascript and Python have already had this, but what about Ruby? For documentation there is [Yard](https://rubydoc.info/gems/yard/file/docs/GettingStarted.md), but these annotations are only at the documentation level. For writing and checking types, there is a tool called [Sorbet](https://github.com/sorbet/sorbet) from Stripe. In Ruby 3.0 native type annotations (RBS) have been introduced, but they are written in separate files (similar to C and TypeScript). Types are kept in separate files because Matz believes that developers should focus on business features rather than writing types. He is confident that in nearest future, programming languages will move away from manually writing types towards tools for automatically generating signatures. Work in this direction has started with tools like [TypeProf](https://github.com/ruby/typeprof).

### 3.4  Learning Curve

Ruby learning curve is higher to compare to other languages. Fortunately, Ruby is often not the first programming language for a big part of developer because its syntax with rich capabilities might be confusing for beginners. That's why newcomers often started with Python or Go, where the constructs are simpler. After gaining initial experience, they tend to explore other languages and explore the capabilities of different technologies. Many Ruby programmers are already experienced in programming before they embrace Ruby. Thankfully for newcomers, Rubocop guarantees a unified coding standard to writing code correctly and with best practices.

## 4 Hiring

![image11](./063_image11.jpeg)

Thanks to the standardization I mentioned earlier, people coming to your project will be familiar with the technologies, significantly reducing the search time. This is important for hiring a new team member. In the onboarding, new developers have to only understand the business domain. With other languages, the situation is different. Looking for a Python or PHP developer, we reject some candidates because they've worked with a different framework and during the interview, we need to understand the technologies they are familiar with. Similarly, when hiring a Python developer, we must understand their specialization: backend, automated testing, ML, data science, ETL, etc. That's why having just one Python developer may not be sufficient to launch an application.

In contrast, most RoR developers are full-stack specialists who can launch a web application on your own. Some of them can solve frontend tasks, some of them follow the latest technologies such as [HTML over websockets](https://hotwired.dev/). However, there are fewer Ruby developers on the market compared to Node.js/Python, and they often ask for higher salaries.


## 5 Community

![image12](./063_image12.jpeg)

I'd like to add what is behind any technology. Of course, there are people and Ruby is famous for its community. The community isn't large as in some other languages due to a smaller amount of developer (though history has examples where the core of an ecosystem was created by [just one person](https://medium.com/@kelas/how-is-tj-holowaychuk-so-insanely-productive-604818b4e9eb)), it is influential in the open-source world. Projects like GitHub has been released to thanks to the Ruby community. The fundamental principle of the Ruby community is sharing code and ideas, therefore RoR has the highest number of contributors compared to other technologies. Perhaps only the frontend community has a similar obsession for open source.

The community follows Matz's idea that Ruby was created for comfort and fun. How else would you explain methods in RoR arrays like `second`, `third`, `second_to_last`, `third_to_last`, and suddenly `forty_two`? Moreover, when Ruby developers switch into other languages, they often bring ideas from the Ruby world. For example, [dry-python](https://github.com/dry-python) and [pyenv](https://github.com/pyenv/pyenv) were influenced by concepts from Ruby. The Ruby ecosystem has inspired other languages; for instance, RoR inspired PHP developers to create [Laravel](https://github.com/laravel/laravel), and Rspec inspired JavaScript developers to build libraries like [mocha](https://github.com/mochajs/mocha), [jasmine](https://github.com/jasmine/jasmine), and [Chai](https://github.com/chaijs/chai).

Ruby's community was fortunate to create a canonical, convenient, and high-quality dependency manager, [Bundler](https://bundler.io/), in 2009. It's a unified, deterministic dependency installer with a lock file, support for multi-environments, and the ability to install dependencies directly from GitHub. Features that ecosystems like Go and NodeJS can't boast about. Of course, Bundler had its problems in the early years, such as speed issues, but they were successfully overcome. Bundler became the prototype for tools like [yarn](https://www.npmjs.com/package/yarn#prior-art) in JavaScript, `cargo` in Rust, and `composer` in PHP.

Ruby's ecosystem used to inspire others, nowadays the trend has shifted. Now Ruby adopts features from other languages. It has received pattern matching, similar to Elixir and Erlang; the transpiler [RubyNext](https://github.com/ruby-next/ruby-next) (as Babel in JS), allowing the use of new language features on previous Ruby versions, etc. Some solutions for the frontend are inspired by RubyOnRails, for instance: [sockpuppet](https://github.com/jonathan-s/django-sockpuppet) for Django is an implementation of [stimulus-reflex](https://github.com/stimulusreflex/stimulus_reflex) from Rails.

The community continues to expand and organize significant conferences worldwide, such as RubyKaigi (Japan), RubyRussia (Russia), Euruko (Europe), RailsConf/RubyConf (USA) and more.

## 6 Relevance of the Technology

![image13](./063_image13.jpeg)

The language and framework were highly popular from 2013 to 2015. However, since 2016, there has been a myth that "Ruby is dead." These claims make little sense because every technology goes through the same [hype cycle](https://tecedu.academy/storage/ckeditor-content/YWWop0OvNadq0RCkc0Fha42gs3Mo8p1QNhxEVA7r.png). Each technology experiences a peak of inflated expectations and people believe it's outdated, but, in reality, the technology enters the plateau of productivity. This stage indicates that the technology is mature enough for use in production. Java, for instance, hasn't died since 2005 and is still relevant in production.

Essentially, it's rare for a technology to die, especially in open source. After the the years of using, a lot of application has been built and each of it requires ongoing support and development. How else can we explain that Fortran is in 16th place in the [TIOBE](https://www.tiobe.com/tiobe-index/) ranking as of 2023? Moreover, how can we consider the actual relevance of technologies? To count the number of questions on Stack Overflow? This indicates the number of beginner developers learning. To count stars or the number of projects on GitHub? But GitHub is like a warehouse of building materials, and in some communities, it's not common to click on the star in GitHub. In bloody enterprise industries, libraries are not often publicly published. Job listings on websites? This indicates a shortage of specialists in this area at the moment. Even TIOBE itself is calculated based on [mentions in 25 search engines](https://www.tiobe.com/tiobe-index/programminglanguages_definition/). Therefore, determining actual relevance is indeed complex.

## Conclusion

![image14](./063_image14.jpeg)

Ruby was initially crafted to be the most user-friendly language for development and it has not changed its core priority. It has launched a rich, stable ecosystem with diverse solutions for a wide range of tasks. It's general-purpose language with great full-stack tools in web development. The community has created large base how to build production ready applications. The language has some "weaker" points, but you can overcome them if you understand how and why they came about.The community continues to evolve the language and the tools around it, observing and adopting features from other languages, making Ruby a relevant tool. That's exactly why it suits my needs perfectly after all these years.

Additionally, I agree with [Grigory Petrov's words](https://www.youtube.com/watch?v=L3YqKI5kcOk) that the Ruby language is an ideal choice for someone if:

* They understand that they enjoy programming and they want the process of writing code to be maximally enjoyable.
* Their area of interest is in helping businesses. Not gamedev, not frontend, not code for physical devices, not code for scientific calculations, not landing pages. Ruby and its ecosystem are well-suited for solving business problems using web applications, integration scripts and more.

If these are your specific needs, then choosing on Ruby is a good decision!

I ♥️ Ruby

[dev.to]()
