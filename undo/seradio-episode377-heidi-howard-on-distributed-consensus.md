**Link** https://www.se-radio.net/2019/08/episode-377-heidi-howard-on-distributed-consensus/


This is Software Engineering Radio, the podcast for professional developers on the web at
se-radio.net.
SE Radio is brought to you by the IEEE Computer Society, the IEEE Software Magazine.
Online at computer.org slash software.
Are you looking for a better way to track your engineering backlog?
Do you want a project management tool that's both powerful and a joy to use?
Give Clubhouse a try.
Designed for software teams, Clubhouse is a fast modern alternative to conventional project
management tools that are either too complex or too simple.
Start your free trial and get two extra free months at clubhouse.io slash se-radio.
Hello, this is Adam Gordon Bell for Software Engineering Radio.
Today I'm speaking with Heidi Howard, a research fellow in computer science at the University
of Cambridge.
Heidi, welcome to Software Engineering Radio.
Thank you so much for having me.
So today we're going to talk about distributed consensus algorithms.
I thought a good place to start is with the basics.
So what is a distributed system?
A distributed system is basically a concurrent system, but even worse.
It's a system where multiple things are happening at the same time, but you can also have failures
and asynchronous.
So the nodes in the system can fail and they may or may not come back up.
And we also have asynchronous.
So when we send messages, we don't know exactly how long they're going to take to get to their
destination or even if they will get to their destination.
So it's sort of like multiple instances of the same program running and communicating
over an hour.
Is that?
Yeah, absolutely.
In the context of a distributed system, what does consensus mean?
So consensus means making a decision in a distributed system.
So that could be deciding a value in a register or deciding an ordering of operations to be
applied to a database.
And what makes distributed consensus difficult is the same thing that makes distributed systems
difficult.
That is the asynchronous and the partial failures.
If we had a purely reliable system that was super synchronous, then consensus would be
really easy to solve, but unfortunately, computers don't work like that in reality.
So what does it mean to have consensus?
Is it just agreement?
So consensus has three components.
So the first one is that the value you agree is the value that somebody proposed.
It should be.
So that's kind of the non-triviality component.
There is a safety component.
So when you decide a value that's final, everyone finds out the same value.
And thirdly, there is a progress component.
So that means that if everything's working okay in the system, the system will reach
your decision.
So the system can't get itself stuck indefinitely in a deadlock.
So when do we need consensus in a distributed system?
Do we always need it?
Is there specific use cases?
We're very lucky that we don't always need it because it could be quite expensive in
practice in terms of the number of messages and stuff that's needed.
You need consensus when you have to have strong consistency guarantees.
So for example, if you have a key values store and you need to have linearizability, you
can't just have causal consistency or something like that, you absolutely have to have linearizability.
So it could be holding some really critical data in your system.
It could be data that's for configuration or orchestration where sacrificing consistency
is just not okay.
And you need consensus if you can't rely on synchrony of clocks.
So for example, Google doesn't always need consensus.
They use a system called Spanner.
This uses clocks basically because Google controls the data center.
So they've put atomic clocks and GPS receivers into their data centers instead of relying
on normal time protocols.
And that means they don't need consensus either.
Is that because if they all agree on the exact timestamps, they can put things in order,
even though they take place in different places?
Yeah, absolutely.
They don't rely on it being perfect, but it's very, very close to perfect.
So as long as the timestamps are consistent to within a very small margin, then that's
absolutely fine that the system will run.
And that allows the system to be much more efficient and much more lightweight than a
proper distributed consensus system.
One thing that I find confusing maybe is just consensus and consistency.
They both start with Cs.
And are they the same thing?
Or I'm familiar with the cap theorem, which I believe has to do with consistency.
How does consensus and consistency relate?
Because is the method of achieving very strong consistency.
And how does the cap theorem?
In terms of the cap theorem, when we're looking at consensus, we're getting strong consistency.
So we're not compromising consistency.
We are, however, compromising availability.
So typical consensus systems use majorities, which means that if, say, three participants
in a system of five participants go down, the system is unable.
And so you've sacrificed availability in that case.
So it's more of a, it's a CP system in that kind of set of terminology.
So in a distributed consensus system, it's not possible to be highly available.
Is that right?
I think the comparison usually, because you're getting the strong consistency guarantee,
the comparison is usually between having one server or having three servers or five servers
and using consensus.
So you have better availability than you would have had with just one server.
So it's improved availability compared to one server, whether it's improved enough to
be considered high availability.
I think depends on the situation.
If those servers are in different availability zones, for example, and you can tolerate failure
of one, that seems pretty, pretty highly available to me.
So because you just need the majority up.
So if you scale up to more nodes, then it can still be sort of highly available.
Am I getting this right?
No, you are getting it right.
So if you have three nodes, you can tolerate one failure.
If you scale up to five nodes, you could then tolerate two failures and up to seven, you
could tolerate three and people don't tend to scale further than that, really.
Why not?
Because whenever you're doing something, you need to use that majority of participants.
You need to get the majority participants to basically agree and back you with what you're
doing.
So as you scale the system, the size of the majority goes up and up and therefore the
latency and the amount of messages that you need to exchange to get anything done gets
higher and higher.
It's interesting.
So the throughput of writes, for instance, decreases the more nodes you have because you
make the majority?
Absolutely.
It's the opposite of a scalable system.
For the best performance, you would have one node.
That is interesting.
So at my work, I sometimes work on this distributed system and it has this like, it's a Java program.
It has this like leadership annotation.
So like I can say, like, I need to do something.
I only need to do it.
I need to make sure it happens on startup, but like only once.
And so there's this.
I just have to say, I think it's called like leader only.
I can kind of annotate that.
And that's where I kind of do things like if I need to do some data migration that's
kind of shared data, I will do that there.
What happens in sort of a process like that?
Like what does that mean to say the leader only does this?
What is leadership election?
Okay.
So leadership election is basically deciding who's going to be the person to make decisions
in a distributed sense.
So there are some political analogies that I'm trying not.
I've probably tried not to draw on too much.
But basically the participants in the system decide between themselves who is able to act
on their behalf.
If they decided this every time anything needed to be done, it would take a long time to get
anything done.
So before the system needs to do things, it decides who's going to be in charge, who's
going to be responsible for making those decisions.
That is the leader of the system.
And typically in consensus systems where we're using majorities, the leader backs up all
their decisions to a majority of the other participants.
And then if the leader fails, there's a backup on at least the majority of the participants.
So is it in fact like a voting process?
It's called leadership election.
Is the nodes voting?
Yeah, they absolutely are voting.
They use some mechanisms to basically allow participants to vote multiple times if they
need to.
And these are things like views, ballots or term numbers depending on what academic paper
you're looking at so that they can still make progress so we don't ever end up in a deadlock
situation where half the participants are voted for one leader and the other half are voted
for a different leader.
So I think that kind of gives me an understanding of what distributed consensus is.
So I thought it might be fun to take like a historical perspective on this in part because
I saw a talk you gave that kind of had that feel to it.
So what was the first solution you're aware of to distributed consensus?
That's quite an interesting and contentious question actually because there was quite
a few different people who proposed consensus at a similarish time.
So we're talking late 80s and a lot of the consensus algorithms were kind of buried in
other academic papers, papers about things like transactions and they weren't necessarily
noticed.
The paper that is kind of iconic for pioneering distributed consensus is a paper called the
Part Time Parliament by Turing Award winner Leslie Lamport.
Yeah, he was a guest on SE Radio episode 203.
So listeners can certainly check that out.
So what is the Paxos algorithm?
So the Paxos algorithm is kind of this algorithm that we've been describing so far.
It's an algorithm that has two phases.
In the first phase, the majority of people get together and just elect one of the participants
to be the leader, to be the person who's going to choose values in the future.
And in the second phase, the leader chooses a value and they back that value up to the
majority of participants.
This means that if that leader fails, they can communicate with the participants, the
participants communicate between themselves, sorry, and decide on a new leader.
And as long as the majority are involved in choosing a new leader, we can be sure that
at least one of those people who votes for a new leader will know about what the last
leader did.
You mentioned earlier, Vanner, what systems use Paxos like practical distributed systems?
So Paxos is very widely used.
It's used in Chubby, which is used as a Google system.
It's used in Cassandra.
It's we can talk about Paxos versus Raft in a bit, but it's the same or very similar to
the Raft algorithm, which is used in EdCD, which is used in Kubernetes.
It's very similar to the Zab algorithm, which is in Zugeeper.
It's very, very widely used typically just for coordination and orchestration.
So if I am using a Paxos system and like let's say it has three nodes, like your previous
example and I go, I'm a client and I try to write a value, like what happens there?
Do I have to go to the leader?
Yes, so you'll send your right request to the leader.
The leader will then place it in a kind of sequence of right requests that it's received
and it will then send a message to the other participants saying that they should insert
that right in a given position in kind of the history of what's happened in the system
into their logs.
The other participants will do that right and then once the majority have got back, the
leader will commit that right.
So if it's got like say a state machine there like a replica or a key value store, it will
apply the request and it will return to you an acknowledgement that it's been successful.
And if I try to read that value back, is how does it work where if I only have to write
it to the majority of nodes, isn't there a chance?
If I tried to read, I could access the value that's not part of that majority.
So it depends how you implement the reads.
The proper theoretically, the proper way to implement reads is to achieve consensus over
the reads as well.
So they also get a position inside the log.
The leader will basically check that they still are a leader.
They're still up to date.
They still know everything about what's happened in the system and then they will apply the
read and that guarantees you linearizable reads as well as writes typically in practice
though because systems of generally read heavy.
We implement, we're a bit more relaxed in the way we implement reads.
So for example, you would send the read just to the master and the master will give, send
you back a response.
And as long as that master is still the master in the system, that will be a consistent response.
And what if I have like two clients and they each try to write the same value in some time
span that's very close, I guess?
Yeah.
So from a consensus perspective, both of those writes will be successful and they will happen
in A order.
It doesn't limit what order will be it will happen in, but it will definitely appear as
if one happened, then the other happened or vice versa.
So there won't be, you won't end up with an inconsistent state in the system.
It's definitely the case that all the replicas in the system will see those writes in the
same order.
But linearizability doesn't guarantee a particular ordering over concurrent writes, only over
writes that happen at different times.
Are there cases where a write could fail?
So it can fail if the system is unavailable.
So if there are two out of three participants or down, perhaps you contact the leader, but
the other two nodes are down, the leader isn't able to commit that write, that would be a
failed write.
But as long as the system's up, the reads will get placed in some sort of order.
So you mentioned Chubby, which I think is a Google paper or system.
It's a Google paper about a Google system, yep.
So what innovation did Google bring to the world of distributed consensus, if any?
Google was probably the first company that publicly came out and said, like, we are implementing
Paxos and we are deploying it at scale, here is the story of how we did it, here's the engineering
solutions we came up with to some of the problems to do with implementing Paxos in practice.
And what were some of those challenges and problems?
So some of the problems would be things like we're reaching agreement about a sequence of
operations.
So that sequence of operations just keeps growing and growing indefinitely.
And at some point you need to kind of compact that sequence or you need to snapshot it so
that you don't end up just using an infinite amount of memory.
I heard this term before, multi-Paxos, what does that mean?
So multi-Paxos is actually what we've been describing so far.
So people tend to use the terms Paxos and multi-Paxos anonymously.
So that's the idea that you establish a leader in the system and that leader, sorry, will
go on to make many decisions in the system.
Single degree Paxos or like the vanilla Paxos or classic Paxos is where you have the leader
just make one decision and then next time you elect someone new and you have them make
one decision.
So it's basically you run a kind of distinct instance of Paxos every time you need to
make a decision.
Each of those instances is basically disconnected.
And was this multi-Paxos, like was this Google's innovation or was this in the original paper?
It was in the original paper.
It was a little bit hidden in the original paper, just like Paxos was to be honest.
But Lamport lays it out quite clearly in his Paxos made simple, which is his paper to kind
of try to explain the part-time parliament paper a bit more clearly.
And earlier you mentioned zookeeper and I think some consistency algorithm that it
used, I thought it used Paxos, is that incorrect?
It depends who you talk to, basically.
So zookeeper uses an algorithm called ZAB, which stands for zookeeper atomic broadcast.
That algorithm solves atomic broadcast, which has been shown to be equivalent to multivite
consensus.
ZAB is very, very similar to the multi-Paxos algorithm that we've been discussing so far.
There's a few kind of design decisions that are a little bit different, but the general
idea of electing a leader using majorities and that leader then taking decisions on behalf
of the system is the same.
What is atomic broadcast?
Atomic broadcast is the idea that if you send messages within a system, everybody will see
the messages in the same order, regardless of the asynchronous system.
So the reason it's the same is what you were saying the leader orders them and then distributes
them out to the nodes?
Yeah, absolutely, so if you had a network that gave you atomic broadcast, you could implement
consensus really trivially by just allowing anyone to send their operation as a message.
And then everyone would see exactly the same set of operations in the same order.
Interesting.
You mentioned EdCd and Raft.
So is Raft a variation on Paxos or something completely different?
Again, it depends who you ask, my perspective is very much that Raft is a really good, well-articulated
description and simplified version of Paxos specifically designed for using with state
machine replication, which is by far the most common application of Paxos in practice.
I know some people talk about Raft and Paxos as if they're completely separate and completely
different algorithms, that's definitely not a view that I would agree with.
But Raft is definitely a very nice paper, it's a very nice algorithm and it was in fact
my introduction to the field of distributed consensus.
So how can it be a different algorithm and be the same?
The devil is in the detail.
The basic kind of core idea is the same.
And because part-time parliament was so underspecified, you can basically make the case that it's
the same as almost anything that you come up with because it was quite vague in what
it said.
The good thing about Raft is that it's quite specific unlike a lot of the theoretical
computer science work.
So Raft is about replicating state machines and agreeing an order of operations to pass
into those state machines.
And it uses lots of examples of key value stores.
And I think that made it the famous paper that it now is.
So is it a good starting place if somebody wanted to understand these algorithms?
It sounds like it's just better specified?
I do think it's a really good starting point for understanding the basics of Paxos.
I think that people do struggle after reading Raft as to where to go next.
And if I needed a recommendation for that, I would say there's a good paper called Paxos
made moderately complex, which describes Paxos and some of the big famous variants to the
Paxos algorithm as a survey or overview of what's out there.
And from then you could then bootstrap into consensus algorithms, different types, different
families of consensus algorithms depending on your needs.
Yeah, I still find this terminology confusing because so EtsyD says it uses the Raft protocol.
And then I think Cassandra says it uses Paxos, but you're saying these are the same really.
It's more of a implementation perspective.
Yeah.
Yeah.
So are there any variations on these algorithms that should be preferred in certain situations
versus others?
Yeah, definitely.
So there's a different kind of family of algorithms.
The main algorithm in that family is called Fask Paxos.
And this group of algorithms basically uses bigger quorum.
So instead of using majorities, it might use say two thirds of participants, but it removes
the leader from the system.
So it goes from a centralized approach to a decentralized approach.
And that family has other algorithms in it like a generalized consensus or a egalitarian
Paxos.
And those are good algorithms if latency is really important to use.
If you were implementing consensus across data centers, that's definitely something I
would recommend.
So if we were implementing it across data centers, would we still want to keep a low
number of nodes?
I think before you said never more than five or something like that?
Yeah.
Yeah.
So using majority quorums, it's the system's only going to get worse, the bigger you make
it.
So I would say five is a good number.
Some people do three, some people do seven.
I don't really see people using nine and above.
If you did want to span more nodes, then some of the work that I did a couple of years ago
is known as flexible Paxos.
And this is about allowing some flexibility in Paxos algorithms, as it says on the tin
by allowing you to use different quorums.
So instead of using majority, you can use different group sizes.
And that allows you to then scale up consensus more than if you were using majorities.
So this is your contribution to the field.
How is it different?
What is the advantage of being able to choose a different majority?
So Paxos gives you one option in terms of trading the performance of the system against
fault tolerance.
If you use majorities, you can tolerate minority failures.
But if you had say seven, a system of seven nodes, what you could do instead of using
a quorum of four, which is the majority, you could use a quorum of three.
This is smaller quorum, which means there's less messages and it can be more performant.
In some cases, it may tolerate one less failure.
So it would only tolerate two failures instead of three, but depending on the system, you
can get some really interesting performance gains from that.
So what Flexile Paxos does is basically open up Paxos and say you can choose that trade
off that you want to make versus Paxos.
So this is the one trade off that you have and this will fit all systems.
So is this algorithm in usage?
Yeah, yeah, absolutely.
So I've seen various kind of discussions of it.
People have implemented it.
So there was a master student that implemented it into zookeeper and there, yeah, if you
Google it online, various other people have implemented it.
There's a team that are using it to do a geo replicated consensus.
Their algorithm is called W Paxos.
Yeah, so lots of, I implemented it myself as well.
I built it on top of implementation called libpaxos.
Although that's predominantly for kind of benchmarking and testing.
Interesting.
We've been talking about these consensus algorithms, I guess, with the assumption that they work.
Like how is that demonstrated?
That's a really good question.
So it's quite hard to reason about what's going on with these algorithms because they
really are quite subtle.
The tool that I use the same tool that a lot of people in the field use is known as TLA
stress.
It was also invented by Leslie Lamport.
So shockingly it's good for expressing consensus algorithms.
And what it does is it allows you to write the algorithm and then it allows you to model
check that specification of the algorithm.
So you can come up with a property, say, at most one value is decided in any slot in
a sequence.
And then you can check for a kind of small configuration that that's true regardless of
what happens in the system.
That's quite a nice way of getting you to think carefully about the algorithm and reasoning
about edge cases.
But it is limited in the sense that you model check for a specific number of nodes.
So you check for five nodes or for seven nodes.
If you want to check more comprehensively, you could move on to using a theorem prover.
So there's T-lapse, which is a theorem prover over TLA+, otherwise there's theorem provers.
The one developed by Cambridge University is called Isabel.
And there's another one, this written in O'Kamel, which is called GOG.
And they allow you to prove properties of consensus algorithms.
This again is very nice.
It allows you to reason about edge cases.
But you then still have to hand over the algorithm to engineers to implement.
And then, you know, who knows, that's implemented correctly.
And so I'm a huge fan of the work of Kyle Kinsbury, which is probably familiar to some
of your listeners who are interested in databases and his Jessup system, where he tests various
databases and key value stores.
And he tests them for linearizability.
And that's a really good, really good work in terms of checking whether these algorithms
actually do what they say on the tin.
And he's tested Zukiper and Edsudio, I believe.
Yeah, I think that he was a guest.
I don't have the number, but I'll find it.
So what he does is actually real world testing.
Is TLA plus like a simulation?
Is that how you would describe it?
Or is it a proof?
It's, I guess it's kind of like a simulation.
So you write the algorithm in the special TLA plus language.
And then you basically, you model check, which is a simulation, but it's just exhaustive
simulation, so you check all possible scenarios.
It's quite computationally intensive, as you can imagine.
Oh, an interesting thing.
I was noticing I was looking at some benchmarks online of Zukiper versus Edsudio versus a
couple other systems, and then looking at them versus like a more traditional key value
store.
And they seem to be not, they're obviously slower because consensus has to be reached,
but not too bad.
But what I noticed is there's big spikes in like latency, like occasionally a write takes
a long time.
I wondered if there was something to do with reaching consensus that can cause like unpredictable
times for updates.
Yeah, so there's definitely stuff that can go wrong in practice that could leave to a
high latency.
So probably the most common would be a failure of the leader or just a suspected failure of
the leader.
They take a bit of congestion in the network and the other participants accidentally believed
that the leader had failed, and they go on to try and elect a new leader that can take
quite really quite a long time.
And it's quite funny because, you know, no failures of actual, of actual participants
in your system need to occur for your system to actually believe that there's a failure
and then suffer the high latency as a result.
Yeah, earlier you mentioned like multi data center consistency.
It seems like that would be a challenge with with the latency and possibly like losing
contact.
Is this being done in practice like consistency across data centers?
Yeah, it is.
It is.
So Google mega store is packs of influence packs across data centers.
It's it is hard.
It is tricky.
What usually happens is that because we don't want to accidentally believe a leader has
failed, we set timers for how often we heartbeat the leader and how long we're willing to wait
before we start reporting this leader is having failed.
We have in a geo replicated setting, we have to set those timers quite high to stop those
false failure detection detections.
The downside of this of course is that when a failure does happen and the leader does go
down, the system will wait a long time unnecessarily just to be 100% sure that that leader has gone
down.
Well, 99% sure you can never be 100% sure if anything can distribute it systems.
So you mentioned time out in a distributed system.
I think I I saw in a talk of yours something called the FLP result, which I believe showed
that reaching consensus could be impossible.
Is that right?
Yes, that's right.
So quite entertainingly, the FLP result was basically the paper that founded the field
of distributed consensus.
So paper that said this thing is impossible led to decades of researchers trying to prove
the authors wrong.
And what that paper says is that it's impossible to guarantee consensus in a system where even
one participant might fail, which is interesting because it doesn't even say one participant
fails.
It's just the possibility that a participant might fail.
You can't guarantee consensus.
So that means you can't be sure that consensus will happen within a bounded time period.
So I don't get then how what we're talking about.
If it's impossible, how is this resolved or was it?
We don't get guaranteed consensus in practice.
In practice, we do things like we have we use timers and we set them quite high and we or
when we send a message instead of we use things like TCP and we say we just assume the message
will get there within a year.
And that allows us to circumvent the FLP result in practice.
And why does the FLP result suggest that it's impossible?
It's impossible because we can't guarantee within like a fixed time period that we will
be able to get consensus.
But we can't definitely say it is going to happen in this amount of time, which makes
sense when we can't even guarantee that a message will get to somewhere within a fixed
amount of time.
So we definitely can't guarantee that many messages will have got to many different places
within a fixed amount of time.
So is it related to this idea?
Like I can't tell if you're still there because you might be there but just not getting my
requests?
Yeah.
Yeah.
So how did you get interested in this field?
So I came across the rough paper whilst I was doing my undergraduate and I read the paper
and I thought it was really interesting and I was really excited by the idea of consensus.
And at this point the paper was just a draft online.
It hadn't been published or anything like that.
And I like many people wanted to go away and implement it.
So I did.
I implemented it in my favorite language at the time, which is was work was is OCaml.
And in that implementation I came up with like what I thought was some cool novel optimizations
to the algorithm to make the performance better.
So I implemented those and I started telling people about them.
Very, very excited.
I was.
And then people were like, this is all known stuff.
This is like what people use with Paxos all the time.
And you need to go away and do a lot of reading about Paxos and consensus before you can start
optimizing consensus algorithms.
So I went away and I read various algorithms on consensus.
I also started giving some talks about those papers that I'd read because I don't think
you really, I think you really know something until you try to teach it to somebody else.
And from there, I got more and more interested in consensus algorithms and how we could optimize
their performance in practice.
Is the danger with your early optimizations just that they would make it, you know, not
always consistent in some way that you didn't perceive?
Not normally.
Mostly they're just complicated and they take away from the point of raft.
The whole point of raft is that it puts simplicity first and they say upfront in the paper they
say, you know, where we had a choice between performance and understandability, we chose
understandability.
So I was basically just putting back in all of the stuff that they had taken out from
Paxos.
And I was putting it back in to try to make it more efficient.
There is definitely a challenge with ensuring the safety of optimizations, particularly when
people start to use clocks and rely on clocks.
So things like master leases.
As a software engineer, chances are you've crossed paths with MongoDB at some point.
Whether you're building an app for millions of users or just figuring out a side hustle.
As the most popular non-relational database, MongoDB is intuitive and incredibly easy for
development teams to use.
Now with MongoDB Atlas, you can take advantage of MongoDB's flexible document data model as
a fully automated cloud service.
MongoDB Atlas handles all the costly database operations and admin tasks you'd rather not
spend time on, like security, high availability, data recovery, monitoring and elastic scaling.
Try MongoDB Atlas for free today.
Visit MongoDB.com slash cloud to learn more.
When we talk about an innovation like your flexible Paxos, what is coming up with that
involved?
Are you using the TLA plus that you discussed or you just have an idea or what is your
steps?
So the story of flexible Paxos is quite a funny one.
So I was in, it was three summers ago, I went to VMware Research in Palo Alto, California
to work with Dalia Malke, who's a very famous computer scientist in the field of distributed
consensus and she wanted me to work on an area known as Byzantine fault tolerance.
So that is the type of consensus that fuels the blockchain.
And whilst I was kind of trying to get my head together with Byzantine fault tolerance,
I was also trying to put together a talk to kind of introduce myself to the team about
consensus and how it works and how do we teach Paxos from real like first principles?
Lamport has this famous quote where he says that Paxos is inevitable.
If you just think about consensus, you would derive Paxos.
And so I was trying to kind of put that to the test.
And whilst I was doing it and I was trying to prove it using kind of pen and paper proof,
I couldn't see why we needed to use majorities for consensus.
And I went through a good few weeks of just thinking, oh my gosh, I don't understand
Paxos.
And I was saying I've been going around talking to people about and like I've been telling
people about consensus and people assume that I know, but I just I can't understand why
we need majorities.
So I was definitely spent quite a while scratching my head assuming I was just missing something.
And eventually I got the bravery together to go to my colleague, Dalia Malke, and say
like, why does Paxos need to use majority quorums?
And initially she was like, well, you know, for safety, it's obvious.
And then I was like, but why for safety?
And she thought about it.
And she was like, you, you don't need them.
You don't need them.
The core the requirement about quorums is weaker than what Paxos uses.
And so then we expressed it in TLA plus.
I was it was pretty easy to express because Lamport actually written a Paxos implementation
in TLA plus so varying that for flexible Paxos was quite straightforward.
And then we wrote up a paper and centered around.
It was very helpful to have done the TLA plus stuff and to have a famous computer scientist
like Dalia kind of backing that works, try and convince other people that it was true
because I think a lot of people's initial reaction was like, what is this?
What are you talking about?
Everyone knows Paxos needs to use majorities for safety.
Wow, it sounds like quite an exciting time.
I think isn't it true that the original like Paxos paper took a long time to gain acceptance?
Yes, yes it is.
So I think it was seven years if I remember incorrectly that the paper, I think some versions
of it had like been distributed to some people but it wasn't formally published for for a
long, long time.
It still I think once it was published, Butler Lampsen I believe wrote a paper about Lamport's
Paxos paper and really kind of and Lamport already got a good reputation for his work
and distributed systems that it really like as soon as it was available, it really took
off very quickly.
If you're using this TLA plus to show something like that the majority is not needed, like
why is getting acceptance hard?
Like isn't that a demonstration?
Because it is very hard to convince people that stuff they believe to be true for a long
time isn't true.
And a lot of the descriptions of Paxos bake the idea of majorities right into the very
definition of what it means to reach consensus and therefore the idea of not having them
is crazy.
And with TLA plus like it does have its limitations.
So like I said, when you check a model, you specify a number of notes.
So you can't show for all possible N where N is the number of notes.
You can say I've checked for three and five and seven, but I can't say that it's true
in a general case.
Did you end up showing it's true in a general case using another tool at another time or?
So I've done a pen and paper, pen and paper proof where you kind of break it down and
write it out over a few pages.
And that's something people have been very, been happy enough with once you once you
find the right way of explaining it, you can easily make quite a convincing case for flexible
Paxos.
But I'm actually one of the things I'm working on at the moment is learning to use Isabel,
which is a their improver so that I can prove these things in the general case.
It's one of those things where it takes a long time to learn and it's quite a difficult
tool.
But hopefully once I've got it, I'll be able to use it again and again to show the correctness
or the incorrectness of consensus algorithms.
And are there future innovations in consensus algorithms expected?
Well, I obviously hope so.
I've written a paper recently, which is called the generalized solution to distributed consensus.
What it does is it breaks down consensus such that it only uses immutable registers.
The good thing about only using immutable registers is that when you have immutability, it's a
lot easier to see what's going on.
And if you read a register, you know that that value will always be the same.
By switching to that representation of using immutable registers, it's become apparent that
there are even more consensus algorithms that are correct than I originally thought.
And so at the moment, it's just a case of exploring specific points in that space.
So the area is not like totally explored?
No, no, definitely not.
And beyond kind of the consensus I currently work on, which is non-bizantine consensus,
there is a Byzantine consensus.
So this is where participants can act maliciously or just arbitrarily.
In consensus has been around for a very long time, but it's now getting a lot of interest
because of blockchain and cryptocurrencies.
How does blockchain and consensus relate?
I'm not clear on that.
So in the blockchain, generally what happens is you use some process like proof of work
to elect a committee.
And that committee makes decisions using Byzantine fault-tolerant consensus algorithms such as
PBFT, practical Byzantine fault-tolerance.
And in that algorithm, it basically uses extra round trips and the signing of messages to
keep a record, each participant basically keeps a record of what all the other participants
in the system are doing so that when you reach consensus, so you can reach consensus whilst
tolerating the fact that some of those participants might have lied to you.
It is even more heavy weight than normal consensus.
Why would you be concerned that nodes might lie to you?
Because you don't know who's operating, who's operating that node.
So with the kind of consensus, I traditionally work on you control all the nodes in there
in your data center and everybody is able to trust everyone else.
But out in the real world, different people are operating different machines.
Also, you know, programs contain bugs.
And if you're using Byzantine fault-tolerance to build, say, a cryptocurrency, what you've
got is some really motivated attackers who would be willing to put a lot of resources
in to try to compromise some of those hosts and take over the distributed consensus component.
Yeah.
So in a zookeeper system, like I control all the nodes and I don't have to worry about
them misbehaving, is there value for kind of the zookeeper scenario?
Some of the breakthroughs in the blockchain area, like, would it ever make sense to have
zookeeper be able to handle adversarial nodes or something?
So people are working on this.
There is Byzantine fault-tolerant versions of Raft.
It's not my specialty.
My PhD was entirely in the non-Bisentine space.
I personally think that the best Byzantine consensus algorithms are leaderless.
They're decentralized because if you have a leader, it just makes it a lot harder to
construct a Byzantine consensus algorithm because you have to deal with the fact that
that leader, that point of centralization in your system might lie.
Whereas if you have a decentralized system, I think it's much easier to tolerate lies
and malicious participants.
Is there performance ramifications if you don't have a leader?
Is it more complicated or slower?
It is generally a little bit slower, like if you take something like fast-pack source,
which is ironic to say fast-pack source is slower, but fast-pack source uses larger quorum.
So it might say two thirds of participants have to agree to something before it can happen
instead of saying just over half of the participants have to agree.
Performance can also be less reliable.
So in an algorithm like fast-pack source, you can end up with situations where, because
you don't have a leader, you have two people trying to do something at the same time and
they conflict and one of them has to back off and try again.
So we've heard about variations on distributed consensus.
What does the future hold?
Like I'm a software developer, the listeners are as well.
Can we expect what a faster zookeeper, some new breakthrough I've never heard of, everything
writes to the blockchain?
Where do you think things are heading?
I think we're heading towards a world where we have more options in terms of distributed
consensus that are specialized to different scenarios.
So at the moment, we really in practice, we mostly have Paxos and its variants and they're
all used majorities and they're all leader-based.
And I think we're going to see more and more different options that come available.
So with fast-pack source, for example, you can use different types of quorums across
where you've got multiple nodes across different data centers and within data centers so that
you could tolerate a data center failure, for example.
And so I think we're going to see a lot more variation as people come to understand consensus
algorithms more and more and also, you know, depending on whether you want to have a right
heavy workload versus whether you have a read heavy workload, you might want to use different
consensus algorithm, whether if you've got a key value store and there's a lot of locality
and accesses, you may want to use a different kind of consensus algorithm to if there wasn't.
So it really depends on the application and the trade-offs you're willing to make.
So my big hope for the future is that, you know, you have to make compromises in distributed
systems but you should at least get to choose which compromises you make instead of having
to take Paxos as it is.
And right at the beginning, you mentioned Google Spanner and how does their approach
from this differ? They're kind of rethinking some of the assumptions about networks or
time?
Yeah, so in Google Spanner, the protocol actually makes use of clocks and relies on
clocks synchrony to some extent for safety, which is very different from consensus.
Google are able to do this because they put atomic clocks in all their data centers.
And so it's a reasonable assumption to assume that those clocks are pretty close to synchronized.
But that's not an assumption that everybody can make with their normal machines that are
using NTP say for time synchronization.
And what about if networks become more reliable?
Would that influence these algorithms?
Yeah, I mean, if we could get a lot stronger guarantees about the synchrony and networks,
if we could say with a very, very high degree of certainty that a message will be delivered
within a fairly small time bound, we would be able to develop much better systems, I
think.
So for example, we could detect failures of participants a lot more quickly if we didn't
have such variation in network latency.
Because we'd have a tighter timeout?
Yeah, absolutely.
And what's your future work?
What are you working on?
So as I said before, I'm working a bit on Isabelle and learning about using theorem
approvals, I'm also looking to learn more about Byzantine fault tolerance.
And if we can apply flexible pack source and generalize consensus in some of the ideas
from non Byzantine consensus to Byzantine consensus.
So you started with reading this Raff paper as an undergraduate.
What did you think it would be like then getting into this field and how has it actually played
out?
I never really had a plan, I did not plan to go into this field.
I just read the Raff paper because I was interested and then I was really interested.
So I implemented it and then I thought about it a lot.
It wasn't my plan to do a PhD on dishbeak consensus.
It was my plan to do a PhD on kind of peer to peer edge network systems and kind of cloud
computing at the edge and to build these massive systems and solving consensus was a tiny,
tiny, tiny component of that.
I guess I expected in my head that consensus would be a couple of pages of my PhD thesis
instead of 150 pages.
So how do you think you for coming on the show where work and listeners find you online
if they want to reach out to you?
So mostly I'm on Twitter.
My handle is Heidi and 360.
And we'll include that in the show notes along with some of these links.
This is Adam Gordon-Bell for Software Engineering Radio.
Thank you for listening.
Thanks for listening to SE Radio, an educational program brought to you by IEEE Software Magazine.
For more about the podcast including other episodes, visit our website at se-radio.net.
To provide feedback you can comment on each episode on the website or reach us on LinkedIn,
Facebook, Twitter or through our Slack channel at se-radio.slack.com.
You can also email us at team at se-radio.net.
This and all other episodes of SE Radio is licensed under Creative Commons License 2.5.
Thanks for listening.
