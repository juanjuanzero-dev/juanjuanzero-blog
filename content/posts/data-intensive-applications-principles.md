---
title: "Data-Intensive Application Principles"
date: 2023-07-24T05:57:45-04:00
draft: false
description: "If you've wondered what principles boslter data-intensive application systems. This is what i've learned"
featured_image: https://images.unsplash.com/photo-1639322537228-f710d846310a
tags:
  - computer science
  - computer systems
  - technology
  - Designing Data-Intensive Applications
---

# What are Reliable, Scalable and Maintainable systems?

Many services today in the digital world are data intensive. Meaning, they use, produce and consume a vast amounts of data. When you are thinking about building data intensive applications the main goal of a data intensive system is to be reliable, scalable and maintainable. When working with larger and growing systems; it becomes critical to adapt to and handle the changes that come your way. Let's discuss this as far as what I have seen in the world and what I am learning from reading [Designing Data-Intensive Applications by Martin Kleppmann (paid link)](https://www.amazon.com/Designing-Data-Intensive-Applications-Reliable-Maintainable/dp/1449373321/ref=asc_df_1449373321/?tag=hyprod-20&amp;linkCode=df0&amp;hvadid=266396064900&amp;hvpos=&amp;hvnetw=g&amp;hvrand=15878266807487173621&amp;hvpone=&amp;hvptwo=&amp;hvqmt=&amp;hvdev=c&amp;hvdvcmdl=&amp;hvlocint=&amp;hvlocphy=9052202&amp;hvtargid=pla-432535594773&amp;psc=1&_encoding=UTF8&tag=juanjuanzerod-20&linkCode=ur2&linkId=2d8f7596358d868de636c9d243aaf3ed&camp=1789&creative=9325).

## Reliable
{{< figure width="100%" maxwidth="40rem" class="rounded-5" src="https://images.unsplash.com/photo-1546545817-27f0fb006153" >}}


Many seen Toyota vehicles as a reliable brand of vehicles. Customers have seen that Toyota vehicles continue to perform well relative to the age, when compared with other cars in the same class. For a software system to be reliable, means that it will continue to perform the way you expect it to behave. To have something reliable means that you can expect something to perform in a way that you expect it to. For a software system, this means it is working correctly, and it performs at the level of performance that you expect, even in the face of adversity. 

You will have *faults*, or things that can go wrong or out of the normal specification that it was made for, for example the car runs out of gas earlier than normal. You will also have *failures*, or when the system itself stops providing the service to the user, like when the car breaks down and refuses to run. While you can’t make a car able to handle faults and failures (maybe one day it can selfheal). You can make fault-tolerant software systems. 

There is a general preference to using software to create fault-tolerant software systems. This is mainly due to the hardware constraints of the physical world. Network issues can happen. Someone can trip on a power cord. Therefore, designing systems with software with a fault-tolerant component is desirable. 

## Scalable
{{< figure width="100%" maxwidth="40rem" class="rounded-5" src="https://images.unsplash.com/photo-1547637589-f54c34f5d7a4" >}}


How can you deal with growth? Scalability is the system’s ability to deal with growth. Scalability is not a binary quality, i.e. “this system is scalable” vs “this system is not scalable”, it is more a question of how it handles growth: 

- “If the systems grows in a specific way, what are the options for coping with the growth?”
- “How can you add resources to the system to handle load?”

### What is the Load?

There are certain parameters that you much look over to understand the load on your system. It all depends in the system that you are designing. It could be requests per second, the ratio of reads and writes to the database, number of users in a chat, the cache hit rate. It depends. 

For example in the case of a Twitter user, it may be the requests / sec when they post a tweet, and also read their home page. When a Twitter user posts a tweet their followers receive it in their own homepage. The challenge here is the fan-out problem of every user being able to also see everyone’s Twitter post. 

Lets outline what happens when a post occurs:

- `juanjuanzero` posts a tweet,
- Twitter’s servers receive the tweet and write the post into `tweets` database. This database houses everyone’s tweets.
- When followers of `juanjuanzero` login to Twitter, each user makes a request to the server and query the `tweets` database.

You can see here that there are two interactions going on:

- each user makes are request to get the posted tweet for their homepage
- any user can post tweets to other’s homepage

`juanjuanzero` only has a few followers so the database is able to handle those requests. However, any user can post a tweet, and there are users with millions of followers (influencers). For the influencers, the current design would reach a limit in this approach in the amount reads to `tweets` database. You can add more resources to the database, but the design itself imposes a limit on the parameter: the amount of reads to the `tweets` database. We’ll get into reads and writes to a database in a later post. If you are unfamiliar just imagine it as another request to the database. 

An alternative design would be to have a homepage cache for each user. Let’s say `juanjuanzero` becomes an influencer and reaches 1 Million subscribers (hey, it could happen). When `juanjuanzero` posts a tweet to the server. The server looks up all of his followers, and updates the homepage cache of each of those followers. The cache will reduce the amount of reads required from the `tweets` database, because now when each user pulls up their homepage, this is retrieved from the cache. 

Lets summarize: 

