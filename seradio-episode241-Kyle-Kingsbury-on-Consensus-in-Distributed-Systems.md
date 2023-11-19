Link: https://www.se-radio.net/2015/11/se-radio-episode-241-kyle-kingsbury-on-consensus-in-distributed-systems/



This is Software Engineering Radio, the podcast for professional developers on the web at se-radio.net. SC Radio brings you relevant and detailed discussions of software engineering topics at least once a month. SE Radio was brought to you by IEEE Software Magazine, online at computer.org slash software.

**Stefan Tilkov** 00:00:36 For Software Engineering Radio, this is Stefan Tilkov Today I'm joined by Kyle Kingsbury, better known by his Twitter handle Aphyr, to talk about database consistency. Kyle has become quite famous in the community because of his tendency to break databases that is, use rigorous testing to show how they deal with various error scenarios. I've been a long time follower of Kyle's posts and talks, and I'm sure you'll be as fascinated as I am, by his insights into the theory and practice of databases handling failure, or, unfortunately more often, failing to do so to varying degrees. Kyle, why don't you start us off by introducing yourself and tell us a little bit about yourself?

**Kyle Kingsbury** 00:01:20 Well, my name is Kyle, I'm a backend engineer by trade, and I work right now at a company called Stripe, where I'm doing database safety analysis, trying to understand whether or not distributed systems, caches, databases, queues, that sort of thing, whether or not they live up to the guarantees that we expect from them.

**Stefan Tilkov** 00:01:43 Okay, so for those of our listeners who haven't really been exposed to those kinds of problems, can you talk a little bit about what issues might arise when people use technologies like that?

**Kyle Kingsbury** 00:01:55 Sure thing, we see consistency problems all the time. For example, you might go to Twitter and read some direct messages, and then go from the website to your phone, and you'd notice the same direct messages are shown as unread. That's a synchronization problem, a consistency problem, where different computers, different clients have different ideas about the state of the system. And this goes much deeper, you can get into errors with bank accounts. For example, there was a famous Bitcoin exchange that lost a lot of money because it's not correctly debit and credit accounts in a transaction. Facebook, for example, has all sorts of interesting out-of-order issues with display. These are problems that arise sometimes because they're simply no good way to do them at scale. And sometimes just because we didn't understand what the necessary safety permits were.

**Stefan Tilkov** 00:02:46 So is this something that only companies that run really, really big data centers are exposed to? Do I have to be running something like Facebook or Twitter to run into those kinds of problems?

**Kyle Kingsbury** 00:02:56 I don't think so. But it's tricky to give good data about it because most people instrument their consistency. They don't measure whether the system is actually safe. So one of the things I'm trying to answer is how likely is it that you might see real-world impact during certain common failure modes?

**Stefan Tilkov** 00:03:15 So one of the things that I often run into is that people who use traditional relational databases, the articles and DBTUs and MySQLs and Postgres, whatever, most kinds of things believe that they don't have to worry about those things. Are they right?

**Kyle Kingsbury** 00:03:32 Yeah. It depends. There are certainly consistency issues in relational databases as well. There was a paper published recently actually on a Postgres error where it's serializable isolation level failed to be serializable in a sort of surprising way. And so I think you'll find these safety issues affect people both at a level of bugs and just because understanding the weaker, anti-ascual isolation levels is a little bit tricky. The spec is not entirely clear about what shouldn't happen.

**Stefan Tilkov** 00:04:02 Can you talk a bit through those isolation levels?
Not necessarily all of them, but give us an idea about what is specified there?

**Kyle Kingsbury** 00:04:10 Sure thing. So as I understand it, as relational databases are coming into their own, different database companies realized that it was impossible to provide complete, nice global orders for events in the system. You want all your transactions to look like they take place sequentially one after the other. But actually doing that is relatively expensive. And so maybe there are shortcuts. Maybe you could avoid taking out a complete lock and allow two transactions and don't touch the same data to complete the same time. When you do that, there are different tradeouts you can make regarding how long your card locks for, whether you lock for reads or writes. If you do multi-version currency control, you can get different sort of what we call anomalies. Their behavior patterns, orders in which transactions could execute, that look maybe a little bit dodgy. In some cases they're okay, but in some cases they're not. For example, we get a problem called phantoms where you might write a new record into a data set. And at the same time you make a query against a table. And that query should have seen the new record, but does not because it's predicate didn't yet match for whatever reason. So maybe access to the same row in the table is synchronized correctly, but that row might not appear in a query that should have seen it. It's a subtle trick you think the reason about. So what happened was the different database manufacturers were starting to standardize and nobody wants to be left out. So we have this standard NCSQL that codifies different levels of safety. And those levels sort of correspond to whatever trade-offs the databases might have made historically so everybody could say they slaughtered them somewhere. So practice you'll get a wide variety of interpretations of those anomalies and then different levels of actual conformance to the spec.

**Stefan Tilkov** 00:06:05 One thing that I find interesting is that you can actually specify some of those or configure them, right? You can make a decision of what kind of isolation level you want. So this idea is nothing new. It's been there for a long, long time that people can make, have to make trade-offs regarding consistency levels, right?

**Kyle Kingsbury** 00:06:22 Exactly. So you can choose to jack up the isolation level serializable for every transaction. But this is somewhat slow. So a lot of people will start off recommitted or even back after it uncommitted depending on what their application needs. That's the ideal case. But some people will just turn it down to recommitted or leave it at the defaults because it's faster without understanding implications and without fully reasoning through the transactions. I think it's sort of an open research problem right now. I'm not sure which transactions, which patterns of access in the database require which isolation levels to be safe.

**Stefan Tilkov** 00:06:57 So how is this related to the famous cap theorem and can you maybe explain that to us a bit?

