---
layout: post
title: Cap Theorm
date: '2019-04-11 15:42:04 +0530'
categories: posts
published: false
---

Instead of choosing different database for explaining CAP theorm, I will choose only Mongo to explain the scenarios.

![CAP theorm](/img/blogs/cap-theorm/cap.jpeg)

Let’s say you have single Node of DB. eg: mongo1

## You chose CA

You have built a cool application, let's say Trip booking application. You use **Mongo** to save those trips. 5000 unique users are using your application and booked more than 1,00,000 trips.

Mongodb is hosted in data-center which is located in **country A**.

Now **country A** got Tsunami. With good heart, let's assume. No one got harmed, only your websever got crashed.

Luckily you have code in ur local or in ur version control. You can re-host again. But where you will get the data which you lost in Tsunami? What you are going to tell to your 5000 users.


![Availability](/img/blogs/cap-theorm/tsunami_country-a.jpg)

Your data is available at one place would make things worse. Due to some accident, if you lose data, it should be easier to recover your data, as your recovered code.

How will you make this happen? When you 



What if that Node goes due to Network issue or Hardware issue. Downtime of your application will increase till your mongo node comes back. Or what if data in that Mongo node is lost. Your application will not work, because you lost your data.

This problem can be solved by having multi node system. In particular, the CAP theorem implies that in the presence of a network partition, one has to choose between **consistency** and **availability**.

*Note*: There are some database like Oracle which even doesn't support P (network partitioning). You can achieve partitioning using third party tools like Shareplex.

when you start scaling your application, you have to make sure that there is no single point of failure. You have to introduce Network partioning to avoid the same. When you introduce network partioning, you might need to chose consistency or availability.

## You chose AP

what if you chose **Availability**?

When you chose Availability over Consistency, doesn't mean that data won’t be consistent across the other nodes. Data will be eventually consistent. Once your application writes the data to master node, replication will happen eventually. In-case if other nodes are behaving bad, replication might take time. Other applications which are reading the data from slave nodes will not receive latest data due to replication lag. Most of DB will replicate to secondary with less lag.

This will be useful in systems where lag does not matters, eg: reporting system can be read from slave DB.

## You chose CP

what if you chose **Consistency**?

When you are working on transaction system, consistency is very important. Let's say if your application write to master, it will immediately write to secondary nodes. If any one of node is down, it will not be saved in any nodes. This make sure that your data is consistent over all nodes.

But when one of node goes down, your API will not be available.