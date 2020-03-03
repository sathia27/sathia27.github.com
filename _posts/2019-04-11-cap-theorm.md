---
layout: post
title: Cap Theorm
date: '2019-04-11 15:42:04 +0530'
categories: posts
published: true
---

## Understanding CAP theorm

![CAP theorm](/img/blogs/cap-theorm/cap.jpeg)

When your database have single node. And when if node goes down due to any issues like Network or Hardware failure. Application Downtime will increase till your node comes back. And other scenario, you might lose your data.

**Partition tolerance** is required in modern distributed systems

This problem can be solved by having multi-node system. In particular, the CAP theorem implies that in the presence of a network partition, one has to choose between **consistency** and **availability**.

*Note*: There are some database like Oracle which even doesn't support P (network partitioning). You can achieve partitioning using third party tools like Shareplex.

## You chose Consistency

Consistency refers to requirement that at any point of time, all nodes should have same data.
In this case, any node will not have any inconsistent state compared to other nodes.

So whenever any writes happens, node will need time to update all other nodes. So node will be busy and not available on network often while making reads.

But this make sure **All nodes see the same data at the same time**.

Example: In case of transactional system, consistency is very important. Even when one node is lost, Other node shouldn't show older data.

## You chose Availability

Availability refers to requirement that at any point of time, node should be 100% operational.

So whenever any read/write happens, this make sure server will always be available. But this doesn't promise that data in other networks are 100% replicated at this point of time. It could be eventually replicated.

Example: In case of Log management server, You are okay with data loss, but you should make sure log server is always available when client wants to log data.