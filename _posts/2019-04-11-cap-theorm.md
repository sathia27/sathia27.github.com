---
layout: post
title: Cap Theorm
date: '2019-04-11 15:42:04 +0530'
categories: posts
published: true
---

## Understanding CAP theorm

![CAP theorm](/img/blogs/cap-theorm/cap.jpeg)

Let’s say you have single Node of DB

What if that Node goes due to Network issue or Hardware issue. Downtime of your application will increase till your node comes back. Or what if data in that node is lost. Your application will not work, because you lost your data.

**Partition tolerance** is required in modern distributed systems

This problem can be solved by having multi node system. In particular, the CAP theorem implies that in the presence of a network partition, one has to choose between **consistency** and **availability**.

*Note*: There are some database like Oracle which even doesn't support P (network partitioning). You can achieve partitioning using third party tools like Shareplex.

## You chose Consistency

Consistency refers to requirement that at any point of time, all nodes should have same data.
In this case, any node will not have any inconsistent state compared to other nodes.

So whenever any writes happens, node will need time to update all other nodes. So node will be busy and not available on network often while making reads.

But this make sure **All nodes see the same data at the same time**.

eg: When you are working on transaction system, consistency is very important.

## You chose Availability

Availability refers to requirement that at any point of time, node should be 100% operational.

So whenever any read/write happens, this make sure server will always be available. But this doesn't promise that data in other networks are 100% replicated.

eg: If you are building log management server.