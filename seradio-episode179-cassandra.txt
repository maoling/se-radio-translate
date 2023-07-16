**Link** https://www.se-radio.net/2011/10/episode-179-cassandra-with-jonathan-ellis/

**Transcript**:

This is Software Engineering Radio, the podcast for professional developers on the web at se-radio.net
SE Radio brings you relevant and detailed discussions about software engineering topics at least once a month.
Thanks to our audience and the partners listed on our website for supporting the podcast.

**Robert** This is Robert for Software Engineering Radio. I'm at the O'Reilly Strata Conference in Santa Clara, California.
Jonathan Ellis. Jonathan is the project chair of the Apache Cassandra project.Jonathan and I will be talking about the Cassandra open source database project.Jonathan, please tell the listeners about your background.

**Jonathan Ellis** My background is for the Cassandra. I built a pretty much steel work for a site.It was like a private site.
Tonight I'm actually where I will have to be a beautiful database in that style of example.

**Robert** Tell us briefly what is Cassandra?

**Jonathan Ellis** Cassandra is a scalable database that makes the best of people big table approach in the Amazon Dynamo approach.
Specifically, this is big table because it's a log structure storage system that's highly efficient on modern storage for writes
as well as to use the form E. Well, the Amazon Dynamo system is the equivalent of the distributed model for replicating and patitioning data that are witnessing the different criteria in Cassandra gives you the best of all for welfare.

**Robert** Tell us a bit about the history of the project.

**Jonathan Ellis** Cassandra was started at Facebook and open source in the summer of 2008, I believe, and I got involved later that year when the project moved to the Apache Foundation and we started building the community from there in the Apache incubator
and graduated in April of last year and we just released our 07 version in January.

**Robert** Why are people now looking at alternatives to relational databases?

**Jonathan Ellis** It's mostly because, well, there's a couple reasons you have this whole most general term for anything that's not a relational database and there's all kinds of systems under that rubric. But I think the really interesting problem is the one of scale that relational databases weren't designed to scale,
don't do it particularly well. And in fact, you can make a relational system scale by partitioning it and you're adding an effect taken by what you end up with is two things. One, it's very ad hoc and one off-ish. You can't take an architecture that partitions a relational database or eBay and run Amazon's infrastructure on top of that, even though they're both, you know, e-commerce site.
The other is that when you do this, when you take a relational system and partition it and you make it scale,
you're giving up, you have to give up the benefits that make you want to choose a relational system in the first place,
maybe you're acid guarantees. So, you know, companies that are running into, you know, big data problems are taking a look at this and saying,
you know, either we can build a one-off that doesn't really take advantage of the strengths of the relational platform,
or I can go with a system that's designed for this kind of new need of application and give me some benefits to make up for giving up acid.

**Robert** 00:04:25 People are trying to rethink the problem in a more fundamental way and not just get stuck with certain trade-offs you get by trying to fit the round peg in a square hole.

**Jonathan Ellis** 00:04:37 Yeah, I think so. I think that more and more companies are running into data sets and query volumes that can't be handled by single machine databases
anymore, and once you start partitioning, then these scalable database like the cassandra is going to be a better fit for your application.

**Robert** 00:04:56 Would you accept the division into the non-relational stores into key value stores, document oriented and big table type column family?

**Jonathan Ellis** 00:05:08 Yeah, I think I wrote an article a couple years ago called the NoSQL ecosystem, and one of the things I talked about there is that there's several different axes on which to evaluate a database. One of them is the data model. That's what you're talking about here with document versus key value versus column family. But there's other important factors to keep in mind when evaluating these systems, one of which is not only what's the data model, but what's the storage model, which are related, but not the same thing. For instance, there's a database called Riak by a company named Basho, and it gives you a document data model, but it's implemented in terms of pluggable key value systems. You can run it on top of BDB or on top of BitPass, which is a new value single machine database that it wrote to the back of the yacht. There's very different characteristics depending on the implementation there. In the same vein, you have CouchDB, also gives you a document data model, but that's implemented in terms of appendomy features. So the performance characteristics of BitPass versus CouchDB versus MongoDB are quite different.

**Robert** 00:06:45 Let's talk about the data model, Cassandra. Give us an overview of the main concepts in the data model. 

