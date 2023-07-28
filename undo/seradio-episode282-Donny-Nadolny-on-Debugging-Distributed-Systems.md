**Link**: https://www.se-radio.net/2017/02/se-radio-episode-282-donny-nadolny-on-debugging-distributed-systems/


This is Software Engineering Radio, the podcast for professional developers on the web at
sce-radio.net.
SC Radio brings you relevant and detailed discussions of software engineering topics
at least once a month.
SC Radio was brought to you by IEEE Software Magazine, online at computer.org slash software.
This March 21st and 22nd, don't miss Tech Ignite in early game California, just minutes
from the San Francisco airport.
Come to Tech Ignite to get rock solid info and forecasts on adaptive cybersecurity, emerging
technologies, machine learning and deep learning, operational intelligence and much more.
On Tech Superstars Steve Wozniak and Grady Booch plus C-level leaders from Netflix, Google,
IBM, Salesforce, GE Digital and Intel to gain valuable insights and learn about real-world
solutions you can start applying today.
Register now for the IEEE Computer Society's premier conference, Tech Ignite, at techignite.computer.org
and discover the truth behind technology.
For our software engineering radio, this is Robert Blumen.
I have with me today Donnie Nadalny.
Donnie is a scale developer at PagerDuty working on improving the reliability of their
back-end systems.
He spends a large amount of time investigating problems experienced with distributed systems
like Cassandra and Zucheeper.
He runs the failure Friday exercise at PagerDuty.
Donnie is a graduate from Waterloo University in Economics.
Donnie, welcome to Software Engineering Radio.
Thank you very much for having me.
We will be talking today about debugging distributed systems.
The idea for this show came from a blog post called Zucheeper's Poison Packet and a conference
talk on the same subject which I saw Donnie give at the Velocity Conference earlier in
2016.
I wanted to talk about the bug itself and some lessons learned that can be applied to
distributed systems problems in general.
First thing Donnie, would you like to tell the listeners anything else about your background?
No I think you've pretty much covered it.
Okay, briefly tell us what is PagerDuty's application that you were working on when
this bug was discovered.
The core of PagerDuty's application is alerting people when their software has problems.
So you generally have a lot of different monitoring systems, they all hook up into PagerDuty and
then our software will call or text or email your engineers to let them know when you have
a problem.
We also have a lot of other things on top of that to make it easier to coordinate these
responses and to fix things quickly.
At the time I was working on a few different back end systems in what we call the notification
pipeline which is the critical series of microservices that we have which are necessary to take
an event coming in from our customers, do some processing on it and then actually make
the notification so call out to our customers.
I'm going to guess that because people depend on this system when their product is down,
it would be very important for your system not to have any unplanned failures or downtime.
Is that correct?
Absolutely.
We care a lot about having highly available software and our goal is to be able to run
our software across multiple data centers and in fact across multiple providers, so not
just AWS also on Azure and the goal is to be able to lose an entire data center while
still remaining available.
You talked about running on AWS.
I'd like to introduce the concept of a region.
What is a region in AWS?
A region is one of the units of fault tolerance that AWS gives you.
So a region is made up of multiple availability zones and an availability zone.
You can think of it as a data center although really an availability zone can be made up
of multiple data centers, multiple buildings but they're very, very close together.
And within a region, different availability zones are also very close together.
AWS has the goal of having each availability zone which is within a region, each availability
zone should be a unit of fault tolerance.
So it should be nearly impossible for all of the availability zones in a region to go
down although it has happened before.
So internally in page of duty, we tend to think of a single region as a single data center
and so we spread out across multiple regions.
The point being then, these units of fault tolerance are things which can fail independently
of each other and you give yourself some redundancy by using more than one.
Yeah, that's right.
Okay, so in AWS their regions are called things like US West One, US East One.
Why would a system span multiple regions?
Well, the idea is that because a region can go down even though it shouldn't go down,
it should really only be an availability zone that goes down, a region can go down.
And so by spreading across multiple regions, it means that if you have proper replication
and things like that, you ought to be able to lose a region.
And a region when you're talking about US East One, US West One, that is a very tight
geographic, that's a really tight grouping of buildings together.
So if there were for an example, an earthquake around there, even though you're in multiple
availability zones, it's possible that it could be a very tight grouping.
It could damage all of those availability zones.
Sure, it could be something even though AWS did a great job building their data centers
out and the electricity and heating and cooling, it could be something totally out of their
control like a natural disaster.
Yep, and a good example of where even if you have lots of redundancy in your power or
things like that, you have backup generators, you still have to have those machines be connected
to other ones through your network.
And you can have redundant paths for your network, but a natural disaster is the kind
of thing that could take out all of your network connections to the rest of the internet or
to other regions.
Sure, are you aware whether Amazon routes across the public internet between regions
or if they have their own dedicated bandwidth that they lease?
I believe they have both, so they, I believe they do have peering agreements that can
go across the public internet, but they do have their own fiber, at least for most regions
and I think for all, and that's what they use primarily.
There's actually a case, I think about a year ago or so, that we experienced where there
was a fiber cut and the latency in one of the regions that we were in to some of the
other ones, it went from being 20 milliseconds or so all the way up to being, I think, around
100 milliseconds because I think that Terry was that AWS was actually routing through
their own fiber as much as they could, but it meant they were routing packets kind of
partway around the world, even though the two places were really close, and if you went
over the public internet, then the network connection would be fast, but because they
were routing over their own internal network, maybe for cost reasons, the latency was quite
high for several hours.
The problem we're going to be discussing it originated in a component called ZooKeeper.
We did an entire show, number 229 on ZooKeeper, I'd like you to briefly tell us what is ZooKeeper
and what is a ZooKeeper cluster?
Sure, ZooKeeper is a distributed system which is useful for building other distributed systems.
What it gives you is it exposes kind of a file system like interface to you, except
you don't store regular files in it, you store little bits of configuration information or
coordination information.
So you can do things like create a directory, create a file, atomically update a file, you
can create a file where the name is guaranteed to be sequential, if you have multiple concurrent
clients that are trying to create the same file, you can also have a file that will go
away when the client disconnects.
Now the idea with ZooKeeper is that you then build on top of these small set of primitives
to build other primitives of varying degrees of how high level they are.
So you can use this to build distributed locking where you want to make sure that only one
node in the distributed system that you're building is a leader for example, or it can
take a lock on some entity or some set of entities.
You can also use it while you probably not recommended, but you could use it for service
discovery where you would create a file with information like your IP in your port that's
have to contact you and all of your different clients would create that and to do service
discovery to pick which node you want to contact, you just look in that directory and you pick
one at random and it will most likely be live because when it disconnects shortly after that
the file would go away.
At PagerDuty we use it mostly for distributed locking where we use it to protect different
entities from concurrent modification.
One of the reasons we do that is because we use Cassandra and it doesn't really give
real acid transactions, they do have what they call lightweight transactions, but they're
not quite as useful as you might think.
So we use zookeeper as a way to prevent, as a way to kind of gain some of the benefits
of a transaction at least some level of isolation from that.
A zookeeper cluster is how zookeeper is highly available.
When you run zookeeper you typically run it as in a cluster of three or five nodes at
PagerDuty we typically use five.
And what zookeeper does is it will first elect a leader and then all operations, all
right operations flow through the leader.
That's how it can do things like give you sequentially named files when you try to create
them concurrently.
And the idea is that if the zookeeper leader fails, one of the replicas can take over because
before it will have told the client that a right was performed, it will have replicated
that to at least a majority of the zookeeper nodes.
So in that way you can have the kind of the extra abstraction of zookeeper be highly available
where any node can fail and you haven't lost any data.
Okay.
I can see how that would be very useful in your business because in order to ensure that
your system is always delivering messages, you need to have a lot of redundancy because
pieces of the system might fail.
But then when you have a lot of nodes that might not be trying to do the same thing or
to cooperate in doing something you don't want, you want to ensure that everything that needs
to be done gets done exactly once and not zero times or three times.
Yeah, that's right.
Okay.
Why does the zookeeper cluster need to elect a leader?
In order to give some of the consistency guarantees that zookeeper offers you, it's very, very
helpful to have all of these operations go through a single node.
That way it can do whatever coordination it needs on its own.
So in zookeeper, any right must go through the leader.
So in order to have your cluster be available, it has to elect a leader first.
So I'd like to discuss a bug that you presented at velocity.
Let's start with what was the first indication that something was not right?
The first indication that we had that there was a problem was actually with our regular
application level monitoring that we had.
What it had told us was that basically just things were becoming stale, work wasn't being
done, and we ended up tracing that during the outage.
We ended up tracing that to zookeeper not being able to do any work.
Your monitoring was telling you that the system was stuck.
How did you narrow down the problem to zookeeper?
I believe what we had done in at least the first instance that we had of this happening
in production was just through the logs we were able to see that we were timing out when
I'm trying to talk to zookeeper.
You have a client and it was trying to connect to zookeeper and it was timing out either
getting a connection or it requested zookeeper to do some work and that request timed out.
Yeah, that's right.
It was requesting zookeeper to do some work either reads or writes.
Both of them were failing.
I'm sorry.
I'm not interested in production, were you able to reproduce that in an isolated environment?
Until we were actually able to figure out what was going on, we couldn't reproduce it.
We did try quite a bit to reproduce it in our low testing environment, but we were never
successful until the very end when we actually knew what was going on.
What was the next step?
One quick thing I should probably mention is that one of the symptoms that we had seen
was that even though zookeeper wasn't able to do any work, all the processes were up.
It seemed like it should be able to, but it wasn't actually able to.
The client was timing out probably the next thing you would want to look at is zookeeper
and see if there are any clues there.
Am I right about that?
Yeah, that's right.
We do have some good dashboards for zookeeper.
What they had showed us was that all of the zookeeper processes were up.
There was a leader for the cluster, but the cluster was completely hung.
When we checked the networking between them, that was also fine, at least at the time that
we checked.
There was a network blip that had happened a little bit before, but when we checked,
the network was completely healthy.
The cluster was up, but for some reason it wasn't actually able to do any work.
When you say the cluster was hung, what exactly do you mean?
The processes were running, but if you tried to do a read or a write, then it would just
time out.
Were there any error messages, log messages, health checks, anything that gave you more
insight into why?
It got into that state.
One of the annoying things about this was that the health checks that we were using,
at least for zookeeper, were all fine.
There's a very popular health check that a lot of people use for zookeeper, which is
you can connect on a certain port.
You can issue it, the four-letter command, are you okay?
It will reply back saying, I'm okay if it's up and if it thinks everything is good.
We did have monitoring on that.
We had that command running regularly, but it actually showed that zookeeper was reporting,
everything was okay.
We looked in the zookeeper logs, and there were some weird looking things in the zookeeper
logs that we did try to investigate.
What's one of those, for example?
One of them was this concept of zookeeper trying to take a snapshot.
Zookeeper, as I mentioned earlier, gives you this abstraction of a file system.
One thing I didn't mention was that you're meant to put in only really small configuration
information or coordination information, and this is meant to fit in memory.
The entire database for zookeeper is usually pretty small, just a few megs or maybe tens
of megs, and it keeps a log of operations that you've done to it.
What it does is, every once in a while, it will write out a complete snapshot of this
in-memory database so that when you crash and recover, it can just read in the snapshot
instead of having to replay a gigantic log of everything you've done since the beginning
of time.
One of the early suspicious log entries that we had seen was this log entry saying that
it was too busy to take a snapshot and that it was going to skip that snapshot.
When we looked into the code to see what that meant, it meant that it was actually in the
process of writing out a snapshot when it tried to take a second one.
Looking at the time out of when it had taken the previous snapshot, several minutes had
gone by before it attempted to take the second one that had been skipped.
Maybe we thought that maybe there was some problem where that first snapshot was taking
a really long time to write out and was then blocking this other one from happening, maybe
because of some disk problems or things like that.
Okay, as far as you could with information that zookeeper was already giving you, you
didn't have a solution.
What was the next step in your investigation?
Well, one of the things that we had tried was, so one of the key things that you want
is obviously to be able to reproduce a bug.
It makes it much, much easier to figure out what's going on if you can reproduce it at
will, because then you can do this in your own staging environment or low test environment,
and then you can be a lot more aggressive at poking at zookeeper or about enabling
reliever-based logging to let you know what's going on.
So we struggled for quite a long time trying to be able to reproduce this.
The closest that we had got was actually radiolated to that log line of trying to take a snapshot.
We ended up reproducing what would happen if there was a really slow disk.
So a nice tool that we used for this was a tool called SSHFS, which lets you mount a
remote directory of any machine that you have SSH access to, so we did that.
We configured zookeeper to put its files on that remotely mounted file system, and then
you can use your normal network tools like IP tables or TC to mess with the network to
make it be slow or to hang, and those things will manifest as disk problems by either the
disk being slow or the disk hanging.
Now we had actually managed to get zookeeper to hang in a similar fashion by slowing down
the disk like that.
Now that seemed quite promising at first, but after we had investigated a bit more, we
have other monitoring and other dashboards showing us disk metrics and things like that.
And even though it seemed quite promising at first, we ended up concluding that that
wasn't what was happening because our other monitoring showed us that really there wasn't
any disk problems at the time.
There were other log files that were being written out fine.
And as I said earlier, this in-memory database, this snapshot that it was writing out was
really small, so it would have to be really severe IO problems for it to actually hang
for as long as it seemed like it was hanging with this logline of it saying that it was
skipping this snapshot many minutes after it had started the previous one.
You went down this path that turned out to be a dead end.
Then you did have a breakthrough that enabled you to make significant progress.
What was the big step forward that got you closer to what was going on?
So the way that we got that to next step was by adding some extra monitoring to zookeeper.
So I had mentioned earlier that this are you okay health check showed that zookeeper itself
thought that it was healthy.
What we had added was an external health check that was just simple.
It would just do a write to a key in zookeeper and then it would try to read that one.
Now we added that health check and the next time that this had happened in production,
we managed to grab a stack trace from the leader at the time.
And it was this stack trace while the cluster was in this kind of up but stuck state that
let us figure out what was going on.
So you added this more intrusive health check.
What was the thinking there of why that would be a useful check?
Well what we had seen was that zookeeper itself thought it was healthy but the application
wasn't actually able to make any progress by reading or writing to zookeeper.
Now having a really general health check on the application as close as you can to what
the user would see is very useful because it means that you can catch any problem or
pretty much any problem even ones that you haven't even thought of, dependencies that
you recently added or anything like that because you have this health check of pretty much
the customer or your user actually being able to do things.
The problem is that that doesn't give you any information about which of your dependencies
have problems or what's really going on.
So that was part of the reasoning of wanting to add this second one and it was also just
the observation that our application wasn't able to read and write.
So we wanted to have this extra independent check of whether or not something could read
or write to zookeeper because it seemed pretty likely that zookeeper did have a problem but
it could have been that our application itself was the one that had a bug that caused it
to not be able to read or write to zookeeper and that zookeeper itself was healthy the
whole time.
I see the built in health check you ask zookeeper are you okay?
It says yeah sure I'm okay but it might be line two and maybe the only thing that it's
able to do is say yes I'm okay not able to perform its other functions that you have
that you really need it to do.
Yep that's right when you look into the code that are you okay check is really just a very
tight mapping of it will take the command as long as it can receive that command it will
reply with I'm okay it doesn't do any extra check itself to make sure that it's actually
healthy it's really not much more than just checking that the port is open and that zookeeper
has somewhat started up.
Okay let's talk about this stack trace what did you see?
What we had seen was one of the threads on the leader stuck trying to serialize a snapshot
of the database to the follower.
So zookeeper runs as a cluster you have the leader you have all the followers and it if
you remember it keeps this log of what operations you performed but when a follower disconnects
from the leader and then starts back up again it has to catch up to where the leader is right
now so often it will just be able to grab the operations that happened recently the zookeeper
leader stores them in memory so it will just replay those operations to the follower but
if the follower is too far behind then it has to send a full snapshot of the database
to the follower.
Now this was where it seemed to get stuck so it was a follower who disconnected it did
reconnected it had requested a snapshot the leader was sending a snapshot to the follower
but it was blocking when it was trying to write to that socket to the follower.
When you looked at the code that was indicated in the stack trace did you get more insight
into what was going wrong?
Yep well the key problem here was that when it writes out that snapshot to a follower
it has to iterate all over all of the items in this in memory database.
Now when it does that it enters a Java synchronized block for each item.
Now because it did that and because it got stuck that meant that this thread that was
trying to write out the snapshot to the follower was holding on to a Java synchronized or holding
on to a Java lock for one of those items.
If you were to try to perform an operation on this item that had the lock taken on it
then of course you would block.
Now the zookeeper works by having a series of threads that process the requests that
come in they each do different things so there's a thread that's responsible for writing out
to the file system to replicating it to followers to actually performing the operation on the
in memory database and these threads or one of these threads would then become stuck because
it tried to take that lock that was being held by one of the by the thread that was writing
out the snapshot.
And this actually explained kind of one of the mysteries that we had seen before that
I hadn't mentioned earlier which is that sometimes the cluster would hang by having the leader
just come to a dead stop it would it would stop performing new operations but sometimes
the leader would actually fall behind the rest of the cluster and operations would still
happen for a little while and then they would hang.
And we ended up realizing that the reason this was happening was because depending on
what kind of operation you try to perform on this item that was that was blocked by
this other thread you could hang near the end of this chain that does things after it
was able to replicate that request to a follower or you might hang right at the beginning of
that chain before it was able to replicate a request to a follower.
Yeah I'm looking at this code right now it was synchronized block then right to a network
that sounds like a dangerous coding practice something that could definitely go wrong when
you're holding locks on things and doing operations that have potential of lasting a long time.
Yeah absolutely so one problem here is that you're taking a lock on an item and then doing
this operation that could in this case a network right which could block for a long time.
Now one of the surprising things to me is how long this operation could block.
So normally you think of a network right sure it could block for a little bit but even if
there was some it would only kind of block until there was a connection problem and then
the socket would disconnect but what we had seen was actually this problem lasting for
well over ten minutes I think nearly fifteen minutes in one case that we saw of having
the cluster in this state where it's blocked trying to do the right and this the TCP socket
that you're trying to write to is just hung for that entire period of time.
When you found this problem in the code the cause of the stack trace and that was not the
final step in debugging the problem.
No it really just pointed us to more things that we needed to figure out to really understand
what was going on here.
So after you found this stack trace you identified where it was failing what was the next
problem that led you to.
Well the next thing that we realized once we knew that the code was blocking in this way
we could figure out why one of the followers wasn't taking over from the leader because
the way that we resolved this when it happened was just by killing the leader.
So restart the leader process and then immediately one of the followers would take over and operations
would continue as normal.
But the thing is that you're not supposed to need to do that manually part of the benefit
of as you keep your being in a cluster and using a consensus algorithm it can figure out
or it should be able to figure out automatically that something is wrong and one of the other
followers should be able to take over.
Now the way that it did this is by having the leader send a heartbeat to the followers.
So every few seconds I think it is the leader will send a heartbeat to the followers and
the followers will keep track of how recently they've heard this heartbeat from the leader
and if it's been too long then they kick off an election.
Now when we dug into the code a bit more we realized why this wasn't happening which
was that this thread that sends out heartbeats to the leader or that this process of the
leader sending out heartbeats to the followers is actually done by a completely separate thread.
So even though most of the important threads on zookeeper are blocked or unable to do
any kind of work you still have this one thread off on the side sending heartbeats to the
followers saying don't take over essentially.
It sounds a little bit like the problem with the health check where the system was doing
just enough to pass certain checks but not doing the things you needed it to do.
Yep I think this is actually a pretty big problem in a lot of distributed systems in
any kind of leader based system like this.
You usually have some kind of timeout or heartbeat or things like that that are being
sent.
Now the problem is that if this heartbeat or whatever it is you're sending if you don't
first check to make sure that you're actually able to make progress then you can end up
in this situation where the only thing that the leader is able to do is send out this
message saying I'm healthy don't take over even though all of the important things aren't
able to happen.
From that point what was the next stage in your investigation?
So the next step was digging into this TCP behavior to figure out why this TCP connection
was stuck for so long.
Now we did have a little bit of network trouble before these outages happened but when we
at the time that we would investigate the network was actually healthy but for some
reason it was still blocked the TCP connection was still stuck.
Okay in my experience debugging these type of transitions are very important where you've
decided we've extracted everything we can from this layer the Java layer the problem
isn't there we need to change where we're looking to another layer.
How did you determine that the problem wasn't or you weren't going to get any more progress
out of looking at the Java and zookeeper and you had to go down to the network layer?
Well it was really about trying to find a full explanation for what had happened so
even though we had gone through the code we had figured out what was happening in zookeeper
and in fact by this point we knew enough that we could fix it and the fix is at least that
we implemented was just not doing a this network blocking right not doing that within the synchronized
block instead in the synchronized block you just take an in-memory copy of the thing that
you want to write and then outside of that block perform the right that way even if it
does get stuck all the important threads are still able to do what they need because you
haven't blocked one of the objects that they're trying to take a lock on so even though we
had somewhat figured out what was going on there we wanted to dig deeper to figure out
why this TCP connection could be stuck for so long.
Yeah and this illustrates another important lesson about this story with an open source
project you can modify the code and deploy your own patched version of the project to
test out bug fixes.
Yep and in fact we did run our own patched version for quite a while while we were waiting
for a new release of zookeeper to be put out which had incorporated our patch.
Yeah I'm going on a bit of a tangent away from the investigation how did the zookeeper
community respond where they where they did you get any inspiration from the community
and were you in communication with them around this issue.
We had opened up a ticket on their ticket tracker describing what this problem was that we had
seen and actually we didn't even open that up until we had a fix for it that we had tried
out and once we had that they were very receptive and quite happy to incorporate it into the
project.
Moving back to the investigation then you determined that the TCP behavior was not well
understood how did you proceed with your investigation from that point.
Well the big thing that didn't make sense was a connection on the leader being in the
established state and blocking this blocking right coming from Java staying there and being
blocked while this connection was established and yet on the follower side it didn't know
about this connection at all.
It had already forgotten about it or something like that and yet it was still established
and we had seen this staying in this state for many minutes.
Yeah so TCP it's a state machine and the two ends should ideally agree on what the state
of the connection is or they should reach an agreement fairly quickly is that correct?
Yep under normal circumstances they should.
What was going on then?
We decided to look into what happened to what happens to TCP connections under packet loss
or under network isolation.
So one of the things that we started experimenting with was establishing a connection and then
introducing a network partitions that these two sides of the TCP connection can't talk
to each other and then having one of the sides kind of playing the role of the leader do
a right and watch what happens with retries, watch what happens to the connection state,
and does it get terminated, how long does it take, those kinds of things.
So what kind of tools did you use to examine this?
What's very helpful for this is using TCP dump which will capture all the packets that
are being sent and then copying that over to your machine and opening it up in wire sharp
to take a look as well as using net stat regularly to look at the state of the connection and
of course IP tables to block packets.
Did this issue have something to do with the cluster having members in different data centers
having to go across a much longer network path than within single LAN?
To an extent, yes.
These kinds of problems can happen when you're running just within one data center but they're
much more likely to happen when you're running across multiple data centers and in this
case it was going over the public internet because these were two data centers that were
being run by different hosting providers.
So even though these things can happen in kind of a local network or in one data center,
a lot of these problems are much more likely or they can persist for a lot longer.
You can have much higher pack of loss lasting for a lot longer when you're running over
a wide area network.
Are you aware in the distributed systems feel generally are do a lot of bugs tend to surface
when people build these bigger clusters that span across the public internet?
Well I think even now it's still fairly rare to run these systems across multiple data
centers across the public internet separated by real wide area network latency rather than
just really small amounts.
So with Cassandra in particular, there are some people who are doing that although their
setup is usually a little bit different from ours, they're doing more of an asynchronous
replication between their data centers doing rights with a consistency level of what they
call local quorum meaning you only care about the consistency within one data center and
then that will be asynchronously replicated across to another one.
In our Cassandra clusters we use the regular quorum consistency which will synchronously
replicate across a different data center and in zookeeper it's basically always like that
it will always be running at kind of the equivalent of what Cassandra would call quorum consistency
where you always write to the majority.
So if your nodes are across a wide area network then you will always be writing across a network.
So in your presentation there's about 15 slides dealing with the TCP protocol.
Could you condense down what you eventually found to be the cause of these long timeouts?
Sure.
The kind of root problem with what we were seeing was you can have a follower and a leader,
they establish a TCP connection.
If you introduce a network partition between them so they aren't allowed to communicate
and you try to write on the leader it will try to send this packet to the follower and
it will wait to hear an acknowledgment back.
If it doesn't hear that acknowledgment back it will keep on retrying increasing the delay
between subsequent attempts of retrying to send this packet and actually gets up to be
quite a high delay in these retries.
So the maximum which is hard coded in the kernel is it will wait 120 seconds between
retrying and it's doing a doubling behavior of each retry that does and it will wait for
many many retries, 15 retries.
And so when you put all that together, when you try to do a write and there's a network
partition that write could block for at least 15 minutes and it could be higher depending
on some other parameters and kind of the initial network latency that was going on.
When you say hard coded was this something you could fix by tuning some of the kernel
parameters about TCP behavior?
You could kind of work around this indirectly but the only tunable parameter that would
affect this is the number of retries that you allow TCP to do when it doesn't hear
back this acknowledgment.
So if you lowered that then you would indirectly get kind of a lower, you could think of it
as a timeout on when you were doing a write.
But it's an indirect way and it's kind of not really ideal because then if you do have
pack of loss you would get fewer retries before it would just give up.
I have been intending to make some of these parameters a bit more configurable because
what I would like to have is keep the number of retries the same or even increase the number
of retries but cap that maximum retry timeout at some configurable parameter so that you
could set it to be quite a bit less than 120 seconds.
Yeah, that's pretty long.
There's one other slide I found quite interesting.
It talks about IP tables and net stat.
Now those are two tools in the Linux world that there's a lot of Linux tools and they're
not all distinct.
There's a lot of overlap between what they do.
There's an interesting point on this side, it says IP tables connections not equal to
net stat connections.
Explain what that means.
I'd mentioned earlier how when you try to do this write if you have a full network partition
between these two sides then you could have that write be blocked for 15 minutes or more.
But that wasn't quite what we were seeing because when we went to investigate the network
health we saw that the network was actually healthy but the connection was still stuck.
So to figure out why that was happening what we ended up seeing was that the follower side
or one of the sides of the connection that wasn't trying to do this write would disconnect.
It would try to close this TCP connection but there was still a network partition so it
wouldn't be able to send this fin packet it trying to tell the other side that it wants
to close connection.
It wouldn't be able to.
Yeah so the fin packet that's part of the TCP state machine where it's trying to the
two sides are trying to agree to close the connection.
Yeah that's right.
It's trying to close the connection but if you have a network partition obviously that's
not going to get to the other side.
But you don't want to have this connection that you're trying to close remain there indefinitely.
So after a certain amount of time you just consider the connection closed even though
you haven't been able to talk to the other side.
Now we were running IP tables as a firewall to only allow certain connections through
only connections on certain ports or establish connections that you knew about.
Now this follower side what would happen is it would close the connection or it would
make the attempt to close the connection it wouldn't be able to tell the other side but
it would eventually market is closed.
But now the network can heal and one of those retries from the other side that's waiting
this 15 minute time one of those could come through but IP tables would then block this
packet because it didn't know about that connection.
And so in this way you can have these two sides where one of them considers the connection
established one of them considers it closed and even though the retries are coming in
that one side is blocking those packets and so you never get this other TCP packet that
would come through a reset packet which is normally what would happen when you try to
talk to a machine that didn't know about the connection but because IP tables was blocking
it it would never get through and so the network can heal but you still have this connection
stuck.
Now to get to the point about these IP tables connections are not equal to net stack connections.
What I hadn't realized was that IP tables has its own state machine that it uses completely
separate from the regular kernel state machine.
So you can and in fact the IP table side just uses simple timers that are different than
the knowledge of the kernel of whatever retries it has and exponential back often things like
that.
IP tables just has simple timers that it uses based on watching packets come in and out.
And it turns out that that timer at least with the default behavior it can actually
consider a connection be closed after one side attempts to close it earlier than the
regular TCP kernel view of that connection.
So you can in fact have if you do this setup of having a network partition have one side
try to write have the other side try to close it wait for a little while you can be watching
net stat and see the connection still be there in the I think it would be the the fin weight
connection or I forget which TCP state it would be in while you're waiting for it to
be closed.
You could see that in net stat but IP tables might have already forgotten about the connection
and so if the network had healed right then you could have packets come in you could see
on that other side it still knows about the connection but IP tables would drop it and
so you would still not get that reset packet going back.
The reason I found that interesting when I was thinking about the outline for the show
one of the points I'd written down on lessons learned was that these tools we rely on lie
or maybe that's overstated but it's definitely a theme in your discussion that you have these
health checks and heartbeats and tools and they tell you one thing but you can't always
take it at face value it doesn't necessarily meaning exactly what you might think it means.
I think one of the things that made this problem difficult to investigate but also really interesting
to investigate is when you have to go kind of go really deep go beyond the regular layers
that you're looking at in this case digging really deep into what happens with TCP behavior
under network isolation and yeah really kind of understanding or in my case learning a
lot of these different tools to figure out what the real state is.
I spent quite a lot of time playing around with a few machines introducing network isolation
and then watching what actually happened with what watching what actually happened to the
TCP state and to the retries that were going on and actually ended up seeing kind of some
maybe undocumented behavior or maybe a bug in the TCP kernel stack where you end up getting
one extra retry than you would think.
Are we still learning more about TCP and how it behaves in edge cases?
Well I think a lot of people if you're never forced to go digging into it you kind of take
it for granted that you establish your TCP connection you write stuff over it and everything
is fine if things go wrong then it gets disconnected.
It's kind of these weird edge cases that come up that can really throw off your understanding
or force you to dig into what's going on there.
So there's one more layer in debugging about IPsec and I'm going to skip that but people
who are interested should look at your slide share and willing to that in the show notes
I want to get more into lessons learned that the rest of us can apply in debugging problems
that we have what is a key lesson that you learned as an engineer from investigating this
problem.
I think the biggest one that I learned is not really a technical thing but it's more that
you need to be really persistent.
So when we were debugging this problem there were several days in a row where I came into
work and the entire day I spent looking at logs going through Zucheper code and basically
not making any progress.
I was making some progress in terms of gaining an understanding of Zucheper and TCP behavior
and other things like that but in terms of making real tangible progress to figure out
what was going on it was just no progress at all.
So I'm very glad that we were persistent in this investigation to figure out what was
really going on.
I've had the experience debugging and I don't have this experience writing code but I've
had with debug chasing where I start to despair and I think I am never going to find this.
It's just too hard.
Did you ever experience despair or the feeling, the thinking, I'm just going to give up or
did you always have optimism that you would find the root cause?
Well I wouldn't say that I had optimism the whole time but I never really had that kind
of despair.
It was a little discouraging to go for so long and to not make any progress in terms of the
Boolean measure of how I figured it out or not but luckily I never really ran out of
ideas of what to try next.
I had lots of ideas of things that it could be or kind of the next step of something new
that I should investigate or some new bit of the code that I should look at or things
like that.
You gave this presentation about the problem, the full investigation.
Were you the sole person working on it or were there multiple people?
Nope there were multiple people so I think I was the one who investigated for the longest
but I had a couple of my co-workers who had were kind of on and off this investigation
and also some others who I asked about networking behavior who kind of gave me some pointers
of where to investigate next.
The problem as you chased it around it was from Java to the Linux kernel to the TCP protocol.
Often people will not have a really deep knowledge of all those layers.
Did you collaborate with people who are expert each at a different layer or how did that
work?
Well, I'm fairly experienced in Java so I didn't have too much trouble going through
the code base for the zookeeper code which is written in Java.
It was really the networking behavior that I needed a tip from one of my colleagues, Evan,
who gave me that tip about TCP being able to or really paying attention to the state
that the connection is in particularly under network isolation.
You have a number of lessons learned.
What is another key lesson you learned from this process?
One of the interesting things that I thought about during this was how interfaces or abstract
methods can make it a bit harder to be confident that your code is correct.
So with this call in zookeeper that was taking a lock and then performing this potentially
blocking operation, this TCP call, which as we learned can block for quite a bit longer
than you might expect.
I thought about how the call that it was performing that ended up writing over the socket was
actually an interface method.
So with that, even though I don't think it was a factor in this specific one, it's easy
to imagine a situation where you're writing code, where you take a lock, and then you
call out to an interface or some abstract method.
And at the time you write that code, all of the implementations of that are non-blocking.
They all operate just in memory.
They're guaranteed to finish in a certain amount of time.
But it's possible for somebody later on to add a new implementation of that, which would
all of a sudden make your existing code incorrect.
And again, as I said, I don't think that came into play here, but it was just kind of an
interesting thing that I noticed that it can make this kind of analysis or this kind of
problem a bit easier to occur than you might think.
Yeah, I have seen some critiques of object oriented programming that raised this point
that by creating abstract methods and interfaces, you can't guarantee that all the implementations
of people will plug in will honor all of the contracts that are assumed by the frameworks
or higher levels of the stack that call them.
And I think this illustrates that point.
I want to summarize in some of the lessons we talked about earlier were about monitoring
and health checks.
What did you learn about that?
So I feel I think even more strongly now than I did before that it's really good to have
high level health checks of your application to make sure that you can catch all these
different things.
But also that it's very useful to have kind of these narrow but still end to end tests
of dependencies that you have.
That makes it very easy when you get paged and you see instead of just getting a message
saying that there's some problem in the application, it's good to get that rather than nothing.
But it's also nice to have either another alert or a dashboard that shows you.
And here's the one dependency that is messing up.
Right, so you want computers to do as much work as they can in collecting useful information
about the problem before you start working on it.
Yeah.
Okay.
Yeah.
And what about health checks?
Health checks are another thing that I think could be improved in a lot of systems.
So it's very easy to make kind of a naive health check that just replies and says that
you're healthy as long as the process is up.
But you can have this problem where that's then the only thing that is healthy.
So I think it's also important to tie in more of a deep health check whenever you're doing
that kind of thing.
Don't just reply right away saying that it's good.
Either perform an actual operation as your health check or something along those lines.
And I think that this is important for one part for monitoring the dependencies of this
is a great way of checking your dependency, perform a simple operation and that's their
measurement of whether your dependency is healthy, but it's also useful in any kind of
leader follower based system where the leader is preventing followers from taking over to
do some kind of deeper health check locally to make sure that when you're getting requests,
you are actually able to process them.
And it's only then if you know that to be true that you would then send out this message
to your followers saying, you know, don't take over everything is good.
Sure.
This reminds me of a retail system I worked on.
We use an external monitoring service called Pingdom that checks that your servers respond
to HTTP.
We also implemented check that would log into the site, create a customer by a product,
put it in the cart and check out or cancel the charge at the last second because if you
can respond to HTTP, but you can't allow people to buy products, then something's not right.
Yep, absolutely.
That's a great thing to do.
You can pick kind of whatever your most important feature is in the case of a shopping site.
Can you go and actually add and buy a product?
And that's a great health check to use.
So PagerDuty also has a similar system we call Watchdog that performs end to end testing
against our system where it acts as a customer would by sending events to a regular public
endpoint using our public API to perform various operations that we think are important and
then making sure that it actually gets delivered in the end.
And that's a system that we run against production.
But because it's kind of a outside of our regular system and only uses our public endpoints,
you can even point that at any of our different environments and use that to check that, for
example, the load test or different staging environments are healthy as well.
Yeah.
And now there's one more lesson which I learned from your experience and also from reading
the blog post about the zookeeper's poison packet, which describes a different but similar
bug.
And I'm not sure if you learned this lesson because I think maybe you already knew it
and so you didn't need to learn it.
So I come from a software development background and the mindset is there's a bug in my code.
So I'm going to look in my code and find where the problem is and I'm just going to
keep looking in my code until I find it or maybe the libraries if they're open source
and they're in the same language or same stack.
I made a transition in my current job to DevOps where one of the big shifts is we're looking
at a lot more layers of the stack.
A lot of the issues I deal with is any possible layer could fail.
Sometimes multiple at once and just opened up my mind a lot more to all the different
places that things could fail and that I can't come into issue with too much of a preconceived
idea about where the failure is.
So that isn't really a question but you can comment on that if you like or we can go to
the next question.
It certainly makes it harder to figure out what's going on when you can't just assume
that it's your code that's wrong.
When you have to go into these other layers or other systems that maybe had guarantees
or maybe you thought they had certain guarantees but it turns out that they don't.
It makes it quite a bit harder to debug for sure.
So this will be my final question, Donnie, in reading the blog post about the keepers
poison packet which has a lot in common with the issue we've been discussing today.
The problems were not the result of one single bug in one behavior.
It was interaction of multiple bugs or not even bugs but behaviors that were not well
understood that interacted with each other to produce the final problem.
Do you have any thoughts?
Why is it that we get these type of multiple failures?
Why isn't it just one thing?
Well in insecurity they have the concept of defense in depth where you want to have
multiple layers of protection and in a way in a lot of distributed systems you often
get that.
When you have this replication usually you can just suffer the failure of a node and
you're fine.
These problems come up.
My question is this because at the level you're building a system like this you have a wealth
of best practices and knowledge.
You know how to plan for okay a node fails.
Okay the cluster loses a member.
The easy cases you've thought of them and that leaves the hard cases.
Yeah I think in a lot of ways it's when your assumptions are violated that then you start
to have problems like these.
So in this zookeeper problem that we've talked about today the assumption that was probably
made that TCP won't block for that long.
So even though it might take a variety of failures or some weird network conditions to
expose this most of the time it wouldn't happen but there is this case when it can happen
and that assumption being violated then means that you can have this bug.
In the poison packet example it was not just assumption but supposed to be a guarantee
that TCP won't corrupt your data.
It has check sums that are supposed to prevent that but in the kind of multiple cascading
problems or multiple combined problems we did end up having TCP data be corrupted and
then the bad part there was that the failure mode of zookeeper when that happened end
up being in this stuck state rather than being able to cleanly crash and let another node
take over.
I think this is another tough problem to deal with.
It's when you have a node in this sort of degraded state where it's not completely healthy
but it's healthy enough to say to kind of occasionally send out heartbeats or healthy
enough to start taking requests but not healthy enough to finish them in a reasonable amount
of time.
That makes me think about how programmers were trained to think in layers.
If I'm implementing zookeeper, zookeeper deals with the network that's another layer
so I'm going to make some assumptions about that layer as a programmer and if I don't
have in-depth knowledge of TCP I'm going to probably make maybe a little more optimistic
or best case assumptions about the network which are okay as long as they're true but
then sometimes they're not.
Yep having these different layers, having it be well encapsulated like that, it means
that you can be a lot more productive because you now don't have to worry about kind of
data being ordered within TCP or about corruption or anything like that.
You just use this nice abstraction which makes it a lot easier for you to build a system
but then when your assumptions are violated you can have problems.
One interesting one that I thought of a while ago is that you have this guarantee in TCP
that data is ordered so when you write it even if the individual packets are delivered
out of order TCP will reconstruct them and give them to you in the right order.
Now the kind of really edge case that I thought of is that what happens when you then disconnect
or if you have some amount of packet loss.
Maybe what your client or what your system will do is it will establish a new TCP connection.
Now the ordering of data between those two connections isn't guaranteed at all because
they're different connections and you can think of maybe a pathological case where one
side will have written some data that will have been, you know it will be, it will suffer
packet loss and it will be retried and in that time you go and you establish a new connection
and you write some new data but now this thing that you sent second will actually come first
and the other side might even get that data that you sent initially later on and that
could cause problems.
So even though you have this abstraction of TCP is ordered you potentially could have
this weird edge case of well maybe it's not.
Yeah it does exactly illustrate that.
Now a minute ago I said I was going to be my last question but I thought of one more
in the programming job market the emphasis it's very focused on what skills people have.
People will list their Java programmer Python Linux on their resume.
Do you think that debugging is a distinct skill set that can differentiate you and make
you more valuable to a company?
It's an interesting question so I certainly think that debugging skills making more valuable
and that they're extremely useful.
I don't know whether I can say for sure that it's a distinct skill.
I think maybe it is but I'm not quite sure.
Okay wrapping up we have talked about the zookeeper blog slide share.
But are there any other resources or anything you personally post that you would like to
people to have a look at?
Well I keep a pretty low profile online so I don't have too much to share from that.
One thing I would like to pitch though is this the concept of doing fault injection or we
call it failure Friday.
Netflix is going to term chaos engineering.
I think that's a very useful thing to do to distribute systems into any system to make
sure that it's actually correct.
Yeah that would be a great idea for a show so maybe we'll do a future show on that.
Sounds good.
Yeah Donnie thank you very much for speaking to software engineering radio.
Thank you very much for having me on the podcast.
For software engineering radio this has been Robert Blumen.
You can reach us by email team at sc-radio.net find our group on Facebook or LinkedIn and
on Twitter at sc radio.
Thank you for listening.
Thanks for listening to SC radio and educational program brought to you by IEEE software magazine.
For more information about the podcast including other episodes visit our website at sc-radio.net.
To provide feedback you can write comments on each episode on the website or write a review
on iTunes.
Mention our messages on Twitter at sc radio or search for the software engineering radio
group on LinkedIn Google plus or Facebook.
You can also email us at team at sc-radio.net.
This and all other episodes of SC radio is licensed under the Creative Commons 2.5 license.
Thanks again for your support.