**Kyle Kingsbury** 00:07:03 So see, well, there's two things. There's actually a cap theorem which is a formal proof by Gilbert Lynch. And that tells us that of consistency, which they mean linearizability, availability, which means total availability, that is every request to a non-failed node, a non-crashed node must succeed. And of partition tolerance, that's the P in cap. You can only have two out of three. Now that actually derives from a sort of conjecture or informal claim made by Eric Brewer that of consistency, availability, and purchasing tolerance, there's some tradeoff and you can really only look at two out of three. So when people talk about cap in a modern context, I think they're typically referring to the proof, which is very strict definitions about what availability consists to mean. But that doesn't mean that by relaxing consistency, you necessarily get availability. There turns to be other proofs that exclude different classes of operations, different business classes from being fully available as well. So cap is a very particular example of the sort of larger class of problems of consistency. A consistency model is a set of allowed histories, allowed behaviors that take basically go through. So if you were to take all the different operations you do, like reading a value and writing it into some other table or incrementing a number or setting three fields at once on a given record, those are all sort of transactions. If you take all transactions in a system, in practice, the different transactions are applied somewhat concurrently, maybe somebody is writing data and somebody is reading data at the same time. What you want is for the operations to take place in some kind of order. So maybe part of a transaction takes place first, another transaction sneaks in, it does a little bit of work, another transaction comes in, does a little bit of work, the first transaction comes in and finishes. There's lots of different ways those events can be ordered. And the consistency model is the class of allowed orderings of those events. So CAP tells us about a very particular consistency model. Just like NCSQL tells us about certain allowed models, like read-uncommitted, read-committed, they prohibit certain classes of orders. They prohibit you, for example, from reading the result of another transaction is right until your transaction is committed. Or you might get snapshot isolation where all of your operations are supposed to take place on some atomic snapshot of the world that occurs at some logical time just before you commit those consistency classes can only be achieved in certain availability models. And there's different theorems that tell us which classes are achievable and which in which availability settings. CAP tells us that one of the strongest classes of consistency, the one which allows the least number of reorderings, well, not the least, but a very small number. It's still infinite, it's complicated. CAP tells us about linearizability, which is this very strict ordering, and it says that linearizable systems cannot be made to be fully available. If you build a linearizable system and it experiences a network partition, a communication failure between nodes, it's possible for transactions to stall forever or for transactions to be incomplete. You either have to deadlock or give up on certain operations.

**Stefan Tilkov** 00:10:23 I think everybody can imagine what consistency means, sort of. You mentioned linearizability and the system goes from one consistent state to another, and you have this feeling that there's no, you don't get a dirty read of some kind, you don't get half committed things and you don't miss anything that has been has been written or has been committed. I think everybody can imagine what availability is about. But I always feel that people struggle with this idea of partition tolerance. Why use a term like that? What does it actually mean?

**Kyle Kingsbury** 00:10:54 I think that the three partition tolerance is the easiest to understand.

**Stefan Tilkov** 00:10:56 Oh, that's interesting.

**Kyle Kingsbury** 00:10:57 Yeah, all it means is that the system should be resilient or should continue to function in the event that messages are not delivered.

**Stefan Tilkov** 00:10:07 Okay.

**Kyle Kingsbury** 00:10:58 And we see this happen in production network all the time. Microsoft ran a really interesting paper in sitcom a couple of years ago where they measured their database packet loss over several years. They have a lot of machines and a lot of routers. They actually gave pretty good quantitative evidence about the frequency and severity partitions. It turns out to be far more common than you would expect even with good change control even with good hardware.

**Stefan Tilkov** 00:11:29 Okay. So a partition can occur for any number of reasons. It doesn't matter whether it's actually the network being down or some other problem along the way. It's for some reason the message doesn't arrive at the node it was supposed to arrive at. Is that what it's about?

**Kyle Kingsbury** 00:11:44 Yeah, exactly. So it could be a delay because delays and drops are sort of instinctionable from one another on a sort of time scale. If I'm allowed to arbitrarily delay a message I can always out wait you as a software system because at some point you're going to say, well, all right, I give up it's not going to happen. So you can get garbage collection pauses. For example, when a run time stops execution of the program to clean up some memory, no messages will flow in or out. So that introduces a delay and that can look like a node partition. The operating system, you know, if a disk scheduler goes south a lot of times that will stall IO operations. And so if you're stuck waiting for the kernel to do some IO, your program might not be running anymore. And so you again have this sort of delay. Network partitions can happen from the actual network. You know, if you have a cable that gets killed by a backhoe or unplugged, you'll get into logical software errors where a race condition occurs and some messages never get sent. There's all sorts of different ways this can happen.

**Stefan Tilkov** 00:12:43 Okay. So if I, if as you said, I can pick any two from those from those three properties. Any walk us through the three options? Do they all actually exist? And what, what kind of, you know, what's the, what's the result? If I have a CA, CP or an AP system.

**Kyle Kingsbury** 00:13:00 So there's, there's been some debate about what two out of three really means. Well, the paper is very explicit. It says, if you ever have a prediction, you must use either AP or CP. And experimentally, it looks like networks do experience partitions on a relatively frequent time scale. Given the regions are going to happen, I think we should make AP or CP tradeoffs. Okay. So there is no CA because there is no system that doesn't, that doesn't run into partitions anytime. There are maybe systems that are not run into predictions anytime. If you have a synchronous network and you're very diligent about maintaining it, you're okay. However, any system which is CP can also be totally available if an algorithm is in a way that you can do a lot of things. So you might as well shoot for AP or CP and simply in the common case when the network is working, do everything correctly.

**Stefan Tilkov** 00:13:54 Okay. 

**Kyle Kingsbury** 00:13:55 So you can have both as long as the network is running and then you degrade to one of the other. It's sort of like asking, if something bad happens to your network, which are you going to preserve?

**Stefan Tilkov** 00:14:04 Okay. So, so let's, let's talk about those two AP and CP. What, what is the, what kinds of tradeoff am I making if I choose one of those two?

**Kyle Kingsbury** 00:14:12 So AP databases offer you availability first and foremost. And that means total availability that you can make a request to any node. You can make a write, you can make a read, you can make an update, a complicated query, a comparison, whatever you want to do against any node is fine. This makes them really good candidates for highly available systems in a single data center. But it also makes them good candidates for distributed systems that are geographically replicated because you can write independently in any given data center and with low latency immediately return a result. So that's one of the property, right? Everybody wants that.