**Jonathan Ellis** 00:06:52 So Cassandra takes the data model primarily from Google Bigtable and divides the world up into key spaces and column families, which correspond to schemas and tables in a relational database.
The difference between a column family and a table is that there's a couple of differences. One is that columns in your rows in the Cassandra column family are sort of, it's not schema-less because you can annotate columns and say, this column will be of this type and I want you to enforce that one.
But it's a sparse model because you can use columns in your rows without declaring them if you want to. And then you can later add the metadata about the data type or you can just leave it as a column full of fights and Cassandra is okay with that. So one of the differences is then that you don't have to reallocate each row in the table to add a column.
So you can think of either alter table as being optional, or you can think of it as much higher performance than what you would have in a BP based system.
And either of those ways of thinking about it is correct.

**Robert** 00:08:29 You have made a distinction between more of a static schema, although it's not really enforced by the server versus more of a dynamic schema where you freely add columns.

**Jonathan Ellis** 00:08:42 Sure. Yeah, so I like to think of modeling applications in Cassandra in terms of two sort of classes of column family that I call static and dynamic column families.
I'm using static and dynamic in kind of the sense of how much changes going on at runtime, rather than in the sense of static versus dynamic typing or something. So in other words, a static column family would be a column family that the rows are representing objects in your application, for instance, so they all have more or less the same columns. You may, you may, you know, six months into deploying your website, decide that you're going to have users provide you with an SMS, a number as well as an email address to contact them.
And so you can just, you know, by not having to do an alter table, your application can just start putting in that SMS number without having to do any extra work on the, you know, on the DBA side. You know, you still have most of most of your rows, you know, share the same set of columns just by convention. Then the other way, the other kind of column family is where we use a column family to represent materialized use in a single row. And this is where when we did our own seven release, one of the things that a lot of new stories picked up on was that Cassandra 0.7 can now have up to billion columns in a single row. You know, the old limitation was you can have up to two gigabytes worth of data in a single row, because we were representing the size field on disk as a 32 bit integer. And so now the, now the limit has become the number of columns is 2 billion, and the size is basically limited by your disk space. So that, so either way we're talking about really large numbers of columns compared to what you would have in a relational system. And the reason is, is that it's not because we're insane, but it's because we're, we're providing you with a tool to avoid joins, because, you know, even in, even in a relational system, you know, people learn to avoid joins for maximum performance. But that's even more important in a distributed system, where rows in different tables or different column families could live on different machines altogether. And so the price of doing the expense of doing that joint goes up even further because you're dealing with network latency is, and not just data that's all local to single machine. So, so we really want to emphasize avoiding joining, and, and having the ability to put, you know, millions or billions of columns in a single row, lets us denormalize your, your query results set and pre compute it into one of these rows. So, for instance, one of the sort of teaching examples we have doing Cassandra, you know, development is a toy Twitter clone called Twisandra. So if you go to Twisandra.com, there's a, there's a live demo and there's a link to the GitHub source, and it's written in Python on top of Django, but it's, it's, it's, it's, one of the nice things about Python is it's quite readable, even if you don't really have a Python background. So, so it's a pretty good example, even if you're a Java programmer or a throat program or whatever. But one of the things, the core, the core query you make in Twitter is, you know, what tweets have my friends made. So the way you would model this in a relational database is you would have a table for tweets and a table for users and a table to relate users to other users to say who are my friends. And then you would, if you want to say give me the 50 most recent tweets that my friends have made, then, you know, what you're doing is you, first you have to say, well, what is the set of tweets my friends have made and I compute that by going through these join tables, and then I can pull out those cheat rows, sort them by time, and give you back the result.

**Robert** 00:13:22 It's pretty simple design problem for anyone who's read a book about relational databases using a normalized model. So how would you do that with Cassandra to take advantage of Cassandra's data models.

**Jonathan Ellis** 00:13:36 So the way you would do that in Cassandra is, I would have a single row, I would have a separate column family, that would be where I answer this query from of what tweets of my friends made.
And twice and recall that the timeline column family. And so I would have a row in that column family for each user. And the columns in that row are going to be the tweets. I will denormalize each tweet that someone makes into the timeline row of the people who follow him. 

**Robert** 00:14:13 In the relational model, the idea you normalize everything every piece of data is in one place. You can always find the data you want if you're willing to do enough joins. With Cassandra, it sounds like what you do is think about the queries that you're going to do and then create one storage class for each query.

