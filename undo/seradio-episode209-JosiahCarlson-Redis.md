**Link**: https://www.se-radio.net/2014/09/episode-209-josiah-carlson-on-redis/


This is Software Engineering Radio, the podcast for professional developers on the web at
sce-radio.net.
SC Radio brings you relevant and detailed discussions of software engineering topics
at least once a month. SC Radio was brought to you by IEEE Software Magazine, online at
computer.org slash software.
For Software Engineering Radio, this is Robert. I'm joined today by Josiah Carlson. Dr. Carlson
has his PhD in Computer Science from the University of California, Irvine. He's been a software
engineer and architect at several technology companies among them, Google and YouTube,
the focus on architecture, design, algorithms and distributed systems. He is the founder
and chief architect of CHOW Now and is currently an advisor. He is now the chief architect
at ZECONOMY. He is also the author of the book Redis in Action. Welcome to Software Engineering
Radio, Josiah.
JOSIAH RODRIGUEZ Thank you. It's good to be here.
Today, Josiah and I will be discussing Redis. Let's start out with what is Redis?
JOSIAH RODRIGUEZ Well, Redis is an in-memory key data structure server. It is most commonly
compared to a system called memcache D, which stores string keys to string values and where
Redis majorly differs is that it maps from string keys to data structures. You can have
one of several types of structures among which being plain strings, lists, sets, hashes and
what are called sorted sets. There recently has been an edition that offers a type of
aggregate or counting unique things statistically called a hyperlog log that was added. But that's
fairly new, literally in the last couple of weeks. Not that many people have used it yet.
JOSIAH RODRIGUEZ You said Redis is a key data structure server. How does that differ
from a key value store?
JOSIAH RODRIGUEZ The primary difference is the types of things that you can do to your
data. You know, a typical key string server, called a math, you can basically get or set
values. Sometimes the values can be incremented, like say an integer or a floating point number.
And sometimes you can even append data to your string so you can keep making your string
longer. With Redis, you can do all those things too. But with the additional structures, say
lists, you can also push and pop from both ends of the list. You can look for items in
the list. With sets, you basically get a unique mathematical set concept. So if you want to
say you add the string hello twice, there will only be one in it. For hashes, it is a typical
hash that matches the values again. And it has all the typical set interactions or I'm
sorry, hash interactions that you would expect. And sorted sets are a little bit funky in
that they behave a lot like a hash where you can also access your items by the sorted order
of the equivalent of a value which are called scores. And so it offers different access
characteristics. It needs one of them to offer different read, write and access characteristics.
So Josiah, you are talking about the ability to modify these data structures server side.
What is the thinking behind server side data structures rather than client manipulating
data structures and pushing them back as an opaque value? So primarily it has to do with
the amount of data you need to transfer. Say for example, you are using strictly memcached
D and you could send and receive strings. Certainly you can store more or less arbitrary
data structures with JSON or message pack. And a lot of people do that. But then anytime
you want to change the structure, first have to read the whole structure, modify it and
write it back. And then in that process, you have to somehow lock that data so that somebody
else doesn't read it, modify it and then write it. While you can certainly do that, directly
manipulating the objects themselves is a lot more efficient in terms of CPU time because
you are not encoding and decoding. A lot more efficient in terms of network bandwidth because
you are not reading and writing the full value. You are just modifying small pieces.
That makes a lot of sense. In your book, you work out a number of interesting examples
of how Redis can do some very powerful things. Could you walk us through a small example
that shows how you might use one or two data structures to develop an interesting feature?
Certainly. So one of my preferred patterns and how I deal with storing and sorting data
is one that is already built in the Redis. It is pretty common. That is the use of a set
and the use of a collection of hashes. Now generally speaking, a lot of people will use
a hash as sort of like the equivalent of a document in a document store or even a row
in a relational database where each field in the hash is the equivalent of a column or
an attribute. People store their data like this and this is great. You can fetch the
whole thing. You can fetch into these fields. But what you really get and you really start
thinking about what you can do, you can then start sorting data that you have if you were
to put an identifier for each of your hashes. Say for example, your hashes are named, user
colon and then an ID number as the key. Then you have a set with each of those IDs in it
and say it was stored as users colon and that is the key name. Then what you can do is you
can get a sorted order of any arbitrary column equivalent on your hashes. You would say sort
this set by and then you can actually provide a string. That string could be depending on
the column. Say you want to sort by email address and you got email in there. You could
say user colon star which is a wild card and then you can draw and you can type a hyphen
and then the greater than sign to create a small arrow. Then for the field you type email.
If you say sort that set by and you provide that string, that tells Redis to sort that
set by fields referenced in other hashes. If you also provide the final word of alpha
to tell it to sort alpha numerically, you then get a sorted, then the items that are
returned by that sort call are ordered by the email. This is all built into Redis already
and these are known patterns. But it's still really useful if you to offer effectively
arbitrary sorts on your data even when you've not set up an index, you've not set up anything
special, but you still get the ability to perform ad hoc sort queries which is very
powerful and it's something that not everybody would necessarily expect.
So bring up another point in that example. Does Redis offer a wild card expansion on
a key substring?
It depends on the context. Right now the only situation where you get it strictly in this
kind of thing is with the sort command. Now there are other commands like you can, Redis
also supports a pattern known as publish subscribe where you can publish the channels and we
can subscribe to channels to receive messages. There is a pattern subscribe call which lets
you subscribe to arbitrary patterns and you can provide the wild card character asterisk
which will then match anything. And you can also use that if you're looking to find keys
using the keys command or using the leave it as S keys or something similar, a new command
to actually traverse the known scan also takes wild cards but those are, that's not so much
expansion as much as it is an out card.
You mentioned a new feature, the hyper log log which I'm not familiar with. Tell us what
problem that solves and how it works.
Certainly. So one fairly typical usage of Redis is for say for example unique visitor
counts at web pages. One way of doing that, not necessarily the best way but one way of
doing that is when you assign a cookie to your visitors to even if they identify them, whenever
they visit you then say for example add them to a set because sets only contain new items.
When you add their user ID to set they're only in the wants. So if you say create a set
and use it for one week then you get unique visitors for that week. This works great.
Of course the problem is that you need to use a bit of memory for every person that
visits and so for large websites that would require a lot of space. There are a bunch
of different ways that you can change it and prove it and reduce the amount of memory that
you use. And hyper log log was recently added to offer this. It's not just for counting
website visitors. What it can do is when you add items to your hyper log log it doesn't
add them. It statistically adds them. It computes a hash on the item that you're adding
and flips and bits in a small structure and then it uses how many bits have been flipped
to determine an estimate on the number of unique items. Salvatore spent a bunch of time
analyzing it and figuring out how you can optimize and somewhat minimize the error. And
I think right now for the large sets that is for large groups of items that you'd like
to count it is within one or two percent of the true actual value if you were to calculate
it by adding to a giant set. But it only takes up I believe around 12 kilobytes. So for 12
kilobytes you can count to really big numbers. It sounds something like a bloom filter if
I understand it correctly. Yes it's similar to a bloom filter. The difference from being
that a bloom filter is meant to be significantly larger and store yes or no this item is in
it or no this item is not in it and that's not really what a bloom, what a hyper log
log is supposed to do. But a lot of the same ideas go into making them both work.
In some of these examples you're giving they involve multiple data structures which naturally
leads to questions about a consistency model and atomic operations. What kind of models
does Redis provide for those considerations? Individual commands in Redis are themselves
atomic. If you push an item onto a list you know another process pushing an item onto
the list is not going to replace the item that you just pushed because they happen at the
same time. Redis is single threaded so each operation individually is atomic because only
one thing can happen at one time. For multi-command consistency you can set up what are called
transactions. Now there are two basic types of transactions. One is just a strict, we generally
call a multi-exact transaction where you say do these things altogether when exec is called.
So you say multi you put all of your manipulation commands in and then you say exec and when
exec hits all those commands are executed one right after another without any interruption
by other connections or commands. You can also perform conditional transactions using
watch. So the typical thing is you say I want to watch these keys for changes and then you
get data from them if necessary and then you type multi and then you perform your set of
operations or and then you send your set of operations and then when you say exec what
happens is before the multi-exact portion is executed any of the keys that you said that
you wanted to watch at the beginning if any of them have changed then the multi-exact transaction
is actually forwarded you get an error message saying hey these keys have changed and then
if you like you can retry. If there were no changes then it will execute and you will
get the response that you were expecting at least no errors anyways. And then Lua scripting
itself lets you do somewhat arbitrarily complex data manipulations and the Lua script itself
is considered more or less one command and so is also executed atomically. There are
some caveats when it comes to if you're in a Lua script and you're accessing a key that
has just been expired but it's not all that surprising.
With Redis being single threaded if you send a batch of commands and executes them all
on the server there's not really any issue of whether it's atomic or consistent it will
all happen at once. This optimistic locking model it must be designed to deal with environments
where the client is going back and forth a few times before committing.
Yes so it is indeed the case that the conditional execution or this optimistic execution those
require a few round trips. I believe that it was originally designed like that because
it's a lot easier to make what are called deadlock free algorithms when you don't explicitly
lock data from modification. One thing that happens periodically in relation to databases
if you are not careful is that you have say for example you have two clients. One client
locks row A and tries to lock row B but second client has already locked row B and tries
to lock row A and then can't. They're in a deadlock situation because they want to lock
data that the other one is already locked. That's very dangerous but with the optimistic
locking or with the watch these things if they change stop it otherwise let it go you
don't have any deadlock scenarios. You just keep retrying until your data modification
is that true. From an execution standpoint it allows Redis to be more simple even if it
takes a little bit more work on the client. In your book you give some examples of building
application level locks. What is your thought process on when you use an application level
lock and when you use the native locking that's supported by the server? Generally speaking
I look at how complex are my data interactions. For Redis style transactions I will typically
use them when the type of data being accessed is simple and my accesses are simple and when
I only need to manipulate maybe one or two structures in Redis. If manipulations are
simple then the number of structures that I'm manipulating are one or two then I can expect
that the number of retries will be low. If the number of retries are low then I can feel
pretty good that it lets you fast and efficiently and I won't have any issues. When I need to
manipulate things in Redis where there are perhaps many structures that I'm manipulating
or where I need to fetch data externally after locking data in Redis then I will use an application
level lock that is provided in the book. I use this because when your data manipulations
include things exterior to Redis then Redis is not going to be able to watch them obviously
and when you are manipulating many things you will increase the likelihood that your
optimistic locking is not going to succeed and so you will have to retry. In various
situations some of which I bring up in the book like the marketplace using a lock is
substantially faster and so sometimes I look at it in terms of performance as well. That
said locking itself if you use a Lewis script many times you don't even need to lock and
if you can embed all of your data manipulation with single Lewis script then you don't need
to lock, you don't need to use conditional locking, you don't need to use optimistic
locking, you don't need to use Redis transactions, you just use a Lewis script and almost always
get what you are looking for and incidentally because those are typically just one round
trip you also end up performing faster as well.
You mentioned the Lewis scripting a couple of times, I'm aware that Redis natively supports
Lua on the server side, tell us more about the language Lua first and then I would like
to come back to what you can do with it that goes beyond the basic Redis commands.
My exposure to Lua is primarily through Redis, years ago when I was playing Warcraft I got
a little bit of Lua experience there but Lua itself is primarily an interpreted programming
language, it's fairly dynamic but also fairly simple, it is designed to be small and so
it is very often used as an embedded programming language in video games and applications and
in this case in Redis itself as a description language.
The variant of Lua that is included in Redis is Lua 5.1 which can be replaced with Lua
JIT which is a just in time compiler for Lua but it is not included with Redis by default
because the interpretation leads to more consistent performance even though Lua JIT can execute
faster in some cases.
While I personally have been a huge Python guy for many years and though Lua is missing
some of the features that I'm used to, I've personally not having problems using it and
so far I've actually advocated for the converting of as many commands into Lua based commands
as possible just to allow for better understanding and better remixing from others but so far
nobody has bitten on that.
And is the primary use case of Lua so that you can do series of operations on the server
without as many round trips and therefore reduce the number of round trips but also the locking
or...
I'm not sure whether it's strictly for that.
I think it's just for the sake of flexibility.
Many relational databases will have internal scripting languages, PL SQL being one of the
more common.
You take Postgres and it's got more than a dozen internal programming languages that
you can use Python, Perl, PHP, Lua, Java and various others and so the concept is not so
much new.
Other databases have embedded programming languages in them.
What this brings to Redis is primarily just flexibility.
If you want, you can use it to do huge data updates in a single round trip.
You can embed some level of your program logic.
A couple years ago at RedisCon, this would be a year and a half ago now, there's a group
called AndYet that had actually built their data validation and scheme of validation
for Redis as effectively their full time data store.
Redis updated all of their data type verification and constraint verification inside Redis using
Lua scripting which is kind of mind-boggling but they did it because Redis doesn't support
data verification by default and they didn't want to write it on the client.
I thought that was very interesting and very innovative to use Redis to verify its own
data internally when it normally doesn't, at least not the extent that they need to.
I want to switch over a bit talking more about the memory and persistence issue.
I think of Redis primarily as an in-memory data store with some persistence capabilities
that we'll get to.
Is that a fair characterization?
I wouldn't call it unfair.
It is certainly the case that by default the Redis that is shipped is everything in memory.
Older versions had what was called virtual memory interface where it would swap unused
data out but that wasn't very efficient and got no love for many years so it was removed
and Redis 2.4.
Okay, now as an application architect, Josiah, how do you think about the role of an in-memory
server in your entire system compared to where you would use a database meaning something
that has a durability property on commits?
So I typically personally and when I give people advice, will tell people to at least
start with data that is perhaps less important to keep 100%.
Say for example if you're running a web service, one of the first things that I recommend
that people use is to use Redis to store their user session cookies.
Typically in web properties, you use a cookie to store user identification information along
with many other things but every time a web request comes in including for your static
assets and anything else like that, you have to send the cookie.
If you have enough data in your cookies, that can actually be measurable, especially on
mobile.
But if you have say a randomly generated cookie identifier that then maps into Redis
where you store arbitrary user data, then you can minimize the amount of data transferred
by your clients while also storing arbitrarily rich information about them inside Redis.
Then you're only concerned if, well, what if Redis dies on me, what if I lose all of
my information, what if I'm not backing up my data, what if I'm not replicating my data,
what if I'm not keeping copies of my data somewhere else just in case, well, you've
just lost your login cookie.
Start up in your Redis and then people have got a login again.
Not that big a deal.
And I find that that's usually a good introduction to Redis for a lot of people that want to
use it.
And then from there, after you see how fast it is, how well it works, how once you get
used to it, then you notice, wait a second, Redis hasn't crashed on me.
The only issues I've ever had with Redis is not done with Redis.
It's been with the server that Redis runs on.
If you're a cloud provider, sometimes those problems are from law often.
But typically that's the way it is.
Redis is pretty reliable because of software and it is constantly being updated and fixed.
And it's been very, very reliable for me.
I've been using it for a little over four years now and I think I've only run into two
or three situations in those four years where Redis has ever had an issue where I've had
a problem.
You talked about using Redis in situations where the data is not absolutely essential
to preserve every piece of it.
Maybe if you lost a little data, it wouldn't be a disaster because you're giving up the
durability property from an architecture standpoint.
It has to give you back something in exchange for giving up durability.
And what is that?
Primarily what Redis gives you for reducing your durability is a data model that is not
currently replicated in any other database.
The rich data structures that are available in Redis lets you model your application like
you do in your typical programming language.
What do you use in your typical programming language to model data?
Well, you've got lists, you've got hashes, and you've got strings.
Those are the typical things.
Now, sets are less common, but they exist in some programming languages like Python.
And sorted sets are not generally available in most programming languages, but you can
use them to build sorted indexes.
So that's what most people do.
But given these structures, you can structure your data in Redis like you would in your
application.
And what you can do is you can just use Redis as your giant shared memory pool.
Instead of having all of your user object that you're manipulating, say you're incrementing
their visit count or you are keeping a history of the recent pages they visited.
Instead of inserting a row in a database, you can just push an item to the end of the list.
And if you want to cap that number of items at, say, 50, you can then trim the list.
Can you imagine doing this in a relational database for your instance?
I'm not a lot.
Exactly.
It's not viable, not even by a long shot.
But because the structures in Redis, you can do this.
You can model your application the way it fits in your head.
If it just so happens that your data model is not just rows and columns in a relational
database, you can certain map a lot of those things into Redis.
And I've actually done so.
I've used a lot of the ideas that were in the book.
I built a Redis object map before Python that embeds a lot of the concepts of relational
database minus the relations.
But there are still minimal relations that you can perform and actually perform a sort
of join between these things.
I confess I had an expectation in asking you the last question that what I was going to
get back was it's an in-memory server.
So low latency is the main thing where you have some kind of state that needs to be shared
in a distributed system, which is largely stateless.
That's only possible with a low latency.
Is the assumption of low latency implied by your response?
No.
The additional features that Redis is very fast and that generally has low latency because
everything is in memory and not doing data from disk.
Most people go to Redis because it's fast.
We get questions on the Redis knowing list all the time asking, hey, I heard that Redis
was this really, really fast thing.
How do I use it?
How would I model this thing?
I understand that people are coming at it because everybody wants things to be faster.
And it certainly is fast.
And that's one of the first things they got into it.
But after you've been using it for a little bit, then you realize, wait a second, the
speed brings me in.
But what keeps me here is that I can build rich data models.
It reduces the constraints in your modeling in some ways by just giving you different
opportunities.
Certainly the speed is terribly important.
I think that if Redis was 1% of its current speed out, really think that anybody would
use it because it would be practically too slow.
But as it is, it's fast enough and you get faster than a lot of other software equivalents.
And it offers the rich data model that helps you in actually getting your application built.
I know I could go on and on on this direction, but I want to switch to discussion of persistence.
Redis offers two different persistence models, BG Save and AOF.
Could you tell us about what those do and how they differ?
Sure.
So BG Save is actually the command that you execute to initiate a background snapshot.
Snapshots are point in time snapshots of your data.
What happens during a snapshot is Redis forks itself using the standard POSIX fork call,
which creates a copy of the process.
And then the child copy will then examine all the data in Redis and dump it to disk into
a snapshot file with the extension RDB.
And after that is done, then it remains the file overriding the old snapshot and then returns
back to the parent process, which then now knows that all the data has been saved.
And so records information about that so that you know when the last time it was saved.
You can make this operation automatic with the Save configuration and there you go.
So the nice thing about that particular form of persistence is that if you need to have
your data at one point in time, you can get it.
The snapshotting mechanism is also used to initiate slaving replication.
I'm probably getting ahead of myself, you could probably ask about that soon.
On the other hand, the AOF or appendaly file that operates is anytime that there's a right
operation to Redis, Redis will write that right command to disk into this appendaly file.
It behaves very much like a traditional database's commit log or write a head log or sometimes
called intent logs.
It behaves very much like that.
The only real thing that you need to be concerned about there when using it is how often you
want to sync the data to disk, how often you want to make sure that data definitely absolutely
got the disk.
There are three options for that.
One is don't worry about syncing the data.
Another is sync the data to disk once every second at least.
The third is sync for every right command.
In the latter slows down the machine.
The one second typically gets you more or less unchanged Redis behavior depending on
the volume of your rights and how large the right volume is.
Say for example, your right commands are taking 150 megs a second while you're going
to have to have a disk that does 150 megs a second right.
Depending on the situation, I will sometimes use AOF, sometimes use snapshots.
I will sometimes even use both depending on how any retentive I want to be with my data.
Even with both, Redis with persistence is not as durable as an acid database which promises
that it will not tell you your transaction has been committed until those bytes have
been written out to a spinning drive.
This is better than losing an entire server with gigabytes of data.
What problem does the persistence solve within the realm of an application as a whole and
when might you use it versus just running it without persistence and saying if I lose
it I start from scratch?
Well, like any other database server, you've got a Postgres server.
Just a single Postgres server, hang out on a machine.
As you use it, data gets written to disk and let's imagine that you lose the machine.
Literally the machine dies, the disk dies, you lost the data on disk.
Well, now you just lost all of your data.
You need to be back up.
Redis doesn't solve the problem of backups.
Redis solves the problem of user server.
We persist data on disk.
As long as your failure is in catastrophic, you can get a pretty good amount of data back.
That is, if you're using your append-only file, notice that when I said append-only and you
can sync every second at least, in practice I've seen servers actually sync the disk dozens
hundreds of times a second sometimes with very fast SSDs.
So while it is not guaranteed, like say a relational database, that the data is on disk
for these acid consistent databases, it does pretty well in practice.
So far, people who have a machine who's power during many writes, they come back and they've
got their append-only file and their question is, hey, I've got an AOF.
When I try to load it, it fails saying that some of the data has been corrupted, how do
I fix it?
And then somebody says, run this command with these options and it strips off the corrupted
data at the tail end of the file and they recover and almost invariably they say, I
think I lost three writes, which, hey, we were doing 10,000 writes a second, lost three
from a data from power outage.
That's not that.
Not perfect, but not that.
Sounds like a very impressive trade-off between durability and performance.
It certainly helps.
Now, it's also the case that if you have, say, a solid-state drive, you can do better.
If you let it sink with every write, then, of course, you can guarantee that you get
all of your data back.
But then even then, you may have to worry more about the solid-state drives ability to actually
write data in the case of power outage.
You've got to worry about all of these other things.
In a lot of ways, when you're to pull ourselves back a little bit, to worry about the particular
details of a byte on disk is important.
When you really start worrying about them, then you start to understand that most computers
out there that already exist are not terribly reliable data storage devices.
And even solid-state drives, there was an article a couple months ago where it was shown
that Intel's solid-state drives, when they say that they're safe for power outages,
really are the only drives that are safe for power outages, and that other solid-state
drives from other manufacturers, even though they say you can pull the plug and all the
data that was claimed to be written as written, they lie, and they get data corruption, and
sometimes irreversible cell damage on the drives.
So whether or not Redis itself actually says it got data to disk versus what you read,
you know, using, you know, say a single spinning disk in real hardware sitting on your desk
to store data, still not all that terrible lie.
I'd like to switch over to talking about Redis from a programmer standpoint.
How widespread is the support for different programming languages on the client?
So last time I checked, there were more than 30 languages covered in terms of clients.
A quick check, let me see, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12.
Okay, I lied, 29, unless I miscounted my quick pass.
They're also, you know, a collect almost countless higher level tools, and most of the programming
languages covered, you know, that half clients have a few clients, I mean, heck, those got
8 or more clients for Redis, which is kind of crazy, you know, and Java's got a similar
number.
The language support is pretty expansive.
It covers more or less all the major programming languages and a lot of what I would consider
to be minor programming as well.
It also, the protocol itself is also not that bad to implement, while the protocol has changed
since version 1 to the now 2.8 and soon to be 3.0, it's not difficult to implement the
protocol if you've done it before, if you've developed protocol communication before, it's
not different.
It's a new project I may take a little bit longer, but it's not bad.
Are people typically doing anything like an ORM mapper or some kind of a bridge from
the data structures in their programming language to Redis data structures, or do they use it
for not necessarily require that?
Usually it depends on the use, there are a lot of people that are using Redis without
any sort of map of one kind or another, an object map at all.
They're using it for leaderboards and video games, they're using it for cookie storage,
they're using it for all sorts of things that don't necessarily require that kind of access.
Most of my usage, relatively recently, was just data structures in the raw.
More recently, after having a need for actually modeling data, let's take a traditional object
relational map or relational database, I looked around for existing object map versus
for Redis, and I didn't particularly like the features or functionality provided or
the way the API worked, and so I ended up building my own.
To be honest, I don't use mine very often.
I will use it here and there for relatively small things.
Sometimes when I'm building a new proof of concept application, I will use it as the
only data store and I will just use my Redis object map for Python.
That will typically get me started in five minutes.
I don't need to worry about database configuration, I don't need to worry about setting up a
new user and all that other stuff on my database.
I don't need to worry about, okay, now how do I initialize this thing here?
It's just a connection string and I just don't worry about it.
It behaves more or less as I expect because I wrote it, but different object Redis map
errors will have significantly different features.
Some are strictly just, okay, so you've got a hash in Redis, you want to be able to pull
data out, put data in, and that's the level of abstraction.
Some improve or offer an abstraction for set manipulation, others don't.
What features you get are very based on programming language and library, of course, but there
are a lot of them.
I know at least four in Python.
I've seen three in Ruby.
I know there are a few in Java, although the people that I've talked to that use Java are
sad that it doesn't have all the features that some other libraries have.
So whether or not an object map exists or is useful, we'll vary greatly.
I'd like to move on and talk about some of the infrastructure and performance issues.
How much memory are people able to allocate to a single Redis instance and what's typical?
If you're using a 32-bit version of Redis, then you get two or three gigs depending on
your operating system.
With 64-bit Redis, which is fairly typical nowadays, you can have as much memory in Redis
as you have memory on your machine.
I've personally used Redis servers with, back in the day when Amazon's largest instance
was what, 74 gigs of memory.
I used those and I was using 40 or 50 gigs of those at one time.
How big is viable will vary based on the types of operations we're performing, in particular,
if you have very expensive operations that you're performing, say you were intersecting
large sets or performing some of these somewhat expensive set or sorted set operations, maybe
a single large server doesn't make as much sense because then you have a lot of clients
trying to manipulate all the same data.
You could become CPU constrained because, again, single threading.
But if most of what you are doing is using it as a data storage platform, the large
Redis instance I was talking about earlier, that was for a Twitter analytics platform.
Though it was performing a large amounts of leads and rights, none of the operations
were expensive.
And sure, we sustained fairly consistent 50 to 60,000 reading right operations a second.
None of them were expensive.
So it would typically run maybe 20 to 30% of one CPU, which was ready for us.
We could do high IO, we could store a huge amount of data and it was fine.
I'm positive that nothing else at the time could do it.
If I'm running a Redis server, then from what I understand, I would look at if I wanted
to add more memory, I would watch the CPU and the number of operations and look at what
point does the CPU start to become the bottleneck because being single threaded, I can't add
more CPUs to it.
Certainly, that is definitely the case.
You would check your operation and see if you can, it would perform.
That said, it's not impossible and not unreasonable actually to run multiple Redis servers on
a single physical machine, virtual machine is the case might be.
The memory overhead of Redis itself is only a couple of megs.
So there's actually groups of people who advocate for running several Redis servers, you know,
this server processes on a single server itself running on different listening ports and pre-sharding
data so that if later when you need to really grow them up further, say you run four Redis
processes on a single machine, if you ever need to grow, then you can use some replication
techniques to actually migrate the data to another server and, you know, say, reduce your number
of Redis processes from four to two and from two to one, as necessary.
There are some that even suggest going way farther if you know you run from drive data
and starting with 64 Redis servers, running each machine and going from there.
I'm not sure I would do that far.
And I've personally had pretty good luck with running a single server and then partitioning
data later.
But, you know, it's all about where you want to do your work.
You want to do it up front or do you want to do it later?
If I'm looking at application with a large number of keys, what is the algorithmic complexity
of Redis key lookup and how does it scale to huge numbers of keys?
So it's order one.
Redis's main table for looking up keys is a hash table.
So you calculate your hash and you look up in the hash table and because Redis actually
uses what is called buckets and linked list chains, you know, depending on how full the
main hash table is, I will have to look through your few items.
This does automatically resize its own hash table so it is possible that you will need
to check two entries, the current hash table or the old hash table and then the other one.
But still, it's very fast.
The actual major performance issues with Redis that stop it from going from, say, 100 or
200,000 operations per second on a reasonable size machine to, you know, send millions,
tens of millions, the same thing that stops just about every other system from doing that,
which is network I.O.
So generally speaking, Redis is usually constrained by network I.O. more than main hash table look
up by a couple orders of magnitude.
We've talked about earlier the idea of a master slave and just a moment ago about running
multiple instances of Redis on the same box.
There are a number of options for scaling out Redis.
Could you tell us what they all are?
Certainly.
The simplest way to scale Redis is to put Redis on a bigger machine with more memory.
And that's the typical thing that most people do until they can't reasonably afford such
a large machine or they can't get a large machine.
The second is if you have not heavy writes, say, moderate writes, but you have a lot of
reads, another common tactic is to add read slaves.
So Redis has this concept of master slave replication.
And you can attach ostensibly as many slaves as you want, although beyond a few, what you
can end up with is app going bandwidth limitations from your master.
Say for example, if you've got a master and 50 slaves, every write that hits the redis
master will then be replicated out 50 times the slaves.
That's not really viable in practice.
But because Redis slaves also can't slave themselves, you can set up a tree type structure
to do that.
There's actually a user that asked about a similar scenario in the last couple weeks
on Redis mailing list.
In any case, so you can slave four reasons.
If you need to handle a higher volume of writes than what Redis could normally handle
or more than it would be reasonably replicated out, you can shard your data.
That is, you can run multiple Redis servers and you can use say key sharding or some sort
of data sharding to partition your keys from a single Redis server across multiple Redis
servers.
And there's actually a tool called TwmProxy, T-W-E-M Proxy.
It's a server that Twitter had actually developed to proxy sharded Redis database calls as
well as memcache decals.
It supports both.
If you want your clients to not have to worry about sharded data, you can instead have a
collection of proxies that handle the sharding and all the connection management.
And then you can just hit those proxies.
If I'm starting to shard the data, then does that shift the balance as far as what operations
can no longer be done on the server, some of the set intersects and complex multi-valued
operations across different shards?
Yes.
Unfortunately, when you partition your data like that, you're no longer able to perform
multi-key commands that involve keys from different shards or at least data that's living
on different Redis servers.
Redis doesn't really have a concept of data migration.
And when you are manually sharding like this, Redis won't even know that it has its own,
it has a portion of the data and not all data.
There is a service, well, there is another way of scaling out called Redis cluster, which
is currently in beta.
It's currently the priority for Salvatore, the author of Redis, which does offer a bit
more automated scaling out and automatic sharding and replication and a few other features.
But it also is limited to two of these multi-key operations that where all the keys live on
the same server, potentially, but must be in the same or I believe at least right now
are required to be in the same data shard.
Although it does let you specify how you want to shard your data.
So you can force data that should live on the same machine, to live on the same machine.
For example, you have user data, so you've got your cookie for that user, they are visiting
history on your web page and a few other associated pieces of information.
You can tell Redis and your sharding mechanism all of this data should be on the same machine,
so it will all be on the same machine.
In relation to the master slave, you mentioned earlier that it uses the same forking process
as BGSAVE, how does that impact the fraction of the machine's memory that can be allocated
to Redis?
Where I'm going with this is do you need to be cognizant of how the fork is going to
require a chunk of memory?
Certainly.
That is definitely a concern.
The typical super anal retentive developer and architect will say Redis must only use
half of the memory on the machine.
Just in case we've got huge volumes of writes when we are snapshotting in preparation for
replication or as part of the typical saving process.
Because of this fork, if we've got enough write operations, we could actually double the amount
of memory that Redis is using.
Potentially even more if the amount of data in Redis is growing fast.
What Redis does is it actually informs you in the log file of how much memory was used
by the child process, additionally.
Depending on your operation, if you pay attention to your logs, you can discover how much memory
is actually being used by that operation.
You can actually appropriately size the amount of memory that you're storing in Redis compared
with how much memory is available on the machine and how much memory that a fresh snapshot
for slaving or for just snapshotting and keeping your data.
So you can properly size all of those things.
It's not perfect, but it can give you a future idea.
I've looked a little bit at Redis cluster and Redis Sentinel, which is a different project
that has some of the same goals.
I've been following some of the discussion on the Redis mailing list.
This is how it appears to me is Redis is a brilliant solution to the things it does
well.
A lot of that is based on the fact that it's single threaded and runs in a single address
space.
When you try to scale it out, it becomes a distributed system.
Then you're moving more in the direction of a dynamo type store, which is a different
animal and requires a whole different thought process in order to do what those data stores
do well.
Am I being unfair here or how would you respond to that?
I think you might be being just a smidge unfair.
There are a lot of operations that the dynamo and generally speaking, a big table-like data
server offers.
One of the most important being you can put a huge amount of data in it, but how those
systems work primarily is put your data somewhere, optionally with indexes.
If you want to perform a query, basically perform a query against every machine data
that you care about.
A lot of ways, it acts like a map-reduce operation across all of your servers.
In fact, there are several databases that that's all they do.
They just store data somewhere.
If you want to query the data, you perform a map-reduce.
Redis on the other hand is not meant to store, not necessarily meant to store terabytes of
data.
It's not necessarily meant to be your archival data storage solution for the next 20 years.
It's not meant to perform cohort analysis on 30 billion video views over the last two
weeks.
That's not what it's for.
It's for storing, in some sense, data that you need to have number one fast access to,
number two, access in certain structural ways that are fast.
If this typically you look data up via keys or via some index that you've manually built
in Redis.
That's it.
You're typically not saying count the number of items with this certain feature and group
them by this.
It's a more typical operation that you perform on a Dynamo or a big data.
The data you're storing is different.
The data access characteristics is radically different.
The data storage, the types of data that you're storing is radically different.
Meaning to get back to your real question, does it get more complicated?
Absolutely.
It's no longer the simple, oh, I can intersect anything or, oh, I can sort anything and I
can access any type of data anywhere.
It stops being there.
But a lot of those same constraints exist on relational databases where you also have
to sharpen.
Sure.
Definitely.
I'm thinking of a clustering solution for MySQL where you can't do joins anymore.
Exactly.
There's a similar package for Postgres where maybe this is a decade ago where this is
the case where you had to perform your queries against each of your sharded servers and then
you had to merge the results yourself.
That was fun.
Exactly.
Talk a little bit about the open source project, how many committers are there, how active
is the community, what resources are out there.
So Redis itself, it has a primary author.
That is Salvatore San Felito, the inventor and original author.
It was his idea he developed almost all of it.
It certainly is the case that there are more contributors now.
As time goes on, of course, more and more people get patches in.
I personally only have a single patch in Redis even though I've been using it for so long.
It was for an early feature that allowed you to perform Z-interest store and Z-union
store on sets as well where you could pass in a set.
A quick check of the statistics from the Redis repository says that Salvatore has got by
far a factor of 10 more total lines committed than the number two, Peter Nordhoose, who at
least in the past was at least partly employed by VMware now pivotal labs as Salvatore's
number two.
But it would seem that he has ramped off the project and some other people have started
to become more active.
But primarily it's Salvatore.
So as an almost one man piece of software, it's pretty amazing what he's accomplished.
There's also a mailing list which you are very active on.
Yeah, yeah.
So the RedisDB community mailing list.
I recently joined it shortly after starting to use Redis in order to ask some questions
and to even offer some feature requests and a couple patches.
And then people started asking questions and I started answering them and have kept answering
them.
And people keep having interesting questions and I keep learning more about Redis and so
I keep answering the questions because they're fun.
I've done this before in some other mailing list in the past but it just so happens that
at least for now I'm hanging out on the RedisDB community list.
And actually that's how I ended up writing the book.
A publisher saw, Manning saw how prolific I was in the mailing list and said, hey, this
guy probably knows what he's doing.
So I got a call and there's some negotiation and I wrote the book more or less based on
the problems that people were bringing up in Redis, in the Redis mailing list.
Common problems, common solutions, common ways of looking at very typical problems.
In addition to your book, was there anywhere else that you would like to direct listeners
who want to follow more about your work?
I should say the book and the mailing list because you're very active on the mailing
list as well.
Other than that, periodically I post blog updates and sometimes I talk about Redis,
sometimes I talk about Python, sometimes I talk about other things.
Recently I even posted about my experiences with Soilint which is a nutritional product,
a new replacement powder that I'd helped kickstart last spring or I guess early last
summer more likely or crud-on funded.
I'm sorry, it was not on kickstart for itself.
I typically post about a variety of things, mostly being Redis or Python.
Sometimes I talk about general text, sometimes I talk about periodically I will talk about
politics but for most people, at least most engineers in the Bay Area or most engineers
generally, I'm not terribly offensive.
We'll definitely link to your book and your blog in the show notes.
Isaiah Carlson, thank you very much for speaking to Software Engineering Radio.
It's been my pleasure Robert.
Anytime you want to talk again, I'm available.
Thanks for listening to Software Engineering Radio.
We want to know what you think.
Go to iTunes and write a review of the show, stating your opinions.
It would help us improve the show.
Thanks.
Thanks for listening to SC Radio, an educational program brought to you by IEEE Software Magazine.
For more information about the podcast, including other episodes, visit our website at se-radio.net.
To support us, you can advertise SC Radio by clicking the dig, Reddit, delicious or slash
dot buttons on the site or by talking about us on Facebook, Twitter or your own blog.
If you have feedback specific to an episode, please use the commenting feature on the site
so that other listeners can respond to your comments as well.
This and all other episodes of SC Radio is licensed under the Creative Commons 2.5 license.
Please see the website for details.
Thanks again for your support.
