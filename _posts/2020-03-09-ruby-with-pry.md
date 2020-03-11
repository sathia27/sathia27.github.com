---
layout: post
title: Ruby with pry
date: '2020-03-09'
categories: posts
published: true
tags: [ruby]
---

If you are ruby Developer, You must be using `binding.pry` very common for debugging rails / ruby projects. Do you know what is `pry` and `binding`? I will try to walk through about `binding` and `pry`. And also I walk through how `pry` is more powerful tool for debugging.

<div style="width:500px; margin: 10px auto"><img src="/img/blogs/ruby-pry/binding-plus-pry.jpeg" /></div>

## What is binding and how binding.pry works?

[Read blog post about binding in Ruby]({% link _posts/2020-03-09-binding-ruby.md %})

## What is pry?

It is one of REPL (Read Evaluate Print Loop) available in Ruby language. But it is more powerful than other available REPL like IRB. It is a powerful tool that can help in debugging and further digging around your codebase.

### Using binding with pry

Calling `binding.pry` is essentially `prying` into the current binding or context of the code, from outside your file. So when you place the line `binding.pry` in your code, that context will be interpreted and opened in shell at runtime.

That means, `pry` can be used in any of the ruby object other than `binding` object.

Let's see try to use `pry` with string object (everything is object in Ruby)

```ruby
2.6.3 :001 > require 'pry'
 => true
2.6.3 :002 > "sathia".pry
[1] pry("sathia")> upcase
=> "SATHIA"
[2] pry("sathia")>
```
In above example, I am calling pry for `String` object. I am calling `upcase` inside the pry console. Which means I am inside the object of `String` which I was `prying`

Pry gives us many other feature other than above examples:

### Source code browsing

```ruby
[3] pry("sathia")> show-source Array

From: /Users/satyanarayan/.rvm/rubies/ruby-2.6.3/lib/ruby/2.6.0/pp.rb @ line 329:
Class name: Array
Number of monkeypatches: 2. Use the `-a` option to display all available monkeypatches
Number of lines: 13

class Array # :nodoc:
  def pretty_print(q) # :nodoc:
    q.group(1, '[', ']') {
      q.seplist(self) {|v|
        q.pp v
      }
    }
  end

  def pretty_print_cycle(q) # :nodoc:
    q.text(empty? ? '[]' : '[...]')
  end
end

[4] pry("sathia")>
```
### Pry as Document browsing. You don't have to visit ruby docs to achieve this. You just can do it ur own ruby console.

Make sure  `pry-doc` gem is installed in your system.
You can install inside pry console directly by below command

To install `gem-install pry-doc`

```ruby
[11] pry(Array):1> show-doc join

From: array.c (C Method):
Owner: Array
Visibility: public
Signature: join(*arg1)
Number of lines: 12

Returns a string created by converting each element of the array to
a string, separated by the given separator.
If the separator is nil, it uses current $,.
If both the separator and $, are nil,
it uses an empty string.

   [ "a", "b", "c" ].join        #=> "abc"
   [ "a", "b", "c" ].join("-")   #=> "a-b-c"

For nested arrays, join is applied recursively:

   [ "a", [1, 2, [:x, :y]], "b" ].join("-")   #=> "a-1-2-x-y-b"
```

### Navigate around different Class/State
You have commands like `cd`, `ls` to navigate and list methods and variables

```ruby
2.6.3 :004 > pry
[1] pry(main)> cd Array
[2] pry(Array):1> ls
Object.methods: yaml_tag
Array.methods: []  try_convert
Array#methods:
  &    []      bsearch        compact!   difference  eql?        flatten   join     min          prepend             reject!               rindex     shift     sort!       to_h       values_at
  *    []=     bsearch_index  concat     dig         fetch       flatten!  keep_if  none?        pretty_print        repeated_combination  rotate     shuffle   sort_by!    to_s       zip
  +    all?    clear          count      drop        fill        hash      last     one?         pretty_print_cycle  repeated_permutation  rotate!    shuffle!  sum         transpose  |
  -    any?    collect        cycle      drop_while  filter      include?  length   pack         product             replace               sample     size      take        union
  <<   append  collect!       delete     each        filter!     index     map      permutation  push                reverse               select     slice     take_while  uniq
  <=>  assoc   combination    delete_at  each_index  find_index  insert    map!     place        rassoc              reverse!              select!    slice!    to_a        uniq!
  ==   at      compact        delete_if  empty?      first       inspect   max      pop          reject              reverse_each          shelljoin  sort      to_ary      unshift
locals: _  __  _dir_  _ex_  _file_  _in_  _out_  _pry_
```

To list all instance variables, you could use `ls -i`, to check available options with ls, use `ls -h`