**Jonathan Ellis** 00:14:34 Right. That's exactly right. And, you know, there's, what we typically see at data stacks when people come to us and say, hey, we're writing this application against Cassandra.
Can you review our data model is, you know, because Cassandra does give you this rows and columns of sort of view of the world, they'll treat it like a slightly brain damage relational database. And they'll do, you know, all their joins client side and still have this sort of normalized data model only in Cassandra. And that's, that's, you know, Cassandra tries to push you away from that into this denormalized model, you know, to save you from yourself,
because that's how you're going to get, you know, really good performance. And having basically best in class, write performance, let's us say, hey, you know, creating those extra copies isn't as expensive as you're used to thinking it as. So go ahead and spend some writes at creation time to pre compute those queries so that they're light and fast.

**Robert** 00:15:56 Suppose you're dealing with an application that has a bunch of different relations and there's many different queries that come first through a to be to see and then be to see to a and so on. And you, you end up having 20 different queries. Is this a efficient way are people having success in building applications with dozens of different copies of the same data based on the number of ways they need to access it, or is that just the wrong application for this data store.

**Jonathan Ellis** 00:16:31 So, well, there's this important thing that I haven't mentioned, which is that, well, Cassandra doesn't support joins, it does support indexes. So this is something that, you know, certainly the key value type of databases, they don't give you that kind of more sophisticated query capability, and Cassandra does. So, you know, that helps a lot where this kind of application where you're saying that, you know, I have a lot of different combinations of things I need to ask, and maybe, you know, pre computing them all is combinatorially impractical. So having those having those indexes that allow Cassandra to, you know, apply. Credicates to your query at runtime can can help with a lot of those situations. That said, you know, if you have something that's truly ad hoc queries at runtime, that's really a super hard problem to scale, and the lack of joins makes it harder in one respect in Cassandra. So, I would say that any system that's going to give you joins is going to give you a different set of problems. So I would say that Cassandra is probably about as good as anything else when you have the tech combination of, I need to be able to do those queries, but I also need to scale.

**Robert** 00:18:07 Let's get more into the systems side of the project, reading the paper, reading a paper from two Facebook guys about Cassandra. They said the major properties of Cassandra are distributed, decentralized scalable, highly available fault tolerant, and tunable consistency. Would you agree with that list? 

**Jonathan Ellis** 00:18:30 Yeah, I'd say that. I'd say that still after it. I'd maybe I'd maybe add high performance. 

**Robert** 00:18:33 Let's talk about how, how does Cassandra achieve distribution.

**Jonathan Ellis** 00:18:40 There's a couple things that kind of fall under the distribution category. One is, how do I spread data across multiple machines? And the other is, how do I, now that I have my data spread across multiple machines? How do I make sure that I have multiple copies of it as well? So the first, that first part we call partitioning, and the second part we call replication. And both of those are pluggable strategies in Cassandra.
And most commonly, the partitioning is done by a consistent hashing partitioner called random partitioner. And that, that, basically, that distributes rows around the cluster by the MD5 of their primary key.
Now replication is a little more interesting because there's so many different things you can want to achieve with it. For instance, you might want to say, I want, I want three copies of my data, but I want to guarantee that there's at least one copy in each of two data centers. Or you might want to say that, you know, I have three sets of data, and I want all of them to live in each of one data center, but then I want to master copy of each of those pieces of each of those sets of data in a fort.

**Robert** 00:20:12 So you see the multi data center deployment at simply one of the possible conditions you face in distributing your data. It's not a separate engineering task. Now we've got the data center up and running. How do we solve multiple data centers?

**Jonathan Ellis** 00:20:31 And this comes out of Cassandra's design as a fully distributed system. There's no special nodes in a Cassandra cluster. And so what you see in a lot of older systems, older designs, is that there will be some kind of master that is responsible for a certain range of data.
And, you know, writes will be fast as long as you're in the same data center as that master. But then, you know, you know, if you want to replicate to another data center, then you can't, you can't do writes in that data center.
You have to send them over to the master. And then maybe we replicate to the second by log shipping or something like that. So it's kind of, you know, a second system rafted on to the original single data center design.
Whereas with Cassandra, you know, a second data center is just more nodes in the cluster that happened to be across a highlight and ceiling.
And so Cassandra, you know, replication by and large, everything works exactly the same as in a single data center with the exception that Cassandra recognizes that, you know, hey, that that gap that that that that that wannling is higher latency and more expensive. So what I'm going to do is if I need to replicate to multiple nodes that are in that other data center, then what I'll do is I'll send that write to a single node with a note that says also forward it to this other node that's local to you so that I so that I minimize my use of that extensively.

