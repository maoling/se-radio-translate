This is Software Engineering Radio, the podcast for professional developers on the web at
se-radio.net.
SE Radio is brought to you by the IEEE Computer Society, by IEEE Software Magazine.
Online at computer.org slash software.
Welcome to Software Engineering Radio.
I'm your host Akshay Manchali.
My guest today is Spencer Kimball, and we're going to talk about Cockroach TV.
Spencer is the CEO and co-founder of Cockroach Labs, a company formed to accelerate the
development of Cockroach TV.
Previously Spencer worked for 15 years at various companies including Square, Google,
and a couple of his own startups.
His experience at each step along the way convinced him of the need for a better open
source database.
Before his professional career, Spencer co-authored the open source GIMP photo editing software
while an undergraduate at UC Berkeley.
Spencer, welcome to the show.
Thank you Akshay.
Happy to be here.
So let's get started.
How would you describe Cockroach TV?
Well, it's a distributed, resilient, sequel database.
Probably the minimum number of words that you can use.
It's resilience why the name comes from.
Exactly.
You know, they had, for whatever reason, government likes to do strange research.
They had a program at some point to measure the effects of radiation on different living
organisms and it turned out that Cockroaches were pretty resilient to radiation.
And so the joke was always that, you know, after World War III, they'd be the last things
surviving.
That's interesting name.
So your experience kind of convinced you to build a new open source database.
So can you talk about the motivations behind that?
Yeah, absolutely.
In my entire career, it's been, I guess since 97 when I graduated university, it's been
a sort of alternating between building applications and wanting more from the database and then,
you know, working on various infrastructure to support applications, you know, especially
to address the needs that I'd seen in my previous iteration as an application developer.
And it became pretty clear early on that databases were going to be a major sticking point in
my career.
I remember taking a class at Berkeley on databases and being, I found it interesting, but it
was kind of something that you do for extra credits, as opposed to something that was
a true passion.
But as soon as I got out in the industry, and I think it was my first startup in the
dot com boom, I started working with Postgres and Oracle and it became immediately apparent
that these things would need to be sharded for the scale that we had.
So that was the first experience that I had of a database not having what it took to meet
the applications sort of business requirements.
And it's a pretty painful experience to sharded database for anyone that's done it, they sure
can attest to this.
When I went to Google in 2002, that was one of the very first projects that I was on,
which was the AdWords database backend.
So they were using MySQL and that had originally started as obviously just a single MySQL
database and it got too big for one.
And so they created two of them and split up the customers that were using AdWords between
those two.
And then that went to four, which went to eight.
And then after eight, it started to really have problems with just scalability.
How many connections was each database node having to service?
And there were other problems like how do you figure out where our customer is across
all of the different database shards and the beginning, they just queried all the shards
at once.
But once they were at 16 or 24 or 32 shards, then the number of queries coming in just
to find out where customers were when they're trying to log in with an email address was
overloading the servers.
I'm just giving you the tip of the iceberg and the number of problems that were caused
by that architecture.
But one way to look at how problematic it was is that it became an anti-pattern at Google.
So it's somewhat of an outlawed architecture and they didn't get off it for a good 10 years
until they moved to Spanner eventually.
And when I was participating in the project, there was what they called the ads war room
every morning where people would try to stamp out the fires that had lit up overnight.
So it was just this endless slog in order to make this MySQL database in the application
level sharding work properly.
And what's interesting is that Facebook has a similar architecture as do many of the other
big tech companies because it's very obvious.
You can put some customers on shard one, someone shard two, just keep track of where they are
and you can scale to a billion customers if necessary.
And Facebook, of course, has scaled to billions of customers and they are using vast, vast
numbers of these shards.
It makes Google's AdWords program or deployment look pretty small, in fact.
So it's a very popular way of doing things, but that was a great example of where databases
just in their original incarnation didn't meet the needs of many businesses.
And that's becoming, of course, even more common in 2020 than it was in 2002.
So at Google, again, I actually didn't just do the backends and the database related
stuff.
I also worked on various applications over that time and eventually found my way into
building infrastructure for distributed file storage.
And then when I left Google, I did a startup, which was private photo sharing.
So definitely on the application side.
And that's where the idea of Cockroach showed up because we'd seen over 10 years and I say
we because it's me and the same co-founders I have for Cockroach, we left Google and
we did this startup and then we went to Square together.
We kind of been traveling as a pack for a long time.
But we came to a conclusion when we left Google that the existing database options and open
source were pretty far behind what Google had been aggressively working towards with
ambitious research and development projects during our tenure there.
And that inspired us, right?
Because we saw what Google had kind of reached by 2012, which was the first time that Spanner
was starting to hit production.
We saw them build Bigtable before that.
We saw Bigtable morph into Megastore.
We saw the early attempts at finding the point in the solution space, which eventually became
Spanner.
And all of that was significantly more advanced than what existed in the open source world
in 2012.
So we tried looking at HBase.
We looked at MongoDB.
We looked at React.
We looked at Cassandra.
We looked at some of the things that were in the public cloud.
And eventually, it was the same kind of story as we examined each of those options, which
was initially excitement and then a bit of a letdown kind of went into that trough of
despair.
This isn't really going to work.
Doesn't have all the things we need.
And that's where the idea of CockroachDB was born.
It's like, can we build something in open source that we could use for this project and
that lots of people could use for their projects, which has a lot of the same capabilities and
guarantees as the advanced databases Google had been building, you know, in particular
resilience, scale, and even this idea that there exists locality to data.
So you might have customers in the EU and customers in the US and you have to service
them differently and store their data differently.
All of those ideas in one single package that was open source, that was the manifesto.
And Cockroach is an example of our kind of, what I call it, a slightly darker interest
in preference for names.
You mentioned that we worked on the GIMP that was paying homage to the pulp fiction character
of the same name, which was the top of mind in 1993 when we started that project.
So CockroachDB sounded pretty funny.
What's actually even more funny is that nowadays I'm explaining that to Fortune 50 CIOs and
CTOs, which is actually a pretty good icebreaker, but it's also a little embarrassing when you
show up at one of these glass-paneled office receptions and have to spell out your company
name.
You mentioned you evaluated MongoDB.
That was something that was raising in popularity, not MongoDB per se, but just no SQL systems
that kind of promised a scale, not seen in relational databases.
What was the problems with that?
Well, there's a lot to like about MongoDB.
But unfortunately, at that time, it wasn't as scalable as you might have expected with
a distributed architecture like it had.
It certainly had some issues around how that worked in practice.
I think the biggest issue, though, that we had, and it was a pain point that was experienced
at Google as soon as Bigtable came out, which was in 2004, it was just immediately apparent
that that no SQL model without transactions was going to be a difficult hurdle for a lot
of applications that really did need transactions in order to maintain correctness.
One of the big lessons at Google, I think this is just true everywhere nowadays, but
when you have scale, if something can go wrong, it absolutely will go wrong and will go wrong
quickly and it will metastasize into a major problem.
The MongoDB of 2012, and it's introduced transactions in the years since, but at that point, it
felt like that architecture was going to cause more problems than it would solve for us.
Cassandra had similar deficiencies.
They also didn't have that.
This is actually pretty much a feature of no SQL not to have transactions in those days.
Now it's increasingly a feature of no SQL systems.
I'll give you an example.
FaunaDB is one where its built and foundation DB was another, where it was built from the
ground up with transactions.
I think it's a realization that no SQL was about allowing elastic scalability, and that
was its big capability that was new to the market.
It gave up a lot of things.
You might not like SQL, but everyone likes transactions if you're an application developer,
assuming you know what they are.
That was one of the things they gave up early on, because it's complicated, difficult, adds
latency, difficult to get the model right.
It was essentially just simpler to ignore that stuff.
You can build lots of interesting applications that don't need transactions, but what you
also realize is that you might start not needing them, but you quickly find ways to
introduce new functionality that starts to break if you don't have transactions.
That was Google's experience, and Megastore introduced basic transactions, and then Spanner
introduced very general purpose transactions.
That was one of the big sticking points for us with no SQL.
In 2012, we'd just come from a world where transactions were seen as a necessity.
To exit Google and find that there was a bunch of open source stuff that looked the way Google
looked in 2004, 2005, was a little bit depressing, honestly.
You mentioned one thing that you were doing was partitioning or sharding your databases,
so do you have transactions across those partitions?
How did you manage that?
Are you referring to Cockroach?
Not Cockroach, like just historically, was that a point of concern that you wanted to
have transactions across your partitions?
Yeah, absolutely.
That was a big problem with the AdWords system.
Because it was composed of multiple independent MySQL databases, you could within one database
have transactions, and believe me, they did have transactions, and that was very important.
The problem was that they quickly found use cases where they wanted to do transactions
across these shards.
What that forced them to do is make other decisions architecturally so that all the data
they could ever do a transaction between how to be on one shard, which didn't always work.
For example, some customers leave like eBay that had so many ads on AdWords, they wouldn't
fit onto a single shard.
Without transactions that are general purpose, inevitably, if you're succeeding, we'll find
ways that your model is going to break.
That was definitely the experience at AdWords.
That wasn't the only problem.
Other examples, SQL gives you this amazing ability to ask the database for data in a
declarative fashion instead of imperative.
The database is going to use its smarts, whatever it's got, to go fetch the data and
combine it in as efficient a manner as it's capable of.
That's only true within a single shard.
You couldn't do queries across these shards because these databases didn't know about
each other.
There was no centralized brain that could schedule this work and have it be done efficiently.
That's essentially what Cockroach is.
It's that brain that uses a lot of machines and moves the data around them and schedules
and optimizes and so forth.
It is that thing that didn't exist.
What you have to do in a case like Google back in 2002 with this AdWords system is you
got to try to build some of that in at the application level.
Put as much as you can on one shard and then where you have to go across shards, you have
to do it as a programmer, as an application developer, which is difficult to get right
and then very difficult to maintain.
Can you walk us to what CockroachDB deployment looks like, a cluster of nodes and how does
that do transactions?
Let me just start by saying, I'll try to give you a pretty high level answer to that, but
there are a lot of blog posts if you search for CockroachDB and transactions and distributed
SQL and things that will explain this in painstaking detail and that can be a really interesting
resource that people want to find out more.
I'll include that in the show notes.
Great.
A little bit about how you deploy Cockroach.
Basically, we tried to make it, this is one of the guiding design principles because it's
very different at Google and I didn't like the way it worked at Google.
We tried to make Cockroach so that there's a single binary and it's symmetric.
All the nodes run exactly the same binary with nearly identical command line parameters.
They only differ where you say this node is on this particular rack and you specify the
location of the node.
That is different between nodes but otherwise it's identical.
Those nodes work together without any external dependencies.
They don't require ZooKeeper to run.
They don't require distributed file system underneath them.
They don't require access to external things like a timekeeping service.
That's symmetric architecture allows for an easier deployment story especially with different
tools.
There was the mesosphere stuff which is DCOS.
That's a very different environment from running something on Kubernetes as an example.
All of these are things where our customers want to run Cockroach or just run it on bare
metal.
Just to give you a practical explanation, you can start a single Cockroach node and you
can basically tell it to initialize itself.
When you want to create the next node, you simply point it at the first node.
That's all you have to do.
Then they'll begin to self organize and replicate data and move data between them so that it's
equalized in terms of the capacity between the two nodes.
If you add a third, they need to talk to either of the two existing.
If you add a fourth, similarly any of the three existing and so on.
You can even grow it that way.
You can also decommission nodes and shrink a cluster as necessary.
It will heal, up replicate the pieces of data that were stored on the node that's been decommissioned
so that you're always maintaining the desired number of additional replicas of any particular
piece of data.
That's sort of how the deployment works.
It's supposed to be very automated, very straightforward and simple or automatable maybe
is the right word.
Once it's running, it automates everything itself or everything that it can.
These actions, there's a fairly involved model for how they work.
This is something that again, it's best to refer to some of these blog posts because the
model has gotten incredibly sophisticated.
It started as an amalgamation of different transaction algorithms from research papers.
It's since morphed into something even more particular to Cockroach's use case.
I'll give you a little bit of detail about that in a second.
Basically, it's a distributed transaction model.
The goal with these things is that when something goes wrong in the system, let's say that the
transaction begins and you modify, you read a bunch of data from different nodes and you
modify data which is all existing on different nodes.
All those things are being touched.
When you abort that transaction, all of those changes should be seen as intermediate and
they should all go away.
Nothing should be left.
If anything happens concurrently while the transaction is underway, that concurrent execution
or worker, reader, writer, whatever it is, won't see anything that's still in an indeterminate
state that the transaction has started.
When you commit the transaction, all of that data needs to be committed and available and
viewable at the same time.
It needs to be a virtual simultaneity.
That's the heavy lift that this transaction model is doing.
There's a lot of particular details in terms of how it actually works.
To give you an idea of where we've been pushing the state of the art, we've recently, in the
last four years, been looking at lots of use cases that have to do with storing data in
different data centers in different regions, like East Coast, West Coast, the United States,
even on different continents.
This is often something that customers might want in terms of keeping the latency low for
a user in Australia, for example, or complying with data sovereignty regulations like GDPR.
Where can you actually legally store the data or where do the user want you to store their
data?
All of those considerations make us to that.
You actually can put data far away geographically such that even if you were beaming a light
through vacuum in order to send a signal, you'd still have hundreds of milliseconds of latency.
It's more than that sometimes through these communication networks.
How do you make a transaction model that looks like sequels work in that environment that
has geographic latencies?
That's a pretty difficult problem.
If you do it in a naive fashion, which I think fairly accurately describe our original approach
to the transaction model, those latencies.
Let's say it's 150 milliseconds between two nodes and they're interacting in this transaction.
You're doing a bunch of different rights.
In the naive way, those 150 millisecond latencies add up serially.
If you did 10 writes, you'd have 1.5 seconds, which is horrific kinds of latencies.
It's not tenable for building an application.
What we had to do is say, are there ways to change how that works in practice?
We've been able to get an entire distributed transaction, even with 150 millisecond latency,
instead of taking 1.5 seconds for those 10 writes plus another 150 for what's called the
second phase of the distributed transaction to do the commit.
It would be about 1.65 seconds.
We're able to get that down to exactly 150 milliseconds.
We do something called pipelining, which is a somewhat obvious response.
There's a very non-obvious part of that, which is something called parallel commits, which
hides the latency on that second phase of the transaction.
The actual application that's doing the transaction only sees 150 milliseconds of latency before
it's guaranteed that everything is committed globally, in fact.
That's an amazing result.
It's a very recent result of years now of work on this transaction model, but you can kind of see how it's one thing to go from no sequel to something like no sequel transactions.
That's a pretty difficult step on its own, which is why it took so long, for example,
for MongoDB to get transactions, because Sondra doesn't even have transactions.
Then the follow-on work that we're now contending with is this realization that everything is
global nowadays.
Customers actually are, and in fact, their data sometimes needs to be domiciled and even different continents.
How can you still do transactions in a way that still works?
That's some of the novel stuff that we're working on now.
Let's say you have a single table, right?
Does geographic distribution actually show up latency within a single node?
That is, let's say I have a single node with some part of the data and one updates something.
I see geographic latencies in working in the context of a single node.
Well, if you just have a single node in a Cockroach cluster, which is obviously something that you could do,
it won't have any geographic latency because it's just one node, so it's only sitting in a single location.
It's kind of the right way to think of Cockroach is to think about the different stages of sophistication that you go through as a startup,
then growing into a bigger and bigger company.
So as a startup, you might think that, well, we're probably just going to have,
it's about standing this thing up, making the application work,
getting some initial users and failing fast.
And that particular case, you probably have one or three nodes of Cockroach in a single availability zone.
It's nice to have three nodes of Cockroach because you have three copies.
You can lose one of the nodes.
Sometimes that happens.
AWS will sometimes lose nodes, of course, from hardware failures,
or they'll relocate your VM or they'll shut it down.
You want Cockroach to survive even when it's in a single data center.
And you may actually want to scale it that way.
And that's sort of the next level of sophistication.
So now you've proved out your model.
You're getting lots of users.
You might still be in the same availability zone.
And now you say, OK, we don't want just three nodes.
We want to have 10 nodes.
And we can service a lot more requests in terms of the reason
the reason right throughput that's necessary to run our application for now
500,000 users or something like that.
And then what happens is your business, whatever it is,
starts to become somewhat mission critical.
Maybe customers are paying you or your customers, your SaaS business,
and your customers are other companies.
And they have SLAs.
You have to meet terms of uptime.
At that point, that's where Cockroach really starts to come into its own.
And you actually do spread the nodes out geographically.
So you potentially put them in three availability zones,
all in the same region as an example.
So it's very fast latency between these availability zones.
You're talking like less than 10 milliseconds.
But now you can lose a data center.
And there's always two additional replicas and two other data centers.
And together, they're able to have the single source of truth,
like the most recent data that's been written.
And then finally, you kind of move into this level of sophistication
where you start to have customers outside the United States.
Or you start to really care about the latency for customers outside the United States.
You might have had them the whole time, but they're always jumping over to Virginia,
or something like that from Australia, which hasn't been a good experience.
So now you're a company like, hey, we want to build something that looks like Facebook,
where everything feels local, no matter where you are.
So in that particular case, you're actually now using data centers all over the world
and replicating sometimes globally,
sometimes you're replicating only in the EU for an EU user's data.
Basically, there's a lot of different kinds of architectures you can build with Cockroach.
It's very flexible as you grow through these levels of sophistication.
But that should give you a little bit of perspective on what the geographic latencies
actually are and what they mean.
So if you really care about latency, you don't want any geographic latency at all,
then you put all your things in a single data center, which works fine.
But if you decide that you want to have additional resilience to lose a data center,
you have to introduce some latency.
That's the trade-off and it's very fundamental.
If you don't want to put all your things in one data center,
in fact, you might want to keep them like 100 kilometers apart,
which I think is a banking regulation.
There's a reason for that, right?
Because two data centers that are right next to each other,
they could be hit by the same earthquake or tsunami or whatever natural disaster,
a meteor strike or a fire or something that knocks out the trunk
that they're both using for fiber optics,
some backhoe and a construction project,
like all kinds of these random things happen.
So you do want to keep your replicas geographically far enough away
that they're what are called non-correlated failure domains.
But of course, that introduces latency.
So there's always a question of how much latency is OK,
and that depends on the use case.
It turns out that even replicating across the United States,
and let's say you have a high-value use case,
and I believe this is true of Google's AdWords system,
at least it was some number of years ago,
they actually had five replicas across the United States.
I'm pretty sure across three different US power grids.
And with a system like Spanner, which is cockroach is very similar to,
you can lose two out of those five replicas,
as long as you've got three remaining,
so three of those data centers across the US,
you have total uptime.
So it's a really cool system because this is actually not something
that never happens, where you put one data center
into a planned maintenance, so you take it down,
and then you have an unplanned outage in one of your remaining four.
But you still have complete business continuity in that case.
So it's a pretty robust, resilient architecture.
And in that case, your latency is usually
going to be about 35 milliseconds to do a transaction.
And for people that are used to building applications
that talk to the database, where the two are literally
machines on the same rack, 35 milliseconds seems like a long time to wait.
But the reality is that for the end-to-end latency that matters to the end user,
you really want to get things under 100 milliseconds.
And in people that worry a lot about the 35 milliseconds of latency
to do a transaction, often aren't really looking at the end-to-end latency
that their service typically does have.
Because typically for a user to get from their cell phone,
to the cell tower, onto the backbone, into the data center,
through the app server, into the database, and then all the way back out
through those layers.
And, god forbid, it's like a microservices architecture
that's doing all these things serially.
And in any case, these things can add up.
And usually, the 35 milliseconds is a bit of a rounding error
in terms of the total length of time that a user's waiting,
which these days is often anywhere from about 500 milliseconds to a second and a half.
The only companies that have really gotten that down are companies
like Facebook and Netflix and Google and Apple
that have built these global data architectures
and worry about that end-to-end latency.
But that's kind of where people get a little scared of the idea of geographic latency.
But it's really important to look at the bigger picture,
which is what's my end-to-end latency when I click a button in this mobile app
to when something happens on the screen after it's interacted with the back end.
And that's the sort of controlling critical path.
And then the beauty of having something that takes 35 milliseconds
because it's replicating to five data centers across the United States
is that thing is never going to go down.
So everything is about trade-offs and computer science.
And this is the sort of next generation of databases,
which is saying, OK, this isn't going to happen in two milliseconds.
You can make it work that way.
You can run it in one data center.
But that's not what you want for your business.
In fact, it's a horrible story for latency
when you have two millisecond transactions
because your database and your application server are right together.
When the users coming from Australia and your application server, microservices,
whatever, and your database are both in the data center in Virginia.
In that case, that user is probably going to have at least a second of latency
to get end-to-end on their service
because they have to jump all the way across the globe.
It just doesn't matter that you're getting two milliseconds to your database.
So there's a perspective shift, which is necessary
when you're thinking about these next generation platforms and architectures.
And I'll tell you, this is going to become really top of mind for developers
because what's coming down the pipe is 5G.
And 5G is going to make it so that the latency that's sort of experienced
through your mobile device can feel instantaneous
if the service that it's accessing or the application that you're using
is within the same region that the user's in.
If that all works well, then you can give them an incredible experience.
Like the way it feels when you type on your computer
and you see things immediately pop up and the letters pop up as you type.
If that didn't happen in less than 100 milliseconds,
it would feel weird and actually would be quite discombobulating.
So that's how it feels with the keyboard.
And people pretty much solve that problem.
It happens in less than 100 milliseconds.
Right now, the average user is very much trained to expect
a really slow experience when they're online on a mobile device.
But as that begins to change, applications that don't change with the times
are going to feel somewhat antiquated.
You mentioned that developers are going to start seeing this.
They're used to seeing a single node database.
And one thing I noticed was that Cockroach is compatible with Postgres.
Can you talk about that?
Yes, we early on decided that back up just one, maybe a couple months earlier.
I'd been working on a design document for Cockroach.
And that design document got fairly wide circulation.
I don't know exactly how it got shared around.
But it ended up on Hacker News.
And people were pretty interested in it.
That early version, it talked a lot about what we're talking about right now.
It did not talk about SQL.
And in fact, I'm not exactly the world's expert on SQL to put it mildly.
And I wasn't super interested in it.
But I did realize that SQL was the, it's called the lingua franca of database systems
and was heavily in use in most of the world's big companies.
So when we actually started the company Cockroach Labs in order to accelerate the
development of Cockroach DB, which was just a GitHub project at that point in time,
we came to the very early conclusion that, okay, great.
We've been thinking about a distributed transactional key value type, no SQL database.
But in fact, it's just another layer to add SQL.
And SQL is a massive market.
Like we're going to make this a commercial entity.
And we eventually want companies to pay us.
We should think about what those companies are actually using.
And while no SQL at that point in time was, you know, very popular.
And it seemed like it was for a lot of people, okay, we don't even need SQL anymore.
That proved to be a bit of a head fake.
And I think we just had a little bit of luck and a little bit of insight that,
you know, listen, the luck was probably mostly that we'd been working at Google for a decade
before. And we'd seen them go through the exact cycle that the bigger ecosystem was going through
in 2012. They'd gone through it in 2004.
And they realized, okay, these no SQL systems need more.
They're not good enough.
And eventually it came back around a SQL.
And that's what Google uses now.
So it was just, you just realized that no SQL was really about scale.
And then they started adding things piece by piece transactions first.
And then other capabilities.
And came back to the realization that you would need SQL.
So that's a little long-winded.
Sorry. But you know, we early on decided, okay, SQL is the right thing for us.
And, you know, what flavor should it be?
And I think we originally talked to Google about doing some sort of open SQL thing that
could be shared between Spanner and Google.
But, you know, we just had different timelines between the two companies.
And so that didn't really work out.
It wasn't good enough communication.
And Postgres is what we settled on.
It's like, why would you create your own dialect in this day and age when there's so
much usage between MySQL and Postgres and Oracle?
Couldn't really do oracles.
Like, we didn't think it was the right thing for an open source database.
Also, Oracle's a very, you know,
a company that likes to use the legal system.
So we didn't really want to get caught in those gears.
So Postgres seemed to be the more considered approach to SQL.
And MySQL is more seat of the pants.
And there seemed to be less mistakes in Postgres.
So we opted for trying to get reasonably compatible Postgres.
And part of the thinking there wasn't just, okay, people know,
large number of developers know Postgres already and they know the dialect.
But also we wanted to work with the tools and the ecosystem.
And there's a lot of those.
And it was, you know, quite important to make sure that we integrated with as many as we could.
Most developers, I'm sure people listening to this show are very much included and including myself,
you know, having to code directly through JDBC or ODBC or whatever you're using.
You know, it's not so pretty because especially the results you get back from the database are
in a particularly annoying format to then convert into the sorts of objects or structures that
you're using in whatever programming language is your choice.
So ORMs and things like that are just critical time savers.
You know, usually they're auto generating code that just takes care of all of that
connective tissue and busy work.
So it's a huge win.
And, you know, building our own drivers and making them work with all the tools if we had
something that didn't look like Postgres be possible.
But just another huge amount of work that we thought, you know what, let's piggyback
on what Postgres has done, right?
If I'm using Postgres, I'm probably used to say expecting something from a single node database.
But here, Cock
True just kind of like a distributed database of sort.
Is there like eventual consistency?
Is there a difference in how I can expect what is visible, what's not visible?
Then Cockroach, there's no eventual consistency.
And every node looks like the whole database.
So if you're talking to one out of 100 nodes and you're asking for data,
that node most certainly won't have the data you're looking for.
Unless you can plan.
So I want to put a big caveat on that.
You can do a lot of things with Cockroach to make things very, very efficient.
If you know what you're doing and you have a real need for it,
but in the general purpose case, every node will do forwarding and so forth.
The other nodes to find the data.
And this is true in MongoDB and it's true in Cassandra.
And it's, you know, all of these distributed systems try to make the lives easier for
application developers.
They, you don't want to have to know exactly what node to connect to to get a particular
piece of data that's just too inefficient, too difficult.
So yeah, I mean, Cockroach, the way to think about it is every node is
a gateway into the larger database that will serve you a consistent view of the database.
Interesting.
Just to add on to that, something you mentioned earlier was that for compliance
reasons, people may want to keep data within a single geographic region,
say GDPR or something.
Is that something that you define when I create a table, for example,
say this range of users should always be in this continent?
Yes, that's exactly right.
It's a, there's a feature in Oracle and Postgres and I believe MySQL has it too,
SQL Server.
It's called partitioning, table partitioning.
And you can specify a column that you'd like to partition the data on in a monolithic system,
like those others I just mentioned.
Usually you use partitioning in order to put certain kinds of data on less expensive
disk drives, for example, on the same database server.
Or to make it so that the most recent data is on its own partition.
So you can easily back it up and restore it without having the full archive
worth of data to constantly back up.
So partition has been around for a long time.
What we introduced is a sort of new twist on it, which is called geo partitioning.
And you can take those same table partitions and you can assign constraints to them.
So I'll just give you a concrete example.
Let's say that in your data model, you include a column in this table that you want to partition,
which is region, maybe it contains strings like eu.de for Germany and eu.fr for France and us.ny for New York.
You could say that for anything that starts with us, the partition should be replicated only
within data centers in the US.
You might say for any US states that are the Mason and Dixon line that they should be
replicated to the three data centers that you have on the East Coast.
And similarly for West Coast and so forth.
And similarly in the EU, you can do the same thing.
You might actually also say, well, Germany, this is PII financial ledger or something like that.
In Germany, you might have to keep all of those replicas only in Germany.
It's like a more specific regulation than GDPR.
And you in that particular case would say anything that has eu.de in that column,
any customer data that's German, in other words, should be replicated only within Germany.
Maybe you've only got one data center in Germany, all three replicas are going to be in that one data center.
That's just what you're going to do.
So it's not super resilient, but you're maintaining your legal responsibilities according to German law.
In the broader context, can you talk about what happens in the presence of failures and partitions?
You know, let's talk about say CAP.
There's a partition, is CockroachDB available?
Is it consistent?
Where do you stand?
Yeah, well, it's consistent, of course.
Available is an interesting word when you talk about the CAP.
It doesn't mean what people think available means.
Available when you talk to most people, they're talking about high availability.
And how many nines do we have?
And Cockroach is highly available in the sense that, you know, it has multiple replicas.
There's a consensus algorithm that does the replication, which maintains consistency.
As long as a majority of your replicas are available, I don't want to use that word again.
Let's just say, reachable, they're communicating.
Then you're able to access your data, read and write it and so forth.
And you can increase the number of replicas and the number of data centers.
And as I mentioned before, how far apart the data centers are, for example,
in control, how many nines you get?
I mean, you can make your nines go out arbitrarily far.
So if you have five replicas across five data centers, and they all have the same probability
of failure, that's better than three replicas across three data centers.
If you have seven replicas across seven replica data centers, that's even better than the five,
right? And so on and so forth.
So you can control exactly what your SLA guarantees are in terms of what would I call it, like,
estimated probability of availability of the data.
So that's the highly available thing that people actually care about in practice.
When you talk about available in the CAP theorem, it means that a node is reachable and on.
And if that's true, it should be able to serve the data that it has.
So in other words, it's sort of like any node off on their own in a system that is, for example,
AP in the CAP theorem, that node on its own, no problem.
It can serve the information.
So that's why it's not consistent because it can kind of go AWOL and be split brain or
whatever you want to call it and serve up super old information or information that hasn't been
corroborated by the other replicas.
So it's maybe tentative information.
So that's the AP side.
Cockroach by contrast, you must have a majority of the replicas.
So it maintains consistency through that mechanism.
But what it means is that if you don't have a majority, let's just take three replicas.
So you've only got one replica left.
That replica is available by the CAP definition available.
And so in an AP system, that replica could just give you the data, which could be wrong.
All right, and cockroach, if you just got one out of three nodes, it can't tell you the data.
It will refuse to because it hasn't has no way of knowing that it has the right data when it
can't get consensus.
So, yeah, I mean, it's a, I think it's a very common misconception when you talk about CAP to
believe that a system that maintains consistency, like cockroach, can't be highly available.
I mean, nothing's further from the truth.
The whole point of cockroach is that it is highly available.
It simply means that singular minority nodes in cockroach that can't get consensus,
they can't operate independently.
Well, initially you mentioned that cockroach is like a single deployment where you have a single
binary that's common across all data centers or regions.
How do you go about upgrading your databases?
Do you bring nodes down and then rejoin the cluster?
Yeah, exactly.
If you have a single node, then you're not going to have a very seamless upgrade process.
You just have to take that single node down and bring back up.
There's no redundancy there.
With three nodes, it's very simple.
What you do is a rolling restart.
And we have an upgrade process that lets you, because as you can imagine,
as you have successive major versions, there are things that change in the underlying data
model and so forth.
So, there's kind of a point of no return if you migrate to a newer version.
But what we do is we kind of allow that to happen in two steps.
So, the first step is you might get access to things that don't need to change the underlying data.
And you can try all those things out, run your system and so forth so that it looks pretty much
like there's no incompatibilities with the previous version, but you're trying the new version.
And then you kind of flip a switch, which commits you to the new version.
And then you can start using the newer features that might write data that's not compatible with
the old one.
This allows you to go backwards.
It's very important when you're doing an upgrade.
So, with Cockroach, what you do, like practically in terms of the deployment,
you have three nodes.
They're all in version one, let's say.
You take down node one.
You can actually give it a command that sort of drains the traffic off it gracefully.
And then when that decommissioning is done, you take it down,
you start the new binary, it starts back up, it joins the cluster.
Now you've got one out of three that's running version two, let's say.
And now you've got three nodes again.
They all have the data they're replicating and so forth.
You know, when you upgrade the version, you still keep the data that's on that node.
So, it's not, it's usually a fairly fast process to put a new binary on there.
And not usually it always is.
And then you do it to the second node and you do it to the third node.
Now, at this point, you have three nodes that are all version two,
but they're not necessarily using any non-backwards compatible functionality.
And that point, you sort of flip the cluster wide thing.
And you can only flip it when all the nodes agree that they're at version two.
And then that upgrades you to the sort of, okay, now we're fully in version two.
And we have all the features available in version two.
Let's talk a little bit about the open source aspect of CockroachDB.
One thing I noticed that is that it's written in GoLine.
So, what led you to, you know, make the choice and how has that been for database development?
Well, I actually think on balance, we're very happy with Go.
It's performance is improved.
In fact, you know, in the first three years of Cockroach,
every time a new version of Go came out, we get like another 15% performance.
It was pretty cool.
Just kind of get that for free.
And the Garbs Collector works pretty well.
The reality though is that Garbs Collector is,
they're an abstraction that you end up having to break if you're building something like Cockroach.
Luckily, you only have to worry about what the Garbs Collector is doing
and how many allocations you're creating that are creating pressure on the Garbs Collector.
You may have to worry about that in certain critical areas of your code.
So typically what we do is we just program normally in Go and don't worry about that stuff.
And then when we start to really do performance optimization,
we look at the allocations and we cut those down in the critical path,
the critical sections of the code basically.
And that's been a very effective strategy, but fundamentally you are breaking the abstraction.
Like you shouldn't have to worry about the Garbs Collector.
But you do in Go, but luckily you can limit it to these small sections.
Rust by contrast, which would be the other language that I think if you pull our developers,
that would be also a pretty heavily favored contender.
I think there's a bigger upfront cost to learning how to program in it and learning how to program
well in it, but you don't have those problems with the Garbs Collector.
So I think if you pulled our engineers, you might get about an even response between those
two languages, but this I think is true in most projects.
Once you start down a road, even if it's Python and you're building a database,
it's pretty hard to back out of it.
So we chose Go originally because it's what was available.
Rust wasn't available back in 2014 when the project really got started.
And Go, I'd been programming C++ for a long time before that.
And that would have been a reasonable choice at Google where they had lots of RPC libraries
and so forth.
In the open source world after Google, it wasn't clear that the C++ libraries were
going to make the job very easy.
It just seemed like there was a huge amount of complexity.
Like the boost library for RPCs was you look into it and you almost start to cry.
It's so big and so complex and over-engineered.
And so Go seemed simpler.
It had very straightforward libraries that were all sort of system libraries.
It turns out we've replaced many of those with things like GRPC over the years.
So not much of that original architecture remains.
But at the time, 2014, I think Go was the best choice for building a new distributed system.
That's interesting.
So what's coming in CockroachDB?
What's the future?
So this is definitely my favorite question.
So I think it's important to think a little bit about open source and what's changing in the world.
In the OTS, open source was as it grew as quickly as it did because it was creating a much
more convenient model for consumption.
I mean, back in the day, open source, I'd say the number one cool aspect of it was that the
code was available.
You could do whatever you wanted with it.
So it was that real like Richard Stalman freedom type thing, which still exists, of course.
But that was back in the 80s and the 90s.
That was the really great thing about open source.
And it was so different from the closed source models.
But in the OTS, what really started to happen and the reason that open source became the dominant
computing paradigm or software engineering paradigm is that developers used to have to go
through procurement.
I mean, for anyone that doesn't work in a big company, this is like a hideous process where
you're going to talk to enterprise software sales reps.
And there's going to be months long legal review and contracts and you're going to get,
I mean, we do it when we sell Cockroach to these big companies.
It's unbelievable.
But the way that you had to get all of your software, that you had to use to build whatever
new application or service was to go through this unwieldy, terrible process.
And once you've got one of those things, you've got these printed manuals.
There was no community that you could ask other like-minded developers how they solve the problem.
So that all changed with open source.
And that's what vaulted it into being the dominant paradigm.
But now things are changing again as they do, right?
All these ecosystems change.
And they seem to be changing faster every year.
Now the consumption model is moving to services.
So people, it's great.
You could just download a piece of infrastructure, a new developer library and open source.
You'd have to learn it, no problem.
But with something like infrastructure like a database, you then have to set up the monitoring
and the deployment, the day two operations and the amount of cost related to that
is actually pretty excessive.
And even as a developer running it locally in sort of maybe a sort of pre-production environment,
there's a lot of overhead to learn those things.
And what Amazon of course has discovered is that providing infrastructure as a service
is a pretty big win-win.
And people are willing to pay for it and the total cost of ownership and certainly the time to value.
Like how quickly can you create something using AWS and all of the services they provide?
The answer is much more quickly than when you had to download the open source
and learn how to run it yourself and run it in your environment.
So it's a big step forward and it's putting a lot of pressure on open source
in the open source model.
You know, what we have to think of when we're building a database is what's the
there's kind of two sides to the equation.
One is what do we have to do to make the Fortune 500 customers happy?
And then what do we have to do to make it so that developers find cockroach the easiest,
most frictionless database to, you know, especially get started with?
And this is really important to look at what MongoDB has done and done very well.
You know, they made developers lives easier.
And so that's something that we're trying to replicate in our own way.
We're obviously not going to get rid of SQL.
That's a big part of what cockroach is.
But we do need to make developers lives easier in order to really solve for that huge audience
of developers out there that every time they start a new project,
they're thinking about, you know, what's the infrastructure I'm going to use to make this thing work?
And what we're realizing is, okay, the average developer out there,
especially one that's in a smaller company,
but even ones that are in big companies on teams that are doing new projects,
they're looking not so much at databases.
They have to run themselves anymore.
They're definitely want services.
And I think what's really wrong with the Amazon model of services,
I mean, there's Amazon's other things wrong with it.
But the thing that I think's wrong with it is that it's actually still pretty heavy weight
to get a relational database.
And cockroach isn't any better right now.
We have a database as a service and it looks like RDS or like Aurora in terms of how you
provision, you know, how big the nodes are, how many nodes do you want.
And it's actually fairly expensive if you're just trying to create a hobby project or something
like that, you know, even the cheapest RDS is not something that everyone wants to put
their credit card down month after month for.
So I think what we're really looking at for our future is how do we bring
cockroach into the cloud in a way that creates the next consumption model,
which to my mind is to make relational databases truly serverless in the sense that
you no longer have to say, I want this many nodes with, you know, these kinds of CPUs and
this much memory.
I need them to be in US, East and EU West.
We don't want developers to make any of those choices nor do we want them not to pay for full
full VMs.
What we'd like to do is make it so that what you get is just a cockroach cluster and that
cockroach cluster is free.
And in fact, it's perpetually free.
So you'll be able to get a perpetually free relational database that can scale up to pretty
decent resource utilization thresholds and still be free.
So let's say that's somewhere between 100,000 and a million requests per day and it can burst
up to pretty hefty amounts.
So it's a nice database, certainly for any kind of hackathon or hobby project or academic
coursework or anything like that.
This is one that you can rely on is always going to be free, kind of like the niceness that
Gmail brought to the email landscape back in 2003 versus Yahoo mail and Hotmail and those other ones.
We even think that you should be able to get these things in such a frictionless fashion that
you don't have to log in to get it.
You can just create a relational database anonymously.
And so to make that all possible, we have to virtualize cockroach.
And the big realization here is that when you take even the sort of smallest size VM,
you know, and it doesn't necessarily run the database that well, but even when even what it
does do is much more than the average project needs or, you know, if you're just doing a little
bit of an evaluation or a test, you're developing something or it's just a little thing that you
standing up that, you know, some of your friends are hitting or you know, you're just doing a hobby
project, those kinds of things don't require the literally billions of requests a day that
even a small VM can handle.
So it's really about realizing, okay, for every VM that's created for one of these hobby projects,
there's lots of wasted cycles.
So if we can virtualize that, then we can actually lower the unit economics and give,
you know, allowed databases to be truly free as services, you know, up to some resource utilization
threshold. So that's a really cool project to be working on because we get this really
pie in the sky opportunity to do something that actually moves the ball forward in terms of how
easy it is for developers to get what they're trying to get done.
And, you know, relational databases are at the heart of most applications around the world.
So it's, it seems like a big opportunity.
Yeah, that seems, that seems like a very interesting way to go about it.
Well, with that, I'd like to wrap up Spencer. It's been great having you on the show and talking
about CockroachDB. Thank you for coming here.
It's my pleasure.
This is Akshay Manchale for Software Engineering Radio. Thank you for listening.
Thanks for listening to SE Radio, an educational program brought to you by IEEE Software Magazine.
For more about the podcast, including other episodes, visit our website at se-radio.net.
To provide feedback, you can comment on each episode on the website or reach us on LinkedIn,
Facebook, Twitter, or through our Slack channel at seradio.slack.com. You can also email us at team
at se-radio.net. This and all other episodes of SE Radio is licensed under Creative Commons
license 2.5. Thanks for listening.