- The load in question is: the amount of reads to the `tweets` database
- The issue with the first design that the system did not scale with respect to the amount of reads to the `tweets` database
- The alternative approach was to add a homepage cache for each user interested in the database, which reduces the amount of reads to the `tweets` database.

### What is the performance?
{{< figure width="100%" maxwidth="40rem" class="rounded-5" src="https://images.unsplash.com/photo-1605313294941-ea43850d9de5" >}}


Once you know how much is coming down the pipeline, you can now determine how that impacts performance. In the scalability example, we described the how one design is able to handle the load up to a certain point. The performance aspect will help us determine what that point is. 

Here are some questions to ask:

- When you increase the load and keep the same resources, how is performance affected?
- When you increase the load, how much do you need to increase the resources to keep the level of performance unchanged?

What is performance anyway? This varies with respect to the system that you are designing. If you have something that processes requests, you may be interested in *throughput*, or how many items get processed through the system. Looking at it another way this is the time it takes to run a job on a dataset of a certain size. 

If you have an online system where communication between processes is important performance may mean: short response times from the server. You’ll hear about certain metrics like the p50 or p99 response times. These are your aggregated response times over a specific distribution. For example, when someone says `p99 is at 5 seconds` it generally means that 99% of your requests are below 5 seconds, and 1 out of every 100 requests are above 5 seconds. This is to say generally speaking you have outlier requests that take longer than 5 seconds to complete. 

Metrics like p99 are what would be described as *tail latencies.* It’s called tail because these are in the tail-end of the distribution, meaning they are relatively rare when compared to other events that happen. Should you care about the tail latencies? It all depends on the business value that the system provides, tail latencies may correspond to high value customers, or perhaps its more important to pay attention the majority of the consumers. You the designer should have this answered.

### How to deal with load

How can you maintain “good” performance when dealing with the load increase. There is a business nuance here, where “good” is a relative term that is typically defined by what the business can handle. Perhaps “good” means “good enough for our customers”, or “good enough that we can afford to pay the cloud bills”. It depends. The cost is also a major component. There are two ways to handle load: you can scale-up or scale-out. 
{{< figure width="100%" maxwidth="40rem" class="rounded-5" src="https://images.unsplash.com/photo-1624213111452-35e8d3d5cc18" >}}


Imagine you, a super hero, are surrounded by your enemies, they have you cornered, there is no way out, except through. You’ve been fighting them off one by one, or even one against three. But this time, there’s 30 of them. How will you handle this?

Scaling-up generally means adding more resources like compute or memory to handle the influx of requests. This is like injecting yourself with super soldier serum and becoming faster, stronger and smarter to handle your enemies. Eventually the effects wear off and you are back to normal handling normal load. 

{{< figure width="100%" maxwidth="40rem" class="rounded-5" src="https://images.unsplash.com/photo-1531989417401-0f85f7e673f8" >}}


Scaling out means spreading the load among smaller machines to handle the influx of requests. This is like cloning yourself to match the number of enemies you are up against. Some systems are designed to be *elastic*, which means that you increase and decrease the number of machines with respect to the number of requests.

## Maintainable
{{< figure width="100%" maxwidth="40rem" class="rounded-5" src="https://images.unsplash.com/photo-1621905251918-48416bd8575a" >}}


Once out, the system must run smoothly, the requirements will change, and customers will use the software in unexpected ways. Software will need constant maintenance. We should design software in a way that will minimize the pain during maintenance and focus on a few principles: Operability, Simplicity and Evolvability.

### How easy is it to operate?

Evaluating how easy is it to run the software system smoothly is good, How difficult is it to get visibility on the health of the system, Is it difficult to track down problems with the software? How difficult is it to diagnose and debug system faults and failures? Are routine tasks difficult to do? These are questions that you can ask to determine where the system is in terms of it’s operability.

### How simple is the software?

Complexity reduces maintainability. When engineers return to the software, how quickly can they ramp up to the system? Even if that engineer is you from the future, how easy would it be for you to jump back into the code and identify the areas that need to change. 

Are there complexities that are not inherent to the problem at hand? This is what is called “accidental complexity” which can create *unforced errors.* Unforced errors are those mistakes in play that is attributed to the failure of the executor rather than an external force. It is a careless mistake. Accidental complexities typically arise out of the implementation. We can look at abstractions at a high level and see how they are used in the implementation. Good abstractions hide complexity in easy to understand interfaces. When a system uses abstractions that is difficult to reason about, this is one area fertile for accidental complexity. 

### How easy is it to iterate on?

Change is the only constant. The ability of your software to adapt to changing requirements, business deadlines and while the issues that come with the software system itself is critical to moving forward. This will be closely link to how simple the implementation is and the use of abstractions in that system. Having a clear understanding of each is important and will impact the level at which the software system can evolve not just in terms of load on the system but also in terms of the needs of the business. 

## Its a good problem to have

When designing data intensive applications aspects of reliability, scalability and maintainability are critical. With this in mind we can confidently keep moving forward to service the growing demand for our system and the value it provides to others. While this seems like a difficult problem, this is a good problem. It’s a good sign that the value the system provides is important. Take pride in that, you are making something that will have an impact in a lot of people.