**Robert** 00:22:16 Do people typically think in terms of one data center being live and the other being a backup or do you see the entire Cassandra cluster as being simply spread over a wide geographic area and you don't really care where in that cluster you write to?

**Jonathan Ellis** 00:22:36 mostly people find that, you know, having that ability to have a multi master system across data centers is really useful because that allows you to have if you have if you're serving users on the east coast and the west coast or, you know, in California and in Europe, you can have those clients do their updates to the nodes that are closest to them.
And so it and you get you kind of get the geographic redundancy for free, but maybe the lower latency to your clients is maybe the more important of those two qualities.

**Robert** 00:23:16 Walk us through what happens on a write?

**Jonathan Ellis** 00:23:21 So on a write, Cassandra's so you can talk to any node in a Cassandra cluster and give it a write and it will take care of sending it to the write replicas. So this.

**Robert** 00:23:32 this is a client, it can connect to any node in the cluster. And do you typically use a load balancer or how does the client find which node to connect to?

**Jonathan Ellis** 00:23:45 this kind of no wrong answer there. There's a bunch of ways you can do it.
My favorite is to use round robin DNS, but we do see deployments using load balancers like HA proxy or hardware load balancers or even a more kind of client side approach by clients like Hector for Java support.
You give it a, you know, any Cassandra node and it will ask that Cassandra node. Hey, tell me about the other nodes in the cluster and then then the client will spread its connections around all of all of those nodes or the ones that are local to it in the same data center. So Cassandra also supports a smart API called the storage proxy API where the client actually routes the data directly to the right replicas. So we de-emphasize that a little bit because that's that's Java only, but it is out there for people who want to use it. We do have, you know, at least a handful of production deployments using that. So by and large, most most appointments use the API and use something like Hector on top of that and Java or Picasa and Python and so forth. So when the node that a client connects to is called the coordinator node. And that's that will be responsible for sending the write out to the right replicas and then reporting back to the client, successor failure, depending on what it gets back from those replicas. So, you know, successor failure is a little bit of a fuzzy concept in Cassandra because we support tunable consistency level, meaning you can tell Cassandra, consider this write, a success, only if it's written to the majority of replicas for that row, or you could require just a single replica to be available to consider that write and accept that it will have to be replicated to the others that are down right now or unreachable at a future point.

**Robert** 00:26:12 Is a client control the number of replicas on a write per operation or do you set that per column family or cluster? 

**Jonathan Ellis** 00:26:22 Yes. So in other words, there's these two factors for the number of replicas. There's the total number of replicas of a row, and that's that's set per keyspace. How many total replicas you want to have data in that set of column them? But the number number of replicas we block for before calling a write of success is per operation.

**Robert** 00:26:46 You have two numbers there. The total number of replicas that will eventually be written and the number that you need for a client to a client operation to return. Okay. Now, Rok is still what happens on a read?

**Jonathan Ellis** 00:27:05 A read gets a little bit more complicated. So, you know, at the high level, it's like a right only, you know, the other direction. A client talks to a coordinator says, I want some set of columns from this row, and the coordinator goes out and gets it. You know, drilling down a little further though, things get a little more complicated.
Because we want to be efficient. In other words, if I have five copies of a piece of data, and I have for every request, I'm calling all of those five copies back. That's using up a lot of network traffic. I don't want to do that. So what Cassandra does, what the coordinator does, is it says, well, first of all, it says, how many requests, what consistency level am I operating at? How many replies do I need to compare to call this read successful and make sure I'm giving the client, you know, as fresh a copy of the data as it's requesting. And once it calculates that, it will go through, Cassandra has a subsystem called the failure detector that it uses an algorithm that's described in a paper called the Phi failure detector.
And it's a very sophisticated probabilistic failure detection algorithm based on gossip heartbeat. So we can say with high accuracy, you know, if I haven't received a heartbeat from this note for a few seconds, and network traffic has been volatile recently, then he's probably still alive. It's probably just another network pickup. But, you know, the longer it goes, the more likely it is that he's actually really experiencing downtime or a long live petition.
So I get this list of note of replicas that are, you know, that are not failed. And then I take the closest one. And in how we determine closeness is a separate subject that we can go into. And I take the closest one, and I say, I want you to send me the data in the columns that are being requested. And the other, the other replicas that I need to satisfy the consistency level, I will send them just to request for a hash of those columns so that
I'm only sending the full body of the result set once. So it's an optimistic system because most of the time those digests are going to match the body, in which case I just say, okay client, here's here's the result. But, but if there's a failure, if there's a digest mismatch of those hash of those hashes, then I have to go and do a second round and I say, okay, this time I'm going to ask for the full body of the results set from everyone. So I can figure out where the, you know, who's got stale data and send him a more recent copy.