**Stefan Tilkov** 00:14:49 For people who, who, who consider that does an AP system give up consistency completely?
Or what is, what does it actually mean to have an AP system?
So A in the sense of the cap theorem is total availability.
And there's a whole bunch of different proofs to tell you what the sort of maximal consistency
models are you can get.
There's actually infinitely many, but the common ones are going to be rights follow reads.
So you guarantee that if you do a read of an upper, a read of a given value that your
write is logically a success of that.

You get monotonic reads where if you read two things in a row on the same session, they're
going to happen sequentially in the sort of logical system.
Monotonic writes where a write will follow a write, you can get read committed monotonic
atomic view and also PCI, which I do not remember the after instance for.
But all those things are totally available.
So you can have these, these guarantees, what your system will do.
They're so consistent.
It's just that they don't exclude all classes operation.
I see.


So having an AP system just means that I don't have 100% consistency.
It doesn't mean that I, you know, I don't, I don't give up consistently completely around
into.
I make some, I make some trade offs regarding consistency to achieve this 100% availability.


Is that a good way to phrase it?
Exactly.
And it's, it's not like it's a fractional consistency.
It's that it is 100% consistent with respect to that consistency model.

So your system is completely safe if it obeys certain rules.
For example, if you use CRDTs, conversion replicated data types, it's okay to do updates
anywhere you like and they'll eventually converge on some value.

If, if that's your data model, you know, a system like Ryak or Cassandra, where it's AP
and it doesn't offer you any sort of guarantees about when you're going to read data, that
is, that is safe.
But you can't do all, all things with it.
If you want to do a counter, for example, in Ryak, you have to deal with the fact that
you might write a value and somebody else might not see it for quite a while until that data
is replicated over.
So you have to be okay with, with allowing more histories, wider windows of concurrency
in the system.


Okay.
So I already mentioned two examples of AP systems.
Do you have a bunch, a bunch more that you could just briefly mention?
Not to explain what I think we'll get to that.
Just tell us what are typical AP systems?
Yeah.
So Ryak and Cassandra are both Dynamo clones, which comes out of a paper called Dynamo
from Amazon.
There's also Voldemort and a few others.
The, the sort of lesser, I mean, a lot of people don't think about them this way, but
I think if you have like a cluster of them cache machines or cluster of BEDIS machines
that are sort of independent and you run your sharding layer on top of them, in a lot of
cases that is sort of analogous to an AP system.
Now it depends.


You know, CP is also sort of rare because in order to be truly CP, you need to be linear
zable and relatively few databases are.
So let's talk a bit about CP then.
That will be another trade off.
I give up some availability.
I don't have an unavatable system.
I have a system that, that makes a higher priority of consistency than of availability,
right?

So this property of having, as you mentioned, any node be able to accept a right or read.
So what is the typical kind of system that I get if I want, if I shoot for CP as opposed
to AP?
So yeah, exactly.
You can still be highly available, but you cannot be totally available.
So in a CP system, the trade off you'll typically see is majority quorums where you must have
a majority of nodes reachable and connected in order to make progress.
If you're on the sort of minority island or if you're on single nodes isolated, you've
given up the ability to do some oral operations.
You might be able to get a reduced set of operations like maybe I can read data, but it's
not guaranteed to be recent anymore.


So the really common ones here are going to be zookeeper.
At CD, even when disco has a passive implementation side of it.
I think foundation DB is claiming to be relatively strong, although I don't mentally promise
serialized full strong serializability.
And there's a broader class.
So linearizability makes certain guarantees about the wall clock time.


If I write a value to a linearizable database and it comes back and says, okay, you're right
as complete, I am guaranteed that anybody else that makes a request after that in wall clock
time, we'll see that right.
But there are other sort of strong consistency models which are also not totally available.
So sequential consistency, which guarantees you have a total order of all events and that
every process sees the same ordering.
That is not totally available.
Serializability and strong serializability, which simply enforced there must be a total
order.


We're not sure what it is.
Those are both preheathers.
You can't even have repeatable reader snapshot isolation or cursive stability, which are
the kind of strong ones on the SQL side of things.
Those ones cannot be totally available either.
So if you were to build a cluster of my SQL minutes, like what is it?
Percona, something like this.
If you have like an Oracle cluster or whatever, there are theoretical upper bounds on how
available that cluster can be in order to meet the guarantees of cursive stability or
repeatable reads so on.
Very nice.


So after all this theory, maybe let's go to some practical things.
So one of the things that I found really fascinating is that you did not only write a bunch of
smart blog posts or well you did, but you did not only write them based on academic theory,
you actually tested systems and the system that you developed for that or the tool that
you developed is called Japson.
Can you talk a bit about what that is and how you ended up building it?

	Yeah, so none of the things that I've said are particularly novel.
This is all, I think, relatively well understood research in the academic field dating back
to really the 80s.
However, I think if you're a practitioner right now, somebody who's going out and building
a backend for a website or something, it's really not common for somebody in that position
to have the time or the interest or the trend to go and read academic papers.
It's not an easy thing to absorb.
Even for me, it takes me a full week to read a paper.
What do I read?
Where do I start?
There's no good books.


And because consistency is not something that's going to bite me right now, if I'm under a
time crunch, which I think pretty much all engineers are, I'm just going to prioritize
building more features and getting stuff out the door.
And if people start to complain, I'll go look at safety and consistency.
So we're in a world where engineers are not cognizant of the literature and don't believe
that the literature describes failure modes that could actually happen to their system.
And it's unclear because we're not measuring our systems and whether or not that's a real
problem, right?
Like, okay, you know, red is sent null, seems to be really unsafe and these sort of academic
wankers say it's unsafe.
But is it really?
Like, it seems to me.
I've added numbers and I've done reads and they seem to be all right.
So I wanted to build an experimental harness to actually measure whether or not these
theorems had sort of measurable real consequences.