```ruby
[10] pry(Array):1> ls -h
Usage: ls [-m|-M|-p|-pM] [-q|-v] [-c|-i] [Object]
       ls [-g] [-l]

ls shows you which methods, constants and variables are accessible to Pry. By
default it shows you the local variables defined in the current shell, and any
public methods or instance variables defined on the current object.

The colours used are configurable using Pry.config.ls.*_color, and the separator
is Pry.config.ls.separator.

Pry.config.ls.ceiling is used to hide methods defined higher up in the
inheritance chain, this is by default set to [Object, Module, Class] so that
methods defined on all Objects are omitted. The -v flag can be used to ignore
this setting and show all methods, while the -q can be used to set the ceiling
much lower and show only methods defined on the object or its direct class.

Also check out `find-method` command (run `help find-method`).

    -m, --methods               Show public methods defined on the Object
    -M, --instance-methods      Show public methods defined in a Module or Class
    -p, --ppp                   Show public, protected (in yellow) and private (in green) methods
    -q, --quiet                 Show only methods defined on object.singleton_class and object.class
    -v, --verbose               Show methods and constants on all super-classes (ignores Pry.config.ls.ceiling)
    -g, --globals               Show global variables, including those builtin to Ruby (in cyan)
    -l, --locals                Show hash of local vars, sorted by descending size
    -c, --constants             Show constants, highlighting classes (in blue), and exceptions (in purple).
                                Constants that are pending autoload? are also shown (in yellow)
    -i, --ivars                 Show instance variables (in blue) and class variables (in bright blue)
    -G, --grep                  Filter output by regular expression
    -d, --dconstants            Show deprecated constants
    -h, --help                  Show this message.
```


### You can list history of commands which you used inside the session

``` ruby
[3] pry(Array):1> hist
 1: upcase
 2: show-source String
 3: show-source Array
 4: show-doc Array
 5: cd Array
 6: show-doc
 7: cd Array
 8: show-doc join
 9: gem-install pry-doc
10: cd Array
11: show-doc join
12: cd Array
13: ls
[4] pry(Array):1>
```

Also you can use `grep`, `tail`, `head`, `replay` options with `hist` commands

eg:
```ruby
[4] pry(Array):1> hist --tail 3
12: cd Array
13: ls
14: hist
[5] pry(Array):1> hist --grep join
 8: show-doc join
11: show-doc join
[6] pry(Array):1> hist --replay 8

From: array.c (C Method):
Owner: Array
Visibility: public
Signature: join(*arg1)
Number of lines: 12

Returns a string created by converting each element of the array to
a string, separated by the given separator.
If the separator is nil, it uses current $,.
If both the separator and $, are nil,
it uses an empty string.

   [ "a", "b", "c" ].join        #=> "abc"
   [ "a", "b", "c" ].join("-")   #=> "a-b-c"

For nested arrays, join is applied recursively:

   [ "a", [1, 2, [:x, :y]], "b" ].join("-")   #=> "a-1-2-x-y-b"
[8] pry(Array):1>
```
### You can open linux shell inside pry prompt

```ruby
2.5.0 :008 > pry
[1] pry(main)> shell-mode
pry main:/Users/satyanarayan/Documents $ .ls
Books		complaint	friends		pdfs		videos
pry main:/Users/satyanarayan/Documents $
```

You can navigate to multiple files, do other activities in file system as well using linux commands.

### There are few other commands like show-routes, show-models which you can use with rails projects (not in Ruby)

Include `pry-rails` instead of `pry` in your project.
```
gem 'pry-rails', :group => :development
```

```ruby
➜  events git:(master) ✗ rails c
Running via Spring preloader in process 11614
Loading development environment (Rails 5.2.4.1)
[1] pry(main)> show-routes
                   Prefix Verb URI Pattern                                                                              Controller#Action
       rails_service_blob GET  /rails/active_storage/blobs/:signed_id/*filename(.:format)                               active_storage/blobs#show
rails_blob_representation GET  /rails/active_storage/representations/:signed_blob_id/:variation_key/*filename(.:format) active_storage/representations#show
       rails_disk_service GET  /rails/active_storage/disk/:encoded_key/*filename(.:format)                              active_storage/disk#show
update_rails_disk_service PUT  /rails/active_storage/disk/:encoded_token(.:format)                                      active_storage/disk#update
     rails_direct_uploads POST /rails/active_storage/direct_uploads(.:format)                                           active_storage/direct_uploads#create
[2] pry(main)> show-models
ApplicationRecord
  Table doesn't exist
ContactDetail
  id: integer
  contactable_type: string
  contactable_id: integer
  user_id: integer
  created_at: datetime
  updated_at: datetime
  belongs_to :contactable
  belongs_to :user
Email
  id: integer
  email: string
  created_at: datetime
  updated_at: datetime
  has_one :contact_detail
```