**Robert** 00:30:31 We know from the CAP theory that data stores either fall into the category that emphasized or distributed data stores either emphasize consistency more availability. Would it be fair to say that Cassandra is trading off more toward availability? And it's willing to give you data that might be a little bit out of data in order to give you something?

**Jonathan Ellis** 00:30:54 Cassandra gets put into that category most often, but it's more, Cassandra lets you choose with when you want. If I ask for quorum consistency level on writes and quorum consistency level on reads, then my readers will always see the most recently written values. So it will be a consistent system. But, you know, that means that I can only, if I have a three node replication factor, then quorum consistency will only let me lose a single node before I have to say, I can't fulfill these requests anymore because the system is, you know, too damaged that I've lost too many machines. So often, you know, production deployments will say, you know, I am okay with seeing stale data during failure conditions. So I'm going to reduce the consistency level I require to get that higher availability so.

**Robert** 00:32:00 what happens when you add a node to the cluster? 

**Jonathan Ellis** 00:32:04 When you add a node to the cluster, then the existing nodes stream over the data that the new node becoming responsible for. So part of using something like consistent hashing and getting, you know, constant time routing with no central master is that we have to move that data to the new one that, you know, it's positioned in the token ring indicates that it should be responsible for. We call that bootstrapping the process of moving data to a new node in the system. The good news is is that because we manage our own data on this, we can do that without imposing any random IO. on the system. It's all just sequential reads from the existing machines and sequential writes on the new node. And so it's actually, you know, it's over very quickly and it's very effectively.

**Robert** 00:33:10 how big are some of the clusters that you're seeing your clients use at data stacks?

**Jonathan Ellis** 00:33:15 The biggest one that I know of is cluster running digital reasoning software for the US government that's over 400 nodes. There's a lot of clusters. That's a, that's sort of the, you know, that the outlier on the size scale. The, the mean, or the mode cluster size and this common cluster size is probably between 20 and 40 nodes. And we do have it at the other end of the spectrum. We have one client that's running just a bit.
They started with just two nodes, because they knew they wanted to scale and that Cassandra would be able to do that seamlessly. And they didn't, they didn't have to start with a huge cluster to begin with.

**Robert** 00:34:00 Is each node in the cluster and running the phi failure detection algorithm? 

**Jonathan Ellis** 00:34:04 Yes.

**Robert** 00:34:06 And then if a node goes down and other nodes would fairly quickly identify it as down using this algorithm. And do they go through a reverse of the node addition process to create enough replicas for data that was on the failed node?

**Jonathan Ellis** 00:34:23 That's a good question, because what we actually do is we don't assume that a node is dead permanently until human in intervenes and says, okay, he's not coming back. Because the most common failure condition is some kind of temporary failure, whether, whether it's a switch that failed and so the nodes unreadable or a circuit was overloaded and the node goes down, but you know, all the data is still there.
Having a node that's absolutely wiped out and the data doesn't exist anymore is relatively rare. So we don't want to impose the overhead of rebuilding that data until someone says it is necessary. And that's the nice thing about having a replication factor that's basically as high as you want, but the most common one is replication factor three, because if you just have a single replica, well, there's a couple of drawbacks of that. One is that a quorum of two is still two. So, so you don't have, if you're doing quorum consistency, you can't lose anything before you have to start denying requests.
But the other is that from a, you know, I don't want to leave my data point of view. If I have two replicas and one of them goes down, then I'm really, really nervous until I can re-replicate it, because now, you know, when one's down, I just have a single copy. And if I lose that, then game over. But if you have a replication factor of three, then, you know, even if I lose one, one copy, you know, I'm still feeling confident because I still have two copies. And, you know, I don't start to get nervous again until, you know, unless I lose the second one before the first one comes back online, or get to be replicated.

