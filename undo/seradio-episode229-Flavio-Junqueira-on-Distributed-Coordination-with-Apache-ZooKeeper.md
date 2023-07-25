**Link**:
https://www.se-radio.net/2015/06/episode-229-flavio-junqueira-on-distributed-coordination-with-apache-zookeeper/

This is Software Engineering Radio, the podcast for professional developers on the web at
sce-radio.net.
SC Radio brings you relevant and detailed discussions of software engineering topics
at least once a month.
SC Radio was brought to you by IEEE Software Magazine, online at computer.org slash software.
Blavio Jumkeira, thanks for coming on Software Engineering Radio.
Yeah sure, nice stuff to you, Jeff.
Thanks for inviting me.
Yeah, so today we're going to be talking about Apache Zookeeper and I guess you could call
it distributed coordination.
So why don't we start off by talking about what is the purpose of Apache Zookeeper?
What problems does it try to solve?
This is a very interesting question because Zookeeper is related to what I would say
meta operations of a cluster.
So you have a distributed application, it has a number of servers running your application.
And there are a number of operations that you have to do that they're not necessarily
related to the application directly but you need to do them to get your cluster going.
For example, you might need to elect a master among a group of servers and the other end
the user is not going to note that there is a master but for your application to run correctly
you need to elect a master.
And so Zookeeper helps you with those kinds of tasks.
And when I say animation clusters, you can think of clusters of pretty large clusters,
say hundreds or thousands of servers, trying to do this kind of coordination.
And although I use master election as an example, there are many of these tasks that are sort
of common when you're trying to implement a distributed application.
You can think of things like group membership, you want to know which members of your cluster
are actually up.
So if you have hundreds of servers running, it might be the case that a small fraction
of those are not up at a given time.
You'd like to know who they are so that you can make assignments accordingly.
A pretty common use of Zookeeper is also metadata management.
So you need to store some metadata reliably, you need that metadata to be distributed across
the cluster in a consistent manner.
And so Zookeeper helps you with all that.
And so there's this A and B and B tech talk that I watched about Zookeeper.
It features Patrick Hunt and I'll put it in the show notes.
But so he talks about Zookeeper and he frames his discussion in terms of fallacies about
distributed computing.
And these sorts of fallacies that he listed were like, you know, you have a reliable network.
There is zero latency, infinite bandwidth.
You have no problems with security.
There's no changing network topology.
And all these things are not actually true in your typical distributed system.
There's some kind of problem that violates one of the fallacies that I just mentioned.
So is it reasonable to look at Zookeeper as a patch for these fallacies?
Like a sort of way of sort of eliminating them.
So you don't have to think about them as much.
The way I look at those fallacies is what developers end up doing in their code when
they try to implement the tasks I was mentioning before in a quick and dirty manner.
So it's not that people actually make those assumptions that the network is reliable.
There's zero latency and so on.
But if it's not your main goal, right, to implement those tasks, if your task is implemented,
I don't know a crawler or a data store or something, odds are that you try to do these
tasks like master election group membership and so on in a quick and dirty manner.
And so you end up simplifying your code and having the problems that those fallacies are
actually alluding to.
I prefer to look at Zookeeper, not as a patch for those things, but more of a structured
and natural way to look at the problem.
So it gives a way to developers to reason about these problems in a much more natural
way which leads to more robust implementations.
So let's like kind of think about, I kind of want to discuss it from the point of view
of a typical user.
So for example, let's say I'm a user that has designed some application that is, like
you said, that treats these issues, these fallacies as just nonexistent.
And let's say I built the way that I built my application was quick and dirty and I wasn't
really considering the fallacies.
So how am I going to start using Zookeeper to solve these problems?
So I have this collection of distributed systems problems and I want to sort of collect all
of them in a single context or look at them through a single lens.
And I guess that lens would be Zookeeper.
So how do I start using Zookeeper to manage those issues across my application?
There are two points for beginners I would say.
One is to look at the API.
The API is actually very simple.
The core API is very simple.
The six calls allows you to manipulate the small files that we call Z-nodes.
You create them, you can read them, you can change their data.
And once you're familiar with the API, which shouldn't take very long, it's again only
six calls, it's good to go and look at recipes.
So recipes are this way, these ways of using the API to implement the various tasks that
people care about synchronization primitives like locks if you're trying to do master election
that has been a number of implementations out there of master election on top of Zookeeper.
And so I think that this is a pretty good way of going about it.
So understanding the API or at least have a cursory understanding of the API and then
go learn from the recipes.
So there is just a lot of experience out there.
So let's say for example, like I'm developing a game and the game has shot up in popularity
and I wasn't prepared to scale and I'm starting to scale now.
And so it becomes this huge distributed problem.
And I realize I want to use Zookeeper because I've heard of it as a solution.
So could you maybe outline what a recipe would look like or what am I going to start?
So I have, you know, Z-nodes and Zookeeper.
You know, what am I going to start to be doing with those Z-nodes?
Yeah, so you can, for example, have metadata of a user, start Zookeeper.
In fact, for all your users, you can have, say, one Z-nodes containing their metadata.
And the metadata could be the profile of the user plus the scores of the user or the status
of the user for a given game.
You know, what level the user is at some point or how many points it has scored up to now.
So that could be a way of doing this.
Other things, and to be very honest, I've never tried to apply Zookeeper directly to
games.
I'm coming up with this on the fly.
You could think of assigning one server to handle one server in your application, not a Zookeeper
server, right?
So one of your application servers to handle the traffic of a given user.
So that could be the master for that user, right?
And if you want to make your game fault-orient, if the master for that user crashes, you don't
want your user to be offline as a consequence of that crash, right?
So you want to go and allocate another server to deal with that user.
And that's where the fault-orient master election that Zookeeper provides kicks in.
And so, just to get into a specific question, when you're putting data in these various
nodes, and these nodes have to be consistent with one another, how is the data being propagated
from a Z-node to Z-node?
Maybe this is a question that would be more appropriate for later in the conversation,
but I mean, is it sort of like a gossip type protocol or how does that work?
So let me start by saying that in a Zookeeper ensemble, so ensemble is the name we give to
the group of servers that is serving a given Zookeeper instance.
So when you connect, you're a client of Zookeeper, right?
So you have an application that binds to the client library and it connects to an instance
of Zookeeper.
That instance is, well, it could be a standalone, but typically for production use, you have
a replicated setting.
And so that set of replicas, we call it an ensemble.
So each of the servers in the ensemble contains a full copy of all the Z-nodes that have been
stored in that instance of Zookeeper.
So what we do is whenever you come and say, create the Z-node, what the service is going
to do is it's going to propose that we create, that the ensemble creates that Z-node.
And in the case that there is agreement on the creation of that Z-node, then all servers
are going to record it.
And so all servers in a Zookeeper ensemble will contain a copy of that Z-node.
Now the specific protocol we use is not gossip.
We use what we call an atomic broadcast protocol.
So it provides stronger guarantees.
So guarantees that all the updates that apply to those replicas, they come in the same order.
It does not guarantee though that they come at the same time.
So we do not guarantee that they come at the same time.
We are guaranteed that they come in in the same order.
And so all those replicas are going to apply those state updates.
State updates being created as a node being deleted, the data being changed.
So those are all state updates.
And so with this protocol, all those state updates are going to be fed to all those servers
in the same order.
Sure.
OK.
And so I guess, zooming out a little and going back to the high level, let's talk a bit
about what is and what isn't the Zookeeper.
So I'm trying to go back and forth between perspectives of people who are a little more
familiar with distributed systems and maybe familiar with Zookeeper and then people who
are less familiar with it.
And so maybe for the people who are less familiar, maybe we could put a little more of a definition
on what is and what isn't Zookeeper.
Like is it a database?
Is it a file system?
Is it a key value store?
How should a developer who comes into a new company and sees, OK, we've got this zookeeper
thing running.
What am I supposed to think about this thing?
So that's why we sort of coined this term coordination service because it shares properties
with all the systems you mentioned, like file systems, key value stores, databases,
even.
But there's none of that.
It's, again, it shares similarities, but it doesn't, I don't think we can say that Zookeeper
is any of those.
So for example, if I think of key value stores, updates, so you can think of Zookeeper as
a Zookeeper has a path.
So you can look at that as a key and the data of the Zooke can be seen as the value for
that key.
But in a typical key value store, you don't try to order updates across keys or you don't
even try to organize to provide a hierarchical organization as we do with C nodes.
And this hierarchy turns out to be to be important because one of the calls that we have is
get children.
So you get the children of a C node inside the hierarchy.
And that turns out to be very important in many of the recipes that we have.
So in the end, it shares properties with all those classes of systems.
And I think you really deserves a name of its own or a class of its own.
So a distributed coordination system is what you would call it.
Exactly.
And in the case of Zookeeper, we also call it a coordination kernel because in reality,
Zookeeper does not expose any of the primitives themselves.
So if you look at the API, you're not going to see anything about mass re-election.
You're not going to see anything about logs.
You're not going to see anything about group membership.
It's about what you can build on top of it.
So the recipes are a very important part of Zookeeper.
But Zookeeper itself only provides the kernel that allows you to implement all that.
So because those things are less exposed, things like leader election, does that make
it like once you start to scale a system with Zookeeper and maybe you want to loosen certain
requirements or tighten certain requirements?
Are there things that are sort of kept from the user because of the simple API that make
it maybe more difficult to fine tune or perhaps to scale?
Or is that not really an issue?
I think it's actually the other way around.
I think the fact that we have an API that does not expose any of those primitives directly
gives the developer more flexibility.
One good example is mass re-election.
There are at least two different ways of implementing mass re-election that has been used in our
cross applications.
So if we were to implement mass re-election in a given way, there would be a single way
of doing it.
And because we have this core API that you can build your recipes with, you can think
of different ways of doing this tasks.
Now with respect to scalability, scalability limitations of Zookeeper are actually related
to the core protocol, which is fairly fundamental.
And I believe any system that implements this kind of coordination would have the same limitations.
But with respect to the API, I think what we did in Zookeeper is actually a great thing
because it gives flexibility into the implementation of primitives.
Okay I understand.
So you're saying that the things that you do need to access in maybe the underlying API,
maybe beneath the API layer, they essentially are accessible but it's just you have to get
to them through the API.
Yes.
Okay.
So let's talk about some specific use cases.
What are some ways that you have seen Zookeeper deployed?
So when you refer to use cases, I can think of the actual recipes that people have used.
Well the examples I can think of with respect to recipes are mass re-election, which is
pre-popular, group membership, metadata management, and a lot of the synchronization primitives,
I don't know, like logs or barriers, or if you're just trying to do conditional updates,
right?
So you have multiple clients trying to update the same Z-node and you want to make sure
that the Z-node is updated in an ecosystem manner.
This is fairly abstract, right?
So I'm just giving you some tasks and the recipes that people have implemented.
Now if I go one level up and think about the actual applications that have used, two applications
that are fairly popular and come to mind are data stores like K-space and Hadoop in both
HDFS and Yarn, Zookeeper has been used for high availability.
So for example, for mass re-election and for reliable store.
So okay, could you maybe provide like higher level, I mean I definitely understand what
you're saying, but maybe you could provide a mapping from some recipes to maybe like
business side use cases.
Sure, so let's pick mass re-election, okay?
So let's say that you have a master for your file system, like the name node in HDFS, right?
Let's say you want to elect a leader.
The simplest way of electing a leader is to establish the path for a master lock, right?
Let's call it a master lock.
So you go to Zookeeper and say create slash master lock.
It doesn't have to have any data at this point, right?
Create slash master lock.
And you could have many clients actually trying to do this, right?
Zookeeper again replicates, right, this right.
And all the replicas see this update in the same order, only one of those clients will
be able to create this Z node, right?
So one is going to succeed in creating that Z node, right?
And the one that gets okay becomes the master.
And so he holds the master lock at that point.
All the other ones will won't be able to create.
They will get an error message and this Z node already exists.
And at that point they know that they are not the leader, they're not the masters.
And so they have to follow the master or watch the master in the case the master goes away.
But then the creation of this Z node is not enough to make your master election fault
aren't, right?
The other thing you want to do is you want to create that Z node with the ephemeral flag.
The firmware flag is a very interesting feature of Zookeeper because it allows that Z node
to be deleted in the case the client goes away.
So say the client crashes, Zookeeper is going to determine that because the session of that
client is going to expire and it's going to automatically delete that Z node.
So although the other clients that are interested in becoming masters, they will be watching
that Z node.
And when it's deleted, they will know that it has been deleted and so they will run another
round of master election, right?
So I have done all that with a single Z node.
And the features I have used again is one API which is to create the Z node.
I created a Z node as an ephemeral Z node.
And the other clients that are not masters, they use the get data Z node to watch the
master lock.
And they get a notification back when the master lock is deleted.
That's one example of a task that you can implement with Zookeeper and it's fairly popular
and you can easily map that to a use case like the one I mentioned, you have the head
of a file system or of a data store like HBase and you need to make that fault warrant.
Okay, that makes sense.
Now that would be relevant on any distributed database type of system, I guess, whether
your Facebook that's making a database of billions of people or some online store that's
making a database of all kinds of products and you need to keep the things consistent.
You want one user to be seeing the same sorts of stuff that another user is seeing.
That was sort of what I meant by the higher level business case, but I think I understand
the mapping.
So how big of a cluster do you typically see Zookeeper deployed across?
How am I scaling my Zookeeper cluster from 10 users to 1000 users to a million to a
billion?
How does that look?
I have seen clusters with thousands of clients.
We're talking to the same Zookeeper Ensemble, Ensembles of say 5 to 7 servers.
The limitation there is mostly the amount of traffic you want to impose on Zookeeper.
But the short answer is that you can have clusters of hundreds, two thousands of clients connecting
to a Zookeeper Ensemble of say 5 to 7 Zookeeper servers.
That's a pretty typical case.
And is it accurate to say that Zookeeper is strong for systems with lots of reads and
fewer writes?
Yes, yes indeed.
So writes, as I explained before, writes required to run this atomic broadcast protocol,
which is essentially writing that state update to all replicas.
In fact, we just need to wait for a quantum to respond, but we do send the right to everyone.
But the reads are served locally.
So if I add more servers to my Ensemble, then I can definitely increase my read capacity.
Not my write capacity though.
So it's a system that is highly targeted to read dominated workloads.
Although it does pretty well on writes as well.
We can easily get these days 20,000 writes per second with the implementation we have.
So it's not that the volume of writes has to be small.
It's just that we cannot really scale it at this point.
So if you go, if you want to go from 20,000 to say 100,000 and eventually 2 million, then
there's no way in Zookeeper today to actually do this.
But for the reads, it can increase as much as you want.
So how do we have a problem with this?
Have there been users that have said, oh, we actually can't use Zookeeper because it
doesn't let us write aggressively enough?
Do people end up using an alternative service if they need some amount of writes like that?
The interesting thing is that most cases I have seen where people were requesting exactly
what it said.
So I want to be able to scale write workloads.
Zookeeper is actually not a good fit for that.
So a good, you know, at a high level, good use case for Zookeeper is one in which the
writes to Zookeeper are not in the critical path of your application.
So if everything that comes through your system, every change that comes through your systems
within to Zookeeper, that's probably not the right thing to do.
You probably need some sort of data store, some key value store to handle that kind of
workload.
Zookeeper is really for the meta operations I was saying before, I was mentioning before.
And so those operations, they, so often what happens is you have one write to say some
metadata element, right?
So let's say that you have some metadata that your cluster needs to know, right?
So you update that metadata, you know, take the value of those variables and by updating
some z-node, right?
But you expect all the servers in your cluster to read it.
So that translates into one write and say thousands of reads, right?
So this is a fairly common use case for Zookeeper.
Sure.
Okay, so let's look at a little bit of history of Zookeeper.
So Zookeeper began at Yahoo and eventually was moved to Apache as part of the Hadoop
project.
So I'd like to talk a bit about why Zookeeper was so important to Hadoop.
That's a very interesting question.
In fact, it was, it wasn't important to Hadoop at the time.
I think we ended up as a sub project of Hadoop because the Hadoop community was, was in general
supportive of the project and there was also this perception that, you know, HDFS and Zookeeper
were supposed to have a similar relation to, to a GFS in Shelby.
But Zookeeper at the time was not, was not used by Hadoop at all.
Okay, so, but eventually it's become an important part of Hadoop, correct?
Yeah, but only after, after we left, after it became a top level project that some time
later, it was, they started using it for, for a, a name, no, they, and today you have
the yarn as well.
Reliance Zookeeper.
Okay, could you maybe give some color on the integration of that?
Like, how is Zookeeper the best solution for that purpose of Hadoop?
So if it's the use cases I, I mentioned before, so master election, historian, metadata reliably,
some of the things that they are looking for in the, in this, in those use cases.
And so Zookeeper is, is in that ecosystem.
It's, it's a very stable project, you know, it's battle tested.
So it's, it's definitely a good, a good element, a good component to, to being their role.
Okay, that makes sense.
Let's, let's drill down a little bit into the, the different roles in a Zookeeper implementation.
So you have the roles of followers, leaders, well, a, a leader, at least clients and observers.
So let's talk about the, the abilities and the responsibilities for the roles.
Sure.
Let me start with clients.
So clients are connected to a single server at a time, right?
So even if you have many, multiple replicas, you have, you have replicas of the system,
the client's going to connect to a single server at a time.
So out of those servers, one is the leader, as you mentioned, and the others could be
followers or observers.
The leader processes all requests that change the state.
So if, if a client submits a, you know, say a creates a node, that's going to go to the
leader, the leader is going to produce a transaction, which is a, a state update.
It's going to propose that to the followers, the followers are going to persist that on
this, they're going to acknowledge the, the leader, and then the leader is ready to come
in.
Fours are different from observers.
So observers do not vote in that process.
I, I just mentioned observers, they just learn the state changes, right?
The, the state changes have been committed.
And by doing that with the main thing you achieve is that we can increase read capacity
without actually increasing the work the servers have to do for, for a given state change.
What else can I say?
So going back to the leader proposing a state change, when you receive the response from
a majority, then it's ready to respond to the clients.
And at that point the client receives, you know, he receives a response back and that's
more or less how the flow works of, of requesting Zucheber.
Okay.
Yeah, sure.
So that makes sense.
So there's a specific algorithm that the consensus protocol for Zucheber is based on.
Like, is it, is it sort of like a Paxos type of consensus or is it maybe like based on
RAF or is there not a close relationship between the way that Zucheber maintains consensus
and the way that these things are outlined in academic papers?
Right.
Okay, so the protocol we use is called ZAB, which stands for Zucheber Atomic Broadcast.
It has elements that differ from Paxos, although you can go through this exercise of mapping
Paxos somehow to ZAB.
But by reading the two papers you notice that there are differences.
And the difference is mainly because of the way we have structured the components of the
service leaders and followers by propagating the state updates and having this dependence
between the sequence of, between state changes in the same sequence, which sort of forces
to stop the consensus protocol during a leader change so that we could guarantee the consistency
of the service.
So it is a protocol that again, that's called ZAB.
It has many elements in common with, with REFT.
REFT actually came a few years later and yeah, that's the protocol we use.
But again, both REFT and ZAB have elements of earlier protocols like Paxos.
I'm sorry, could you repeat again with the differentiating requirement that incentivize
you to make your own protocol?
We have this need of, when we do a leader change, we need to have agreements on the
initial state of the new epoch.
Okay, so the protocol is structured in epochs.
Right, so a leader comes, proposes a bunch of things, a bunch of state changes, those
are committed.
And at some point that leader goes away, he crashed or he got suspected of crashing,
and the new leader comes.
So when that new leader comes, before he can start proposing new state changes, he needs
to agree with the other servers, what the initial state of the new epoch is.
And so that required us to come up with a different protocol.
Okay, yeah, I understand.
So I know that implementing anything in a distributed system can be hard or implementing
a distributed system can be hard specifically.
So how did you perform validation and testing on zookeeper?
And how did you achieve a level of confidence in the correct behavior?
Because anytime you're trying to implement a distributed feature of something, if you
want to test it, there's always sort of the question of if I test this on three servers,
is that equivalent to how it operates on five servers?
Is that equivalent to how it operates on 10 servers?
And so on.
So how did you do this validation and testing?
It's a combination of reasoning and writing real tasks.
So we did a lot of reasoning about not only the code, but the protocols behind it.
And we have a lot of tests.
So in Apache in general, we are on generalized Apache.
In zookeeper, we use extensively JUnit tests to test cases that could be problematic or
they have been problematic in the past.
And then in time a change goes in, it's checked and we have to run those tests to make sure
that it's not violating or it's not breaking anything that's already in there.
And there is also the test of time, right?
It has been running production in a number of places for a long time.
This is not to say that there is no bug, right?
Or there has never been bugs quite the other way around.
We always find small things that we can fix.
And that's part of the work of the community, fix this bug that we find every now and then.
But in general, I would say that the customer zookeeper are really happy with it.
Over the years, we have heard lots of good comments about this, the big of zookeeper and
how it runs and so on.
Are there any known edge cases that result in failure?
Edge cases that result in failure.
So could you be a bit more specific about failure?
I guess I was unspecific on purpose because I wanted to see if that evoked any, is there
anything that happens frequently maybe because it could be just because people tend to implement
something in an erroneous way and that leads to a vulnerability at an edge case or maybe
it's something that is inherent in zookeeper that leads to an edge case.
I'm just curious as to how you would define failures and the edge cases that would result
in those.
Right.
Well, I guess I can see failures in two ways.
One is, for example, if we ignore the notifications that zookeeper sends up to the application,
right?
So one example is if you implement your master election, let me actually step back and talk
about sessions a bit.
So when a client starts talking to an ensemble, he has to establish a session, right?
A session can live through multiple connections.
So initially, the client connects to a server, but later on, that client may move that session
to a different server, right?
It disconnects from the first server, connects to a new server, but the session is still the
same.
If you implement a master election, what we strongly recommend is that if you disconnect
from a server, right, and you receive that notification from the client library, the
master should stop doing any kind of master work because at that point, it doesn't know
what the state of the cluster is.
It might be the case that each session has expired, right?
Or it's going to expire soon.
In that case, it could be, you know, its decisions or its steps might be conflicting
with the steps of the new master, right?
And this is something that is problematic.
So the bottom line is ignoring the notifications that the client library sends up is a bad
idea and doing that could lead to, to say, bad cases in your application.
That are figures on, say, the operational side of Zookeeper.
Zookeeper today is sensitive to losing disk state.
It heavily depends on servers not losing disk state to guarantee the consistency of the
data.
This is not to say that there is no way around it, so if a server loses its disk state, then
you need to remove that server from the ensemble and reinsert it to guarantee that there is
no consistency problem.
This is a sort of a deeper argument into the protocol, but at a high level, this is what
happens.
And so if I'm a user that's implemented in my Zookeeper system, do people typically
do fuzz testing or, you know, like is there a barrage of random and unusual events that
you can just impose on the system as a means of testing it?
As I mentioned, we run JUnit tests, right?
And that we are continuously running it, and every time we check in that we also need to
run it before it's checked in.
Users of Zookeeper tend to have their own set, especially the big customers of Zookeeper
tend to have their own set of tests.
At some point, we talked about fault injection.
This is not present in Zookeeper today.
So I think the short answer to your question is that we don't have anything like your
suggestion, although we have various test cases and test suits that various customers
run on Zookeeper distributions.
Okay.
I want to talk about the CAP theorem.
What is your definition of CAP, and do you actually find it useful?
Do you follow it as a design principle?
And could you elaborate on consistency, availability, and partition tolerance?
Right.
So the CAP work is a bit old by now.
In the past few years, a few people have actually criticized it, but trying to uncover some of
the dangers of using the CAP definitions loosely.
On my hands, I would say the CAP has always confused me a bit, mainly because it defines
terminology in the way that is very different from the way I typically use.
So if I think about Zookeeper, I would actually say that it's consistent, it's available,
and it's partitioned on.
It's consistent because realizing this atomic broadcast primitive, which makes sure that
the state updates are totally ordered and delivered in order to the server replicas.
It's available because a single crash or a minority of the server replicas being unavailable
is not going to cause the whole system to be unavailable.
So availability is about the total amount of time that the server says a whole is up.
And finally, it's partitioned tolerance because it does not admit split-brain.
So split-brain is a situation in which two or more parts of the system end up making
progress independently, which causes the state in which one of those parts to diverge when
compared to the others, which is clearly bad, at least in the case of Zookeeper.
Now, if going back to the CAP, to the CAP work, CAP, as many people expose, is essentially
says that we have just three part properties and when our building is a disability system,
we pick two.
So CP, CA, or AP, I would say that CA doesn't make a lot of sense because CA essentially
assumes that partitions do not occur.
Well, I'm not sure how one can control whether partitions occur or not.
I think the best one could do is to assume that partitions won't occur in the case they
occur.
You just suffer the consequences.
But I'm not sure how much one has to gain by just doing this.
It doesn't sound like a great way of thinking about systems.
Does Zookeeper fall cleanly into any of the models provided by CAP, CP, CA, or AP?
Be strict about the CAP definitions.
Zookeeper is reading none of those.
So the workload isn't necessarily linearizable because the regular reads, so the basic reads
we have, they do not make the workload really linearizable, so it's not consistent.
And it doesn't accept split brain.
It doesn't admit split brain, so it's not available either.
But a quorum-based systems, replicated state machines, which zookeeper is an instance
of, typically falling to the category of CP, consistent partitioned owners.
So that's really the closest to Zookeeper.
How useful is CAP when it comes to trying to reason about systems?
One of the great benefits of CAP is that it provides a common terminology for people
to talk about their systems.
But I really think that is great on just our conversations.
So a UCP person or a UAP person.
But thinking deeply about systems and its relation to CAP seems to be pointless to me.
Now I think the CAP work is very relevant.
So if you read the CAP proof, the CAP theory improve, it's very interesting.
It provides a lot of insight.
But it's a really narrow result, although people keep using it in a very broad manner.
Just to give an example, if I think about consistency, if I pick the definitions in the
disability systems by Andy Tanningbaum, so over that there are over 10 definitions of
consistency.
So narrowing it down to just linearize a biggie doesn't sound very wise to me.
Although granted that linear is a big, it's a very important property for many of the systems
that we build and use.
Would it make sense to extend the CAP theorem to try to define more finer-grained classifications
for systems, because these three dimensions, C, A, and P, consistency, availability, and
partition tolerance, they don't seem to capture the richness of systems these days.
I mean, maybe they do provide a framework in which you can write a proof.
But maybe that's not as useful as having a more rich terminology.
Yeah, as I said, I think one of the interesting and nice things about the CAP work is that
it has achieved to create a common simple terminology that developers of disability
systems have been used, have been used into discuss our projects.
I don't think everyone doing disability systems fully understands the implications of the
CAP definitions though.
And so if you are to come up with something different, I think this new thing should have,
should make it a bit more clear what consistency of the ability and partition tolerance or
whatever other times you come up with are.
So that developers of disability systems builders do not end up using these terms loosely and
misinterpreting what's actually going on just by using terminology loosely.
But at the same time, as I mentioned, I don't think such as small terminology or such a
concise model is great for something else other than just starting a conversation.
I really think that when we talk about systems, there are just so many set of things and
ones as to the things we do that you need to go deeper to understand what needs to be
done, what kind of properties you're looking for and so on.
As I was making this reference to, to consistency properties, there are many consistency properties
one could think of when implementing a system.
And so just thinking about one or maybe two, that doesn't sound great.
So a terminology that allows us to convey in a concise manner, but in a precise manner
as well, what we are trying to do in a project, that wouldn't be a bad idea.
But again, keeping in mind that that's down the road, we need to talk in more detail about
things so that we can think more deeply about what we're trying to achieve.
So how does how does Zucheber guarantee that clients will never detect old data?
Zucheber actually doesn't make that promise.
What Zucheber tries to do is actually to avoid discussions about time altogether.
So the replica is guaranteed that state changes come in the same order.
Also, guarantees that replicas receive the state changes in the same order.
But it doesn't really guarantee that a client is going to read some older data.
So it could happen in a system that at some point in time, two clients read different
values for the same zenode.
But over time, the client, the older data eventually will be able to see that state
update, the newer version of the data.
So Zucheber guarantees the order of updates, but not when they're going to happen.
Right.
And so Zucheber keeps watch over something that sort of looks like a file system in some
sense.
And I'm curious, are the observers costly?
The notion of observing this file system, is it costly?
Are you constantly polling and spending energy?
And this question may lend itself to maybe I have a kind of loose understanding of how
a zookeeper works, but maybe you could address that question.
Sure.
So when a client says, so watch an on a zenode, right?
You say, I want to receive a notification when something happens to this zenode.
And so that request goes to Zucheber and to the servers.
And the server that receives that request annotates that particular path for the watch.
And so when you receive the updates, it triggers the notification to the client.
So the client has to do no work on the watch, right?
Why it's connected to that server?
And when the state change comes in, the server will produce a notification and ship it to
the client.
The burden is mostly on the server side.
Okay.
How horizontally scalable is Zucheber?
If I'm growing, how rapidly can I just like, can I just like throw new servers on Zucheber
and say, hey, rebalance this?
Or is it more of a complicated, difficult process?
If you're trying to scale writes or the amount of state of a zookeeper ensemble, then we
don't have any good mechanisms for doing this today.
What you can scale, horizontally scale in Zucheber today is read capacity, right?
As I was mentioning, you can throw in more observers or just increase the number of followers
and that will increase your big enough doing reads, right?
You can do more reads per second.
Okay.
Makes sense.
So I guess we can start to close off.
I'm curious about where Zucheber is going.
So what is the future?
Are there features that need to be implemented?
Are there changes that need to be made or that are going to be made, new features being
added?
The main feature we have been working on recently is reconfiguration.
It has been made recently available in the 35 branch, which was released in August if
I'm not mistaken.
So that branch is not stable yet and so the community has been focusing a lot on that
feature.
Okay.
For somebody that's getting started on Zucheber, what would you say are the best resources
to consider?
I know you've mentioned the recipes several times and certainly we'll link to some of
those and how to pursue them.
But what are some other resources that people should look at?
And maybe other projects that people should look at.
Right.
So the Apache Curator Project is definitely a very good source of not only information,
but you can actually use the recipes that they have implemented.
So Apache Curator, the proposal of Curator is to implement the best practices known for
Zucheber.
And so you can learn a lot by just looking at the recipes and you can even be a beelase
and just use them directly.
And so it's definitely a good source of recipes, right?
And knowledge about how to program against Zucheber.
Great.
And how can people find out more about you or follow you on social media?
Oh, yeah.
So I have a Twitter account.
It's Fpjomkater.
So that's my hand on Twitter.
I'm also on LinkedIn.
So if you search me for my name, you'll probably find it.
And so those are a good ways of following me.
And I also strongly encourage the ones interested in Zucheber or learning about Zucheber, contributing
to Zucheber, to sign up for the list, for the dev list and the user list.
We always interested in onboarding new contributors, new commuters, and growing the community.
So I would be happy to see more people signing up.
All right.
Thank you.
You're welcome.
Thank you.
Thank you.
Thank you.
Thank you.
Thank you.
Thank you.
Thank you.
Thank you.
Thank you.
Thank you.
Thank you.
Thank you.
Thanks for listening to SE Radio.
An educational program brought to you by IEEE Software Magazine.
For more information about the podcast, including other episodes, visit our website at se-radio.net.
To provide feedback, you can write comments on each episode on the website or right
on iTunes.
Mention our messages on Twitter at se-radio or search for the Software Engineering Radio
group on LinkedIn, Google+, or Facebook.
You can also email us at team at se-radio.net.
This and all other episodes of Se Radio is licensed under the Creative Commons 2.5 license.
Thanks again for your support.
