---
layout: post
title:  cap-theorm
date:   2019-04-11 15:42:04 +0530
categories: blog posts
---
Instead of choosing different database for explaining cases like CP, AP and CA. I will choose only Mongo to explain the scenarios.

== You chose CA

Let’s say you have single Node of DB. eg: mongo1

What if that Node goes due to Network issue or Hardware issue. Downtime of your application will increase till your mongo node comes back. Or what if data in that Mongo node is lost. Your application will not work, because you lost your data.

This problem can be solved by having multi node system. In particular, the CAP theorem implies that in the presence of a network partition, one has to choose between *consistency* and *availability*.

*Note*: There are some database like Oracle which even doesn't support P (network partitioning). You can achieve partitioning using third party tools like Shareplex.

== You chose AP

what if you chose *Availability* in this case?

When you chose Availability over Consistency, doesn't mean that data won’t be consistent across the other nodes. Data will be eventually consistent. Once your application writes the data to master node, replication will happen eventually. In-case if other nodes are behaving bad, replication might take time. Other applications which are reading the data from slave nodes will not receive latest data due to replication lag. Most of DB will replicate to secondary with less lag.

This will be useful in systems where lag does not matters, eg: reporting system can be read from slave DB.

When you are working on transaction system, consistency is very important. when your application write to master, it immediately writes to secondary nodes. If any one of node is down, it will not be saved in any of those nodes. This gives you data consistency across over the nodes. But you lose availability of DB, when one of the node goes down.