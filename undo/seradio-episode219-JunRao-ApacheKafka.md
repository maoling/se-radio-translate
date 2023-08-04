**Link** https://www.se-radio.net/2015/02/episode-219-apache-kafka-with-jun-rao/


This is Software Engineering Radio, the podcast for professional developers on the web at
sce-radio.net.
SC Radio brings you relevant and detailed discussions of software engineering topics
at least once a month.
SC Radio is brought to you by IEEE Software Magazine, online at computer.org slash software.
Welcome to Software Engineering Radio.
My guest is June Rao.
He's a software engineer and researcher formerly of LinkedIn who is embarking on starting a
company that is largely based on the Apache Kafka project.
He's a researcher and developer who has spent much of his time researching MapReduce, scalable
databases, query processing and other facets of the data warehouse.
And for the past three years, he's been a committer to the Apache Kafka project which will
be a focus of our discussion.
So June Rao, welcome to Software Engineering Radio.
JUN Rao Thanks, Jeff.
Thanks for inviting me.
SC Radio So to get started, let's talk about a couple definitions.
So how would you define streaming and how would you define messaging?
JUN Rao Yeah, that's a good question.
So streaming, based on sort of my view of it, historically has been mostly referring to
systems or frameworks that provides the capability of processing one or more sequence of infinite
events.
So those frameworks typically provide things like the ability of plug-in customized logic
for processing those events.
You can do things like filtering, window-based aggregation or stream-based joins.
That framework typically also provides things like parallelism.
It will parallelize the events for you.
You don't have to worry too much about it.
And those frameworks typically also cover things like fault tolerance.
So if one of the processing unit is down, you don't have to worry about it because the
framework can restart the process in some other machines for you.
So this is my view of what this stream processing system refers to.
So based on that definition, things like Apache storm and Apache spark.
And there's another system that has been built at linking called Apache SAMZA, forced
into this category.
These are the system that provides a framework for processing sort of sequence of infinite
events.
Now one of the things those systems have to worry about is where do they get the data?
That's where the messaging system comes in.
So one of the key ways for those stream processing system to get data from is from a messaging
system or a pop system.
So if you look at Apache storm, the most popular way for you to get data is from Apache Kafka,
which is messaging system.
And there's integration with Kafka in other streaming systems like spark and SAMZA as
well.
So that's sort of the I view as the difference or the relationship between streaming and the
messaging system.
So another sort of analogy, if you want to map that to the Hadoop world, the Hadoop
part has two systems, right, one is the MapReduce part, which is really the processing framework
of those offline data.
And then there's HDFS, which is the storage and delivery mechanism of those data.
And these two systems are coupled together in the loose way to provide this offline computation
framework.
And you can view sort of this streaming processing system as the equivalent of MapReduce, but
more in the real time world.
And the messaging system like Kafka are sort of equivalent out of the counterpart of HDFS.
But again, it's for the real time world.
So that's interesting that you kind of describe it as the Kafka and storm projects is sort
of being an unbundling of things that are frequently found together or in other domains
found together.
So, and I guess let's talk a little bit more about specifically what Apache Kafka is.
The summary that I found is a published, subscribed messaging system, rethought as a distributed
commit log.
So what do you think of this definition?
And why is it useful to draw an analogy between messaging and a distributed commit log?
Yeah.
So yeah, it is true, I think Kafka at the high level of the definition is really sort
of messaging or pop-sub system.
So the line between messaging system and pop-sub system is a little blurred these days.
Traditionally, messaging system has been mostly referring to systems where you sort of do
this point-to-point message delivery.
So messages published from one place and it's delivered to another one.
And if you look at the pop-sub word there, I think the analogy is the difference is messages
can be consumed multiple times, potentially by different consumers.
So, but these days, I think the line is blurred because a lot of systems can support both.
Now, why is the, why do we draw an analogy between a messaging system and a commit log?
So there, I think there are two things.
One is at the very high level, both messaging systems and a commit log are dealing with
sort of a sequence of a pandemic data.
So the access tab pattern is pretty simple.
To write the data into those systems, you just keep appending new data into your system.
There's typical or no updates.
You just keep adding new things.
The way you read it is also pretty simple.
Typically just read those data sequentially.
Now, because of such simplicity, it comes a lot of benefit.
Basically, you can implement this system in a very efficient way in terms of scalability,
reliability and performance.
So that's the first sort of analogy.
The second thing is, so I came from the database background.
So if you look inside a database system, right, most of the database has a commit log.
Now, that commit log is actually the source of truth of all the data you store in a database.
Even though you may not be kept forever, but that's actually where the initial data comes
in.
And from that, you can derive all the other data you have in the database systems, like
data you store in a table, data you store in the index.
So this is also true for messaging system.
If you think about all the consumers or the downstream applications that get data from
a messaging system, messaging system in some sense serves as the source of truth for all
those downstream systems.
So that's sort of the second analogy between a messaging system and a commit log.
Okay, and so how does Kafka compare to more traditional messaging systems like ActiveMQ
or RabbitMQ?
Yeah, that's also a good question.
That's sort of a question that we got a lot.
I would summarize, there are two key differences.
One, these are sort of the differences where Kafka has a little bit of advantage.
The one is Kafka is really designed for high volume data.
If you look at traditional messaging systems, those systems typically are only responsible
for storing data that's generated in the database.
But if you look at Kafka, it's really designed for systems, it's really designed for storing
data such as business metrics, service logs, operational metrics.
And those data, in terms of volume, is easily a hundred times or a thousand times bigger
than the data you store in the database.
And it's not something traditional messaging systems are designed for.
And Kafka is really designed for those.
For example, Kafka is designed as a distributed system from ground up.
So if the volume of data increases, you can just easily add more machines into the cluster
to handle that volume of data.
And there are quite a few optimizations in various components where we do batching and
compression so that we can handle this volume of data more efficiently.
Okay, so that makes a lot of sense.
So kind of looking at the size of the pipe from the ground up that you need and saying
we need something that's just way bigger than something like ActiveMQ or RabbitMQ.
That's right.
So you can think of it as a big data version of the messaging systems.
So that's the first difference.
The second difference, which is also important, is traditional messaging systems are really
designed for, and only for real-time use cases.
But for the Kafka use cases, we not only have the more real-time use cases, but an important
consumer or downstream application of Kafka is to feed data into those offline pipeline.
So one of the most important one of course is Hadoop.
A lot of people are using Kafka to get data into the Hadoop system.
So in that case, what we have is a system that you need to handle both the real-time
use case as well as the non-real-time or the batch or inter-use cases.
A lot of traditional messaging systems, because they really optimize for the real-time use
case, they do things like buffering all the messages or unconscioner messages in memory.
So now imagine if you have a downstream consumer like Hadoop that's under maintenance for a
couple of hours, then you may not be able to buffer all those data in your message broker
for that long.
So would you?
Sorry, go ahead.
And because of that, for a lot of the offline consumer use cases, a lot of traditional messaging
systems, the performance force way short of Kafka.
So would you describe Kafka as a data warehouse messaging system?
Or is that too narrow of a definition?
I think it is a warehouse of data.
You can think of it as warehouse data, but it's really a place where you can collect all
more types of data and you can feed that into different platforms.
One important type is to feed that into the warehouse or the offline system, but you can
equally use the same system to feed data into a more real-time platform.
So it's sort of the integration point for both the offline and more real-time consumption.
Okay, that's great.
So now that we have sort of a framework of an idea for our listeners for what Kafka
is, let's talk a bit about the basic vocabulary of Kafka and kind of drill down into how it
works.
So maybe you could talk about how some of the central components work.
So could you maybe talk a bit about topics and producers and consumers and brokers?
Yeah.
So at the very high level, Kafka is very similar to traditional messaging systems.
It has this three-tier concept.
In the middle, we have this broker layer.
It is actually where messages are stored and served.
And of course, we have the producer tier.
These actually are the publishers of the messages.
And we have the consumers.
These are the systems that make subscription one or more of those feed of data and the
data just get delivered to those consumers.
So these are at a very high level, so there's three layers of entities around the Kafka.
Topics are essentially the unit of, the logical unit of defining individual feeds.
You can think of each topic corresponds to like a virtual queue.
And when you publish messages, of course, you publish a message to a topic.
So then your data is added into one of those virtual queues.
And when you consume a message, consume data, you typically subscribe to one or more of
those topics.
So then all those data stored in those corresponding virtual queue or virtual channel will get fed
into your system.
There is also this low level concept called partitions, which is associated with topic.
Now each topic physically can have one or more partitions.
And the reason we have this partition is really for parallelism.
You can think of a virtual channel physically dividing into multiple physical queues or
channels.
And those sub-cues or sub-channels can be stored independently on different machines.
And that's a way of getting more parallelism out of our system.
And when you say parallelism, do you also mean redundancy or fault tolerance?
Or is it mostly just to speed things up?
I think it mostly has to do with speeding things up.
So logically, different partitions can be sort of consumed independently.
And whether you replicate those data or not.
So the more partitions you have in terms of both the producer and the client, the more
parallel channels you can have.
So that is a way of increasing the degree of parallelism.
Now each of the partitions can be further replicated.
And that is a way that we can use to provide a reliability of the system.
But it's not related to what we have in terms of parallelism.
So messaging typically has two models, at least there are two that you can discuss,
queuing and pubsub, publish, subscribe.
And with queuing, you have a pool of consumers that read from a server.
And each message goes to only one consumer.
So in queuing, maybe you have a list of tasks that the server needs to accomplish just once.
And it just has consumers that are receiving each of those tasks.
And then in pubsub, the message is broadcast to all consumers.
So Kafka has a single consumer abstraction called a consumer group that generalizes both
of these.
Could you talk a bit about consumer groups and how and why they were conceived?
Yeah, certainly.
I think consumer group, yeah, it's pretty interesting.
So as you said, it actually covers both use patterns.
In one case, what you can have is you can have just a single consumer group.
And in that group, you can have multiple consumers.
And what they do is they sort of jointly consume one or more of the topics.
So for example, so you have like two consumers in the same consumer group, then a topics
data will be hopefully evenly divided half and half between these two consumers.
But that's sort of a way of increase the degree of parallelism in the consumer in a lot of
use cases where the logic in the consumer can be a little bit CPU intensive or time consuming.
Of course, if you have more degree of parallelism, then you can use more CPU power and other
resources to speed up the processing of the data.
So that's one use case.
Another use case, of course, is you can have multiple consumer groups on the same topic.
And this is sort of the multi subscription model.
And in that case, each consumer group will get a complete feed of all the data in a topic.
So this is actually intended for there are just lots of applications.
They want to consume the same data, but they just want to process them in a slightly different
way or for different applications.
And this multi subscription model is a very powerful way for those applications to sort
of consume little data independent of each other.
And you can actually have even for multiple consumer groups within each consumer group,
you can still have those multiple consumers.
So you sort of can combine the benefit of the multi subscription and the degree of parallelism
within each instance of the consumption.
So that's how we sort of combine both concept into this consumer group.
And what other facets about the relationship between producers and consumers are unique
to Kafka?
Yeah, in terms of, yeah, I think that actually one good question.
I think one of the things if we compare Kafka with traditional messaging systems, one of
the things we don't have is some of the more traditional messaging APIs such as like the
GMS.
GMS has like a rich set of features in terms of like all the delivery priority is kind
of stuff.
That we don't have in Kafka.
As I said earlier, Kafka is really designed for high volume of data and for dealing data
in the efficient way.
So we don't have a lot of features that's existing in traditional messaging systems.
And so does this lead to some trade-offs where there are sort of like explicit use cases
where Kafka just doesn't really make sense as your messaging system of choice?
Right.
So it depends on your application.
On the one hand, Kafka does support some basic features that a lot of applications want to
have such as the guarantee of delivery.
There's like a producer-side API allows you to configure when you can get an acknowledgement
when you publish a message.
And ways of acknowledging gives you different guarantees on durability.
That actually a lot of the users are using for the applications.
Now what we don't have is some other things like the priorities, order to deliver like
various out of order delivery.
So if you have requirements of that, then either you have to implement in your application
or you just have to use some other messaging systems that gives you this capabilities.
So talking more broadly or historically, there have been a lot of platforms over the
last few years.
We've mentioned some of them so far.
So what do you think are the factors that are driving the emergence of this important
technology and sort of like where you're kind of at the forefront of developing this kind
of stuff.
So what do you see changing in the future and how do you see what evolution of this domain
do you see?
Yeah, that's also an interesting question.
So I see there are two important factors that have changed.
One is sort of the volume of the type of data that people are collecting.
Historically, most of the data people that have interest are just database data.
But in the last 10 or 15 years, people start collecting.
So other those what we call big data.
So what are those data?
These are data such as business metrics.
I like the page views, impressions, clicks, search keywords people type in.
This also includes operational metrics or the GM max, IO stats, CPU stats.
This also includes all the service logs like log for J logs, this logs or maybe some of
service calls.
Now in terms of volume of data, these data is easily two to three orders magnitude bigger
than the data stored in traditional database systems.
In terms of value, these data provide a lot of insights that's actually valuable for lots
of downstream applications.
So that's trend number one.
The second thing which is also related is which is the question why people are only starting
to collect this data more recently instead of like 15 or 20 years ago.
This actually has to do with another trend which is the emergence of a lot of scalable
specialized vertical systems.
If you like to do it, it's a specialized scalable system for doing offline processing.
Traditional a lot of those things you can do this kind of stuff is in Oracle or Teradata
which is expensive.
But what can do it is very special.
They basically take just take one aspect of it which is offline processing of the system
and made it more scalable on commodity hardware.
As a result, they can provide an offline processing solution in a more economical way.
And because of that, people actually cannot afford to store this large volume of data
if they want to do offline processing in Hadoop.
They don't have to buy those expensive Oracle or Teradata instances.
So this kind of trend is not only happening in the offline world, but it's actually starting
to happen in many other vertical specialized domains.
So if you like how to search, there is a Alexa search that's doing the sort of scalable version
on commodity hardware in the search domain.
And if you look at key value stores, there are lots of system data provides capability
of storing and retrieving like point data in a scalable and economical way on commodity
hardware.
And if you look at there are lots of graph engines, those are systems that can do graph
processing.
But again, sort of commodity hardware in an economical way.
There are lots of like a streaming system that also is redesigned to be designed on commodity
hardware that you can scale in a more economical way.
And do you think that the big cost of storing this kind of data is sort of trending towards
the fact that you have to maintain it as opposed to whether or not you can afford to store
it?
Yeah.
I think both aspects are important.
The first of course is upfront cost, right?
How much does it cost for you to acquire the software and the corresponding hardware?
That of course, if your software is like free like open source and your hardware can run
or depend on commodity hardware, then it's going to be a lot cheaper than you have to
pay a premium on proprietary software and you have to pay a high premium on expensive
storage boxes and just commodity boxes with large local set of drives.
That's the first aspect of it.
The second thing of course, once you have your system in place, you have to operate that,
right?
So there are of course, the more instances or the larger the class you have, potentially
the more overhead of monitoring or operationalizing it.
So there, I think that a lot of tooling would definitely help.
If you have a lot of tooling and monitoring in place, then it doesn't really matter whether
you are monitoring a class of 10 nodes or a class of like 100 nodes because a lot of
things can just be integrated together with the right alert and monitoring set in place.
So that we're certainly going to help to drive down the operational cost of those systems.
Okay.
So let's get back to talking a little more about Kafka specifically.
So maybe you could describe a, you know, I know you've touched on some more abstract
examples of Kafka.
Maybe you could talk about a more specific use case when somebody would want to deploy
Kafka and sort of describe how they would go about deploying it and connecting the different
components.
Right.
So at a high level, this sort of goes back to the changes of industry trend I mentioned
a bit earlier.
So because of those existence or the emergence of those very specialized scalable systems,
so a lot of those systems, if you look at those, they need to be fed with the same type of
data.
So for example, if you have a collection of log data, right, of course you want to feed
that data into a doop for offline processing, but equally important, you probably want to
feed that data into your search system so that you can search any of the log occurrence
quickly.
That's a lot of the business, what Splunk does, right?
And equally, if you have, for example, stream or like operational data, of course you want
to feed that into offline system, but you probably want to feed that into some of your
real time monitoring system so that you can monitor and graph those data.
So the question is where do all those independent system get their data from?
So you sort of need like an integration point where all those systems can get its data from.
Now a lot of those systems are actually unlike a doop, they are actually more real time.
So you can't really just integrate that in your warehouse or offline system because in
terms of latency, these systems may not be good enough to provide data feed to some of
the more real time applications or frameworks such as real time search and graphing.
So Kafka sort of fuels in that role because it's a system actually designed for collecting
and storing high volume of data, it actually can provide feed to any number of downstream
applications and those applications can be sort of more real time, but those systems
can also be offline.
So you can sort of it as an integration hub for all this big data.
Now in terms of Kafka adoption, what we have seen of course is people are picking up sort
of various pieces.
One common use case of course is people are using Kafka as this ingestion pipeline to
get data into Hadoop, but at the same time they also have application, one or more of
the application for real time use case, be it a specialized real time application or
some of the streaming processing framework they are adopting or some of the real time
search system they are adopting.
So because of those need of more than one application of the same data, so Kafka is
very important for them to get those various pipelines working.
So that's one of the common pattern we see Kafka gets adopted.
Has there been any implementations you've seen that have really surprised you and maybe
ended up maybe even changing the direction of the project or sort of introduced a reason
for adding some new features you wouldn't have predicted?
Well I think so far I think there are lots of features that we know are required as Kafka
gets more and more adopted.
It's like security which is important and in that domain we don't see too many surprises
in terms like future requirements.
We are sort of surprised occasionally just the way people are using system.
There are a lot of things that we sort of didn't plan to have people use it in that
way.
But you know people use it in any way.
Sometimes it's not really a idea use case and sometimes it's just we surprise that Kafka
actually works when people use it that way.
For example one of the things we are surprised at is that there are at least one use case
that are using Kafka as a message delivering mechanism and the way they send messages each
message is sort of gigantic those are like each message can be a gigabyte.
Of course you can configure Kafka to do things like that but it's certainly not something
that we really designed it for.
But we are just surprised that people are using it and then sort of worked in this limited
context.
There are other use cases like people are sort of trying to use Kafka more like a key
battle store where they just sort of randomly consume messages from different places.
That also sort of can work but it's not really something that Kafka is really designed for.
So that I would say architecture wise if you have a use case like that probably a key
battle store is more appropriate for those use cases.
Yeah that's really surprising.
That sounds like the type of thing where you have somebody with a hammer and everything
looks like an ale.
That's right.
So in the 2011 paper about Kafka there is a distinction that is made between application
data and log data.
And so this distinction has become less clear over time.
Do you think Kafka is still viewed as a log record messaging tool or is it less acceptable
now compared to three years ago if some of the log records are dropped?
Yeah so one of the things we have seen of course you know the way Kafka get adopted typically
starts from people putting in log data.
And that's actually what Kafka is really designed for and it's a powerful way to store and serve
this kind of large volume of data.
And arguably maybe for log data you can argue maybe it's okay to lose a few messages from
time to time.
But over time of course what people want to have is once you have a pipeline that can
serve high volume of data right then people would think oh why can't we move all the data
including some of the more application or maybe database data into this pipeline.
But then for those data of course those are more important data than log data some of the
durability and reliability that guarantees will be stronger.
So that's one aspect of it.
Another thing is when you publish data to Kafka one of the things you can do is sort
of do this semantic partitioning instead of just random distribution.
So for some applications maybe what you want to have is for example all the messages associated
with like a particular user ID right you want that to be consumed by one consumer.
So the most convenient way you can do that is to partition the messages by this partitioning
key which can be the user ID.
Then you guarantee all the messages belonging to the same user ID will show up in one partition
and as a result will be consumed by only one consumer.
Now to do things like that it actually requires each of the partition to be made highly reliable
and available right because now if you imagine if one of the partition is offline right now
you have a message semantically based on the key it has to be able to do that partition.
If that partition is not available what would you do?
You know you probably have to drop it or if you map it to a different partition then essentially
that changed the meaning of your semantics which may not be ideal either.
So that's why for those cases reliability and durability is actually equally important
when you have even just log data.
So for that in the last couple of years we actually have spent a significant amount of
time on Kafka just to add reliability and durability support in Kafka.
So what we did is for each partition of Kafka you can have multiple replicas and when you
publish a message a message will be redundantly stored on multiple replicas typically in different
replicas.
Then if any of the broker failures there is a logic in Kafka that allows the system to
continuously use function when there are individual broker failures.
So from the end users perspective most of the producers and consumer it's as if nothing
has happened because the underlying systems will route the connection from the client to
the alive brokers that has the same copy of the data.
So that's one of the important things we added in Kafka.
Is this performed by a zookeeper?
Yes, I think the way this works is we use zookeeper for storing some important state
information for partition.
In terms of the amount of data it's not a lot there's only like maybe a few bytes of
information that we need to store per topic partition.
And what we saw there are critical information as who is the leader of a partition.
I think in the world of replication typically you will have multiple replicas associated
with a partition one of them is desiccated as the leader which takes all the rights and
the other replicas are followers which just takes data from the leader.
So when the leader fails of course you need to be able to quickly elect a new leader and
then continue to serving data from the new leader.
So we rely on zookeeper to store information like who is the current leader of a partition
and who an equally important who are the correct or possible candidates for becoming a new
leader had the original leader fails.
What's interesting is it seems like the ability to just offload a little while I'm
not sure how much energy is offloaded onto zookeeper but it sounds like a significant
amount and it makes me think back to what you said earlier on in the conversation about
how the Kafka is analogous to if you look at Kafka and Storm it's analogous to unbundled
components of Hadoop.
And it makes me think it's interesting that you have zookeeper that's essentially just
like a Lego block that is now a central component of Kafka.
And so all that makes me think about what are like can you conceive of future type of
components maybe open source or not that would be built on top of Kafka?
Yeah.
So that is a good point.
I think for now, Kafka you can think of it as like a place where a feed of data is provided
and given the sort of reliability and durability guarantees we have it is possible for people
to store like the source of truth data in Kafka.
Now in some sense you can view it as sort of the commit log in the database world.
Now from the source of truth of data, there are lots of system that you can potentially
build on top of it to get a feed of data and then sort of convert data into a format that's
easier for processing for a particular vertical domain.
So there has been systems for example in the key value store domain where it relies upon
sort of a distributed commit log as the centerpiece for storing and recovering data.
Now we haven't tried this in Kafka but it is possible for someone to take Kafka and then
use it as sort of a commit log as a source of truth and view other surrounding systems
like maybe another key value store or other storage systems around it.
Okay.
Interesting.
So to go back to a little bit more about the PubSub model because I see that as fairly
important to this discussion and I'd like to just maybe even if this involves rehashing
some stuff that has already been discussed, I think it would be worthwhile.
So coming back to the topic of PubSub, so each topic is analogous to a published channel
in the PubSub model and as I understand each of these topics has one or more partitions
which is a block of memory that is allocated to it where new messages are appended and
at any given time you have each subscriber to a topic that has consumed up to some point
on that partition.
Would you describe that as, do you think that's an accurate description of how partitions
and channels work?
Yeah, that's actually, it's not 100% accurate.
One of the things what we do in Kafka is remember earlier, I think Kafka is really designed
to optimize for both the real-time consumption as well as the offline consumption.
So one of the things that requires a think about is in terms of buffering messages.
So if you buffer or unconsume messages in memory, of course that just limits how offline applications
you can support, right, because memory is limited, you can't essentially buffer everything
if a consumer is gone for a long time.
So what we do in the broker is we don't actually allocate a lot of heap space for storing messages.
So every time we get a message or a set of messages from the producer, we simply just
append it to the corresponding fire channel on the local disk.
Of course we don't flush data immediately for better performance, we sort of flush data
in those fire channels to disk only periodically.
But we don't really keep a copy in the JVM heap space.
We actually immediately just put the data into the fire system.
But most of the time, fire system has its own page cache, so chances are those data
are still in memory, but just in the page cache managed by the fire system for us, you just
have to manage that in the JVM heap space, which is a lot better for GC tuning and other
things.
Now in terms of consumer, we do the same thing, we do similar things.
We use the sort of Unix Santa Fire API to send data from a local fire channel to a remote
socket directly.
Again, from the consumer's perspective, we don't have to take any heap space by first
copying the data from the fire channel into an application space, then copying the data
from the application space into the remote socket.
We sort of rely on the operating system to do this for us more efficiently.
So because of that, Kafka, at least a broker, is actually very efficient in terms of JVM
heap memory usage.
Typically, we don't require a lot of heap space, and we only need sort of limited the
kind of tuning for GC management because we don't have a lot of a permanent space that's
taken by the JVM heap.
A lot of the memory consumption are like transient per request.
Okay, that makes sense.
So let's talk about another aspect of, I guess, distributing systems in general and
how it pertains to Kafka specifically.
So let's talk about durability.
So are messages recoverable across restarts and failures?
Right.
So yeah, there are different failures.
On the broker side, as I mentioned earlier, we added this replication support.
So within a single cluster, messages can be redundantly stored on multiple brokers.
So now, if one other broker is synonymous with a topic partition?
A broker is actually think of like, yeah, just like a server or a node where multiple
partitions data can be stored upon.
Okay.
So it's sort of like an engine where we store those messages.
And in the cluster, you can have multiple storage engines, but the topic partitions are
the replicas.
They are sort of spread around into those storage engines.
And each storage engine or broker can store, typically stores multiple partitions.
Got it.
Partitions from different topics.
That's right.
Of course, each partition typically corresponds to a local file directory.
And sort of takes this, has its own set of files from other topics.
So there's going to be isolation at a disk level.
Now in terms of reliability, if you configure a topic to have multiple replicas, then we
can, the broker on the broker side, it can tolerate broker failures.
The common failure in the couple class of course is those record, you know, soft failure.
The broker is actually a healthy, you just want to deploy a new code on making new config
changes.
Because of that, you have to bring one or maybe each of the broker in the cluster down
and restart it.
So that's actually maybe 80% or maybe 90% of failure cases in a Kafka cluster.
So with replication, of course, we can handle this kind of failure very effectively.
We have the logic to automatically just fire over the leader of each other partition to
another replica and all the clients will just behave seamlessly in these cases.
So it sounds like it's not really within the spec of Kafka to backup messages to a durable
store.
Is that accurate?
Yeah, actually that's sort of a orthogonal.
Yeah, so this, yeah, the replication really deals with for message, you know, where we
store those and how we keep, make sure we keep them in sync among those different replicas.
And a second issue, which is sort of what you are pointing out is how long do we keep
a message around in a Kafka cluster.
So that, which this is also a bit unique with Kafka is the retention of a message is actually
not driven by consumption as in typical messaging systems in the sense that a message is not
much like deleted from the Kafka ones or consumers have finished consuming of this message.
Rather messages are retained based on retention policy.
And that policy can be driven by time.
You can, for example, keep the messages for like seven days or maybe a month, whatever
you feel that you need those for.
And then of course, can also be retained based on sizes.
So you maybe you want to keep like, let's say, a terabytes of data, right, around.
So the one of the important aspect with this kind of retention is actually a pretty useful
feature.
What it gives you is what we call the rewindability from the consumers in the sense that, you know,
a consumer after finished consuming a message, if it wants to, it actually can consume the
message again as long as that message is still within the retention window.
There's some use cases of this.
I think one important use case is this can deal with like application error or logic error
very effectively.
So imagine like you have application that consumes a feed of data from Kafka and do
things like standardization, right?
You standardize each of the data and into some canonical format.
But let's say at some point you push some new software and you have like a serious bug.
And you realize that only the later maybe an hour into it, you say, oh, or the data that
processed in the last hour is not usable because of the bug.
So now what you do.
So with this sort of a retention policy, what you can do is you can actually rewind the
consumption of your messages from like an hour, maybe two hours ago, the point where
the bucket code is introduced.
As long as those messages are still retained there, you can actually just reapply the new
logic, which I assume presumably have fixed that bug and just run through those same data
again.
Then you can recover or you can correct all the mistakes you have made on the previous
computation logic.
That sounds like a super important feature.
That's right.
So that's something that not all the traditional messages provide and we realize.
Right.
The ability to go back in time.
That's right.
The ability to re-consume a lot of messages over time and fixing application logic is
one of the important use cases for this.
Cool.
So one of the words that we hear a lot on this podcast is the word microservice.
So what does the word microservice mean to you?
And is that something that's appropriate to this conversation as what Kafka is and what
it provides?
Yeah.
So actually I'm not 100% sure what those microservice means.
What's your interpretation of those microservice?
Sure.
So what I would think of a microservice as is something that is a service that can be
put it in this context.
A service that could have a call made to it over a message broker.
Just a small service that X needs to get done.
You could make a call to it over whatever message broker you have.
The service gets performed and the advantage to these sorts of services that you can modularize
them and it becomes easier to scale more instances of them.
As I understand about the word microservice is that it means different things to different
people.
So that's part of why I brought it up.
If you're not even really familiar with it or comfortable discussing with it, that's totally
fine.
Yeah, I'm not 100% but I think it sounds like is that sort of an ad hoc use of the message
broker where you want to start out some services just for a short window of time and then you
want to do some processing of the message and then after that you can just stop the service.
That's exactly what I'm saying and I'm just kind of wondering if Kafka serves that purpose
ever.
Yeah, actually one of the things actually this actually has to do with the multi-subscription
model that Kafka provides.
Because one of the powerful things that we see people is enabled with Kafka is the ability
to do debugging on Kafka.
So this can happen in the production environment but more often it happens in the sort of a
testing or the pre-production environment where of course you have all the applications
that's written up to consume like a feed of data but sometimes it's actually if the application
doesn't work as expected, right, how do you know what went wrong.
So the ability of doing multi-subscription allows you just to easily tap into like a
topic, use a command line tool on a console where you can actually see what the data are
and then based on that potentially we have a bad idea where the problem is that you don't
get the right data or there's something else in the application.
But that's sort of like a special case of maybe microservice that you are referring
to.
Sure, sounds good.
So in terms of the, I'd like to talk a little bit more about the development of Kafka.
Why was the Scala the language of choice?
Like did you, and more specifically did you consider Erlang?
Right.
I think, yeah, when we started this, I think J. Kras was the first developer of Kafka around
that point, you know, he was, Scala is getting a little bit popular and he's thinking of
like a learning Scala.
So that's why I thought he picked Scala.
It's sort of a way of just to try it out and then see how it works and from developers
perspective, it's always like interestingly exciting to try some new like promising programming
languages.
Now for Scala, I think what we benefited a lot from Scala is mainly the sort of the
conciseness of the syntax.
We use a lot of the functional support on the collections, which allows writing code
of iterating collections very convenient.
And because of that, if you look at the Kafka code base, it's actually sickness-spore compared
with some of the similar system but written in Java and we've identified a lot of just
from the Scala-concise syntax.
But one of the things we found is a little bit difficult with Scala, this actually mostly
has to do with the clients, which is because Scala is sort of emerging technology is not
as mature as Java in terms of preserving binary compatibility.
Just because it's early in some of the early releases, Scala really have to break the binary
compatibility.
So some of the byte code generated in an early version of Scala won't be able to run on
a new version of Scala.
So this actually creates some problem for upgrading our clients because a lot of our
clients maybe they are, of course, using Kafka because of that, they have a Scala dependency
but they may use some other components which could also be Scala dependent.
Maybe we're dragging a different version of Scala.
Now if the Scala version is not compatible, then it makes the upgrading of those clients
very painful.
So that's sort of one of the things that we find is a bit hard with Scala.
So we're trying to address this issue by sort of rewriting the client components just
in pure Java.
So hopefully this will still allow us to get most of the benefit from using the concise
syntax in Scala but at the same time makes the client upgrade a little bit easier going
forward.
That's great.
So this has been a great discussion of Kafka.
Where can listeners go to learn more about Kafka or maybe even your work on the project?
Yeah, I think the best place to do that, of course, is just go to our website.
See if you search Kafka in our web, just in Google, I think Kafka probably is Apache
Kafka is probably the first or the second link.
So there it has some documentation that describes what Kafka is and how Kafka is designed for
and some of the examples of how to use our APIs.
And you can download a code and there's a quick start that you can follow, just try
it out.
And from that webpage, we also have a link to our Gura system in Apache.
If you are interested in contributing, you can just search for some of the newbie labels
in the Gura system that probably will give you some low-handing fruit that you can give
it a try.
And we also have a wiki that describes some of the future release plans and future features.
So those are more substantial development work that's required.
So if you are more experienced contributor to Kafka, some of the future projects may
be of interest to you.
And now some of the new things that are coming out of Kafka land, one is we are in the process
of rewriting some of those consumer APIs as I mentioned to reduce, to remove the Scala
dependency and to just make the API more feature complete and better performing.
Another thing that's coming up soon is the security support in Kafka.
This includes probably authentication and maybe sending some of the data in the encrypted
way.
We also have thoughts on how to sort of protect Kafka in multi-tendent environment.
So hopefully different users don't affect each other, especially if one of the applications
in the head of bug and becomes like a runaway user.
So we have some thoughts on how to use a quota system to prevent things like that from happening.
So all those information can be found on our Apache website.
And we also have like Apache mailing list for both the user and developer.
And if you're interested, you can just subscribe to those mailing lists and get a feed of what's
happening in the Kafka world.
That's great.
All right.
Well, June Rao, thank you so much for coming on Software Engineering Radio.
It's been a pleasure.
Yeah, thanks a lot, Jeff.
Okay.
All right.
You have a great day.
Good to you.
All right.
Thanks, Jim.
Thanks for listening to SC Radio, an educational program brought to you by IEEE Software Magazine.
For more information about the podcast, including other episodes, visit our website at se-radio.net.
To support us, you can advertise SC Radio by clicking the dig, Reddit, delicious or slash
thought buttons on the site or by talking about us on Facebook, Twitter or your own blog.
If you have feedback specific to an episode, please use the commenting feature on the site
so that other listeners can respond to your comments as well.
This and all other episodes of SC Radio is licensed under the Creative Commons 2.5 license.
Please see the website for details.
Thanks again for your support.