So how did you go about that?
I mean, you would have to set up all of those systems and then re-drive some abstract things.
What was the actual process that took you there?
So Tom Centaro and Mark Phillips asked me to give a talk at recon, distributed systems
conference in New York in 2013 and I scrambled the Figaro what to do.
I knew I wanted to talk about consistency but I wasn't sure how to build, how to really
do this verification thing.
So the software that I wrote was very simple.
So just said, let's add a bunch of numbers to a set and then we'll read the set and if
all the numbers we inserted are there, it's good.
That sounds easy.
So this doesn't require order.
You can reorder something between but it's something you can implement on most databases
and get some understanding of the city properties.
Okay.
So how did you build that?
What database did you pick and how did you end up building this kind of thing?
Originally I wanted to do REOC because there was some mailing with discussion about the
left-right lens behavior.
So I did REOC.
There was something from Antirez that I thought was a little dodgy and sentinel.
So I was like, all right, let's take a look at Redis 2.
I wanted to do MongoDB because that's sort of a very common and popular database that
a lot of people are shawling out.
But historically, it's public perception has not always been so positive as far as safety
goes.
And then Postgres, which has this reputation of being completely safe and ironclad.
In fact, they're doing very good engineering work, but there are some subtleties that you
might not think about.
So I took those four databases and wrote harnesses for each one that hooked them up to this
little set machine.
It's going to add ad ad ad numbers and then do a final read and we'll check to see how
many were dropped and how many were present in the final set that we thought might have
failed and it has all the statistics and computes.
And then I started out by running a proxy actually to everybody's database would talk
to my proxy software and it would route the connections for you and drop them.
That turned out not to work very well.
And so what I had to back off to was running each one on a separate VM and I tried to be
slow.
So I ran them in LHC instances and that kind of worked.
And so I wound up in a sort of horrible mishmash of shell scripts and Ruby and enclosure that
would cut off things with ID tables and do slowness and mesh results.
Okay.
So essentially this software is out there and anybody could test your results and verify
your results, right?
And just have it set up a cluster of whatever database will all the databases that you've
taken a look at so far.
And how many databases did you take a look at by now?


We did four in the first round.
I think three in the second and three in the third.
So I guess we're up to 10 now.
Maybe 11.
I'm almost lost track.
And there's been some sort of side work that was more theoretical looking and modeling.
The latest round I'm working on right now, it's probably going to be another three databases
and hoping that will show up and some talks this spring.
So if you had to categorize those results, was it like, you know, 90% just a pass with
flying colors, everything was great.
And that was just a very, very few that broke or was it all were bad or what was the general
kind of the type of result that you got from those tests?
It's always surprising.
I think every time you go and measure something, you discover something new and fascinating
about a system that you trusted and oftentimes you discover more questions you never thought
to ask.
This is what I love about experimentalism.

Like you get to just throw random stuff at a machine and unusual behaviors occur.
In MongoDB, for example, I knew that the defaults weren't safe.
They just wrote to a socket and did not check to see if it actually went anywhere.
They just assumed you were right with safe.
That's obviously wrong.
And you could show that without too much trouble.
But they were in the process of migrating to a stronger one where you would verify that
at least one machine had accepted it.
And I also knew that wasn't safe and was able to demonstrate that.
What I didn't realize was that even the majority level where it confirms act from the majority
of nodes in the cluster, even that isn't safe.
And I actually discovered this bug and worked with the MongoDB people to fix it where if
the network failed, it would check off the OK box and send the request back to the client
just to get it out of its hair.
It's like, all right, I don't know what to do with this thing.
Check, OK, you're done.
Send it back.
And so that was a sort of unexpected and delightful behavior.
And in RYAC, nothing was that surprising.
We knew last right, wins was bad.
The documentation was the last right, wins is bad.
But it was still the default because Bachelors has a bunch of customers who for various historical
reasons expected in one's behavior.
And having an actual blog post that gets pointed and says, look, you're going to drop 70% of
your data if you use this thing in this way was sort of a way to call it.
OK.
So maybe we can spend just, I don't know, one minute at most with each of them.
Just remember what you thought of your results and what your general impression was and we
can go through all of them.
So I have a list before me and I will of course put the list of the blog posts in the show
notes.


So let's start with Postgres.
So the Postgres finding is it's a subtle and not terrifically scary phenomenon.
It just leads to false negatives.
The reason is that Postgres sort of does this too big thing where I'm going to tell Postgres,
OK, can I transaction?
Postgres goes and checks it's OK.
And then sends back a response saying, OK, you're done.
And then we both agree that the transaction is complete.
If that final message is lost, 2BC is not a partition tolerant.
It will just stall forever.
And so the client eventually does his time out and said, OK, your transaction failed.
When in fact, the transaction couldn't succeed it.
And I just managed to show that experimentally.
You get into situations where you say, hey, I did not write the number two.
The number two failed at the transaction.
But in fact, the number two would appear in the final read because Postgres had committed
it, but the response had gotten lost.
OK.
Redis.


Redis was a complete disaster.
The sendopodical had numerous race conditions and all sorts of issues with safety.
It would enter permanent split brain.
You would get independent chunks of the cluster that evolved separately.
And I think it never actually recovered.
I think it just kept going and split brain forever.
Yeah, a mitigate disaster.
It's better now, somewhat.
And in fact, Antiras has actually built his own testing harness for doing a partition tolerance
test.
So I'm really happy with how that's progressed.
OK.
So Antiras, for those who don't know, is the author is the author of the chosen Philippo
of Redis, right?
OK.


So MongoDB, you talked a bit about it, but maybe give us some more detail about it.
Yeah.
MongoDB lost data at every consistency level.
The unacknowledged rights, of course, are not safe because you're not checking to see
if they actually went anywhere.
You're just throwing them out into the void and hoping they arrive.
If you write to a single load or two nodes, those are also not safe because if that node
crashes, it may not have replicated its data to the remaining nodes in the cluster.
And so a new node, a new leader can come up and help knowledge of those rights and then
overwrite your original rights.
You have to write with right-conserma-jority.
And that also turned out to be broken because it would claim things accessible when they
weren't.
That bug was since fixed.
I believe Tokutek has published additional analysis of the Mongo replication protocol.
I think there are actually still bugs in it, but I haven't gone back and lived personally.
One of my goals for this next round of Jepsen is to do a second attempt.
OK.


