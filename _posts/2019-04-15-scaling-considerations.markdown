---
layout: post
title:  "Scaling Considerations, for Cocktail Parties"
date:   2019-04-15 12:00:00 -0400
categories: coding
---

I've been trying to dive more deeply into scaling web apps. When I want to internalize something, I often make up a little song or phrase to remind myself of the main points.  Below are a few sentences I came up with for scaling, as well as an explanation of what each word means.  I wanted to share it in case helpful.  This way, if either one of us finds ourself, say, at a cocktail party where scaling comes up (sure, it could happen), we can silently repeat the phrase to jog our memories about some of the major considerations.  

The memory-jogging phrase is nonsense, but there's an organization to its sentences. I find it helpful to think of them, and read their cadence, as poorly-translated furniture assembly instructions:

<p class="text-centered bold">"Load horizontally or vertically. Separate shards sequentially.  Cache your queues."</p>

The discussion below describes what this all means in a very general way.  At the very bottom is a bibliography/list of resources if you would like to check them out.

-------------------------------------------------------------------------
<p/>
<h4 class="text-centered">First Sentence: Think Server Structure - "Load horizontally or vertically." </h4>
-------------------------------------------------------------------------
<p/>
1. **"Load"** - Stands for think "load balancer".  This is a machine that handles multiple simultaneous browser requests and routes the requests to different servers to handle them.  This "balances the load" of incoming requests and avoids one server being overloaded.  The different servers that are assigned requests by the load balancer are often clones of a master server.  Each clone must be updated every time the master server is updated, but there are services that handle this.  

2. **"Horizontally"** - Stands for think about horizontal scaling.  Horizontal scaling means to increase the number of servers you use to handle incoming requests. This would tend to go hand-in-hand with the load balancer set up described above.
You might also consider the use of a **content delivery network, aka a CDN**. Very generally speaking, CDNs are meant to deal with the fact that internet speeds are affected by the physical distance between a requesting browser and a responding server (**keyword: latency**).  CDNs put clones of your master server in different geographic areas and ensure that each clone handles requests made nearest to them in order to minimize user wait time.  For example, if you are a U.S. company with a U.S. website and a Canadian customer base, you might want to make use of a CDN to have a clone of your master server in Canada, so that Canadian customers don't get frustrated waiting for a distant, U.S.-based server to respond to them.

3. **"Vertically"** - Stands for think about vertical scaling. Vertical scaling simply means beefing up each server you use -- adding more disk space, more RAM, more processing power, etc.  There will always be limits to this, so I get the sense it's only a band-aid, not a fantastic solution, when trying to scale.

-------------------------------------------------------------------------
<p/>
<h4 class="text-centered">Second Sentence: Think Database Structure - "Separate shards sequentially." </h4>
-------------------------------------------------------------------------
<p/>

1. **"Separate"** - Stands for separate your web hosting server(s) from your database server(s).  This is good practice in general.  Don't forget to think about having **redundant databases** - if you put all of your data eggs in one basket and that basket breaks, you've got a bad scene on your hands.

2. **"Shards"** - Stands for consider "sharding" your database, also known as **partitioning**.  Sharding means to divide your database up among multiple machines. There are different ways you could do this.  For example, if you run a blog, you might have one machine for posts, one for reader profiles, and one for comments.  You might also utilize "directory-based" partitioning, meaning there is a separate machine with a table for looking up information housed on other machines. As you consider sharding, you're going to want to avoid problems from maxing out a machine's physical memory (so you have to reconfigure or move data) and avoid bottlenecks (for example, all requests must pass through one machine, which could overload it and slow performance).

3. **"Sequentially"** - Stands for think about SQL vs NoSQL. SQL is a relational database schema, and "join"
 lookups can be slow. NoSQL schemas (such as MongoDB and Couchbase) were born out of a desire to work around the limits, constraints, and timing issues of traditional SQL databases.  NoSQL schemas utilize alternative ways of relating data and do not support join lookups. They also have a better reputation for scaling in general. If you are going to stick with SQL, one potential way to improve performance is **denormalizing**, which basically means to repeat commonly-used data across different tables to avoid using a join lookup every time this commonly-used data is needed.

 -------------------------------------------------------------------------
 <p/>
 <h4 class="text-centered">Third Sentence: Think Improving Performance for Complex Tasks - "Cache your queues." </h4>
 -------------------------------------------------------------------------
 <p/>

 1. **"Cache"** - Stands for consider storing the results of complex processes in a cache for fast lookup, rather than running the processes each time a request is made. Cache in this context means a memory layer in between your application and your database. When a request or query is made, the application first checks the cache to see if the results are already stored there, and if so, the application can provide the results quickly. If not, then the application looks to the database per tradition.  Storing commonly-used, complex queries in the cache can provide a major speed boost and keep users from having to wait for complex query results.  But there's a tradeoff: the cache results might be slightly stale, because, after all, they were already pre-computed and stashed away in the cache.

 2. **"Queue"** - Stands for plan to have a queue, hand-in-hand with utilizing a cache.  The commonly-used, complex requests or queries that are stored in the cache have to get updated somehow, to keep them only *slightly* stale and not *majorly* stale.  The updating is handled by a queue, which keeps track of the processes to re-run for the purpose of updating the cache and dutifully performs these processes on a first-in-first-out basis.   

 -------------------------------------------------------------------------
 <p/>
 <h4 class="text-centered">Bibliography / List of Resources </h4>
 -------------------------------------------------------------------------
 <p/>

* <https://blog.hartleybrody.com/scale-load/>
* <https://aws.amazon.com/blogs/architecture/scale-your-web-application-one-step-at-a-time/>
* <https://www.keycdn.com/blog/scale-website-traffic>
* <https://www.cloudflare.com/learning/cdn/what-is-a-cdn/>
* <https://www.imperva.com/learn/performance/what-is-cdn-how-it-works/>
* <http://highscalability.com/start-here/>
* <http://highscalability.com/blog/2016/1/11/a-beginners-guide-to-scaling-to-11-million-users-on-amazons.html>
* [Chapter 9 of CTCI](https://www.amazon.com/dp/0984782850/ref=cm_sw_em_r_mt_dp_U_uODZCbZ8AD0BA)
* <https://www.hiredintech.com/classrooms/system-design/lesson/60>
* <https://www.lecloud.net/post/7295452622/scalability-for-dummies-part-1-clones>
