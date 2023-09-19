**Link** https://www.se-radio.net/2015/12/se-radio-episode-243-rethinkdb-with-slava-akhmechet/

This is Software Engineering Radio, the podcast for professional developers on the web at
sce-radio.net.
SC Radio brings you relevant and detailed discussions of software engineering topics
at least once a month.
SC Radio was brought to you by IEEE Software Magazine, online at computer.org slash software.
RethinkDB is an open source database for the real time web.
Slava Akmashit is the CEO of RethinkDB.
Slava, what is RethinkDB?
RethinkDB is an open source distributed database designed from the ground up for the real time
web and real time applications.
The idea is that when we started the company, we noticed that the world is really moving
towards real time.
People build much more collaborative experiences, for example, applications like Slack or Google
Docs where people collaborate online and when someone makes a change to anything, it's really
important for other participants to see it right away.
We noticed that the database architecture really wasn't written for this.
The developers that make all kinds of really complicated decisions to build these apps
and we thought, hey, this could be dramatically easier.
Let's build a product that makes building these kinds of apps much, much easier to do
because that's where the world is going and that's how we started Rethink and that's
what it's for.
At a slightly lower level, what problems does RethinkDB solve?
At a lower level, the problem is, so imagine, let's talk about the example of RethinkDB.net
example of Google Docs.
You have a document that one user is modifying and you have another document that someone
else is looking at.
What you have to do is whenever a modification and browser A happens, you have to store it
somewhere and then push it back to browser B.
There's been tremendous innovation on the front end with things like WebSockets and
Socket.io and all kinds of client libraries to make that possible.
On the back end, it's still really hard because when you're working with a database, you typically
just, the way you query data is you send a query, you get a response.
If you want new data, you have to send another query and get another response.
On a very simple level, if you were to write this application in a naive way, what you
do is you start querying the database over and over again in a loop, let's say every
few milliseconds.
Of course, that turns out to be a really bad user experience because the latency is too
high.
You can send a query every few milliseconds, people typically set it to seconds.
The second problem is if you have a lot of concurrent users and you start querying the
database in a loop, you just bring down the database.
It's really hard to do this naively.
You can talk a little bit about how people build these applications today.
On a lower level, what we think it would be does is the developer can say instead of just
sending a query, they can say, okay, I'm subscribing to this information.
For example, I'm subscribing to all anything that happens in this document.
When someone writes to the database, the database sends notifications to all the subscribers
saying, hey, this particular piece of data you were interested in is different.
Now it's changed.
This is done via a push message.
If you're building your collaborative application, each client can just say, okay, I'm interested
in this specific piece of data or these multiple specific pieces of data.
Anytime any client does anything, any modification, we push a message and you can get it.
Anybody who's interested in it can get it.
What that does is it makes it dramatically, first of all, it solves the problem of bugging
down the database.
It's very scalable because the database knows at all times what happens and it can efficiently
push these messages.
It solves the problem of your infrastructure getting much more complicated because it
can't do it in a naive way, typically.
The way people do it now is pretty hard.
You have to add a lot of different pieces and we take all of the complexity out.
The code becomes much simpler.
The infrastructure becomes much simpler.
You don't have to add a lot of different moving pieces.
The way we do that is by pushing these messages about data updates.
Does that make sense, Jeff?
Yes.
The push notification.
We will get into that.
But what differentiates rethink from other databases on the market?
What are the most specific points of differentiation?
The biggest piece of differentiation is rethink it completely designed around push architecture.
Anything that happens in the database, we can push a message up to the user.
I brought up a specific example of the user's interest in this piece of data.
It's not just data, it's done for computations too.
For example, you could say, I'm interested in the top 10 selling products in my store.
Then anytime something happens, the database will recompute this.
It isn't just looking at a specific document in the database, it's looking at a computation.
At the core, there is a computation engine that's designed for push notifications.
Whenever something changes, we'll recompute things efficiently, push the message.
What that does is it makes building real-time apps dramatically easier.
It's the main piece of differentiation in rethink to be.
There's a lot of other things like we cared a lot about both the developers and operations
people.
The query language is extremely convenient.
It's very nice.
We think to be much, much easier to scale than a lot of competing systems.
It's really designed with apps people in mind.
The guarantees that we offer are typically much more significant than what you get with
a lot of database systems.
There's actually a lot in the internals that differentiates and rethink to be one of the
small technical details.
On a high level, the push architecture is extremely important.
I think it's one of the first databases in the world to do this.
We will get into the engineering that underlies that push architecture.
But first, how did rethink DBL originate?
What is the origin story?
The origin story, so my background, I grew up in New York City.
I was a software engineer.
I was working on Wall Street in the financial industry.
I was responsible actually for building custom databases for some of the high-frequency trading
use cases we had and nothing of the shelf would do.
I worked there for a while, I learned a lot, but I didn't really like the environment.
I went to grad school.
On grad school, I was studying brain simulation on supercomputers.
I learned a lot about clustering.
I learned a lot about distributed systems.
In the meantime, I met my co-founder Michael, who was mainly doing a lot of web development,
web applications, human computer interaction, stuff like that.
We met, we started talking, and I told him about what I was doing.
He said, hey, this is a huge problem on the web, and you seem to know a lot about it.
We just started brainstorming ideas.
We noticed that the way people build and deploy web applications is changing, and these changes
are going to procolate through the whole development stack.
We decided to start a company.
This was back on the east coast in New York, and we packed our bags and moved to California
and incorporated it and never looked back.
Everything2B supports a push model, as you mentioned, rather than the request handling
model.
Could you give a broader explanation of how that compares to application development of
the past that's based on request response?
Yes, I talked a little bit about how if you use request response, you have to continuously
query the database, which is really hard because you end up bringing it down.
It's hard to scale, and the latency turns out to be pretty bad.
If you think about it, if you squint your eyes for a bit, the reason why a request response
model used to work for databases on the web for so long is because HTTP was request response.
You send the browser sends an HTTP request to the web server, the web server sends a
request to the database, guess the response sends the response.
But now that we have web sockets, and the web server often has to push data to the browser,
the top of the stack is no longer request response is much more complicated.
But people try to do it, they try to shoehorn the request response database model on top
of a much completely different model for building web applications.
That's why push is very important.
I mentioned in the Eve way of doing it, where you just query the database over and over
again, and clearly people learn very quickly that that doesn't work.
What they start doing later is they start, they figure out, okay, this doesn't work
we have to optimize it.
And there's a couple of ways of doing it.
But generally when you start doing that, your code becomes much more complicated because
you have to split your code.
You have to write to the database and then push a message somewhere else.
And then you have to have a piece of infrastructure for keeping track of all these messages.
You have to have code to route these messages among your clients.
And if you have multiple web servers, if you're scaling your web servers, then the web servers
have to talk to each other.
They have to filter out the messages so you don't saturate your network.
You have to do all these things and it's really, really hard.
And the experts in this space can definitely do it.
It's clearly doable, but it's really hard.
And rethinking makes it really, really easy because you just use your database directly
and it's designed to handle all of that complexity.
So the software engineer pushes it and pushes the burden into rethinking to be and they
don't have to worry about that stuff.
Does that make sense, Jeff?
Absolutely.
So many applications build push in the application layer on top of a database.
Why is it better to have that layer within the database itself?
There are a couple of reasons.
So the first reason is the database itself has a lot of knowledge that the application
doesn't have.
So one canonical example of this is suppose you're trying to get top 10 players in your
game in real time.
You can build and you want to push that leaderboard to the clients in real time.
Show it to people in real time.
So you can do this in the application, but it's really, really hard because if you, for
example, learn that your number 10 player drops out, it's very hard for you to know what
the next player is because the application just doesn't have that data.
So it would have to send the query to the database and get it and it would have to know
which user to get.
So it's quite challenging to do in the application.
It's actually doable.
I mean, people have definitely done it before, but you have to deal with a lot of custom
codes.
So for each interaction you want to implement, you have to write a lot of custom code to
deal with it.
You have to deal with routing and you have to add a message bus.
You know, if you ever want to scale and you have to write routing code above that message
bus.
So you have to do all these things and a lot of it becomes custom work.
And you know, in software engineering we talk a lot about abstractions and we talk about
the fact that if the abstractions are right, then everything is easy and everything kind
of flows.
And if the abstractions are wrong, then everything becomes really, really complicated.
And doing data manipulation in the application like that is kind of the wrong abstraction
because the application just doesn't have that information.
It's all custom code.
And you could do this much, much more elegantly in the database that's both more efficient
and results in a dramatically simpler application code.
What is the full stack architecture when the database is pushing rather than pulling?
Okay, so people do this in different ways.
Usually we think the B users, they tend to use Node.js.
Sometimes they use Ruby or Python.
Sometimes other languages like C sharp or the JVM stack.
But generally, and you know, sometimes it's not always the browser.
Sometimes it's the mobile phone.
So it's kind of hard to generalize, but I'll try.
So generally what happens is you have your web browser, you have your web client, and
it's communicating with the web server via some kind of a socket protocol.
So web sockets, typically people use socket.
I owe you could use different libraries.
So on the front end, it's usually a framework like React or Angular.
Then there's a web socket connection to the web server.
And the web server is usually Node.js or Python or Ruby.
And then that communicates with the database why are they rethink GDB drivers.
So that would be the stack, the real time stack if you use rethink GDB.
And if you didn't use rethink GDB, the stack would be much more complicated because you'd
need a database, you'd need a message bus, you'd need a lot of custom code in the application
to push messages on the message bus and to read from the message bus.
And people often add a lot of other custom code to hook into the database to kind of
get updates and do computations.
You've mentioned messaging a couple times now.
Push is usually handled in a messaging layer that is responsible for durability and delivery
guarantees and other features is rethink really a database with a messaging layer bundled
in?
Okay.
So I think thinking about it as a database with a messaging layer bundled in is kind
of a good first approximation.
If you think about it that way, you'll probably be right quite a bit about what we think
GDB can do for people.
But it's not really designed with a message bus in mind.
So message buses are very different.
They have different purposes.
You know, as you mentioned, Jeff, they guarantee delivery.
They have a lot of different features.
They have pops up features.
We think does this a little bit differently.
What we do is instead of thinking about everything as messages, we think about it as changes
in the data.
So for example, if you're building a web application and you want to show, you want
to re-render your DOM every five milliseconds and show new data every five milliseconds.
Because if you do it faster than that, the user doesn't care, right?
It doesn't matter.
They can't see changes faster than that.
So what we think GDB will do it as a feature called squash, which will squash updates.
So if multiple things, if multiple relevant changes happen within a five millisecond period,
then rethink GDB instead of sending multiple messages to the clients that saying, hey,
stuff keeps changing.
It will wait five milliseconds and then send a single message saying, here's the changes
to the data.
And if the changes intersect and it will figure all of that out and it will send you the latest,
you know, the latest relevant piece of information.
So for web developers, you actually want that.
And of course, all of that is configurable and people can turn it off and change that.
But it's much more, it's much better to think of it as differences in data that we inform
people about instead of messages because the use cases are different.
And there are some things we think you can't do right now.
You know, like the traditional message process will do.
So if you need guaranteed delivery and you need to make sure every message that you push
on the boss you see, like we think GDB is not the best product for that.
There's much, much better solutions.
What we do is we just make sure you get the latest piece of data you're interested in.
So it's not about messages, it's about differences in data.
Does that make sense, Chair?
Yes.
Does rethink support PubSub?
Is there like a PubSub API somewhere?
So a couple of people have built a PubSub system on top of everything.
GDB change feeds and it's actually pretty easy.
There is some code.
If you go to everythinggibby.com slash docs, I think we have an example of how to build
a PubSub system.
So it's very easy to build on top of feeds, but we don't ship that natively.
It's not part of the product.
Realtime is a buzzword that gets used a lot.
How does rethink define a realtime application?
We define a realtime application as basically an application when multiple users generally
collaborate and they need to see the changes made by one user.
The other users need to see it within a very short period of time, probably under whatever
is possible in HTTP, let's say under 20 milliseconds.
Although there are some applications where the requirements are stricter and you have
to get to five milliseconds or so.
When you have those more strict requirements, does the usage of rethink change or does rethink
tend to just provide that latency and the bottleneck is more on the application developer?
No, the usage doesn't change.
It's pretty much the same.
So for our change feeds, the users can say which timeline they expect and you'd be surprised
sometimes you want it to be a little bit higher like if you're rendering DOM and there's a
lot of stuff going on.
Let's say you're updating market data about stocks and you have this pipe and it's changing
very quickly.
You don't want to re-render the DOM more than let's say every 20 milliseconds because if
you do that the experience on the browser becomes bad.
So all the user has to say is I care about updates within this bound of time and then
we'll bundle them, we'll dedupe them, we'll do all the work and send that information
to the user and the use is exactly the same.
You've mentioned the word change feed a couple times.
What is the definition of a change feed?
A change feed is a query in RethinkDB just like any other query except you tell Rethink
like hey when you send me the response don't stop keep sending me updates when something
changes.
So it's basically one way to think about it as a cursor it's like a cursor that never
terminates.
So you open this cursor and then whenever there's updates to the results of the user
gets these updates.
What is the data model of RethinkDB?
RethinkDB is a document store more specifically it's a JSON store.
So what you store in the database it's usually JSON documents and it's extended a little
bit so you could store a JS spatial data, GeoJSON binary data we extend JSON to deal
with time a little bit better and of course it's all backwards compatible so you could
get JSON out.
So it's definitely a document model and then you could build relationships on top of that
and the joins which is a huge deal Rethink is one of the only I think no SQL products
that supports distributed join queries.
So it's a document model but you could also model relationships very effectively.
What about the transaction model if I update the database and as a result some pushes occur
to my application at what point is the transaction considered complete?
So the push model in RethinkDB is a synchronous with respect to the rights.
So when you do it right there's a lot of guarantees that have to happen in a distributed database
to make sure all the nodes, all the relevant nodes got the right and so on.
Then the client gets the acknowledgment whenever the data is safely committed to disk.
The push notification happens asynchronously we send it to all the relevant clients that
are interested in it.
And what the clients can do is there's some provisions and we're extending that up quite
a bit actually to correlate the right that happened to the message that happened.
So you can for example tell like I've received this message that means I've received the
messages relevant to all of the rights like this one and before that.
So we do a lot of that but there is no transaction with respect to push and messages because
this doesn't really work well on the web.
What about the consistency model?
Sir, you think it is immediately consistent.
When you do it right you get the response notification, you read it, you generally you
always get you know you can always see the right that you made.
And with respect to a single document you get all of the consistency and durability guarantees
that you would expect a relational database.
We're very very serious about that stuff.
So you know we're very serious about people's data and you get you pretty much get all the
guarantees that you get in a relational database but only as long as you're looking at a single
document not across documents.
Is rethink DB distributed or monolithic?
You mean with respect to like the network?
In terms of how people tend to implement it.
If you help help people tend to use it.
Oh it's definitely distributed.
So people typically start with a single you know the download you think you'd be typically
under a laptop.
So you develop you can develop in one node and then when you need to scale people generally
build out clusters and they move up to three nodes then five and then they keep going up.
So it's definitely distributed.
So once they distribute what kind of cap trade-offs does rethink DB make?
Okay, so rethink DB is hugely biased towards consistency.
If you have a cluster or even multiple data centers with a lot of machines and then you
have a split brain scenario you can only write on one side of the split brain that's guaranteed.
You can read consistent data on one side of the split brain.
On the other side you could read outdated data but not by default.
So you have to pass a flag saying I'm okay with outdated data.
So whenever there is a partitioning scenario we're very highly biased or it's consistency
than availability.
And the reason we do that is because it makes building applications much simpler because
the application developer doesn't have to deal with conflicts.
It just results in a dramatically better developer experience environment.
So we tell people who need availability in the face of partitioning, network partitioning
with all them to use other products.
And there's really good products for that they're typically much harder to program
against but they solve that problem much better.
What are the invariants that rethink DB provides?
Okay, so this is a little hard because rethink is a big product but I'd say the biggest
invariant is if you write the data and you get the acknowledgment you can be sure that
the data is on disk and you're going to be able to get it later.
If you make multiple changes to a document those changes are atomic.
So in a single document they're both going to get written or neither of them are going
to get written.
If the power goes out it's not going to corrupt your system.
So it's all of the traditional kind of asset guarantees that you get in a regular database.
These are invariants and we think you'd be except in rethink DB it's only true with respect
to a single document whereas in a relational database you could do it across multiple rows
and get transactions across rows which isn't possible in rethink.
Is there a scenario where rethink DB can see stale data?
Yes, so this is actually extremely rare but there are situations where if you have a multi-node
cluster and you have what we call cascading split-brain scenarios.
It's not just one failure in an network it's multiple cascading failures and some of the
nodes fail while some other stuff happens.
There is a possibility of stale data.
So this is very rare and multiple things have to go wrong but it's possible and we actually
have modes and rethink to deal with that stuff so the user can specify, okay I'm okay with
seeing stale data but get high performance reads in these rare situations or I don't
want to see stale data I always want consistent views but the reads are a little bit slower.
So we'll let the user control all of the behavior and kind of trade off performance and consistency.
Could you talk more about how rethink DB handles right durability?
Yeah, so I mentioned a couple of times that we care very deeply about people's data because
it's obviously the core of everything and you get the same guarantees for right durability
that you get in a traditional relational database but on a lower level what happens when the
user does a right is so let's say it's a distributed scenario and you have three replicas which
means you've asked the database to maintain three copies of your data.
So what happens when a client does a right is rethink the people send that right to three
different nodes in the cluster and it will wait for the majority of the nodes to acknowledge
the right.
When the majority of the nodes acknowledge that right did rethink everything to be sent
acknowledgement to the client.
So then if anything fails you have majority of the nodes contain new data and with timestamp
everything and stuff like that.
So if one of the nodes fails and then restart it will contact all the relevant nodes ask
for the latest updates and figure all of that stuff out.
And that's a right to handle on the in the distributed layer and then on disk we have
a storage engine that you know that deals with it pretty much in a similar way as any
other traditional database what there is a couple of differences but I don't know if
we have time to go into that.
Does that make sense Jeff?
Absolutely.
Do you agree with that?
I mean you've mentioned a couple of times that there are scenarios where you need a
different store than rethink.
Do you agree with the premise of polyglot persistence that applications generally need
a variety of databases with varying sets of features and guarantees?
So I don't know how I feel about it ideologically but empirically we always see that in our
user steps.
That's exactly what the mem sequel CTO said.
I really didn't know that.
I didn't listen to that podcast but yeah I mean it shows up it's not just rethink though
it's people use three, four, five different systems to do what they need and sometimes
in really big organizations what we see is they have a single store for example HBase
where they stole all their data and then for custom use cases they take slices of data
from HBase that are relevant to that use case and put it in a completely different product.
And in a big organization I mean they can have dozens and dozens of different products
and hundreds of instances across teams.
So yeah I'd say I agree with polyglot persistence.
You're saying you agree with it empirically though.
So what is, I mean is there some bright future where developers only need one database
product that does everything for them?
Oh man ideologically I think it's very very hard because the use cases are really divergent
for people.
People are doing a lot of very very different things and it's just you know this diversity
hasn't been around here in the 90s like in the 90s all of the things that we did were
the same and now there's just much more stuff that happens in typical infrastructure.
So I honestly you know I look at a lot of products like we were very proud of what we've
done with rethink and we think it's amazing.
It's an amazing product for the use cases that we've built it for but I also look at
a lot of other things people are doing and you know they're very very impressive and
they're so for example elastic search is just phenomenal for what it does and I don't see
you know I don't see how they could do what we do and I don't see how we could do what
they do well in a single product.
So I think the web and storage is probably going to move closer to the Unix philosophy
where it's not quiet you know do one thing and do it well but it's do a small set of
things that are relevant to each other and do that well.
So in the polyglot reality that we live in how does rethink DB operate alongside other
databases what role should it play in a polyglot architecture?
So rethink to be integrates with a lot of other database systems really nicely and in
particular change feeds make that make that really really easy and there's some libraries
that have been written in the community to integrate rethink to be was pretty much any
database in existence.
We tell people to use rethink for this real time use case it's very often the authoritative
source of data.
I can tell you a little bit about what happens in the wild you know in real deployment so
people will very often use elastic search for some of the full text search functionality
they'll use sometimes message bosses to integrate with other applications that use relational
systems relational systems are typically there for financial processing where transactions
across multiple documents are important.
So that's generally how it tends to work out.
Interesting rethink DB is written entirely in C++ this might be kind of a naive question
but why did you choose C++?
We chose C++ because performance was very important and there were a couple of like if
you look at the options I think there were a couple of options we could have picked and
probably on the top of the list was C and C++ so we chose C++ instead of C because
it gave us greater flexibility to deal with kind of bigger architectures and a lot of
the early people that started rethink to be new C++ you know really well so it was a
natural choice.
Right now there's more options there is Go I hear Rastis is developing pretty nicely
and I really like that language so Go still has some garbage collection issues that are
you know probably good enough for most applications but not databases and Rast doesn't have GC
in the same way but it you know a lot of that stuff is still being developed.
So I think for a low level infrastructure software like this where latency is really
important and performance really important.
I mean there's been a lot of development but C++ is still to this day probably the best
language and probably will stay that way for a while.
Do you have much insight into what the things that Go or Rast need to overcome in order to
be resilient and flexible enough as languages to have databases written in them?
Yeah so I actually I love programming languages I'm kind of a PL nerd and if you look at
everything to be real language you'll see that it was designed by a lot of people who
hopefully understand programming languages really well.
So I think about that a lot.
I think with Go there is it's just fundamentally there is a garbage collector and they keep
you know they keep doing updates and it keeps getting better and better but for database
garbage collectors are really bad because that means there are stalls and you can get
a variance in the latency you know for queries and that's really really bad for a database
product.
So for Go I don't think it will ever be a good systems language for the kinds of products
like databases just because the garbage collector is baked in like the moment you get a garbage
collector even if it's really really good and even that takes you know decades getting
that to be a good language for system projects like databases really hard.
For Rust I think Rust is a lot more promising because it's designed for more lower level
use cases it's just a new language you know it's going to take some time to develop people
have to learn it they have to learn best practices and you know it's easy to learn a language
it's much harder to learn all of the icebergs that you're going to get in production when
you're architecting your application.
So yeah I think Rust is a lot is a lot more promising it's just new.
So since you're saying that you don't need garbage collection or you don't want garbage
collection when you're writing a database to application developers who are listening
to this who all of the work that they've done has been in garbage collection based languages
they probably understand that garbage collection is this hazard it can slow you down it does
have problems but how does your like what is the paradigm shift you need to make when
you are writing an application that does not necessitate garbage collection like rethink
DB.
Okay so so the problem with garbage collection is wonderful of course and the problem with
garbage collection is latency.
So if the GC kicks in it's going to stall your thread hopefully only a single thread and
then when the query comes in that stall is going to be visible in the latency that the
user is going to see.
So you can't do that right that's very very important it's very important in the database
to have as little latency variance as possible.
If you don't use garbage collection for example if you use C++ then you're really switching
to a whole slew of other techniques and so some of the best you know the best thing you
can do is you can know really well the lifetime of all your objects and if you can tell the
lifetime of your objects you can just do manual memory management and you can you know
deallocate memory when it makes sense and it works really well for example with short
queries because if I do a short query I know when it begins and I know when it ends and
I can just deallocate all the objects at the end of the query.
So that you know that's extremely fast it's very efficient it works really well you know
and you don't need garbage collection you don't have to deal with latency issues.
If you have longer queries of course things get more complicated because you might need
to deallocate memory in the middle and there's all kinds of ways of doing it you could just
share pointers you could you know you sculpt pointers and shared pointers are really cool
except you can't have you know cycles so you have to design your code in a certain way.
So you really you're going down a rabbit hole if you're not using garbage collection it's
a whole other thing.
So speaking of rabbit holes we can go down rethink DB is written in C++ but drivers exist
for Ruby and Python and JavaScript and other languages could you describe the functions
of a database driver and how a database driver interacts with a C++ database application?
Yes so the function of a rethink DB driver it's interesting because it was very important
for us to give people a query language that doesn't feel like they don't have to manipulate
strings right it should feel like an API like a library.
So when you're querying rethink DB in Python it feels like you're just writing Python code
if you're querying it in Ruby it feels like you're writing Ruby code and so on.
So what the driver does is it's really a thin layer that integrates extremely well into
the development environment that you're used to so it feels like you're writing in the
language you know that you're writing in.
That's the function of a rethink DB driver to provide people with the best possible user
experience to query the database.
Now the way the driver communicates with rethink DB is we have a wire protocol and it's public
you know it's on rethink DB.com slash docs anybody could write a driver that interacts
with this wire protocol and it will work with the server.
The wire protocol is it's you know it's for TCP so it starts out as binary but it's basically
JSON so you could you know you just send JSON on the wire the database interprets it and
sends back a response and it's very efficient it's very fast and it's relatively easy to
implement so so that's why actually there's been so many community developed drivers.
So we've dive down deep I'd like to zoom out again how does a developer get started with
rethink DB?
Well the easiest way is to go through rethink DB.com and kind of read a little bit about
what it does for you and if it makes sense you can download it very easily it's available
in OSX right now via brew or you know package manager DMG.
It's also available in pretty much all flavors of Linux we're currently working on a Windows
port that's going to be available pretty soon but you could also do it in Docker you could
deploy it in the cloud you could use compose IO which is one of our partners that will
let you deploy you think in the cloud and a lot of people actually use them just to get
everything to be server for development you know so there's many many ways to just install
it it's really simple it only takes a few seconds and then we have really good documentation
and we have a lot of tutorials on rethink DB.com and a lot of people in the community
have built a lot of tutorials so I'd get started that way go through our guide it's
quite simple and then if you have a specific stack like for example let's say you use Angular
JS there's some specific tutorials for how to get started with rethink to be an Angular
for example and you know some the same thing for other frameworks.
Is there a notable developer experience difference between a user on a rail stack versus an Angular
or a spring stack?
Yeah so we'll see we're kind of jumping a little bit because Angular is a front end
framework and rail.
Sure okay we express JS stack or a mean stack.
Yeah so so generally the user experience is really nice in Node JS because Node JS is
in the synchronous language right so it works much much better with change feeds.
If you use Ruby or Python if you're just using we think you'll be without change feeds it's
pretty similar to using any other database and it's really nice.
If you start using change feeds in Ruby and Python not only languages like that that don't
have a synchronous behavior built in you have to use some of the libraries like Twisted
or Event Machine so or you know I think I own Python there's a lot of options.
So yeah there's a little bit of difference a little bit of difference in each stack but
generally it's very very similar like the driver is local must exactly the same.
So if you have a Python Python Ruby code and you look at it you can convert it to Ruby
or better it trivially.
So Meteor is a JavaScript application framework for building real time apps that leverages
Node JS and it seems to me that Meteor would have a lot of synergies with rethink DB but
maybe I'm off base here is that accurate or do you know much about Meteor?
Oh that's totally accurate so we actually know the team and when we were designing change
feeds the first thing we met was the Meteor team and we said hey guys you know what do
you see in the wild what are the important use cases for users.
So we kind of collaborated with them to design the feeds in rethink to be when it comes to
the API in architecturally all that happened a lot earlier but the API and the user facing
stuff was done in conjunction with the Meteor team and some people have done an integration
with rethink to be in Meteor there's still a little bit more work that needs to happen
before it's production grade but people can probably start building apps on top of it
now.
So yeah we definitely I think what Meteor has done is all of that complexity they've
baked it into the Meteor framework and took it away from from application developers
but because they're dealing with database systems it's hard to scale for example Meteor
doesn't work with like sharded sharded databases right now.
So yeah it's very close we work pretty closely with them and they've been immensely helpful
in providing information about what needs to happen you know in the database layer to
have a phenomenal user experience for the application developer.
Is Meteor a leading indicator for how application development is going to be done in the future?
I think there is a lot of innovation going on in application development and people are
trying so the world has clearly changed like the kinds of apps we're building is totally
different and people are trying a lot of different ideas so if you look at paradigms you know
Angular does it one way Facebook's react does it a different way Meteor does it a third
way I think it's very hard to tell which way it's going to go but it's I think it's pretty
clear to everyone that it's it's gonna change it's already changed.
What is RIKL?
RIKL is a rethink to be query language and it's a query language we design with a couple
of important design goals so one is we wanted people to have a really really good experience
developing in their native language so you know you're writing Python RIKL code looks
just like Python code for example.
The second thing we wanted is we wanted it to be very efficient so when you write a
RIKL query and you send it to the cluster it all executes in the cluster the cluster
figures out where to send it how to give the data how to do all that stuff and you
just get a response.
So RIKL is really really wonderful to use and again if you got everything to be the
conslash docs you can learn a lot about how it works, why it works and how easy it is
to get started.
What does it mean to have chainable queries?
Chainable queries is for listeners who've used jQuery before you're already used to
this to this idea.
Or actually for listeners who've used Bash before on the command line you've also.
You're currying right just currying and programming.
Yeah so this idea actually shows up in a lot of different cases but generally in rethink
so imagine you know you have your table that has a bunch of documents let's say you have
a table of users that stores the user data then what you can do is you can chain a command
on top of that so let's say you wanted to join it with an organization they work with
so you say tableusers.join you know company and then after that let's say you wanted
to pull out only the fields you wanted to so you say.pluck you know user company and
then so you can keep doing this you can keep adding commands and then the data flows from
left to right to sort of like pipe in Bash or like.in jQuery and you can basically manipulate
data in very very sophisticated ways but the user is feels extremely natural because you're
just building up this query from left to right and anytime you get more data or less data
if you narrow it you figure out okay here's the next thing I'm gonna do so it's an extremely
intuitive way to building very complicated data queries that you couldn't really do so
intuitively in something like SQL for example.
To emphasize this could you explain what happens when I write a series of chained queries
maybe you could give an example and then you call run on the query chain explain it in
a little more detail what happens.
Yeah so let's say I say tableusers.join you know company.pluck username company name
run so what I've done there is I took all the data from my user's table I've joined
it with my company table I only pulled out username and company name and then I typed
run and passed it a connection which means okay it's time to run you know the run to
run the query so what happens is the driver will take this whole query the whole thing
so not just the first piece then the second piece it will take the whole thing package
it up in the JSON wire protocol and send it to the cluster so the whole thing gets executed
in the database server nothing happens on the client.
The database server will get the query and the data could be on one node or if you're
sharded or replicated it could be on multiple nodes and so the database will get this query
it will figure out where all of the data is it will send the relevant messages inside
the cluster do the computations get the data back and then send it to the user so from a
user's point of view you don't have to care if it's distributed or not you just you know
you just write your query everything gets executed on the server and the cluster and
it can actually get really complex like you could do map reduce you could do aggregations
you could do computations you could do a ton of different stuff and all of that is handled
by the database server.
RethinkDB is in production at hundreds of startups and consulting studios and Fortune
500 companies could you give an example about how a user from one of these companies is
taking advantage of RethinkDB?
Yeah so one example we give is Jive software so Jive software is a public company they've
built a product called Jive Chime and it's basically it's a messaging product for their
users it's similar to Slack or HipChat if people have used that before but it's designed for
you know Jive, Jive Cool Chain or Jive Sweet, Sweet of Products.
So messaging is an example that comes up a lot we have a lot of really interesting users
and a lot of different verticals some of one of my favorite ones is a company called Seamium
that does gaming in particular multiplayer, multiplayer gaming and the use we think you
be quite a bit we're actually working right now on video case studies with a lot of our
customers that we'll be publishing pretty soon and what we do is we go out we interview
some of the folks there and we ask them hey what do you like about the product what do
you not like it you know how did it work out for you was the use case so we'll be publishing
video series pretty soon and I'm very excited about it because there's some really cool use
cases everything to be that I can't wait to tell people about.
Can you give any previews on that like what are the commonalities between what people
say they really like about rethink.
So the common things people say is they really like the push model because it makes their
lives dramatically easier when they build the application.
They really like the flexibility of the career language the fact that you could do joins
and how easy it is to write and they really like the fact that when you deploy everything
to be and you need to scale it the operational story is actually really really good.
So operations people love the product because they no longer have to you know wake up in
the middle of the night to deal with problems.
So these are generally three things people say I can't talk about specifics though I
don't have to coordinate it with some people here but yeah I'm very excited and we're going
to publish a lot of that stuff really soon.
Could you talk anymore about how you would make a messaging application using rethink
or like what are the things that you don't have to do because you have rethink or you
could talk about games or maybe the commonalities between those two verticals.
Yeah so messaging well in any one game in case of games if you have a multiplayer game
and let's say you have a character in a specific location in the game and lots of other players
around them and then the character moves let's say in a strategy game you need everyone else
to find that out.
So you need to push that notification to all the other players.
In messaging whenever someone says something you need to send a message to everyone who
is interested.
So for example you know it's like there are channels and if someone says something in
the channel anybody who subscribe to the channel needs to see the message.
So that's kind of the commonalities the fact that you're pushing pushing data.
So if you were to do this the naive way you'd have to query the database all the time to
get the messages and we talked about how that doesn't work.
So what people start doing is they start editing pieces of infrastructure like PubSub and then
anytime someone sends a message what they do is they split the code base.
So the first part of the code says okay write this to the database because I might need
this later.
The second part of the code says push it on a message bus and then there's a separate
piece of code that has to maintain all of the channels and all of that information.
You typically also store that in the database and then whenever someone joins the channel
leaves the channel you have to configure your message bus to account for all that stuff.
So that's the kind of code you'd have to write and then in the UI and the web servers
it has to write a lot of code to propagate that back you know to the user.
Which we think you don't need the message bus you don't need to write any of that routing
code you just say so when someone you don't have to split your code so when someone you
know sends a message you just write it to the database and on the receiving end you say
you know this logged in user cares about these channels and then anytime someone writes to
those channels the part of the code that deals with that user is just going to get a message
saying hey here's messages and all the channels this particular user cares about.
So that becomes much much simpler.
We talked about this some at the beginning but I want to ask again after we've delved
into the technical details what is the key breakthrough what are the breakthroughs of
rethink DB what is being rethought.
So we started with kind of throwing out all of the assumptions about database systems
and starting from scratch and what we learned after a long time of doing this is that and
we kind of suspected you know suspected a lot of this would happen but so when you're
dealing with disk and you're dealing with queries and you're dealing with all of the
traditional like database storage stuff the systems that were designed you know twenty
thirty years ago are absolutely rock solid and you want a lot of that you don't want
to rethink any of that stuff like when you're storing data you know we've solved those
problems those solutions are really really good you don't want to be rethinking anything.
It's just I mean those things are just really well done and people have put a lot of effort
into that and a lot of thought a lot of brain power a lot of money and those solutions are
really good.
When you're dealing with APIs that is the part where you need to rethink a lot of things
because because the way people build applications right now you know in 2015 it's just so different
from the way we were doing it even five or ten years ago all of that stuff is different
the requirements are different.
So rethinking that is really important and another thing that another piece of innovation
that happened quite recently is the idea of distributed systems because you can't fit
data you know on a single machine anymore you can't meet performance requirements in
a single machine anymore.
So you have to rethink that part and you have to you know build a distributed system which
is really really hard.
So the parts that we've done that I'm really proud of is the distributed system and everything
to be you know I think it's excellent it's really well designed it's very safe it's very
very easy to use so I'm very happy for that.
Requel I think is very well designed very easy to use very intuitive to write queries and
the idea of change is the idea of building the full architecture around push I think
I'm also really proud of because it's legitimately like makes people's lives a hundred times
easier if they're building these kinds of apps.
So these are the things we've done differently and on the storage side and just kind of traditional
database stuff like asset guarantees and all of that we try to stay as close to all the
work that people have done you know in the past probably half a century in the database
world because if we think that part doesn't need to be retilt and it's really really well
done already.
What is the business model behind rethink DB the company and is there a close analog
maybe red hat or cloud era and how does this affect the developer?
Okay so we think it'd be open source and we're open source you know both functionally because
the code is online but also in spirit because all the development happens in GitHub so anybody
could go and get a bit issues comment and features as they're being designed and we're
collaborating really closely with our developer community and like these are communities absolutely
wonderful it's very vibrant we get a lot of feedback so the whole thing is open source
as far as business model we provide a lot of services to organizations we provide training
both on site and online to help people bring their developers up with everything to be
and bring them up to speed.
We do developer support so as people build applications we can help them learn things
and get the market faster we do production support so something goes wrong you know we
help people out.
Generally we do support over slack so they get a real time channel to record development
team and we can use other you know other message systems too so people really love the support
model so that's what we're doing right now we do health checks for people you know when
they deploy in production things like that and we also have a lot of plans for different
services on top of everything to be that will make people's lives a lot easier and I can
talk about that right now but hopefully that's going to start coming out you know next year
so there's a lot of really exciting exciting stuff we're working on but right now it's
been a lot of just basically doing services for people and helping out with getting you
know getting apps developed getting into production and making sure everything stays
great when the ones they have is up.
So you've mentioned a couple resources that aren't quite ready for publication yet where
can people where can I put what in the show notes that if people click on these pointers
in you know a few months or three months or five months these resources will be available
where should I point people to if they want to find out more about rethink DB.
I would point people to rethink to be.com that's the biggest you know biggest repository
of information on rethink to be and it's pretty easy to read as well design there's
a lot of info there people should should be able to get you know a lot of information
we're also on Twitter so everything to be on Twitter we're very active there respond
the most questions so if you just have a question you can just bring us everything to be on
Twitter will respond my personal email address here is lava at rethink to be.com so feel
free to send me an email if you have any questions and I'll you know it gets sometimes quite hard
but I'll try very hard to respond to every email so yeah I'd say use these resources
is just kind of also get up get up is another good one so we think to be on get up is very
active and everyone's very responsive.
Are there any closing thoughts on rethink DB or application development or computer science
that you'd like to provide to my listeners.
Yeah well we're very excited about what's going on in the web because there's a lot
of innovation there's a lot of innovation happening in the browser and this is really
important because it's really changing the way the way people build web applications
you know there's been angular and meteor and reactors doing some quite amazing things and
as people build more and more sophisticated from an applications it becomes quite clear
that the rest of the stack needs to change to because things get very complicated so
that's why we started we think to be in and we're very excited about a lot of the work
that we're doing.
So that's the first part I wanted to say the second part is that I wanted to thank
all of the open source community people that have been engaged with everything to be that
have been so helpful you know going to hackathons and doing tutorials for people going to meet
up giving feature suggestions, bug reports you know helping with feature design.
Thank you guys so much the product couldn't couldn't have been nearly as good as it is
now without your help.
Thanks for listening to SC Radio and educational program brought to you by I-Trip Police Software
magazine.
For more information about the podcast including other episodes visit our website at se-radio.net.
To provide feedback you can write comments on each episode on the website or right or
review on iTunes, mention our messages on Twitter at se-radio or search for the Software
Engineering Radio group on LinkedIn, Google+, or Facebook.
You can also email us at team at se-radio.net.
This and all other episodes of S-T radio is licensed under the Creative Commons 2.5 license.
Thanks again for your support.