React.
React lost staggering amounts of data in last right wins mode.
This is not a new result.
The docs tell you, like, it is not safe to do this.
You can only write once in last right wins.
Last right wins essentially means any right wins when a person occurs.
However, if you use CRDTs and set an addition as a safe operation to do in a CRDT, you recover
100% of all rights and complete availability.
So it does behave exactly as you'd expect for an AP system.
You just have to make sure you use siblings.
OK.


So the next one is ZooKeeper.
That's a different kind of system, right?
Well, it wouldn't have categorized it as a database, but clearly it is a data store with
a particular purpose or typically used for a particular purpose, right?
Yeah, ZooKeeper typically is used more for configuration and state data.
It all has to fit in RAM, so it's not what you would think of as a traditional OLTP database.
But it has the advantage that if you use sync commands, it is fully linearizable.
And if you don't use sync, you get linearizable, semi-linearizable rights and possibly stale
reads.
So it's really a terrific system for doing distributed state machines.
That turned out to pass with fine colors.
Now, again, the fact that I don't find a problem is not evidence that no problems exist.
No obvious problems exist, obviously.
Yeah, it's just that it happened to pass that particular set of tests.
And I hammered on it for quite a while and wasn't able to find another problem.
OK.


You would be one I haven't heard of before.
You would be, yeah, that came out of attention because one of the founders wrote some really
questionable things about the cap theorem.
And the documentation was extremely confused.
The approach they took was essentially to block forever and just accrue rights until memory
exploded.
Obviously, that doesn't work particularly well.
It'll be fine, so long as your right volume is sufficiently low, but you also stall all
your clients.
So it's unlikely that your clients are willing to tolerate 10 milli-an-lensies or whatever.
They're probably going to give up and fail.
So it essentially becomes unavailable completely during a position as opposed to having partial
availability.


Kafka.
Kafka had a really interesting bug.
I was reading the replication design document for, I think, maybe O8.
I think it was O8 at zero.
And they had this new replication design where you had a set of replicas that were kept in
single each other, they proceeded in the mock step.
But if a replica fell behind or was unreachable, it would be ejected from the instant replica
set.
And this is all negotiated through zookeeper, which is, I think, a great choice for keeping
your cluster membership state.
The tricky bit is that they allowed the instant replica set to shrink below a majority of nodes.
So if you were careful with your partitions, it's possible to get the instant replica set
down to a single node, which will still acknowledge all of the writes that are performed.
So the single node is busy saying, okay, yep, I'm taking your data, everything's fine,
everything's fine.
But if that single node then crashes, or if it loses a zookeeper connection, the cluster
will have an election and choose one of the other nodes, which has no knowledge of the
writes that were acknowledged only in that single node.
And at that point, you've lost data.
That new node will take over and overwrite the previous ones.
So to find those kinds of scenarios, did you have to, you know, think of that once you
notice there's a single node, happily acknowledging that everything is fine.


Did you change the tool to actually look for that case and kill that one node as well?
Or how did you find it?
How did you provoke those blocks so you make them happen?
The Kafka test was the most tricky.
I was looking at the algorithm on the wiki and kind of I had this like gut sensitive
wasn't it?
But I wasn't sure exactly.
I thought a bunch of things on a whiteboard and I thought I found this pattern that would
lead to that problem.
And so I actually had a chat with, I think J. Crops actually that day and he was like,
yeah, you know, that might be plausible.
We're not sure.
We'll get back to you and talk about it.
And so I went and built the test.
And because it requires that you shrink the in syncretics set first to a particular node,
you can't just partition stuff randomly anymore.
It's not like a simple person would do it.
You had to have this sort of one to punch of getting it down to a single node or minority
nodes and then making sure those loss of connections.


So luckily, J.U.P.S.N. has like a place you can hook in arbitrary failure scheduler.
And I was able to write one that caused that particular pattern.
And indeed, it did lose tons of data.
So I think I was, if I recall correctly, that got changed.
There's now a configuration switch, which you can flip that controls whether or not elections
possible in that mode.
Okay.
So the interesting thing that Kafka, not even the Kafka people claim in any way that
it's a database, right?
Still, you can show the problems occurring in anything that stores data in some kind
that is distributed across multiple nodes, right?
Anything will exhibit those kinds of problems.
Yeah.
It's still called Kafka database.
It's a log.
It's maybe not what you think of as a traditional database, but it still was a data store, for
sure.


What about Cassandra?
Cassandra had just introduced a new transaction system called Byway Transactions.
When I was able to show they would deadlock hard, which was a dead to life result.
And when the deadlock was fixed, it also lost data.
So that found two super bugs in their transaction system, plus a whole bunch of interesting
isolation problems and terminology issues in the docs.


Okay.
What about RabbitMQ?
Now that's not a database that I have to insist.
Yeah, it's a queue.
Much like Kafka, you know, I still kind of think of those databases.
They're just different shapes.
Here's a blog post which suggested building a semaphore mutex on top of RabbitMQ.
And I thought, well, that can't be right.
And of course, of all the data, and then we talked about this back channel.
He's like, yeah, this is obviously not right either.
I wrote the thing.
We'll quietly change that.
But it was still a good example of here's something some people do, you know, trying
to shape one database into another.
How can we demonstrate with amount of city properties?
Hold true.


Okay.
Just pick, well, let's go with the last few as well.
So we have ad city and console.
ad city and console are both very similar systems.
Zookeeper, they're configuration stores.
And then for small in-memory data sets that are supposed to be linearizable.
And both of them made the same interesting choice.
So they're based on the raft protocol, which is so far analysis looks to be solid.
The difficulty is that they're consistent updates and their consistent reads.
Oh, yeah, sorry, it's consistent reads did not check to make sure that the leader they
were talking to was still the leader by the end of the read operation.
And so a cluster transition could occur and you could wind up reading stale data.


