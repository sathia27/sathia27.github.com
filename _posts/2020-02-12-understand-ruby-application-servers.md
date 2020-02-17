---
layout: post
title: Ruby Application servers
date: '2020-02-12'
categories: posts
published: true
---

Ruby have variety of application servers which are used widely in production.
For this blog post, I would like to cover below 3 application servers which I have used recently.

1. Unicorn
2. Phusion Passenger
3. Puma

This article needs understanding of below few topics.
1. Why ruby needed Application servers.
2. What is **Rack** based applications. (not rake)
3. Global interpreter lock (GIL)

As far as MRI (ruby < 3),
**Parallelism**:
You cannot achieve parallelism using single process/Threads. Due to GCL (Global interpreter lock).
But this was address in Ruby 3 using *Guild*.

**Concurrency**:
Concurrency in ruby applications are achieved using ruby Threads.

Read more about to understand differences between *Parallelism vs Concurrency*

### Unicorn
This is one of available open source application server available in Ruby on rails community.

**Advantage of using Unicorn:**
1. This provides parallelism by forking multiple process. You could configure process count provided by unicorn configurations.
2. You don't have to worry about Thread safety. As multiple process doesn't share memory.

**Disadvantages using Unicorn:**
1. Let's say your ruby process takes around 10MB of memory, using process-based scaling will consume lot of memory in your single instance.
2. In today's containerized world, it's recommended to have one process per container. If you want to handle throughput, you need to do horizontal scaling. Which increases more memory (includes cost per instance)
3. Unicorn is used for MRI based ruby. Other run-time like Jruby is not be supported in unicorn
4. Does'nt gives you multi-threaded environment as far now.
5. As ruby does copy-on-write while forking, you have to make sure connections like DB, redis are re-established.

### Passenger Phusion
There are two versions in Passenger Phusion. 
1. Community Editions (Open source)
2. Enterprise Editions

**Advantage of using Passenger:**
1. In community edition, Parallelism can be achieved using forking multiple process.
2. In Enterprise edition, Concurrency can be achieved (Multi-threaded environment for enterprise edition).
3. Passenger supports other programing languages other than Ruby. If you are company who uses programing languages like ruby, node.js etc. You can go with Phusion Passenger instead of Unicorn.

**Disadvantages using Passenger:**
1. You need to pay to use Enterprise.

### Puma
This is one of favorite Heroku's favorite application server. Now it's Rails default application server.
This is open and free software.

**Advantage of using Puma:**
1. This support multi-process and multi-threaded server. Hence you can gain, parallelism and concurrency as well. (which is same benefits as Passenger Phusion enterprise, But puma is free)
You can configure workers (process count) and thread count in puma's configuration.

2. With multi-threaded configuration, you can even gain parallelism with JRuby/Rubmine runtime environment.

3. By configuring with threads, your application will consume less memory.

**Disadvantages using Passenger:**
1. No disadvantages, Make sure you write thread-safe code. If your code is not thread-safe, still you can use Puma with worker configurations, with thread count as 0. (which would work like Unicorn)

Few other application servers which are notable.

### Thin
Single threaded concurrent web application server.
This uses event-machine internally to gain concurrency. IO calls will not block your main thread to receive the request.

### Falcon
This is fibre based instead of thread based. As Fibres are weigh lesser than threads, Hence application will consume lesser memory than other application servers.

Read about Thread safety in Ruby [here]({% link _posts/2020-02-15-writing-thread-safe-with-ruby.md %})