**Robert** 00:36:20 And about the case where you have two data centers and you lose the data center or they lose communication, you've configured it to put one replica in the other data center, then at that point, would you want the, if you think you've lost the data center for a while? Would you want the surviving data center to re replicate data so you have your desired number of copies within that data center?

**Jonathan Ellis** 00:36:47 Usually, usually not, because again, you know, if I lose the entire data center, then it's probably some kind of network problem, and that data center will be back online within probably hours, worst case, probably days. And so, you know, you replicate in Cassandra to, you know, to protect your data, not to improve performance. So maybe that, you know, people coming from a relational database background, you may be thinking, well, you know, I'll create more replicas to handle more traffic.
But the right way to handle more traffic in Cassandra is to add more machines and spread your data across more machines, rather than increasing the replication count, which would just put more copies across the same number of machines. And because remember that, you know, we're doing that comparing of digests of the different replicas with the data on each we eat.
So, and so in general, unless there's some caveats you can turn off with there, for instance, but in the out of the box configuration, you don't add more capacity by adding replicas. Because you add more capacity by adding more machines and spreading those replicas more thinly across the machines.

**Robert** 00:38:28 You just mentioned read repair. Tell us how that works.

**Jonathan Ellis** 00:38:32 So read repair means that even if I say, even if I do a read at consistency level one, which it says, you know, just give me, you know, the result from the closest replica of Cassandra will in the background.
So go ahead and after it gives you that reply, it will go and ask the other replicas for digests of what it just read so it can make sure that that they are up to date. So that's one of the many ways that Cassandra keeps its replication in C. So there's, there's no way to do great Cassandra's replication such that it has to rebuild from scratch. Which is, which is a really nice property coming from a relational background. I've seen a sloney replication for Postgres as well. And just, you know, the only way to make it recover was to rebuild the slave from scratch.

**Robert** 00:39:37 You really don't want to have to do that in a production environment.

**Jonathan Ellis** 00:39:42 No, no, no, no, because not only does it, does it impose a very large performance penalty while you're doing that.
You know, it also means that you're in that situation where now I just have that one copy of my data and I'm nervous. You know, if something goes wrong while I'm rebuilding, I'm not in a happy place. So the three ways that Cassandra uses to keep its replication up to date are read repair that we mentioned, hinted hand-off and anti-entropy repair. And all of those terms come from the Amazon Dynamo paper and they're a little, they don't mean anything to you unless you've read that paper.
So briefly, what those other two, hinted hand-off means that if I'm doing a write and the failure detector lets me know, hey, you know, this replica of this write is down. Then, well, it, well, then one of two things will happen. If I don't have enough replicas alive to satisfy the consistency level, then I'll tell the client, you know, I'll give it an unavailable exception that says, you know, there aren't enough live replicas to satisfy your request.
But, but if I do have enough live replicas to satisfy the request, then I will send one of the live ones a piece of metadata with the write that says replay this right to this, this replica that's down right now when he comes back up, replay it. So that's called, that's called a hint and the replay process is called hand-off. So that's where that, that's where that turns comes from.
Then the third way to achieve consistency is the anti-entropy repair. And that's basically, you know, it basically are seen for databases. So what we're going to do is we're going to build a tree of hashes for each range of data in the replica. And we exchange those hash trees in such a way that we find out which data blocks are out of sync in logarithmic time. And we're only exchanging those hashes over the network until we find what is actually out of sync. And then we, then we repair that. 

**Robert** 00:42:06 So efficiently compare trees without having to send the entire trees back.

