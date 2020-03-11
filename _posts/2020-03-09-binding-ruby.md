---
layout: post
title: How binding works in ruby
date: '2020-03-09'
categories: posts
published: true
tags: [ruby]
---

### What is binding?

Any Objects of class can be encapsulated into context and you can retain this context for future use. The variables, methods, value of self, and possibly an iterator block that can be accessed in this context are all retained.

To keep track of current scope, Ruby uses `binding`, which retains the execution context at each line of code.

`binding` method returns `Binding` object which describes the binding at current line.

```ruby
2.6.3 :001 > a = 10
 => 10
2.6.3 :002 > binding
 => #<Binding:0x00007fe8270c7698>
2.6.3 :003 > binding.local_variables
 => [:a, :_]
2.6.3 :004 >
```
Code can be evaluated in `binding` using `eval` method

```ruby
2.6.3 :001 > a = 10
 => 10
2.6.3 :002 > binding.local_variables
 => [:a, :_]
2.6.3 :003 > binding.eval("a")
 => 10
2.6.3 :004 >
```

Example to demonstrate `binding`

```
2.6.3 :001 > class MyTemplateEngine
2.6.3 :002?>   def initialize(template)
2.6.3 :003?>     @template = template
2.6.3 :004?>     end
2.6.3 :005?>
2.6.3 :006?>   def result(binding)
2.6.3 :007?>     @template.gsub(/<%=(.+?)%>/) do
2.6.3 :008 >               binding.eval($1)
2.6.3 :009?>           end
2.6.3 :010?>     end
2.6.3 :011?>   end
 => :result
2.6.3 :012 > @title = "My title"
 => "My title"
2.6.3 :014 > description = "I am description"
 => "I am description"
2.6.3 :015 > template = MyTemplateEngine.new("<%= @title %> -- <%= description %>")
 => #<MyTemplateEngine:0x00007f8f4b8fbd80 @template="<%= @title %> -- <%= description %>">
2.6.3 :016 > template.result(binding)
 => "My title -- I am description"
 ```
Real life example will be how `ERB` templating system works in Ruby

```ruby
2.6.3 :001 > require 'erb'
 => true
2.6.3 :002 > title = "Sathia"
 => "Sathia"
2.6.3 :003 > ERB.new("Hi <%= title %>").result(binding)
 => "Hi Sathia"
```

Now you know `binding`, you must have used `binding.pry`. But what is `pry`? Read [here]({% link _posts/2020-03-09-ruby-with-pry.md %})
