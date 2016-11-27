+++
title = "The demise of spinning disks"
date = "2012-03-18"
slug = "the-demise-of-spinning-disks"
categories = [ "Technology", "Storage" ]
tags = [ "HDD", "SSD", "Server Side"]
+++

It was quite some time ago when my company started using Fusion-IO (enterprise SSDs) on our database servers. We are happily using them till this day. The great part is that we could scale vertically and leave software layer unchanged. Even better: we **still** have room for growth, despite getting more users and our application stack getting more and more demanding. We were one of the first to switch to SSDs on server side - so right now it's interesting that this idea propagated so quickly. We got to the point where people start loudly saying "disk is the new tape". And it really is.

The gap between CPU speed and external memory has been widening for a long time, and got to the point of being simply ridiculous. It's worth noting that it's not only about gap between CPU and disk drive but also about gap between CPU and DRAM (which is slightly less known fact). For CPU accessing L1 cache takes single digit amount of clock cycles. Subsequent levels of memory hierarchy are slower and slower. Accessing main memory gets us into area of hundereds to even 1000+ CPU cycles (this is strongly architecture dependent). This is very bad and low level programmers have been battling this issue for a long time now. One of approaches is using access patterns that minimize CPU cache misses and using low level prefetch instructions. We are essentally talking about linearizing access to memory that is supposed to be "random access". If latency is the problem in case of DRAM, how bad it is in case of disk drives? Very, very bad.

Let's start with simple analogy. I need something and: 

  * it happens to be in my room (that's my L1 cache) - it takes a few seconds to get what I need
  * ok, it's not on my desk (I have to hit DRAM) - it takes a few minutes to get what I need, an equivalent to, say, walking to a neighbor and borrowing something
  * hmmm, actually my neighbor doesn't have it and I have to hit the store - how much would it take if the store was the disk?

This is where 99% of people totally fail at estimation saying something like "few hours". The real answer is: much, much longer.

Let's assume that DRAM latency is at 100 ns (so about 300 clock cycles of 3GHz CPU), and average disk seek time is 6 ms (a very fast disk!). The difference is 60 000 times! We are not talking about "get into a car and drive to the store". In our hypothetical scenario we are actually talking about **months**! But it's only part of the story, because it's not just about latency - it's also about parallelism.

For a hard disk drive latency determines the upper number of random I/O operations per second because disk has to do everything serially. The serial nature of HDDs is the real problem and source of pain as CPUs are becoming more and more parallel, putting more and more processing power into roughly the same space. Because CPU throughput is skyrocketing (high end Xeon E7 is 10 cores/20 threads), we need more and more I/O operations per second. Single hard disk cannot provide these, and will never be able to :( In fact, most of server side programming nowadays revolves around optimizing for I/O (in general), not for CPU. This happens at many levels - operating system kernel, file systems, database storage engines, applications ... Tons of software engineering work was performed just to hide simple fact that the disks cannot keep up. Interesting consequence is the proliferation of Hadoop (and other implementations of MapReduce) as an alternative to relational databases in many data processing tasks. Hadoop takes what's good about disks (cheap bandwidth) and gets rid of the latency problem by streaming all the data, instead of randomly accessing it.

SSDs still have latency of ~0.1 ms - it a lot faster than HDDs, but at the same time a lot slower than DRAM ... What's special about SSDs is not the access time - it's the parallelism. Enterprise SSDs offer hundreds of thousands of I/O operations per second - the difference here is a lot bigger than the difference in access times. Additionally, while the performance of disk drives is not really improving any more, Flash memory based solutions have huge room for improvement - and their I/O performance is going to improve as fast or even faster than CPU performance. One very simple reason for that is that right now we are still trying to access them through traditional file I/O interfaces that weren't meant for this kind of usage ... In fact, when we start treating SSDs for what they really are, a memory, and forget about block device+filesystem nonsense, we are going to see massive performance gains. It's already happening in the form of Auto Commit Memory technology from Fusion-IO. Surely more will follow, because right now interfaces and API layers are huge source of inefficiency. Also, it's good to know that NAND Flash is not the last word in technologies of non-volatile memory and there are others on the horizon (although I am not aware of any commercial applications right now). We are heading towards a world where speed gap between HDD and SSD is measured in thousands of times, and getting wider every day.

Unfortunately, because it's a revolutionary shift, the price of Flash memory does not go down as fast as one might want it to. Simply speaking, now **everyone** wants all of their hot data to be in Flash memory. Massive demand for Flash memory in consumer electronics does not make things any better - Apple is likely sucking in significant chunk of world production :) Interesting question is: where are the SSDs in the cloud? Recently, as it turns out, Amazon used a lot of these to launch DynamoDB. Which from my point of view creates interesting problem - because it becomes essentially impossible to build comparable service on "raw" EC2. That means getting locked into AWS. Will Amazon introduce some sort of "high I/O EBS"? I am sure I am not the only one interested in such product.