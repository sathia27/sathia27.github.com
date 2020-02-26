---
layout: post
title: Ruby Application servers
date: '2020-02-12'
categories: posts
published: true
---

Ruby have many application servers which are widely used in production.
For this blog post, I would like to cover below 5 application servers.

1. Unicorn
2. Phusion Passenger
3. Puma
4. Thin
5. Falcon

This article needs understanding of below few topics.
1. Why ruby requires Application servers.
2. What is Global interpreter lock - [GIL](https://en.wikipedia.org/wiki/Global_interpreter_lock)
3. Also read differences between *Parallelism vs Concurrency*
	You could refer below links:
	[Parallel computing](https://en.wikipedia.org/wiki/Parallel_computing)
	[Concurrent computing](https://en.wikipedia.org/wiki/Concurrent_computing)

### Unicorn
This is one of available open source application server available in Ruby on rails.

Unicorn spawner does fork to handle multiple requests. If you are running your application in multi-code systems you could achieve parallelism, by configuring process count in unicorn.

As this is process based, memory consumption per server instance will be more. Let's say if your ruby process takes around 10 MB, and if you are running with 4 process (workers). Your default memory consumption per server instance will be 10*4 MB.

But if you use >= Ruby 2.0, Forking is possible with less memory consumption due to Copy-on-write advantage of Operating system. You need to make sure your are reconnecting DB, Redis, File-system connections after forking.

Unicorn doesn't give your multi-threaded capability for handling requests.

### Passenger Phusion
There are two versions available in Passenger Phusion.
1. Community Editions (Open source)
2. Enterprise Editions

In Community Edition, Passenger can be configured as Multi-Process server (fork based). Above description for Unicorn will be applicable for Passenger as well.

In Enterprise Edition, Passenger can be configured as both Multi-Process and Multi-threaded server.

In Multi-threaded server, with MRI as runtime, you will not be able to achieve Parallelism due to GIL (Global interpretor lock) implemented inside MRI. MRI will not allow you to execute your threads parallel even in Multi core system. But you could achieve concurrency across the request. If your application is IO bounded, configuring your application in multi-threaded instead of multi-process will improve your application performance per server instance.

If you are using multi-threaded server with JRuby run-time, You can achieve Parallelism. Jruby, Rubinius runtime doesn't have GIL implementation.

But you need to make your code is thread-safe if you have configured your application server as multi-threaded server.
Read about Thread safety in Ruby [here]({% link _posts/2020-02-15-writing-thread-safe-with-ruby.md %})

Passenger supports other application like Python, Node.js other than Ruby. This is one of advantage of using Passenger over Unicorn/Puma.

### Puma
This is one of favorite Heroku's favorite application server. Now it's Rails default application server. This is open and free software.

Puma gives you both Multi-threaded and Multi-process server capability for Free. This is bigger advantages for using Puma over Passenger/Unicorn for Ruby application.

### Thin
Single threaded concurrent web application server.
This uses event-machine internally to gain concurrency. IO calls will not block your main thread, so your main thread will not be blocked to receive the request.

### Falcon
This is Fibre-based instead of thread based. Fibres in Ruby consumes very lesser memory than Thread. It would consume very lesser memory than Other application servers.

