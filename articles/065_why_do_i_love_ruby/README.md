# Why I love Ruby

<!-- Photo by <a href="https://unsplash.com/@ashlynnephotos?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Ashley Levinson</a> on <a href="https://unsplash.com/photos/a-red-heart-shaped-object-sitting-on-top-of-a-table-_x9IVPFH4VI?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a> -->

![Why I love Ruby](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/95xa6pvhhf2zbkeatt9w.jpg)

I'm a back-end developer with over 8 years of coding experience. As a backend developer, I use lots of different languages to do my stuff, so I've tried a lot of them. I've done a lot of research and decided to focus my efforts on Ruby. This essay is about why I like using Ruby for work, and why it's still relevant in 2023.

## 1 The Reason for the creation

![Ruby - The Reason for the creation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/386x4wa82trkvkxzcurf.jpeg)

I start the discussion about a programming language by talking about its history. A lot of programming languages have been created for a practical purpose. For instance, C was made to be a more abstract tool than assembly code. PHP was created as a way to quickly make personal websites using C macroses. Python was developed as a simple tool for system administrator tool, replacing Bash/C scripts for the Amoeba operating system. Some languages were created specifically for some runtime environment.

In Japan, Yukihiro Matsumoto, a programmer with lots of experience in different languages, was searching for a language that suited well. He wanted to spend 8-10 hours per day comfortably coding. After not finding the perfect language among existing ones, he decided to make his own - a language that he would find comfortable and enjoyable to work with. His goal was to create a "true object-oriented language". The first version of Ruby came out in 1995, and now let's take a look at the pros and cons of this language.

## 2 Strengths of the technology

![Ruby Strengths of the technology](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hu5jp98gzu49hljos9li.jpeg)

### 2.1 Syntax

![Ruby - Syntax](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0boctewlcpudqxfwrfqk.jpeg)

In order to achieve his goal, Matz has paid close attention to the syntax and object structure. Ruby inherited the object model from Smalltalk 80. which added scripting capabilities and advanced regular expression support from Perl. As a result, Ruby became a strict object-oriented language where everything is an object with methods. Ruby's syntax offers a lot of flexibility, emphasizing syntactic sugar. There are many features such as function calls without parentheses, "code blocks" (similar to anonymous functions that can be passed to any method), and metaprogramming (code that writes code). The code in Ruby is easy to read like English text.

Ruby is very powerful with its syntax, but it can be a bit harder for newcomers, unlike languages with simpler syntax such as Python. Ruby isn't just simple and straightforward like Python or Golang (where if you want to do something, you usually just have one way to do it). For a new Ruby developers, the syntax can feel a bit confusing. For example, there are many methods that basically do the same thing, just with different names - they're called 'aliases' (`filter` and `select`, `exit` and `quit` do the same. They are here only for convenience.).