**Jonathan Ellis** 00:42:12 Yeah, I mean, worst case, we'll send the entire tree back and forth, but you know, that would be the case of, I, you know, I, Rm-RF, my data directory is nothing there anymore.
When there is data that's shared between them, even if it's only a little bit, then Cassandra will be able to avoid replicating that part.
I'd like to switch gears and talk about the storage model you mentioned very early on in the interview. Tell us, give us an overview of the storage model that Cassandra you do.
So so far, we've been talking about in terms of replication and consistency level of Cassandra's kind of dynamo heritage and then moving to the storage model, then we were kind of switching gears to the big table influence.
And so actually big table actually wasn't the first to describe this kind of storage model, which is a log structured storage model for databases.
That comes from a paper in 1996 called the log structured merge tree.
And that paper didn't get a whole lot of attention until Google cited it in big table 10 years later.
But what, what happens at the storage layer when a replica, when a write comes into a replica that belongs on that node is, you know, so, you know, most, most databases use a B3 based storage model, that's more or less updated in place.
You know, there's, there's a little bit of a wrinkle from things like multi version and currency control, but you can pretty much think of it as as an update in place storage model, where there's a certain block of data that of row lives in and then we follow an index down to find that block.
And then we update the columns for for an update request. Now, in Cassandra, what we do instead is when we have updates come in for a column family, we put them in a structure called a men table.
And we that we collect, we collect updates and we don't, we don't turn those into a file on this until the men table is full. And then we, we sort the men table by row, and then we write it out to disk where we call it an SS table.
You know, it's a, it's a data file. And once it's written on this, it's immutable. We've never do updating place. We only, we update by putting a new version in a new men table and eventually writing that out to a new data file.
This is the node serve rights or region rights out of the men table. Yeah, so the data in the men table is live for requests immediately. And what we'll do is if, if, you know, if I have a row for my user record on on a website.
And so my company changed its name recently from Ritano to data stack. So in my profile, I have my URL, it says, ritano.com. And then I come in and say, hey, no, I changed my company name. So I'm going to change my URL to data stacks.com.
Well, I have my original record on disk in an SS table file. Now I have my new URL in a men table that hasn't been written to disk yet. So if someone comes in and says, you know, the equivalent of select, start from users where username is JBLs, then I will, you know, Cassandra will merge that new value that's sitting in the men table with the row that's on disk and say, you know, this is the
right to the end of the list and say, you know, this new value supersedes the one that was in the row on disk. The rest of the data that was on disk is still current. So I'll, I'll present that in the result set as is.
There's a right complete successfully before anything's been written to disk.
So short answer is no, because before we put the data in the men table, we append it to a commit log. So this is the same kind of architecture that, you know, traditional databases use to provide your ability is when we append to commit log, and then
the tunable parameter here is how often do we F-sync that commit log. In other words, how often do we tell the operating system to actually send that data in the commit log to disk.
And so if you have your commit log on a separate device, then the data files that you're doing reach from, then, you know, you're just constantly doing a pens. That's a very, you know, that's a very fast operation because I'm not having to seek the disk head around.
But, you know, and so this is the same for relational databases as for Cassandra, or at least commit log oriented relational databases that want to have that on that on that separate device and performance is worse if it's not.
Now, there is kind of a workaround for, you know, having it on a non-performant environment, which is that we're only F-sync every so many seconds.
Instead of instead of grouping up writes into groups of say, you know, 10 milliseconds worth of writes and F-syncing those together for 1 millisecond worth of writes and F-syncing those together, we will, because, you know, even if it's on its own device, you don't want to F-sync for each individual
write, you want to batch those up somehow. But we can relax that and say, we'll only F-sync more every 10 seconds, no matter how many writes come in, to give you better performance if you have to have your commit log on a shared disk with your data files.
What you're trying to do here, then, is support a lot of writes without having to do a lot of seeks on the spindle.
Exactly. Are there clients for all the popular programming languages?
Well, I guess that depends on what you call popular. There's two levels to the client story in Cassandra. There's the raw communication protocol is an RPC framework, RPC and serialization framework called TRIP.
And that generates code for, you know, around a dozen languages. And yeah, I think that does cover pretty much all the popular ones.
Actually, the only one that I've had someone request a client for that TRIP doesn't handle is Delphi. So that, but C-sharp, Erlang, O-Caml, small-pot.
Java, Perl, Ruby, Python. Yeah, I mean, you know, even some clearly obscure ones like those others, you know, TRIP does support.
But then, the RIP does really, the right way to think about the RIP is that it's kind of the driver. So sort of like libbq for PostgreSQL.
You don't really want to write your application code with RPC calls if you can possibly help it.
That's where it's a little more sparse. We have clients for Java, for Python, for Ruby, for PHP. And those are the ones that are really well supported.
Oh, C-sharp as well. But C++ is a little more touchy. There's a client out there that needs a little bit of fleshing out to be a really full-fledged environment.
Perl is another one where there isn't a really good client that's up to date for the Cassandra 07 features.
Those are the two main ones that we'd like to see, I think, a little better story with the C++ and Perl.
Cassandra, support anything like transactions? It supports something like transactions, but not transactional per se. So when we think of transactions, we think of the atomic, isolated, consistent, and durable.
Cassandra gives you durable. If it wasn't clear in my description earlier, you absolutely can configure Cassandra so that every write is 100% durable.
And every write that was acknowledged will be there when we come back up. So we do durability. And we do atomicity within limits. So atomicity means that if I give you a write, either all of it will happen or none of it.
And we do atomicity within a single row. That's the level of what we put in a commit log and that's the level of routing of data and partitioning.
More broadly, when people think of transactions, they say, well, I want updates to these different tables and I want all of those to happen as a group.
So Cassandra doesn't give you atomicity across multiple different rows. But what it does give you is this concept called a batch.
So that's sort of the thing that's like a transaction to Cassandra is you can have a batch of updates and you send it off to Cassandra.
And if what you need to worry about is the coordinator node failing part way through. That's the edge case there.
If the coordinator node fails part way through, then the client needs to fail over to another node to talk to a new coordinator. And that's one of the things that good clients handle for you like Hector and Picasa, they're going to handle that for you.
And then you retry the batch. And the property of the batches is that even if part of it got applied already retrying it will be item potent. And only the parts that weren't applied will happen.
And the parts that already did, no harm done was still cool.
If you're designing a highly normalized database where there's multiple copies of the same data for different queries, you could have batches that are partially completed.
Then isn't it the case that clients would have to be prepared to deal with what we would call referential integrity failures during reads.
And you have to worry about that if you're not fully normalized. If you're only partially normalized, then you have to worry about that because partially normalized means I'm doing some kind of joined client sided and that's where you have to worry about that.
So, you know, each query is coming from one of these materialized new rows, then the data in that row, because I said data within a row is atomic, you don't have to worry about that.
Okay guys, so it either succeeds or it doesn't. But if we could proceed to talk a little bit about the project, tell us about the structure of Cassandra as an open source project.
And then we started getting engineers from dig and Twitter working on it. Today we have substantially over 100 people who contributed at least one patch and more who contributed bug reports and so forth.
So, in this case, for instance, wasn't looking to build something solo. It's really been, you know, today, you know, data stacks has quite a few people on it full time, but it's still a very healthy community where it's developed in the open and people from different companies are working on it together.
So, it's one of the healthiest that you need in that respect that I that I've seen.
Do you have mailing lists, wiki, forum, things like that.
There's there's a user mailing list, a developer mailing list, a client developer mailing list, a lot of the action.
And to see this in the open source world in general in the last few years is projects have moved a little bit more towards meal time communication.
And so younger projects. I think I think there's a common trend towards emphasizing something like IRC, a little bit more than mailing lists.
And we have we do a lot of our development in the Cassandra dev channel on keynote.
And there's there's a just count Cassandra channel for users there that consistently has about a couple hundred people there.
And basically pretty much any time zone in that real time help in IRC.
And you know there's there's also the mailing list for more involved discussions or more more more highlights, I guess.
People want to find out more about the project work.
Should they go?
I think they should go to Apache dot Cassandra dot Apache dot org.
And I would I would particularly recommend the wiki page called articles and presentations.
And that has several high quality articles about Cassandra at the top of the page.
Tell us about your company data stacks.
So I started data stacks about eight months ago we called it with Tano and we see the change our name.
And we provide professional products and support with Cassandra.
We have over 50 support customers and then then there's the customers that we've done.
Product design and training with it's it's going really well this week at strata where we're introducing our Cassandra management platform called off center.
And and starting to give that out to more interested data users.
Do you personally write or blog anywhere?
I do most of my blog in these days on the data stacks website.
So that is I think it's that's the next time slash blog, but it's definitely from the main site.
And I'm I'm spiced on Twitter as py for sort of obscure historical reasons.
We will have a link to that and all the academic or research papers that you've mentioned in the show notes.
So Jonathan also thank you very much for talking to software engineering radio.
Thanks for having me on the program.
Thanks for listening to software engineering radio.
SEO radio is an educational program brought to you by a whole site Europe.
If you want more information about the podcast and the other episodes visit our website at s c-radio.net.
If you want to support us, you can donate to the SEO radio team via the website or you can advertise for SEO radio.
For example, by clicking on the dig ready delicious and slash dot buttons or by talking about us on Twitter and Facebook.
You can also support us by joining team and shouldering some of the work.
To contact the team, please send email to team at se-radio.net or if your feedback is specific to an episode,
please use the comments facility on the website so other people can react to your comments.
This episode of SEO radio as well as all other episodes are licensed under a creative comments 2.5 license.
Please see the website for details.