And a final one on the list is Elasticsearch.
Elasticsearch was a complete disaster.
It lost data from network positions that were intersecting from network positions that were
not intersecting from single nodes being isolated, even a single node reboot could cause data
loss.
It was a spectacular heavily.
Sounds like you had fun breaking down.
Yeah, but I think the results would be good.
A number of these database companies using Jepsen internally to help instrument the software.
And a lot of them have sort of turned around.
They're documenting these failure modes.
Elasticsearch in particular now has a phenomenal status page, which talks about all the sort
of data loss issues and their priorities and links throughout the tickets.
I'm really happy with that.
So that is one thing that I found utterly fascinating.
It seems that you broke stuff and then told people their systems are just bad and completely
broken and shouldn't be built that way in the first place.
But nobody really kept cursing at you that badly.
Obviously, people took a look at the results and found them to be valid and actually used
to their own advantage.
Was there a lot of anger at you?
Some people are mad.
Some people are not.
And it has to do with how politely I approached the database and how closely I put them in
tangers.
Again, this is a social problem.
Humans are humans.
The results by and large have been constructive.
This has been the goal to help users figure out what systems are safe and how to analyze
their own systems, also to help vendors improve their safety.
But I'm one person before I was doing this in my free time on nights and weekends.
And these analyses take hundreds of hours per database.
And you factor all the time right in the tooling and researching papers and writing
out the results, it's a lot of work.
So I can't beat a QA department for the internet, right?
Ultimately, I had to inspire the companies to care themselves.
And then it becomes a competitive advantage for them because they can claim, hey, we're
actually safe according to these metrics.
So I think you mentioned that before, there are a lot of papers that people can read.
They may not be the most interesting nighttime reading, but there is a lot of research out
there for that stuff.
So do you feel that some people didn't do their research properly before building a
distributed consistency resolution protocol or implementation?
Keep in mind that everybody who is building a database is under tremendous pressure.
Financially, in terms of time, in terms of staffing, there's a ton of things that have
to happen and it's an incredibly difficult job.
So just because people didn't read literature or weren't aware of it or didn't design something
correctly, it's not that they're bad engineers per se.
It's just that these problems are sort of poorly understood and there's not a culture
to support their understanding yet.
One of the things I want to do with Jepsen is change engineering culture to be a little
bit more aware of failure modes and how to reason them up.
So it depends on which sort of company you talk to, what their engineering culture is
like.
Basho places a very high value in papers and it shows in their engineering, while I do
have data, allows the corruption bugs from time to time, I think their general architecture
has proven to be solid.


That company like MongoDB, they made some obviously terrible choices as far as safety
goes, but they also made those choices because they knew they had to get this database out
door quickly to build traction in the market.
So in some cases, it's a deliberate trade off.
We're going to do less safe thing.
People won't notice until later and by that time they can fix it.
Which is a valid business strategy, I suppose.
My principle concerns that people document and explain and communicate their uncertainty
or their possible failure modes to users.
I don't need people to be 100% safe, I just want them to be honest.
Right.


So that is something that I find very interesting.
I mean, the different trade offs, sometimes there are even valid trade offs, right?
As you mentioned, if it's documented and if it's very openly communicated that there
are certain restrictions that you have to be aware of, you obviously have to address
them in your application layer, right?
You have to do something to work with that database.
It might be okay to lose some data.
Maybe if you're using a search engine, maybe it doesn't matter that much if something that
you're right to it is lost, right?
Depending on what kind of stuff it is that you're searching for or that you built this
index for, right?
So what are some other strategies that you can use in your own application or let me
phrase it differently?
What's a good strategy?
I mean, for developers, if they look at all those things, are they supposed to pick the
one that got the best results in your testing with Japsin or what are the kinds of trade
offs that people have to make and how do they address them once they've made them with
the data store in their application layer?
You can make two answers out of that if that was too much of a question to ask.
I really want to avoid making specific endorsements because not every database is suitable for
all problem classes, right?


Typically, you're going to choose several databases that your particular needs as far
as availability performance and distribution go.
And also, just because they didn't find problems doesn't mean there are problems, and you
also have to consider other factors like ergonomics, availability of tooling, availability of
expertise, can you hire people to run the database?
Can you afford the licensing piece?
So there's a lot of things you balance in addition to safety.
Just because something is theoretically unsafe does not mean you'll be bitten by it.
So I've been at several companies now with Elasticsearch and prod, and every one of them
has been a tire fire.
I would not recommend Elasticsearch at all if you care about having your database cluster
or having your search system actually available and trying to request a data in it.
However, it doesn't sound like an endorsement.


Oh yeah, it's definitely not.
However, it has a phenomenal API.
It is probably the most popular of the available tools.
There's plenty of library support.
People continue to choose it because search systems don't actually have to have all your
data.
Now I've seen it do some interesting things like lose a third to an eighth of the data
set from a node reboot, and that stage starts to be really problematic for us.
But in certain cases, that's still okay.


And if you're refilling it, it's fine.
So this is one of the things where I want people to just think really carefully about
what kind of safety they really need.
And then you choose the database that will satisfy that level of safety.
In certain cases, you can kind of make a more loose cultural decision.
Like Zucheeper, SCD, and Console, Zucheeper's API is complicated.
The clients are not easy to understand.
It has a very complicated connection model, and I mess it up all the time.
But Zucheeper and Console, we can see the are excellent tools for just, you know, for
the purpose that they're intended for, I would assume they are, right, for the configuration
use case.


It's a different one than the string.
Yeah, it's like one of those three, right?
Even though SCD has, I think, a very nice, simple HTTP API, it has only been around for
a couple of years.
And so it has not yet hit all of the various interesting production failure modes that
Zucheeper has, and it's going to take time to stabilize.
So I think people are finding out with Console and SCD, like, oh, there's a highly surprising
behavior that developers haven't seen yet because it's like, oh, it only happens once
you're at this request volume, this cluster size, this sort of latency pattern.
And then you get this weird explosion of memory and then GCPOS or whatever.
And Zucheeper, you know, is also seen as things.


