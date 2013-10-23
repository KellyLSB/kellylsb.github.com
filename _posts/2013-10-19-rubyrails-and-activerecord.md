---
layout: post
title: "Optimizing Ruby/Rails: AR Sucks"
description: "Part 1 in a series of responses about Ruby's scaling capabilities"
category: [optimization, startups]
tags: [scaling, ruby, rails, activerecord, optimization]
---
{% include JB/setup %}

_The code to run this example is avilable at [https://github.com/KellyLSB/objective_oriented_ar_sucks](https://github.com/KellyLSB/objective_oriented_ar_sucks), this example uses MySQL. After bundle installing you can run `bundle exec rake db:create db:migrate run` to run the test._

So I'm over at Toorcon San Diego and I was thinking about a project I was working on this past week at work.

So if you ask most senior engineers, CTO's and various other business people within the tech scene you will find most of them will tell you that Rails applications do not scale and most of them have proof of this fact; but the problem is not Ruby, Rails or the application needs at all. It's the engineers working on the application and their knowledge of writing data structures properly, to use the least amount of resources, be the shortest readable code, and scale efficiently and to an endless degree.

Now this is just a small example of a small change that can make a big difference in your application scaling and processing. There are many other methods to improve performance and I will get to those later.
If you have any suggestions you would like me to share feel free to fork my repository at http://github.com/KellyLSB/kellylsb.github.com and make a Pull Request.

* * *

Over the past couple weeks I have been thinking and preparing different benchmarks to show the difference between using the built in active record relations without joins, with joins and directly.

For this post I created a simple demo (which is available for download at http://github.com/KellyLSB/objective_oriented_ar_sucks).

I created four models titled ConnectionOne, ConnectionTwo, ConnectionThree and ConnectionFour. The idea in this example is to reach ConnectionFour from an instance of ConnectionOne using the least amount of DB and application power. For the purpose of this post I will be focussing specifically on SQL time benchmarks and the amount of instantiated classes in Ruby (data loaded).

Connection Table:

- ConnectionOne -> ConnectionTwo - `belongs_to -> has_one`
- ConnectionOne -> ConnectionFour - `belongs_to -> has_one`
- ConnectionTwo -> ConnectionThree - `belongs_to -> has_one`
- ConnectionThree -> ConnectionFour - `belongs_to -> has_one`

This is a pretty relationship schema with not to much complication in it. Now you might be wondering "Why would you try to retrieve ConnectionFour through Two and Three?"
Some companies like to "normalize" their data and specify explicit relationships to specific models rather than make as many connection as possible. There are merits to both methods.
From a database optimization standpoint however denormilization generally works a little better even if it is a little ugglier, but hey were talking about scaling here not a perfect world.

Let's run this test. (You can do it your self by running `bundle exec rake db:create db:migrate run` on the code base listed at the top).

    with AR
      ConnectionTwo Load (0.3ms)  SELECT `connection_twos`.* FROM `connection_twos` WHERE `connection_twos`.`id` = 10545 LIMIT 1
      ConnectionThree Load (0.2ms)  SELECT `connection_threes`.* FROM `connection_threes` WHERE `connection_threes`.`id` = 10545 LIMIT 1
      ConnectionFour Load (0.1ms)  SELECT `connection_fours`.* FROM `connection_fours` WHERE `connection_fours`.`id` = 10545 LIMIT 1

    with Joins
      ConnectionFour Load (0.5ms)  SELECT `connection_fours`.* FROM `connection_fours` INNER JOIN `connection_threes` ON `connection_threes`.`connection_four_id` = `connection_fours`.`id` INNER JOIN `connection_twos` ON `connection_twos`.`connection_three_id` = `connection_threes`.`id` INNER JOIN `connection_ones` ON `connection_ones`.`connection_two_id` = `connection_twos`.`id` WHERE `connection_ones`.`id` = 10544 LIMIT 1

    with Direct
      ConnectionFour Load (0.3ms)  SELECT `connection_fours`.* FROM `connection_fours` WHERE `connection_fours`.`id` = 10545 LIMIT 1

So in the first example here "with AR", essentially what I am doing is this `connection_four = ConnectionOne.first.connection_two.connection_three.connection_four`. Now if you look at the timestamps the time to load those records is really not that bad, but it is being done in Three queries (four if you specifically called ConnectionOne to reach Four). If you look at the timing there just loading those three records it took 0.6ms; mind you I'm running this in a development environment with almost no load.

So we know that we can access ConnectionFour through Three, Two and One. So we can actually break that down a little using a join and doing the whole things in a single query. The second example here labeled "with join" is that. The ruby call to make that work is `ConnectionFour.includes(connection_three: {connection_two: :connection_one}).where(connection_ones: {id: id}).first`. The code is a little bit longer but the time to run the query 0.5ms is shorter than the other method. There are other advantages other than the load time of the model. It also uses less database calls, which is especially important if you have a very high load database server; especially if you are using replication (repliaction generally replays the SQL log on the replica; this is not true of all SQL servers)! Lastly you are only instantiating one model in the Ruby environment which saves heavily of memory.

The final method `ConnectionOne.first.connection_four` and in my opinion the best method to use whenever possible is direct relations using has_one or has_many relationships (avoid many to many and polymorphic relationships as much as possible!!!!!!). If an object belongs to multiple objects ALWAYS connect them directly and do not require joining between those tables. It makes it easier to not only to code for, but also cleanup later (if you need to delete records). It uses one short database query that does not require too much processing on the database server and it also only took 0.2ms which is Much Much Much better the previous queries. Not to mention it is even less data loaded into ruby then previously.

Summary: Use direct relationships when possible, they are clean, short and have very little resource impact. If you can't do a direct connection use a join and absolutely always avoid doing model recursion in order to find a record. It is a waste of time, resouces and creates imense technical debt. Ruby is a great flexable, mutable, multi-faceted language; It makes it wonderful and terrible. To properly scale Ruby it needs it
s engineers to understand it though. There is almost always a better way to do something, and since Ruby is continuously evolving it's very important to take some time out of your day to keep up with it.

Thank You's: Thank you [Nothus](https://github.com/Nothus) for his time helping me write the example.
