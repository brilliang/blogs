---
layout: researcher
---

## scalability

scalability: System's capabitity of load is increased in a manner proportional to resources added -- heterogeneous resource could be supported properly.
* If you have a performance problem, your system is slow for a single user.
* If you have a scalability problem, your system is fast for a single user but slow under heavy load.

[a serial](https://www.lecloud.net/tagged/scalability/chrono)
1. [clone](https://www.lecloud.net/post/7295452622/scalability-for-dummies-part-1-clones): every server contains exactly the same codebase and does not store any user-related data, like sessions or profile pictures, on local disc or memory. They need to be a centralized data service which is accessible to all your application servers.
1. [database](https://www.lecloud.net/post/7994751381/scalability-for-dummies-part-2-database):  denormalize right from the beginning and include no more Joins in any database query, or you can switch to a better and easier to scale NoSQL database like MongoDB or CouchDB. Joins will now need to be done in your application code.
1. [cache](https://www.lecloud.net/post/9246290032/scalability-for-dummies-part-3-cache):
1. [Asynchronism](https://www.lecloud.net/post/9699762917/scalability-for-dummies-part-4-asynchronism):
    * doing the time-consuming work in advance and serving the finished work with a low request time, e.g. turn dynamic content into static HTML files on every change.
    * use message queue: [RabbitMQ](https://www.rabbitmq.com/getstarted.html),  ActiveMQ or a simple Redis list.

### cache
cache policy: LRU, FIFO
cache pattern:  Cache aside, Read through, Write through, Write behind caching

#### cache aside
read: hit, return from cache, otherwise, read from DB and on the way back save in the cache
write: directly write into DB, ***after*** the writing success, invalidate the item in the cache.

#### read/write through
app only read/write from cache. and cache will update with DB ***synchronized***

### write behind (write back)
app only read/write from cache. and cache will update with DB ***asynchronized***. Cache could write back to DB only when this block of cache is invalid.

reference:
1. https://coolshell.cn/articles/17416.html
2. https://zhuanlan.zhihu.com/p/42276548


## CAP
when you are designing a distributed system you can get cannot achieve all three of Consistency, Availability and Partition tolerance.

Networks aren't reliable, so you'll need to support partition tolerance. You'll need to make a software ***tradeoff*** between consistency and availability.


### Consistency patterns
* Weak consistency: A best effort approach is taken but nothing is guaranteed, e.g. memcached, video chat, and realtime multiplayer games.
* Eventual consistency: After a write, reads will eventually see it, e.g.  DNS and email
* Strong consistency: After a write, reads will see it. Data is replicated synchronously. transactions 事务


### Availability patterns
* Fail-over
    - Active-passive
