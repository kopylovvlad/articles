# Write your own scss-compiler

![Photo by Artem Sapegin on Unsplash](image01.jpeg)

If you want to be a software developer, you have to study. On the internet, you can find a lot of online courses and books. After that, you can write a pet-project, npm-package or another library and add the project to GitHub.

I think that every software developer must write CMS, compiler/interpreter or JavaScript library for building web-interfaces. It is excellent practice because the 10,000-hour rule is not a myth. I wrote CMS twice. And now is a great time for writing a compiler.

## Why do developers write compiler/interpreter?

A compiler can transform code written in one computer language into another computer language. If your program can’t be executed directly, you have to use a compiler to produce an executable program.
For example, moderns browsers can run only JavaScript-code. But if you don’t like JavaScript, you can write code in other languages and [compile it into JavaScript](https://github.com/jashkenas/coffeescript/wiki/list-of-languages-that-compile-to-js).

An interpreter doesn’t compile the code. It parses code and executes it directly. There are many compiled computer languages like Ruby, Python, Javascript and others.

## Writing a compiler

Between compiler and interpreter, I decided to [write a compiler](https://github.com/kopylovvlad/lite_scss). I didn’t want to write a compiler for programming languages. I had a choice to write a stylesheet language like SASS or markup language like HAML. And I chose SASS.

Then, I had to choose the functionality: Multiline comments, Variables, Mixins and Nesting selectors.

## The structure of the compiler

The compiler consists of two parts (machine and CLI).
CLI is a simple command line interface.
The Machine is the main part that includes: parser, transformer and compiler.

## Parser

![image02](image02.png)

Here is the logic of parsing text to nested AST ([abstract syntax tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree)). It is a tree representation of the abstract syntactic structure of source code. AST is a useful structure for further manipulation.

For example:
We have a CSS-file like this.

```css
/* example 1 */
nav ul, nav ol {
    margin: 0;
    padding: 0px;
    list-style: none;
    font: 12pt/10pt sans-serif;
}
```

The parser will parse the text, destroy comments and then, split the text by symbols ';', '{', '}'. After that, we will have an array of strings.

```ruby
[
  'nav ul, nav ol {',
  'margin: 0;',
  'padding: 0px;',
  'list-style: none;',
  'font: 12pt/10pt sans-serif;',
  '}'
]
```

After that, the parser will wrap each element of the array to an instance of abstract node.

```ruby
[
  #<SelectorStartParserNode @name="selector start", @value="nav ul, nav ol {">, 
  #<PropertyParserNode @name="property", @value="margin: 0;">, 
  #<PropertyParserNode @name="property", @value="padding: 0px;">, 
  #<PropertyParserNode @name="property", @value="list-style: none;">, 
  #<PropertyParserNode @name="property", @value="font: 12pt/10pt sans-serif;">, 
  #<SelectorEndParserNode @name="selector end", @value="}">
]
```

We have an array of objects, but the structure is not nested. After that, the parser will transform the array to AST, and we will have AST for further manipulation.

```ruby
[
  #<SelectorAstNode
    @class_name="Selector.new”, 
    @name="nav ul, nav ol”, 
    @children=[
      #<PropertyAstNode 
        @class_name="Property.new", 
        @name="margin", 
        @children=[], 
        @value= #<AstNodeValue 
          @class_name="VirtualString.new", 
          @value="0">>, 
      #<PropertyAstNode 
        @class_name="Property.new", 
        @name="padding", 
        @children=[], 
        @value= #<AstNodeValue 
          @class_name="VirtualString.new", 
          @value="0px">>, 
      #<PropertyAstNode 
        @class_name="Property.new", 
        @name="list-style", 
        @children=[], 
        @value= #<AstNodeValue 
          @class_name="VirtualString.new", 
          @value="none">>, 
      #<PropertyAstNode 
        @class_name="Property.new", 
        @name="font", 
        @children=[], 
        @value= #<AstNodeValue:0x007fa5b9046760 
          @class_name="VirtualString.new", 
          @value="12pt/10pt sans-serif">>
    ], 
    @value= #<AstNodeValue 
      @class_name="", 
      @value="">>
]
```

Another example of more difficult structure.
Before, SCSS-file

```scss
/* example 2 */
$other-color: red;
$main_color: green;
@mixin border-radius {
  -webkit-border-radius: 10px;
     -moz-border-radius: 10px;
      -ms-border-radius: 10px;
          border-radius: 10px;
}
div {
  color: $main_color;
  @include border-radius;
}
p {
  color: blue;
  border-radius: 5px;
  border-color: $other-color;
}
```

After, AST

```ruby
[
  #<VarAssignAstNode 
    @class_name="VariableAssign.new", 
    @name="other-color", 
    @children=[], 
    @value= #<AstNodeValue @class_name="VirtualString.new", @value="red">>, 
  #<VarAssignAstNode 
    @class_name="VariableAssign.new", 
    @name="main_color", 
    @children=[], 
    @value= #<AstNodeValue @class_name="VirtualString.new", @value="green">>, 
  #<MixAssignAstNode 
    @class_name="MixinAssign.new", 
    @name="border-radius", 
    @children=[
      #<PropertyAstNode 
        @class_name="Property.new", 
        @name="-webkit-border-radius", 
        @children=[], 
        @value= #<AstNodeValue @class_name="VirtualString.new", @value="10px">>, 
      #<PropertyAstNode 
        @class_name="Property.new", 
        @name="-moz-border-radius", 
        @children=[], 
        @value= #<AstNodeValue @class_name="VirtualString.new", @value="10px">>, 
      #<PropertyAstNode 
        @class_name="Property.new",
        @name="-ms-border-radius", 
        @children=[], 
        @value= #<AstNodeValue @class_name="VirtualString.new", @value="10px">>, 
      #<PropertyAstNode 
        @class_name="Property.new", 
        @name="border-radius", 
        @children=[], 
        @value= #<AstNodeValue @class_name="VirtualString.new", @value="10px">>
    ], 
    @value= #<AstNodeValue @class_name="", @value="">>, 
  #<SelectorAstNode 
    @class_name="Selector.new", 
    @name="div", 
    @children=[
      #<PropertyAstNode 
        @class_name="Property.new", 
        @name="color", 
        @children=[], 
        @value= #<AstNodeValue @class_name="Variable.new", @value="main_color">>, 
      #<MixinAstNode 
        @class_name="Mixin.new",
        @name="border-radius", 
        @children=[], 
        @value= #<AstNodeValue @class_name="", @value="">>
    ], 
    @value= #<AstNodeValue @class_name="", @value="">>, 
  #<SelectorAstNode 
    @class_name="Selector.new", 
    @name="p", 
    @children=[
      #<PropertyAstNode 
        @class_name="Property.new", 
        @name="color", 
        @children=[], 
        @value= #<AstNodeValue @class_name="VirtualString.new", @value="blue">>, 
      #<PropertyAstNode 
        @class_name="Property.new", 
        @name="border-radius", 
        @children=[], 
        @value= #<AstNodeValue @class_name="VirtualString.new", @value="5px">>, 
      #<PropertyAstNode 
        @class_name="Property.new", 
        @name="border-color", 
        @children=[], 
        @value= #<AstNodeValue @class_name="Variable.new", @value="other-color">>
    ], 
    @value= #<AstNodeValue @class_name="", @value="">>
]
```

## Transformer

![image03](image03.png)

Here is the logic of transformations nested AST to not nested. The transformer works by the algorithm:

1. Traverse AST and transform nested node to not nested
1. Traverse AST and transform each AstNode

For example, CSS-file like this:

```css
/* example 1 */
nav ul, nav ol {
    margin: 0;
    padding: 0px;
    list-style: none;
    font: 12pt/10pt sans-serif;
}
```

After passing through the Parser and the Transformer, will be transformed into:

```ruby
[
  #<Selector 
    @name="nav ul, nav ol", 
    @declaration=[
      #<Property 
        @name="margin", 
        @value= #<VirtualString @value="0">>, 
      #<Property 
        @name="padding", 
        @value= #<VirtualString @value="0px">>, 
      #<Property 
        @name="list-style", 
        @value= #<VirtualString @value="none">>, 
      #<Property 
        @name="font", 
        @value= #<VirtualString @value="12pt/10pt sans-serif">>
    ]>
]
```

Another example:
before, SCSS-file

```scss
$other-color: red;
$main_color: green;
div {
  color: $main_color;
  p {
    color: $main_color;

    span {
      font-size: 9px;
    }
  }
}
p {
  color: blue;
}
```

after, AST

```ruby
[
  #<VariableAssign 
    @name=:"other-color", 
    @expression= #<VirtualString @value="red">>, 
  #<VariableAssign 
    @name=:main_color, 
    @expression= #<VirtualString @value="green">>, 
  #<Selector 
    @name="div", 
    @declaration=[
      #<Property 
        @name="color", 
        @value= #<Variable @name=:main_color>>
    ]>, 
  #<Selector 
    @name="div p", 
    @declaration=[
      #<Property 
        @name="color", 
        @value= #<Variable @name=:main_color>>
    ]>, 
  #<Selector 
    @name="div p span", 
    @declaration=[
      #<Property 
        @name="font-size", 
        @value= #<VirtualString @value="9px">>
    ]>, 
  #<Selector 
    @name="p", 
    @declaration=[
      #<Property 
        @name="color", 
        @value= #<VirtualString @value="blue">>
    ]>
]
```

## Compiler

Here is the logic of compilation the AST to a file.
The compiler works by the algorithm:

1. Traverse AST and reduce each reducible node
1. Traverse AST and print each printable node

After all of this, we will have valid CSS-file.

## Conclusion

Unfortunately, my version of SCSS is not supporting all original features. But it was a great experience for me and a personal challenge. You can see [source-code in GitHub](https://github.com/kopylovvlad/lite_scss). All information is in README-file.

[Medium](https://kopilov-vlad.medium.com/write-your-own-scss-compiler-68269278dcce)