There are also some methods that do the same things but with slight different behavior (like `each` and `each_with_index`). But despite this, Ruby allows you to overload operators; so you can declare how unary operators work for your classes (this feature is not available in many languages). Also, Matz done a lot of work to make error messages easier for developers to understand. If you get an exception, it won't be a super-long backtrace like you get in Erlang or Java. It won't even be a truncated one like you get in NodeJS ([default is 10]((https://nodejs.org/api/errors.html#errorstacktracelimit))).

Thanks to those cool features, the language has caught the attention of people all over the world.

### 2.2 Domain-Specific Language (DSL)

![Ruby Domain-Specific Language](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wlxref2z9oifqc7185mz.jpeg)

Ruby has a lot of syntactic sugar with features: function calls without parentheses, "code blocks", etc. It gave us the capability to create its own little language for a specific domain (DSL). When building a modern app in Ruby, programmers don't write code in pure Ruby. They use small task-specific dialects for things like testing, routes declaration or writing database migration scripts. It simplifies the code, provides a high level of abstraction and the code reads like English text. As a result, creating a program in Ruby, you typically need [about 3-4 times less code](https://syndicode.com/blog/why-is-ruby-still-our-choice-in-2020-2/) than in Java or Python.

When I saw a Ruby code for the fist time, I was amazed by its conciseness. Having experience in C, C++ and PHP, I was considering Ruby like LEGO bricks. The fact that so much can be done with so little code really reminded me of the jQuery motto 'write less, do more,' and it inspired me to learn Ruby deeper.

### 2.3 Web development

![Ruby - Web development](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vrg6iexymx509c6mdkty.png)

Ruby is a general-purpose language. You can use it for all sorts of things, from writing system administration scripts to program electronics. Ruby has evolved the most as a web development tool thanks to the RubyOnRails framework.

In 2005, David Heinemeier Hansson (DHH), an experienced web developer, posted a video on YouTube about to make a blog in just 15 minutes using RubyOnRails ([video](https://www.youtube.com/watch?v=Gzj723LkRJY)). This presentation of the RoR framework was a revolution for its time. Rails quickly gained popularity in contrast to the development speed of other stacks. In the same year, Java ex-developer Bruce Tate released the book [Beyond Java](https://www.oreilly.com/library/view/beyond-java/0596100949/), discussing the downsides of Java and highlighting RoR as a big thing in IT. The peak of Ruby's popularity in the web domain lasted from 2007 to 2013. This peak gave rise to an extensive ecosystem of production-ready solutions and libraries for web programming and beyond.

The RoR is an example in the tech world when the tool is highly opinionated. It means that using this technology, it dictates you the project structure, where different parts of the application should be located, how components should be named and more. The framework has its own [doctrine](https://rubyonrails.org/doctrine). This approach is great because it means you don't have to think about all the configurations for your app. But it also has its downsides, which I'll talk about later.

RoR is a still a full-stack framework that lets you quickly set up a web service from scratch with base tools for building the user interface (UI). Some people say that programming in Ruby is on average 30-40% faster than in other languages. Moreover, many Ruby developers are full-stack developers, it reduces costs for businesses. The speed of development doesn't compromise quality. The time saved can be invested in refactoring, updating libraries or looking for better architecture abstractions instead. In reality, developers often have lack of time to write code with high quality.

Ruby has become the perfect tool for testing business ideas. Although it's slow, people often overlook its real strength - the speed at which it can create new app/feature. Since a business needs to adapt quickly to changes in order to survive, Ruby allows you to adapt your app to new tasks and changes quickly. That's why so many Y Combinator startups are launched using Ruby on Rails.

### 2.4 Standardization

![Ruby - Standardization](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ymd6rglm6kiriba4wec6.jpeg)

The core of the Ruby ecosystem focuses on the single framework (RoR). It gives us a big amount of tools for it. For example: test writing is standardized, the library for web interfaces is standardized too (Rack). In the infrastructure level, the application server doesn't care whether it's a Rails application or another Ruby web-framework. Code style is also standardized by rubocop, so a new developer won't get confused about the code style. This is similar to Python and Go, but in some languages you need to [choose from N options](https://betterprogramming.pub/comparing-the-top-three-style-guides-and-setting-them-up-with-eslint-98ea0d2fc5b7)). Many third-party libraries have already been adapted for Rails. This reduces the time it takes to integrate code into the app. There aren't any situations where you have to deal with dependency conflicts or spend time configuring external libraries. That means we can hire a new Ruby dev who can easily understand the app. The devs already know the file structure, namespacing, etc. On the negative side, sometimes people get bored with all the apps looking the same. But there are other, less popular frameworks out there.

### 2.5 Knowledge Base on Application Development

![Ruby Knowledge Base on Application Development](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u9yxibwbgtdetm07bx1p.jpeg)

Over the years, the community has collected a huge knowledge base about how to build web apps. It's because apps like Cookpad, GitHub, GitLab, Shopify, Basecamp, and Stripe (among others) are all written in Ruby. The community has gained expertise in building and maintaining monoliths (big apps) that can easily scale in the future and how to maintain them long-term.

The Rails community promotes the concept of the [majestic monolith](https://m.signalvnoise.com/the-majestic-monolith/). We've figured out a way to write these big apps without the problems that come with them. We're using architecture namespaces to make sure everything is organized and there's no need to rewrite everything into microservices later. This way, we can avoid problems like having a distributed monolith or having to write a bunch of schemas to describe how microservices talk to each other and track requests. If you want to learn more, you can attend community meetings or conferences or read dev's blogs.

The technology also follows all security concerns. Ruby is well-suited for applications that handle personal data, such as financial platforms and marketplaces. The ecosystem follows Security Development Lifecycle to control vulnerabilities also there are solutions for the license checking. Rails 7 includes built-in support for encrypting database data. For this reason, fintech products are also launched on Ruby. In this case, the situation is close to Java and the C languages.

### 2.6 A tool to solve business problems

![Ruby A tool to solve business problems](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8od5qe8x04u3sljechs5.jpeg)

When you are coding in Ruby, you don't often worry about things that your colleagues using other programming languages keep in mind. You don't need worry about type declarations, memory usage and etc. You can just focus on the actual problems you're trying to solve. Plus, routine tasks (routing, database structure changes, tests etc.) aren't programmed, but they are described by DSL. That's why it's so great to use Ruby for creating new business features. 8 out of 10 developers are involved in business automation or solving business tasks by the code. RoR guides the developers on how to do it and dictates the structure, allowing the developers to focus on solving the actual business problem, writing code specifically for it, rather than "struggling" with the framework.

However, I want to add that if you're working on atypical tasks, you might find yourself "struggling" with the framework or dealing with legacy issues, like on other languages. Fortunately, in RoR, these limits are pretty wide, so you rarely step outside them, especially if you have a lot of programming experience.

## 3 Technology Weaknesses

![Ruby Technology Weaknesses](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hboh4a8thvxkfm596d6b.jpeg)

Now, let's discuss the weaknesses of the technology or the aspects it's criticized for.

### 3.1 Application Performance

Every developer has heard that Ruby is kinda slow, but actually, a lot of languages are considered as "slow". The execution speed of a computer program depends on memory management features. It's a feature of all technologies where memory management is delegated to a virtual machine (VM). For more details, you can watch Grigory Petrov's video [Why is Ruby slow? (rus)](https://www.youtube.com/watch?v=m73VjmjAnuw), based on which I wrote a [short article](https://dev.to/kopylov_vlad/why-programming-languages-are-slow-1b2d).

For many developers, all languages that are labeled as "interpreted" (or scripting) are considered as slow because the interpreter reads the program's source code line by line and executes it. However in 2023, most modern "interpreted" languages arn't really interpreted. Languages like Ruby and Python are compiled into bytecode and executed in a VM. In Ruby since version 1.9 (2011), the interpreter precompiles Ruby into bytecode before execution. With Ruby 2.6 (2018), Just-In-Time compiler (JIT) was introduced, it compiles frequently used instructions into binary code (similar to nodeJS). This approach significantly speed up mathematical calculations by tens of times, bit it struggles to speed up complex projects. Matz paid the price for it because he prioritized on large and diverse syntax feature not speed.

Also in the core of Ruby there is Global Interpreter Lock mechanism (GIL). It slows down program execution as it needs to pause the execution to clean up memory. This is inherited from the time when computers had single-core processors. Although the language has evolved, now Ruby has the Global Virtual Machine Lock (Global VM Lock) inside. Fortunately, technology continues to evolve. In places where the Global VM Lock causes a problem, developers can use features like Fiber, Ractor and others. If your goal is to write a speedy "number calculator" (a script for matrix multiplication, machine learning (ML) or rendering calculations), it's better to replace Ruby with languages where memory management management is delegated to a developer. That's why Ruby isn't the best choice for ML. However, in web development significant portions of time is spent on I/O (waiting for responses from the database or HTTP connections). For these cases your can use Threads where it's critical. For example, Rails 7 introduces parallel execution of database queries with [load_async](https://blog.skylight.io/rails-7-load_async/).

In 2019 Basecamp (the main developer of RoR) [published an article](https://m.signalvnoise.com/only-15-of-the-basecamp-operations-budget-is-spent-on-ruby/) saying that only 15% of their operating budget is spent actually running Ruby code. So, if you write your app and configure it well, you won't really notice any slowdowns. Even if they spend a lof of resources making the language twice as fast, that won't actually reduce their company costs by much.

The speed of code execution isn't critical for every application. You will notice slowdowns when your application processes hundred requests a second. Most developers don't face the same load in the early stages of creating an application/product. If your MVP starts with less than hundred requests that's quite good. If it's more, RoR can handle it because the application is scalable by increasing amount of processes and threads.

In computer science, there are some areas where speed is crucial, like parsing XML or working with encryption. For these tasks, Ruby works well because the language allows to write native extensions in C++. Popular libraries for these things are written in C++ and Ruby just provides only a wrapper/interface. The similar situation happens in Python with machine learning. Python code doesn't do matrix multiplication, it only manages the data flow. All mathematical operations are performed by C++ extensions. The Python community has really worked hard to make that happen.

### 3.2 Other Usage Areas

Historically, Ruby has been mainly used for web applications. If you need ML or machine vision, the Python's ecosystem is much advanced. For data analysis R/Python are best suited. But that doesn't mean you can't use Ruby for simple data calculations. Also there still areas where one technology dominates by historical factors: frontend, mobile development, etc. In reality, Ruby is not limited for the web. Check out how it's used in Japan, where there's a massive [Ruby community](https://www.jetbrains.com/lp/devecosystem-2021/ruby/). For them Ruby is more than just Rails. They use it to programming controllers (with mruby), as a scripting language for other systems, in search engines and so on. If you wish talks about Ruby, not just Rails, feel free to explore [RubyKaigi](https://www.youtube.com/@rubykaigi4884/videos).

Ruby is actively used in infrastructure tasks. The popular software for installing applications and packages on macOS [HomeBrew](https://brew.sh/) is written in Ruby, which is why Ruby pre-installed on macOS. Additionally, if you look at sysadmin job listings, experience with Ruby is often among the requirements. There are also solutions/libraries for other platforms, such as gamedev and frontend. While they might not be very popular, you can explore and play with them if you're interested.

### 3.3 Type Annotations

Languages like Javascript and Python have already had this, but what about Ruby? For documentation there is [Yard](https://rubydoc.info/gems/yard/file/docs/GettingStarted.md), but these annotations are only at the documentation level. For writing and checking types, there is a tool called [Sorbet](https://github.com/sorbet/sorbet) from Stripe. In Ruby 3.0 native type annotations (RBS) have been introduced, but they are written in separate files (similar to C and TypeScript). Types are kept in separate files because Matz believes that developers should focus on business features rather than writing types. He is confident that in nearest future, programming languages will move away from manually writing types towards tools for automatically generating signatures. Work in this direction has started with tools like [TypeProf](https://github.com/ruby/typeprof).

### 3.4  Learning Curve

Ruby learning curve is higher to compare to other languages. The reason is that Ruby's syntax and capabilities can be a bit overwhelming for beginners, so a lot of developers start with Python or Go instead.  After gaining initial experience, the most developers tend to explore other languages and explore the capabilities of different technologies. Many Ruby programmers are already experienced in programming before they embrace Ruby. Thankfully for newcomers, Rubocop guarantees a unified coding standard to writing code correctly and with best practices.

## 4 Hiring

![Ruby - Hiring](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/97nbt2wo1omzb1rc1dgp.jpeg)

Thanks to the standardization that was talking about  earlier, people who come to your project are already familiar with the technologies, significantly reducing the search time. This is important for hiring a new team member. In the onboarding, new developers have to only understand the business domain. With other languages, the situation is different. If we're looking for a Python or PHP developer, we reject some candidates because they've worked with a different framework and during the interview, we need to understand the technologies they are familiar with. Similarly, when hiring a Python developer, we must understand their specialization: backend, automated testing, ML, data science, ETL, etc. That's why having just one Python developer may not be sufficient to launch an application.

In contrast, most RoR developers are full-stack specialists who can launch a web application on your own. Some of them can solve frontend tasks, some of them follow the latest technologies such as [HTML over websockets](https://hotwired.dev/). However, compared to Node or Python, there's not as many Ruby devs and they tend to ask for higher pay.

## 5 Community

![Ruby Community](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3zfosmhtnqyk6tio4cf8.jpeg)

I just want to say that there's more to any technology than meets the eye. Sure, there are the people behind it, and Ruby is known for its amazing community. The community isn't big as in some other languages due to a smaller amount of developer (though history has examples where the core of an ecosystem was created by [just one person](https://medium.com/@kelas/how-is-tj-holowaychuk-so-insanely-productive-604818b4e9eb)), it is influential in the open-source world. Projects like GitHub has been released to thanks to the Ruby community. The fundamental principle of the Ruby community is sharing code and ideas, therefore RoR has the highest number of contributors compared to other technologies. Perhaps only the frontend community has a similar obsession for open source.

The community follows Matz's idea that Ruby was created for comfort and fun. How else would you explain methods in RoR arrays like `second`, `third`, `second_to_last`, `third_to_last`, and suddenly `forty_two`? Moreover, when Ruby developers switch into other languages, they often bring ideas from the Ruby world. For example, [dry-python](https://github.com/dry-python) and [pyenv](https://github.com/pyenv/pyenv) were influenced by concepts from Ruby. The Ruby ecosystem has inspired other languages; for instance, RoR inspired PHP developers to create [Laravel](https://github.com/laravel/laravel), and Rspec inspired JavaScript developers to build libraries like [mocha](https://github.com/mochajs/mocha), [jasmine](https://github.com/jasmine/jasmine), and [Chai](https://github.com/chaijs/chai).

Ruby's community was fortunate to create a canonical, convenient, and high-quality dependency manager, [Bundler](https://bundler.io/), in 2009. It's a unified, deterministic dependency installer with a lock file, support for multi-environments, and the ability to install dependencies directly from GitHub. Features that ecosystems like Go and NodeJS can't boast about. Of course, Bundler had its problems in the early years, such as speed issues, but they were successfully overcome. Bundler became the prototype for tools like [yarn](https://www.npmjs.com/package/yarn#prior-art) in JavaScript, `cargo` in Rust, and `composer` in PHP.

Ruby's ecosystem used to inspire others, nowadays the trend has shifted. Now Ruby adopts features from other languages. It has received pattern matching, similar to Elixir and Erlang; the transpiler [RubyNext](https://github.com/ruby-next/ruby-next) (as Babel in JS), allowing the use of new language features on previous Ruby versions, etc. Some solutions for the frontend are inspired by RubyOnRails, for instance: [sockpuppet](https://github.com/jonathan-s/django-sockpuppet) for Django is an implementation of [stimulus-reflex](https://github.com/stimulusreflex/stimulus_reflex) from Rails.

The community continues to expand and organize significant conferences worldwide, such as RubyKaigi (Japan), RubyRussia (Russia), Euruko (Europe), RailsConf/RubyConf (USA) and more.

## 6 Relevance of the Technology

![Ruby Relevance of the Technology](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y3etg7lj2kam0isveuny.jpeg)

From 2013 to 2015, the language and framework was super popular. But since 2016, But since 2016, people have been saying that "Ruby is dead". That's just not true, though, because every technology goes through the same [hype cycle](https://tecedu.academy/storage/ckeditor-content/YWWop0OvNadq0RCkc0Fha42gs3Mo8p1QNhxEVA7r.png). Each technology experiences a the peak of high expectations and people believe it's outdated, but, in reality, the technology just enters the plateau of productivity. This stage indicates that the technology is mature enough for use in production. Java, for example, isn't dead since 2005, and it's still used in production.

Essentially, it's rare for a technology to die, especially in open source. After the the years of using, a lot of application has been built and each of it requires ongoing support and development. How else can we explain that Fortran is in 16th place in the [TIOBE](https://www.tiobe.com/tiobe-index/) ranking as of 2023? Moreover, how can we consider the actual relevance of technologies? To count the number of questions on Stack Overflow? This indicates the number of beginner developers learning. To count stars or the number of projects on GitHub? But GitHub is like a warehouse of building materials, and in some communities, it's not common to click on the star in GitHub. In bloody enterprise industries, libraries are not often publicly published. Job listings on websites? This indicates a shortage of specialists in this area at the moment. Even TIOBE itself is calculated based on [mentions in 25 search engines](https://www.tiobe.com/tiobe-index/programminglanguages_definition/). Therefore, determining actual relevance is indeed complex.

## Conclusion

![Ruby Conclusion](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jodekh86w1h5kqinwqe4.jpeg)

Ruby was originally designed to be the most user-friendly language for development and it has not changed its core priority. It has launched a rich, stable ecosystem with diverse solutions for a wide range of tasks. It's general-purpose programming language with great full-stack tools in web development. The community has created large base how to build production ready applications. The language has some "weaker" points, but you can overcome them if you understand how and why they came about.The community continues to evolve the language and the tools around it, observing and adopting features from other languages, making Ruby a relevant tool. That's exactly why it suits my needs perfectly after all these years.

Additionally, I agree with [Grigory Petrov's words](https://www.youtube.com/watch?v=L3YqKI5kcOk) that the Ruby language is an ideal choice for someone if:

* They know they enjoy programming and they want to make the process of writing code to be maximally enjoyable.
* Their area of expertise is in helping businesses, not gamedev, not frontend, not code for physical devices, not code for scientific calculations, not landing pages. Ruby and its ecosystem are well-suited for solving business problems using web applications, integration scripts and more.

If these are your specific needs, then choosing on Ruby is a good decision!

I ♥️ Ruby

[dev.to](https://dev.to/kopylov_vlad/why-i-love-ruby-44g9)