So maturity is, I think, you know, even though you can be based on a solid theoretical understanding,
it takes a while to iron out the case.
So maybe let's talk a bit about how you actually built Japsense.
So because one of the things that I found interesting was that your choice of closure
for it, was that just coincidence because you happened to be familiar with that language
or was it a conscious choice to use it for that purpose?


I wrote the first proxy drilling, actually, and it didn't go very well.
I wound up using a Ruby deployment tool called Soltysid that I wrote to do the network automation,
like killing nodes and causing network failures.
And then I wrote the analysis system enclosure because it has really nice facilities for
working with data and because pretty much every database has a JVM client somewhere.
So it access all the regular Java clients, their decent quality that Riezid interrupt with.
And the performance and closure is acceptably good.
So it was an actual choice for those reasons.
OK, so there was no particular strength of closure that you could exploit, except for
its general purpose, you know, usefulness.
Yeah, it's a nice general purpose, back-end language.
And it's especially good with databases because while it's certainly not as fast as Java,
it's certainly a lot more concise, which makes it easier to sort of reason about the
correctness of your code and to write complex data analyses to the queue lines.
The newer versions of Jepsen actually are written in closure for good reason.
They have certain macros and DSLs and so forth that take much more advantage of closure's
concurrency primitives of its syntactic transformations.
Those things would be a little trickier to port a different language, I think.
OK, do you know of any comparable work?
Are there other tools like Jepsen, rather by vendors or by other researchers?
It's very much a system I built because I needed it, but it only addresses a very particular
subproblem.
There are quite a few academic network modeling tools, but most of them are aimed at you
write your algorithm inside their language and it checks the any of it.
The spin and promo are both algorithm checking languages.


You can write something in C or a formal language like TLA+, and it will check the model to
make sure that it satisfies whatever variance you describe.
You can run your entire network in a simulated system that does accurate message passing
and stuff.
The problem I have is that, of course, I don't control any of these databases.
They're just programs that run on Linux or whatever.
I had to have this black box system where I could induce the failures externally and
insert the system externally and hopefully get some useful results out of it.
Jepsen has a sort of funny shape to account for that problem.
I'm not sure.


Antirez, so it has written his own partition testing tool for Redis.
There are failure tools for measuring Cassandra that are internal.
Rheoc has extensive stuff written in QuickCheck, which is a generative testing tool for Erlang
and Haskell and so forth.
I think most vendors have these sorts of things.
Actually FoundationDB is quite impressive.
They have not only extensive software simulation of their message passing system to verify
its safety, but also they have dedicated hardware that can have induced failures.
They can actually have a robot flip a switch on the switch or whatever and turn it off
and see what happens.
Wow.
Okay.


So can you elaborate a bit on how Jepsen works internally?
Yeah.
So Jepsen has gone through a few iterations.
I mentioned the sort of set addition problem earlier where you add numbers to a set in
the database and read it back.
But I've wanted to test more strong CZ classes like linearizability.
And for that, I can't just do unordered operations like a set.
I need to have more complex data types like a lock or integer with compare and set operations
or maybe even an SQL table.
I'd like to be able to model these complicated things and verify their correctness.
So I've written a tool called NASOs, which does model checking against a history.
It will evaluate with a model like a register or a set or a queue whether that model is
consistent with the operations we observe from a database.
And then Jepsen is responsible for building a bunch of random operations like set the
registers number to two if and only if it's currently three or add the number three to
the queue, the queue something.


And then it has clients which will apply those operations to the system.
So the combination of the randomly generated operations, the history of how those operations
played out in the system, what the different clients actually saw that it was due.
And then the model checker verifying that that history is consistent with what we think
the system should do.
Those three give you a measure of safety.
It reminds me a bit of the approach taken by things like QuickCheck.
It's the same.
Exactly.
Yeah, it's very much inspired by QuickCheck where you have a model that describes what
the system should do in simple concrete terms.
Okay, if I add a number to register, the new number should be, the new value of the register
should be that number plus the yield value.
Or if I add them with the set, it should be one bigger in it.
If I do a read, I should see that number.
You write down simple laws about the system and then generate a bunch of random instructions,
execute them and see what happens.
I'd go back to some theory if that's okay with you.
Sure.
There has been a whole lot of research into algorithms for resolving this, for finding
a strategy to resolve the consistency problems that arise.


Are there some basic ones that people should know that you would like to point people to?
The goal standard is Paxos.
No, if you don't know what you need, choose linear accessibility.
It's going to be the safest of all your options.
Okay, can you explain what Paxos does?
How does the algorithm work?
If that's explainable in a podcast?
You know, honestly, Paxos is a subtle and dark magic, which I am not qualified to explain
verbally and do whiteboard.
Okay.


Even then, it's a little dodgy for me.
The end result is that Paxos gives you a system of participating nodes in a cluster which can
come to agreements on certain decisions.
In practice, you can translate that consensus algorithm into other things like a replicated
state machine or a log.
There's another protocol which is much simpler called Raft, which is as far as we know, equivalent
to Paxos.
There's also Zab, which is the zookeeper atomic broadcast paper.
And there's Zab as actually implemented inside of zookeeper, which are some of the different
things.


All of those also offer linear accessibility.
So as a lowly application developer, I don't actually have to implement them because hopefully
I have some, I use some tool that has implemented those things correctly.
Do I have to know how they work?
Or do I have to know the consequences of using them?
I mean, I have to have some, you know, some insight into what it is that I'm using there,
right?
I do.


