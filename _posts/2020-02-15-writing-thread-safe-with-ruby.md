---
layout: post
title: Thread safety in Ruby
date: '2020-02-15'
categories: posts
published: true
tags: [ruby]
---

When you are using Unicorn or Passenger Phusion based (community edition), you don't have to worry about thread safety, Because these servers are multi-process (workers) based servers.

Read about application server [here]({% link _posts/2020-02-12-understand-ruby-application-servers.md %})

But when comes to Puma, It comes with Multi-threaded options as well. From MRI background, we tend not to write thread-safety code. This will cause issues in multi-threaded environment like Puma.
If you are going to migrate application server from Unicorn/Passenger to Puma (with multi-threaded code), make sure you follow few tips.


### 1. Good practice to use constant, Freeze it!
Ruby doesn't gives you immutable data structure, Even constant is mutable in Ruby's world.

```ruby
2.5.0 :001 > IAM_CONSTANT = "test"
 => "test"
2.5.0 :002 > IAM_CONSTANT = "new_test"
(irb):2: warning: already initialized constant IAM_CONSTANT
(irb):1: warning: previous definition of IAM_CONSTANT was here
 => "new_test"
2.5.0 :003 > IAM_CONSTANT
 => "new_test"

2.5.0 :001 > IAM_ARRAY_CONSTANT = ["Haha! I am constant, no one can change me!"]
 => ["Haha! I am constant, no one can change me!"]
2.5.0 :002 > IAM_ARRAY_CONSTANT << "Seriously?"
 => ["Haha! I am constant, no one can change me!", "Seriously?"]
2.5.0 :003 >
 ```

```ruby
2.5.0 :001 > IAM_ARRAY_CONSTANT = ["Haha! I am constant, no one can change me!"].freeze
 => ["Haha! I am constant, no one can change me!"]
2.5.0 :002 > IAM_ARRAY_CONSTANT << "Seriously?"
Traceback (most recent call last):
        2: from /Users/satyanarayan/.rvm/rubies/ruby-2.5.0/bin/irb:11:in `<main>'
        1: from (irb):2
FrozenError (can't modify frozen Array)
```

### 2. Don't mutate Global variables

I have worked with Legacy code-bases before where we used to determine current user using Global variable. It worked fine when we have used Application servers like Passenger. Those code will not work with Puma like servers.

```ruby
class UsersController < ApplicationController
  before_action :authenticate_user

  def authenticate_user
    $user_id = fetch_user_from_cookies
    sleep(10)
    if $user_id == "admin"
      Rails.logger.info("Show admin template")
    else
      Rails.logger.info("Show normal template")
    end
  end

  def index
    render json: ["Hey i am working fine"]
  end
end
```

```
Started GET "/test?user_id=admin" for ::1 at 2020-02-16 15:48:46 +0530
Processing by TestController#index as HTML
  Parameters: {"user_id"=>"admin"}
Started GET "/test?user_id=admi" for ::1 at 2020-02-16 15:48:48 +0530
Processing by TestController#index as HTML
  Parameters: {"user_id"=>"admi"}

Show normal template
Completed 200 OK in 10ms (Views: 0.2ms | ActiveRecord: 0.0ms)

Show normal template
Completed 200 OK in 10ms (Views: 1.2ms | ActiveRecord: 0.0ms)
```

### 3. Class variables / class instance variables is not thread safe.
Even class method is not thread safe, If you mutating class variable, that will make your code smell

### 5. Memoization is good till you are in right sense.
Memoization is for sure good practice, when you want to cache across multiple calls inside object. 
Make sure whichever memoizing inside class method can shared across multiple request as well.

```ruby
class UsersController < ApplicationController
  before_action :fetch_user

  def fetch_user
  @user = User.find_user_once(@user_id)
  end

  def index
    render json: ["Hey it works"]
  end
end


class User
  def self.find_user_once(user_id)
    @test ||= User.find(user_id)
  end
end

```

### 6. Check your gems before you use.
Read documentation/code to confirm if library that you are going to use is thread-safe