You don't need to understand how they work.
All you need to know is what classes of history do they allow?
In a linearizable system, you're guaranteed once your operation completes, it's going
to be visible.
Every operation will appear to take place between the time that you send the message,
starting it, and when you get back the result saying, hey, it's done.
Now there are other algorithms.
If you have sequential consistency, you no longer get that real time guaranteed.
Your message could come back saying, hey, it's done, but it can logically take effect
later.
Or in the case of recommitted, you can get certain read anomalies for, okay, I recommitted data,
but it might be possible for you to still see a phantom for a predicate not to be consistent
with the current state.
One acronym you mentioned a number of times was CRDT.
Can you give us a brief introduction into that concept?
What does it stand for?
What does it mean?
So CRDT is a convergent replicated data type.
We always use data types, right, like sets or strings or arrays.
A replicated data type is one that can exist in multiple places.
So you can have a copy on one, you can get rid of a copy on another.
And if it's convergent, we want to guarantee that if those copies communicate, they will
come together and to some final value that we can agree on.
So this is really useful for AP systems where you might update a counter on two different
nodes that don't communicate with each other.
And then later they exchange information and come into a new counter value, which should
include both counter increments.
So taking that example, how would that be implemented?
What does it do?
Do you use some version cracking or timestamps or is that the internal implementation of
that particular CRDT then?
The really nice thing about CRDT is that if you follow these simple rules, you basically
get safe convergence for free.
Everything else becomes a performance optimization.
So the rules you have to do to be a CRDT are to be associative, commutative, and idempotent.
associativity means that as you're able to change the parentheses around the function
application, F of A and B, and then applying that to F of that C, that should be equivalent
to switching the nesting order.
Commutativity means that it's argument order independent.
So F of A and B is equal to F of B and A. So those two things mean that I can apply the
operations in any order on any replica to get the same results.
The last one, idempotent, means that if I apply the same operation multiple times, it only
takes effect once.
And that's critical because it means I don't have to only send a message to my peers once.
I could keep sending them my update over and over and over again, and it will have the
same safety properties.
So the convergence strategy for CRDT, once you follow those rules for your merge function,
as you just view comp as yourself at everybody else, and they spew copies back at you, and
you just call the merge function to combine them, and you're guaranteed to come up with
the same answer.
So are there then standard implementations of CRDTs of various kinds in standard libraries
that people can use, or is that just something for database vendors to make use of?
I think it's most efficient if the database vendor does it.
So there are CRDTs built into React, and there's something kind of like CRDTs in Cassandra.
I think some of the SQL data types aren't some art.
Things like sets in, I think, Redis cluster might end up looking somewhat like CRDTs at
some point.
And you can certainly implement your own on top of the database.
So in React, all data types should be CRDTs, and once you're using the strong consistency
buckets.
In Cassandra, all your data types should be CRDTs.
And you can implement that logic just by having your own merge function in your code, so you
read a value, merge in the right back.
There are some common ones.
The in-re-up papers describing CRDTs, and also, what's the second paper?
Anyway, if you Google for CRDT papers, Chris Mitchell-John has a bunch of these, and they
describe a few common implementations.
You can build last-right wins registers.
You can build sets that grow only, sets you can add to, and then remove from once, sets
where you can observe anything you've already removed.
You can build lists that you have in ordering relation.
You can get maps of certain types of maps.
It's a little bit tricky, because in order for them to be converted, you have to give
up with certain properties about the disabilities.
Your counters, for example, maybe all of your inserts to a counter are going to be safe
and eventually seen, but you don't know when they'll be seen.
Maybe the space guarantees are different.
In a counter, you have to have a slot for each and every actor in the system for a typical
role in the counter.
For an observed remove set, you have to carry with you a tag that identifies all the inserts
and all the deletions.
The space complexity requirements are typically worse than a linearizable system would be.
We've covered a lot of ground.
We've talked a lot about theory, and we've talked about a lot of actual systems.
Maybe first of all, is it a reasonable approach for a software engineer to just pick one solution
and stick with it, or maybe two if they have different needs?
Can they use your approach as a guideline as to assess the qualities of the systems?
I think the posts will be useful in figuring out how to think about a system.
I don't think you can use them as, oh, by this database.
Ultimately, I try to stress this in each post and in every talk, it's up to you to measure
your system's correctness and to figure out if the invariance you need are provided by
the database.
You can read a Jepsen post to figure out, okay, can I get the system to be linearizable
or not?
But those may be out of date, right?
I can't keep updating things all the time, and you might discover that there are certain
requirements that I just haven't talked about, which you need for your database.
So ultimately, I want to give people the tooling in Jepsen and push the vendors to provide
tooling as well so that you can evaluate these for yourself.
I think in more general terms, as a software engineer, you're going to need to combine
multiple data stores.
So you might have maybe Kafka as a log store and Cassandra for immutable AP storage and
be re-out for a CRT store and maybe Zookeeper for a linearizable store.
And you might be shuffling operations into and out of these different data stores.
We did postgres, maybe this was sharded postgres in there for strong serializable operations
over subsets of the data set.
I think you'll find that your use cases, once you start getting up to scale, require the
EU's different data types and different data stores for different data volumes.
Typically, you'll have really high volume stuff that can't fit in Zookeeper or Postgres.
And so you have to start sharding it yourself or you have to start using an AP data store
like Cassandra for your high throughput.
I can just.
Okay, so what are some good resources, some places people could start if they want to
familiarize themselves with this topic more?
I think CMEIK, he's got a great reading list for distributed systems engineers.
And that's where I would go for theory.
I've written a blog post on strong consistency models and Peter Bayless has written much
better formal work on topic consistency.
So those are good resources if you're interested in more theory.
If you're interested in a short overview of Jepsen, I think the ACMQ article on network
addition frequency by the EU if you use because it tells you, here's a bunch of different
failure modes that we saw in production systems.
It's an overview of the different host more numbers that we've seen about network addition
failures.
And the Jepsen posts themselves show you exemplars of here's how a system can fail.
But I think it's best to look at those case reports and not as sort of generalizable
results.
Okay, so thank you very much for this, for all this information.
My head will explode in very few seconds.
So thanks a lot for your time and happy to have had you in the show.
Thank you very much for having me.



For software engineering radio, this has been Stefan Tilghal.
Thank you for listening.
Thanks for listening to SE Radio, an educational program brought to you by IEEE Software Magazine.
For more information about the podcast, including other episodes, visit our website at se-radio.net.
To provide feedback, you can write comments on each episode on the website or right of
view on iTunes.
Mention our messages on Twitter at SE Radio or search for the Software Engineering Radio
group on LinkedIn, Google+, or Facebook.
You can also email us at team at se-radio.net.
This and all other episodes of S.C. Radio is launched in some of the Creative Commons
2.5 license.
Thanks again for your support.
