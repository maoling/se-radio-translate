**Link** https://www.se-radio.net/2010/05/episode-162-project-voldemort-with-jay-kreps/


This is Software Engineering Radio, the podcast for professional developers on the web at
se-radio.net.
As your radio brings you relevant and detailed discussions and interviews on software engineering
topics every two weeks.
Thanks to our audience and the partners listed on our website for supporting the podcast.
This is Robert Blumen.
I'm here at Q-Con San Francisco 09 with J. Kreps of Project Voldemort.
J. Welcome to Software Engineering Radio.
Hey, thank you very much for having me.
It's a pleasure to be here.
Tell the listeners a little bit about your background.
I guess I have been involved in data-related programming problems for websites.
I used to work for a comparison shopping site called NextHag.
Now I work for LinkedIn, a social network.
Originally my interest in educational background was more in machine learning applications.
I found that more and more I was pulled just into scalability problems.
One of the things that I worked on at LinkedIn was this project of Algomart which is a storage
system.
It's purely a distributed application that has no analytical applications.
It's just for storing your data and getting it back out.
It's something we've put to use there for these projects.
What is Project Voldemort?
It's a distributed storage system.
It's a dumb down version of MySQL or any database.
It supports much weaker queries.
We'll scale across many machines.
We'll support much larger data and a much higher rate of queries.
Give us a brief history of the project.
I started the project actually right about the time I came to LinkedIn.
At that time, LinkedIn didn't really have any distributed storage solution.
We just had databases, centralized databases, some caching in front of them sometimes, but
that was it.
There was room for improvement.
We had a lot of projects which required storing a lot of data or required serving a lot of
data.
When you have only a single database, that can be a big challenge because you're trying
to store much more data than you have memory.
Provide a brief history of the project.
Project Voldemort was something I started working on right about the time I came to
LinkedIn.
We had a number of problems that I thought this would be applicable for.
It seemed like a good thing to do.
It wasn't really officially a project of LinkedIn.
It was more like a 20% project, but it became an official project as more and more applications
came out.
There are a lot of popular databases out there.
Obviously, you thought that none of them really solved the problem you had.
Tell us a bit about what is the unique problem domain that you think Voldemort solves that
wasn't adequately solved by other tools that already existed.
Sure.
There are a lot of good open source databases.
There's two categories.
There's fairly stable, well tested systems like MySQL or Postgres, which are centralized
database systems.
They do a great job on a single computer.
They provide very rich query capabilities and a lot of features.
Those were not what we were looking for because we needed to scale over many computers.
Doing that on top of those systems is somewhat difficult.
That was the problem we were trying to solve.
Recently, there's been a lot more distributed databases.
This is in that space.
It's a very new space.
There's a lot of new project every week.
Could you name some of the other well known projects in that space?
There's a whole kind of no-sequel genre, I guess, of open source projects.
It breaks down into a couple different areas.
People who are doing real time distributed storage, there's also Cassandra, there's HBase,
there's a number of other ones.
A lot of people are trying to solve this problem at the same time.
This must be an emerging problem.
What real world factors have driven the emergence of this problem?
Probably the most obvious thing is actually quite obvious, which is the internet.
The internet, one of the things it ends up doing is centralizing a lot of computation.
It's the opposite of what you would expect.
You would expect if you connected computers with fast networks that you would decentralize
computation.
In fact, it means that everybody wants to go to Google or they want to go to LinkedIn
or they want to go to some other website.
They want that company to do some computation for them, which means that you end up having
millions of users coming to your site every day wanting to do some computation, which creates
a huge load problem.
If you imagine the larger applications before that were probably enterprise application,
you could imagine having hundreds of people all using it at the same time, but it was
still just a few hundred people.
I didn't really warrant changing the architecture of a database to support that.
You just bought a slightly bigger machine.
That's the main force.
The other forces, things are changing.
Discs are getting slower.
Programming languages have become safer.
You don't necessarily need a specialized query language.
There's a lot of other factors, but I think the main one is just the demand for more scalability.
In your talk at QCon, you said something very interesting.
80% of caching tiers are fixing problems that shouldn't exist.
What did you mean by that?
There's two things you can cache.
You can put a cache in front of your database, meaning you pull something out of the database
and you put it in your cache.
The problem with that is your database is also a cache.
Your database machines have a bunch of memory.
Your caching machines probably have also a bunch of memory.
Probably you have the same stuff in both systems.
That's kind of a bad solution.
You could have twice as much memory, say.
There's another kind of caching, which is maybe you have pulled together many different
things from different databases and you've done a lot of computation on top of it and
you want to cache that result.
That makes a lot of sense.
You don't want to redo that computation if you can help it.
That's what I meant.
Most of the time when people are using caching, they're just pulling something out of a database,
putting it in the cache, and then looking in the cache again.
That's a problem that database should solve.
You shouldn't have to build a storage system on top of your storage system to make it faster.
What would be first hit project Voldemort is a replacement for what people are doing now
in a lot of sites with a sharded relational database, some read replicas and some mem
cache D servers?
Yes.
That is the goal, is to replace that kind of a setup.
Primarily you get a cost savings or is there something else driving it?
Yes.
You would hope to get a couple things.
One is simplicity and not having to build that.
If you have to build the layer that deals with caching and deals with partitioning and
deals with migrating of shards back and forth and all that kind of stuff, it's a lot of
work.
This provides you something which is somewhat simpler, you can kind of mock it out for your
testing.
A lot of these problems are solved.
In addition, we would hope it would be faster because we were able to drop some of the
features that we don't need and the simpler queries you would be doing in that partition
system anyway.
There should be some cost savings and performance improvement as well.
When you build this, similar projects did not exist, but were you influenced by some
other things that were already out there that you liked?
Yes, probably the biggest influence was the Google Bigtable and Amazon Dynamo Papers.
The Bigtable was just a very popular system.
Design wise, there is not that many similarities.
The similarities are mostly to this Amazon Dynamo system.
Those both existed only as published computer science papers.
There's nothing we'd ever used because they're proprietary systems, so that was the goal
was to provide something like that that you could download off the Internet and use.
What are some typical use cases for Project Voldemort?
What I've seen and what we're doing at LinkedIn is primarily things that are under scalability
pressure.
There's a couple of ways I guess how storage problems can be hard.
One is it could be financial transactions.
It could be very important that you have precise consistency.
It could be very important that you never lose the data.
There's other problems which are primarily just about reading the data as fast as possible.
We're primarily focusing on problems that have a lot of scalability pressure.
It could be very, very simple storage problems.
We use it for tracking activity data.
That helps us do relevance features.
It helps us do anything else like that.
It does require doing a right multiple times per each page that's served on the site.
That's a very scalability intensive problem to solve.
Those are the kinds of things that we've looked at first primarily because we had no other
way to solve those problems.
Whereas some smaller problem, you can still just put everything in a MySQL instance or
Oracle instance.
It will certainly work for something.
It's primarily driven by the size of the problem.
Yeah, that's right.
That's right.
In cases where you wish you could use a relational database because you're very unhappy to give
up all the general purpose query functionality, but you're really forced to go this way because
of scale.
Yeah, so probably all of them.
So I think the comparison for us is really, when we're using a relational database, it's
usually an extremely dumb down way.
So compared to that, one of these no SQL systems like Volomard is much nicer to use.
But compared to how you might use a relational database with an ORM system on top of it, if
you were building just a very small application, that's very convenient because you make no
commitments about how you access your data.
Probably all of your data will fit in memory on the database and it provides a very flexible
solution.
Now once you're either putting things on disk, on a centralized system or spreading them across
machines in a decentralized system, you're going to have to make choices about how you
access your data.
And that's going to make it much harder to use.
At that point, I think it becomes a good trade-off.
If you don't need to do that, then I think probably there's nothing about MySQL.
You're unhappy with unless you just don't like the SQL query language.
So if I'm using Project Volomard and I need to look something up based on secondary key
that poses a little bit more of a difficulty.
Yeah, so I mean the way these systems work is you push all that logic for different look-ups
into the right.
So when you're doing a right, you're going to need to update a secondary list of items
that points to it.
So if you have maybe users by a primary user ID and you have users by an email, you're
going to have to update both and you're going to have to deal with the fact that they could
be slightly out of sync because those updates don't happen in a single transaction.
So meaning when you fetch by primary key, you always get the up-to-date result, but when
you fetch by email, it could be a few milliseconds behind and you'll have to deal with the fact
that there's somebody updated their email right at that instant and you get back no results
when you fetch.
You have to handle that correctly in your application.
So it's a trade-off.
I think what you're saying is because of these scale issues, the programming model that we've
all come to know and love, where the data store appears as this mass of undifferentiated
centralized data that that illusion is really going away and as programmers, we're going
to have to deal with some of the things we wish we didn't have to deal with like the
structuring of the data and underlying consistency problems of a distributed system.
Yeah, so that's true.
I mean, the hope with these systems is actually to paper over it as much as possible so that
you don't deal with those.
Now you end up thinking about how to do things with a weaker query model rather than thinking
about how many machines are we going to have in production when we release this.
But my experience has been that you end up thinking a lot about what the database is
doing.
If the data is large, no matter what, you'll spend a lot of time thinking about how is Oracle
answering this query.
What is it doing?
You're going to spend a lot of time testing that even if you're using a very traditional
database, right, you have to think through that otherwise your product won't work.
Some more provides a weaker consistency model than what most of us are used to with the
relational database.
First tell us what is the consistency model that relational databases provide?
Sure.
So I mean, the idea of consistency is kind of when you ask a question, what guarantees
about the answers you have?
So traditionally for databases, you're talking about acid, which is kind of atomic, consisted,
isolated, durable.
So what does that mean?
It means essentially that when you do some operation, your operation will either happen
or not.
We're talking about a right.
If you never write to your data, then it's kind of very easy to keep it consistent.
You just put it everywhere.
But the fundamental problem that we're trying to solve is if you have more than one machine
and you do a right to one machine, you then need to do the same right maybe to another
machine to keep the two in sync.
And of course those can't happen at the same instant in time.
So all of the problems that come up in these systems is how to make it appear as if it
happened at the same time, what's the best you can do?
So for a traditional database, you don't have that problem.
You have only a single machine.
You can simply lock around the update in memory, which makes it appear that it either happens
or doesn't.
So the properties that you get is that it's atomic, meaning it happens or it does on it.
It's isolated.
It's durable, meaning it's written to disk in practice.
But particularly that it's consistent, meaning you get the same answer twice.
You discussed academic research called the CAP theorem, which showed in a formal way
how this consistency property cannot be extended to scale.
Tell us a little more about that.
Sure.
So there's a couple of these impossibility results.
So it's always funny when something, you know, a proof of impossibility becomes a practical
topic because, you know, all it does is prove that you can't do something in a certain mathematical
model.
So the CAP theorem is one of these results.
I don't know if it's the most fundamental one, but it's the one that has been made the most
sense to people.
So what it's saying is it kind of goes along with acid.
It's talking only about a distributed system, meaning you have stayed on more than one machine.
And what it's telling you is that you have to choose between consistency, availability,
and partition tolerance.
You know, in particular, you can get two of those, but not all three.
So consistency is the same as in the case of acid.
Availability is the, you know, can you get at your data?
Right?
So it's very easy to provide consistency and partition tolerance if you don't have availability
because you can just shut down the system any time there's a problem.
So that's an easy way to get to.
The one that's a little bit tricky is partition tolerance.
So this is something which is, they're not that well understood.
But the idea is, the simplest idea is what if you had, you know, three machines here
on one network and three machines on another network and they suddenly couldn't talk to
each other?
How are you going to be able to take updates on one set of machines or the other, right?
Because you can't keep the state in sync between them.
Now that problem becomes, is not just a, you know, what if there was a problem with the
network, but also, you know, in any case where you're taking an update on one machine, it
may have some idea of what other machines are currently working and currently not working.
How can you correctly propagate that update if everyone doesn't agree on which machines
are working and not working?
And so the key problem is, can you tell which machines are working and not working?
That's a, it depends, right?
So it's, it's very thin and key problem in practice.
People are kind of detecting it, meaning they're guessing.
So that's, that's the fundamental problem.
So, so people have proven, you know, in some mathematical setting that you can have it
best to have this and that's, I think, motivating a lot of the more relaxed consistency models
that you see for these newer distributed databases.
We would all, as application developers like to have strong consistency, now as we're learning,
we can't really have everything we want.
You have to weaken the consistency model in such a way that you can still provide a reasonable
user experience.
So what are some of the directions you would go with that?
Sure.
So, so I think eventual consistency is the idea that, you know, your, your data will
come into consistency, but, but maybe not immediately upon the instant of the right.
So, so clearly the, the weakest guarantee is that it will someday happen.
Weaker still is, is no guarantee of consistency, meaning, you know, hopefully it will happen,
but if, if some server crashes or something, then there's absolutely no guarantee at all.
Right?
So, so, um, Volomorged and a lot of these more kind of systems that are, they're modeled
on Amazon Dynamo, try to provide something better than that.
So in particular, they try to provide guarantees about being able to read your own rights.
So, so it's, it's tunable.
You can make it faster by having less consistency or you, or you can make it a little bit slower,
still not slow, but somewhat slower, with, with better consistency.
So the guarantee you can give is essentially around reading your own rights, meaning if
you did an update, you want to get that back.
So in practice, this means, you know, if I, if I enter some information on a form on a
website and I hit submit on the next page, when you show me what I just entered and you
ask me to confirm, I better get back the same thing I put into the form, or I'll be a very
confused user.
And so, so I think you have to be able to provide at least that.
Otherwise, you provide a pretty bad user experience if this inconsistency stuff happens
at all frequency, frequently.
If it's one in a million, maybe you can say, well, that user will be a bit confused, but
it very rarely happens.
I don't know.
But, but we can provide that kind of read your own rights consistency, which I think is the
guarantee most people are looking for.
So if I enter a comment on a blog, somebody else does a read of the blog comments, are
you saying we don't necessarily care if that guy is 300 milliseconds behind the most recent
blog comment?
Yeah, so you can get into a lot, you can think of a lot of scenarios where consistency is
not as important.
And so, so things like a search system will almost always take advantage of this.
So if you enter a comment on a blog, it may actually be, you know, a minute before you're
able to find that in a search because, you know, a search index needs to be updated with
this.
The more you update the index, the more inefficient the system is.
So in fact, you know, a lot of things are doing that already.
So yeah, that's, that's exactly the idea is that, you know, in some cases you don't
need as strong as, you know, read your right consistency.
You just need, I get the update pretty soon.
Now in practice, pretty soon is going to be pretty soon, like, you know, less than a second.
It's not, it's not, you know, 20 minutes.
So we're forced by the growth of data and scale in these web applications to weaken the
consistency model.
We're learning that this really strong guarantee of consistency was a little bit over engineered
for some cases where it wasn't really required.
If you need it, you need it.
There's no, there's no arguing with that, right?
If you have an application which requires you to, to have, you know, a precise view
of the state, then that's, that's what you need.
Now there are distributed algorithms which provide something like that.
They provide, you know, a perfect consistency on right, but they're slower.
So, so, so those are kind of the traditional distributed database algorithms, two-phase
commit, two-phase commit, paxos.
They're in some ways stronger algorithms, but they are, you know, depending on the system,
more, more prone to getting stuck and, and, you know, giving you, giving you nothing or,
you know, sacrificing availability.
So, so you have to kind of actually make that trade off of, you know, is it better for the
website to go down sometimes or is it better for someone to get, you know, a stale result
sometimes?
Those are kind of the, the trade offs you have to make.
Unfortunately, you do have to make some trade off.
You can't, you can't get it off.
Voldemort relies on a concept called consistent hashing.
Tell us about that.
It's, I think a really simple idea that, that's been explained in a complicated way.
But the idea is, it's, it's hashing, meaning you, you, you make up some nonsense number
or bytes related to your value and that is going to control the location where your value
is, right?
So that's, that's what hashing is.
In a normal hash table, you would put, you know, you would, you would make a little array
of size, you know, 10 or whatever and you enter things into that array till it gets
too full and then you resize the array and you copy everything out of that array into
a bigger array, right?
That's how, you know, whenever you had your last implement a hash table, not something
I do that frequently anymore, but that was, that was how you did it, right?
So the problem with that becomes, it's, it's a great system.
That's how most, you know, in memory hash tables work.
It becomes problematic when, when data is on disk somewhere or over a network because
obviously recopying all the data around would be a big problem.
If your hash table is split up onto multiple machines.
Right.
So that's, that's exactly the idea.
So actually this problem was first encountered with disk hashes, which were not distributed.
It was just, you know, instead of a hash table, you'd have a big file and you would hash
data into the file as opposed to a, a B tree or something.
And they came up with a totally different solution from consistent hashing, but it provides
similar properties, which is you don't want to have to move all the data.
You don't want to have to recopy everything.
So in the case of Voldemort, we're using hashing to locate stuff.
So you know, where is my data?
Oh, it's on this machine, that machine and this other machine over here.
So we need to be able to answer that question.
So there's, there's two ways to answer.
One is hashing, which is you kind of predict the location, you know, calculate the location.
The other is store the location, right.
You can keep a big lookup table and you say, okay, well, where is such and such?
Hashing is much, much easier to scale.
So that's, that's why we're doing that.
The problem again becomes, you know, well, okay, when you filled up the machines you
have, how can you move data to the other machines without moving everything?
So, so when I bought my, you know, I had 10 machines, when I bought the 11th, how can
I fill it up without rearranging everything?
You'd really like to move about one tenth of the data in that case, ideally.
That's right, that's right.
So that's, that's the guarantee that, or that's the property that these consistent hashing
methods provide.
So a couple of ways to go about it.
But, but essentially what you're doing is you're, you're mapping keys to some kind of
arbitrary partition, and then you're mapping partitions to machines.
So the mapping of key to partition is static, but the mapping of partition to machine is
allowed to change and that allows you to, you know, have more flexibility in arranging
your data.
So we've been talking about these distributed systems concepts for a few minutes.
Let's come back to Voldemort.
What does a programmer see when they sit down to write an application that uses Voldemort?
Sure.
So, so essentially the model is, is just like a hash table.
And if you can't iterate over all the keys quite as easily, but, but that's the idea.
Right?
So, so what can you put in a hash table?
Well, you can put a simple object, or you can put a list of objects or whatever, but
that's, that's the interface.
So, so for, for simple purposes, you can actually mock it out with an actual hash table and
we can write some helper class for doing that.
But presumably you're going to have something persistent across multiple machines when you
use it.
So, is it the responsibility of the programmer to take whatever data they have, some complex
graph of objects, serialize that, and then Voldemort doesn't care what it's storing?
Yeah, so just some degree.
So I guess there's a, there's a couple ways to design this.
So some people have done just kind of a dumb byte array.
So the system has no idea what it is.
It's just bytes.
It's up to you to get the bytes in and out correctly.
Other people have kind of committed to a more traditional data model where they say,
okay, you can have strings, you know, like you would have in a database, you have Varchar
and you have Big
and whatever, you know, numeric types and string types you have.
So the downside to dumb byte array is it becomes very hard to peek at your data if you're the
operations team and it becomes very hard to remember what so-and-so put in there.
When you're the poor guy who's maintaining so-and-so's code, the downside to committing
to an actual schema like you would have in MySQL is if you want to do something crazy,
like store a personalized search index for your personal inbox or if you want to store
like an image, then that's obviously not going to work as naturally in some other serialization
scheme.
So what we do is we instead kind of make it pluggable.
So there's a number of good serialization systems out there.
We added one which has a JSON data model but is a more efficient binary format.
But there's a number of good systems out there.
There's protocol buffers.
There's a new one called Avro which is being used for Hadoop.
There's Thrift which I think was one of the first open source ones.
So these are pretty good systems.
They provide some code generation capabilities.
They support the kind of data types you would want.
You could store totally opaque data or if you use one of the provided serializations,
then it gives you a little more self-describing property on the data.
That's right.
So in particular, I guess the server knows the serialization type.
If you mark it as a protocol buffer object, it will know that it can be serialized as
a protocol buffer object.
So that's useful.
The main reason we went this kind of middle route is a feature we just recently added
which is views.
So a view is a whole thing and a relational database but for us it means just being able
to do some server side transformation, some processing on the server.
So in order to do that processing on the server, you need to know what your bytes mean.
You can't just process an opaque byte array in any meaningful way.
Does any support provided for an XML data type?
Not directly.
So what we found is XML is not a great way to store.
It's a great way to store documents.
We haven't found it to be a great way to store values, all of which will have the same schema.
So in particular, the reason being it's just a bit inefficient.
It's costly to process and it's costly in terms of the number of bytes you store.
But you can certainly do that.
I mean, what we have is essentially a string serialization type which would allow you to
put in any string.
It doesn't check the XML validation.
What programming languages do you provide an API for?
So the original one was just Java, but we've added a number of others.
So we have one for C++, we have one for Python, and we have one for Ruby.
And if anybody else knows another language that wants to write a client, it's not too
hard to do.
It's nice, it makes it accessible to people in other languages.
Project Voldemort implemented entirely in Java?
Yeah, yeah.
I mean, only the other clients are outside of Java.
Now we're going to talk about the architecture.
Could you break it into pieces for us at the high level and tell us what the pieces are
and how they work together?
Yeah, sure.
So, I mean, there's a couple of problems you have to solve.
One is where to put your stuff, which machines are going to hold what?
And so that, we talked about the consistent hashing.
That's kind of the routing layer.
It's going to deal with the fact that there's multiple machines that each reader right needs
to go to.
It's going to choose those machines using consistent hashing.
There's some other, it's actually pluggable.
So if you have a different hashing mechanism, you could kind of implement your own.
Another problem you're going to have to do is solve this, is getting data back and forth
over the network as quickly as possible.
So there's kind of a networking or a remoting layer.
And there's a storage layer, which is, where are my bytes on this machine?
Are they in memory, on the disk?
How can I get at them?
So those are kind of the main layers, the things that are happening at the routing layer.
Choosing the machines a thing goes to getting back all the results, repairing any inconsistent
data on those machines, and handing back the single result to the client.
The network layer is just kind of very simple socket access.
We have also an HTTP version of that layer, but hasn't proven to be greatly beneficial,
because it's a bit slower.
If I had a programming language I wanted to use it, you didn't have a client for it.
I could use HTTP, correct?
You could.
There's still some stuff you'll need to understand.
There's some versioning of the data, which you'll need to kind of deserialize.
So even if you're communicating via HTTP, you still need to be able to understand the
versioning of the data and what values you're getting back.
So it's not the simplest kind of RESTful model, but it is much more efficient than
giving you back a XML description of whatever.
So that was kind of the trade-off.
We felt like this was a storage system is inherently strongly coupled to the application.
It's not something that's going to be publicly accessible on the internet.
It's inside of your application.
We felt like it was more important for it to be fast than do something more RESTful,
which would provide the most easiest compatibility model with every other language, but would
have other drawbacks.
So you were partway through the layers, and I ask you that question.
So let's finish that up.
Yeah.
So that's what the network layer is doing.
The persistence layer is probably the thing that the most thought in databases has ever
gotten into is what's the data structure for this.
In practice, everyone has ended up with B-trees, but there's a lot of other stuff out there,
in particular kind of variants which are more like a log structured file system that kind
of right ahead.
What we're using at LinkedIn is something based on BDB's Java edition.
So they have a very good kind of right ahead B-tree implementation.
BDB, that's Berkeley database.
Yeah, that's Berkeley database.
Or SleepyCat, I guess some people call it because that was the company that made it.
So that's what we're using the most.
The other thing that we have is something which is probably more efficient than that
for kind of read only batch computed data.
And then we've done a number of experiments to try to, you know, the idea of having a kind
of pluggable layer for this is there are real trade-offs that have to do with how many
writes you have, how many reads you have, what are the size of the values, and a number
of other things like that, you know, how variable is the request stream.
So we've done kind of a number of experiments on improving this.
And I think we haven't gotten something we're completely happy with as a solution to BDB,
like it just isn't as good all around.
So we've stuck with that for now.
The ordering of the layers is not necessarily fixed.
There's some flexibility in the type of network architecture you choose and how you arrange
the layers.
What are the choices that you have there?
In particular, one thing we did when we built this, which, you know, it has some advantages
and disadvantages, is we tried to build each layer as kind of a decoration on top of the
other layer, which means you can actually reverse the layers to some degree.
So clearly you need to serialize your data before you send it over a network.
You know, you need to turn it into bytes before you can send it over a network or put
it on a disk.
But it is actually possible to reverse things like routing and network, which has some
interesting trade-offs.
In particular, it can make it so that you can have a very dumb kind of client.
And the dumb client just sends the data to whichever server it knows about.
And that server sends the data to the right places.
Or you can have a smarter client, which has one less hop, and just sends the data directly
to the servers that will be eventually storing this.
So the trade-off has to do with the implementation complexity of the client versus the latency
and performance and also how tightly coupled the client is to the server.
In what situation would you want to put the routing on the client rather than on the server?
When you can, it makes sense to spread the load to the client and avoid the network
hop just because what you pay for the network hop is probably not worth it.
So I guess currently today, it makes sense to do that if you're doing it in Java or C++
because those are what we have a rich client for.
So for LinkedIn, that's what we're doing.
We've had pretty good results for that.
What software runs on a server node in a cluster?
So essentially, there's kind of a server process that can be deployed as a war in a
server-like container also if that's better for your setup.
And what that's going to have is it's going to have the storage layer, the storage engine
that we talked about.
It's going to have the network server or servers if you have both the TCP IP and HTTP servers
configured.
And it's going to have some of the background consistency stuff.
So we have kind of administrative API.
We have a few other things which are running in the background that might do background
cleaning of the data and stuff.
That's the main software on the server.
And in addition, we're going to have this routing layer for doing server-side routing
which sits there in the background, waits for a request which is as yet unrouted.
And then we'll send to the right servers from there.
Are there any distinguished nodes in the network that place a special role or are they all
uniform?
Yeah.
So one of the decisions we made and it has real trade-offs is that all the nodes are
uniform.
So it's completely pierced.
There's nothing different.
The only thing that distinguishes one from the other is it's ID.
There's no master.
There's no election of masters.
So that makes certain problems a little bit harder and then certain problems a lot easier.
So in particular, we think it's a lot easier to operate that kind of a system because you
don't have this special node or tiers.
So other systems I've seen that some of which are very good have a very layered architecture
with five or six components which are actually in completely different programming languages
and work differently and they fulfill some role in the system.
We don't have that.
We just have a bunch of computers which are all identical and they may be doing one or
two different types of things but all the nodes are identical.
From an operational standpoint, that's a simple one.
Usually, in particular, if you have six different pieces all performing some role, then your
operations team is going to need to know what all the pieces are, how they work together,
how they talk to each other and you then need to, you know, you have six possible bottlenecks.
If you can do it in such a way that you just have one type of machine, then your only bottleneck
is how many machines do we have, hopefully.
Can you add or remove nodes from a live system and have the system self-organizing?
Yes, so that's something we're working on.
We haven't really had that feature.
Rather, you can always add data to a system but your data would become unavailable.
That key would be the consistency guarantees would be lost while it was being moved.
We haven't really had that feature implemented correctly.
That's something we're working on.
I think we almost have it.
We have in trunk now, we have a very, very alpha version of it.
The main features we needed to get that was the ability to stream data between machines
efficiently.
You can't just do a million puts and gets to move data.
You have to stream it efficiently.
Also the ability to change some of the metadata on the fly and expire the metadata that clients
have so that they now talk to the right machines and so on.
Have that feature then if you want to increase the size of a cluster, do you need to take
it down and run some kind of a script to reorganize the data?
People typically build clusters out of homogeneous hardware or can the system use different sized
hardware efficiently?
It can use different sized hardware efficiently.
Whether or not that makes sense is another question.
We are doing that to some extent but I think we're trying not to.
What we have the ability to do is keep more or less data on a particular machine and in
practice that will affect a few things.
It will affect how much disk space you're using, how much I-O and I-O operations you're
going to have.
It will also affect how many network requests you get, how much CPU you get because that's
the number of overall requests you're going to get.
That's kind of the tuning knob for supporting different hardware.
Now in practice that may be a bit of a pain to manage if you have to remember O, Node 21
has extra stuff because it has seven hard drives in there fast and Node 22 has two hard
drives and it's really slow so it has less stuff.
It also kind of makes the performance of the system harder to characterize because if you
draw your monitoring graphs in this one's different than that one it's a bit of a pain.
But it may turn out to be a good feature.
So yeah we support it but we're trying not to use it at LinkedIn just because it's easier
to manage homogeneous systems.
Do you have any modeling tips for anticipating the size of a cluster that I'm going to need
before building it?
Yeah so it's actually kind of an easy problem.
So usually performance tuning for storage stuff is very difficult.
The reason is in general the variance of I-O is very high.
You have your actual disk operations which are extremely slow and then you have your
cache disk operations which are instantaneous and so you have some mixture of those and
you don't quite know what you're going to get for a particular query.
But since we're only providing a very limited set of fast APIs it's pretty predictable.
So the things that you need to understand are what percentage of my hot data is in memory.
So the best way to estimate that is what percentage of all the data is in memory.
So if you have a terabyte of data and you cut it up among ten machines and then you store
two copies of everything or three copies of everything it's very easy to figure out okay
how much data is there on each machine.
And then the other question you would ask is well how much memory do I have on those
machines?
And that's kind of the raw proportion of data you would have in memory if there was no
locality of reference or whatever.
So if you're running at a ratio of 50% in memory you would expect pretty quick in terms
of performance.
If you're running at a rate of 10 to 1 on disk then you would expect much slower because
most of the time you're going to be doing a 10 most second disk seek to find your stuff.
So that's kind of the basic guideline.
If you want to try a real problem the way we always do it is we say okay roughly how
big is my value and then we try it.
And we see the performance and then we extrapolate out.
So if I had because it scales linearly you can say okay well this is performance for
this many on one machine if I had five more like that this is be what I would see.
So we essentially hold that ratio fixed and then multiply out by the actual amount of
data and the actual amount of machines.
And that's been pretty realistic.
I mean there's always some surprise but you're expecting more like plus or minus 10% rather
than oh my god I have a full table scan and nothing works at all.
This seems to me an advantage of this type of system compared to relational databases
which scale very non linearly.
Yeah so in practice I think it's one of the biggest advantages that if at least for us
if we're in the process of developing a feature you can think before you've written any code
you can you know take out your notebook and you can do a few paper and pencil calculations
which will give you a very accurate estimate of how feasible is what you're doing what
are the hardware requirements going to be.
And knowing that very early on in the process instead of right at the end when it's all
done and it's too late to order any hardware anyway that's hugely advantageous because
then you can you can rethink about some of your requirements or whatever if it turns
out that you know it's just an unfeasible problem to solve or you can you can buy more
machines or whatever you need to do early on.
Let's walk through what happens when you do or write.
Okay yeah so I mean it's fairly straightforward.
I'll walk you through it with the client side routing so essentially you know you're going
to call put and you're going to pass in you know kind of a key and a value.
It's going to your value is going to be serialized into bytes if you've enabled compression it's
going to be compressed.
And then the routing layer is going to take that value it's going to take your key it's
going to calculate you know which machines might might have this you know are responsible
for this and the the order to do the rights in and it's then going to hand those off to
the the lower level kind of network stores which are going to start doing these rights
to all the machines.
And then you know each each the network request gets sent to the individual machines the machines
you know de-sterilize the network request and perform the right to disk.
How many rights does it do of each piece of data?
That's a configuration option.
So what you want kind of depends on what performance you're hoping for.
So there's a couple there's a couple of knobs that you have.
So the first knob is you know how many times is each value written overall.
So that that's obviously going to control how many machines you can you can have down
and still get your data right.
So if you're right at three times and and you lose three machines then somebody's missing
their stuff.
The other knob is how much do you the person calling put how many of those rights do you
wait for.
So right you might you might not care at all in which case you may be wait for no rights
or you might wait for just you know the first right to occur the second right to occur before
you consider it done.
I'm coming to you with an application that I want to build what are some questions you'd
ask me to determine how to set those knobs appropriately for my application.
Sure.
So I mean I guess the primary thing that I ask is what you know what are the expectations
for the data.
You know what is the data is it something a user took time to enter or is it something
that we've just calculated typically something that you know your user has created is pretty
valuable and you don't want to lose it.
It's something that you've calculated is kind of less valuable and so that's usually it's
you know what how important is this how important is this data.
The more important it is the more the more correct the more correctness guarantee you
want right so the more replicas.
The more rights you wait for that's going to introduce latency from the user standpoint
correct.
That's correct yeah and it will increase the load on the system right so if you if you
do twice as many rights you'll have twice as much load from those rights.
Now what happens when you want to read the data.
So essentially the same thing will happen the key will be serialized the routing layer
will choose the you know the same nodes that you wrote it to.
You'll perform you'll perform a read from some number of those nodes which is again a configuration
property those requests will be sent off to those machines the machines will you know
look it up in the storage engine in provider result the result comes back with a version
attached to it so then on the client side or whoever is doing the routing server side
routing this would happen on the whichever server was processing your request the versions
will all be compared any version which is kind of obsolete will be scheduled to get
repaired.
It's doing multiple reads.
Yeah and looking for the best one.
That's right that's right.
So so essentially this kind of a system is actually postponing the consistency meaning
it's not consistent immediately on right but we guarantee the consistency properties whatever
you've kind of set on read meaning you have to go to more than one place if you want
the kind of read your own rights consistency.
So so yeah it's it's going to do multiple multiple reads maybe on a two or three it's
going to compare the versions on those objects you're going to get returned the most up to
date and it's going to schedule a repair on anybody who's out of date.
What is a repair?
It's just a put operation so so let's say that let's say that you know one machine was
down or or had some error for some reason couldn't get the earlier right well what happens then
clearly it needs to get the the right value eventually so there's a couple of mechanisms
by what you can get which you can get that but the simplest is if you just do a read we
get back three three versions and one of them is is old then we can schedule you know a
put operation that will happen in the background and update that value.
All right guaranteed to finish within any bounded amount of time?
Yes so all the operations come with a timeout so I mean one thing we found is for real time
applications it's usually better to fail in a guaranteed amount of time than to always
succeed eventually.
So in particular if you have a web page and you know maybe your web pages does 10 or 20
or 50 or 100 service requests to get all the information it needs it's really important
that you have some kind of fixed guarantee of how long each of those pieces takes otherwise
you know even a very trivial piece can break the whole thing you know you're as slow as
your slowest component so so we each each kind of operation has a timeout and when that
timeout's reached the operation will fail even if it might have succeeded had you waited
longer.
That enables application developers to write applications with some kind of latency guarantee.
That's right so in particular it lets you tolerate slowness on a single machine a little better
so we try to do as good as we can with that it's one of the harder problems but in particular
what you would like I mean in a typical application if you if you have one database and your database
is slow then your application is slow right what you would like if you have more than one
server is you would like to be you know not as slow as your slowest machine you would
like to somehow know that that machine was slow and not use it so so what we have in
place is this kind of an SLA that you set and you say okay it must must happen within
this time period if it doesn't that machine is going to be taken out and only periodically
used to retry it to see if it's fixed itself.
If I do a read or a write then there are multiple machines if enough of them finish the operation
will complete and the client can continue.
That's right so you get your result back as quick as the you know the fastest two or
three or however many you've configured that you must have.
Suppose I configure it to return from a write when zero storage is have completed because
I don't exactly care what happens on the back end is the storage layer guaranteed to finish
persisting things to a permanent state within any bounded amount of time.
No, no so so you if you don't wait then you have no guarantees whatsoever of what happened
right so so could be for example that the client you were on lost all network capabilities
you know the the very instant that you called put in so no no writes were done whatsoever
and there's nothing there's nothing that can be done about that so you have to wait for
some rights to occur before you have any guarantee.
Now second question you had is when does data actually get to disk it's an important question
so so the the interface within our system is that it's up to the storage and to decide
so you can write a purely in memory storage engine of course when you restart the server
it will will lose your stuff so you have to be aware of that we do have such a thing but
I think it's probably not that practical for for usage but in particular one of the the
huge performance benefits of a system like this is not not writing your data to disk
immediately so what that means is a your rights can be they can be done in kind of an append
only fashion and they it can happen kind of when when many rights have accumulated you
can you can kind of do a single update to disk rather than doing random rights all over
the place and so that's that's that's that's the first benefit is the disk layout proofs
but also the latency of our right is much better so so in particular if you're waiting
for you know a single disk C plus right you're talking probably at least 10 milliseconds
if you're waiting for that on multiple machines then you're waiting for the the longer of
these operations right because you know even the 10 milliseconds assumes that you get you
start right away but you may be waiting for other rights to complete or whatever else so
so if you're taking the maximum of three of those or something maybe much worse so you
want to you know we would want to avoid that for our production systems we usually see
you know very good very good performance so from client side could be you know millisecond
five milliseconds right so it's you know faster than a disk C from the client side even though
that includes network overhead serialization the designer needs to make some choices then
about the number of reads and the number of rights which you could look at independently
of each other but is there any significance to the number of reads in comparison to the
number of rights yeah so I mean the the two important guidelines are a you know really
do put it on more than one machine even though you're not used to that with the database
it just most of the features don't work if it's only if everything is only written to
one machine because with that machine fails and has a disk error the data is really gone
number of rights greater than greater than or equal to sorry strictly greater than one
so we found first of all people who thought that was a good idea was not a good idea probably
three is the right number I don't know why but two seems to be slightly too little because
sometimes you do actually have two machines that fail at the same time and then that's
you know you don't want to have to you know if you're waiting for a new machine to come
in or something or somebody doesn't have time to deal with it you don't want to have to
deal with that right away the other thing is you know the consistency guarantee of reading
your own rights that would require that you know the number of reads plus the number
of rights is at least the number of replicas so if you what I'm saying is you can configure
it in such a way that you're you're guaranteed to always read one of the newer versions that
you wrote you're never going to have a situation where there's no overlap between those two
so if you do that you're going to get back what you what you put in which is usually
what people want the system designed under the assumption that failures are short and
machine boots back up pretty quickly um no no so so so in particular we don't have that
uh very often if we don't sometimes we have we have extra hardware meaning if if a machine
fails it's uh you can swap in such and such I mean there's two kinds of failures one one
kind of failure is like oh we're just pushing new code so we we do a rolling restart across
the servers um the system stays up but a particular server is down for for a few seconds or you
know a few minutes um but another kind of server is like well another kind of problem is I
don't know we had some problems with uh raid controllers or we had some problems with disks
or whatever and the server is actually down for hours or in some cases you know a week um and so
in that case um yes it's definitely down for a extended period of time. The relational database
we push most of the computation out to the server it understands all the data it knows how it's
structured from what you're saying with this application you have some decisions to make about
do you want to do more of this processing on the client or on the server how do you think about that?
Um so most of the time there's there's not a problem uh it's fine to just you know send the
data to the client let the client deal with it um the the real case where you get into trouble is
if you just want a little snippet of that data or you're going to somehow aggregate the data maybe
by maybe you want to count everything that meets some criteria these are things which are kind of
very traditional SQL uses you know where x greater than two and uh whatever right so the first
assumption is you know because this is kind of a key value storage you've already denormalized that
you have some blob that has the stuff you want um but but you might say well it's it's sort of
inefficient for you to transfer me the whole blob then me to filter out just the things which meet
my criteria come up with a single integer which council them all and then use that right because
you transferred me maybe you know uh 50 kilobytes but I only computed something that was you know four
bites long in the end um so so your complaint would be well you know it's stupid for for for
you to send me all these kilobytes I should just you know send my code over to you once you do that
on your side um and and that will be more efficient um now in practice network is not always the
bottleneck so may or may not actually make anything faster but it does make it does make sense to
do this in a lot of cases in particular we say that you know our storage nodes are usually um
not CPU bound so so they have additional CPU capacity um and it it's always nice to have have
less network IO it's always nice to um to speed things up that way so so so that's the scenario
where you would want to kind of create a view which has your code now the downside to that is you
you're kind of coupling your application a bit more strongly to the to the system um I guess in
that in that respect it's kind of like a stored procedure as much as a view right because you've
you've sent some piece of your application over to the database server which is now performing
that for you so is that a good thing well it may it may reduce the the amount of data that's transferred
back and forth a view is a computation that you want performed on the server from a Voldemort
yeah yeah so so the the primary reason you would do that is to avoid uh network IO
the server needs to understand at least something about how your data is organized in order to
implement a view that's correct otherwise you would just have a bunch of bytes and it would be up
to you to kind of get it right uh obviously that could be very difficult to do um so so
because you you know when you create the the way it's organized is this it's not just one you know
the the whole cluster of machines is not just one hash table it's kind of like multiple hash
tables each hash table has specific settings about the number of reads and writes and other things
like that one of those settings is what's the serialization type for this data and so that
that kind of guarantees that whenever you interact with that data it's it's correctly serialized you
don't have you can't accidentally serialize it into the wrong thing and get some nonsense data back
so so as a result of having that uh metadata associated with the store we can correctly
serialize the data on the the server and apply the user's view transformation to it and then give
give back the transformed result you say that the system has multiple hash tables that would be
in a relational database like the customer table tables tables tables table so so i guess i didn't
call them tables just because they're not tabular but in fact that's much uh more straightforward
we call them stores because they're non tabular but but yeah they're just tables because the view
functionality extends to being able to join across separate stores or tables um so no no it doesn't
right so i guess the if if you're going to join data um you're going to go across the network
because the data you're going to cut across uh what's on a particular machine um because the data
is partitioned up amongst machines and if you're doing that there's no real savings of pushing it to
uh pushing it to the server side because the server is going to have to go you know to multiple
machines and transfer the data across the network anyway now um that said that's that's a perfectly
finding to do and in fact it's you know very common use cases to do do a bunch of gets uh for
different things and kind of assemble it all into one this would put on the client the burden of
what a query optimizer does in a relational database um yes yes it does yes so so whether or not you
think that's a good thing kind of depends um my perspective what this is all online i have also
spent a lot of time on offline systems in particular you know uh we have kind of hadoop and we have a
bunch of these distributed databases uh we've tried a couple of them we have astro data right now and
their their selling point is always the query optimizer and my pain point is always the query
optimizer because hadoop has no query optimizer you just tell it to do something and it does it and
if you tell it something stupid you know oh this is the stupid thing stop doing that and um so so
i found especially for for kind of a real-time website you need to know what it's doing um you
know if it optimizes it wrong and you get something which would scan every table across you know
20 machines or something that will completely crash your site and you know that's a terrible
terrible bug and so you it needs to be you need to somehow be able to program things in a way that
the optimizer couldn't choose that um and so as a result i'm not a huge believer in that problem i
think it's a very fun computer science problem because it's uh there's no right answer and you know
you see you can always work on it and write more papers about how to optimize that particular case
better but i think in practice for for a system you're better off uh thinking about the problem
and understanding your your data and then making making a good decision what you're saying now is
that things that we wish were hidden and that are hidden in a small system are not really hidden in
a large system even if you have some part of the system that's claiming to do that for you yeah so
i guess i guess the one another way to say the same thing that that's true and another way to say
the same thing is that um the the this these kind of key value stores they're they're
they're hiding things which were uh not hidden before like the number of database servers you
have and that kind of stuff so it's it's making it's some in some ways it's it's a higher level
system that's not making you think about that but other things are lower level like like how you're
doing your table operations uh is now lower level because you you're breaking it down into very
predictable operations on multiple stores which previously would be hidden for you and and just
batched up into one declarative statement so it's kind of uh simultaneously i think getting higher
level and lower level at the same time uh the the one kind of unifying theme is it's it's bringing
the api more in line with the um what the machines can provide performance wise so so i guess i
i like to think about databases versus the file system api file system api is a bit of a pain to
work with but it provides exactly what disks and files can provide in terms of performance meaning
you can seek and you expect when you call seek you expect that to take some time when you open a
file you you expect a certain amount of time for that so it hasn't abstracted over anything but
it's extremely you know predictable uh versus uh database query where you know you're going to have
one sentence of text it may take an hour it might take you know a millisecond it's very hard to tell
by looking at it which of those two it would be are there integration layers that make it easy to
plug Voldemort in in place of a relational database with rails or spring or any of the more popular
web frameworks um so so probably not i mean uh i think the way these things come about is something
becomes very standard and then people figure out what the pain points are and then they develop great
great libraries that kind of work around those pain points and i think for all these new databases
there's not that much i mean you know there i have seen a few little things here and there
um but i think they're they're probably not nearly mature enough that it's really
uh doing anything for you that you couldn't do yourself rather quickly so i mean uh obviously
integrates with spring fairly easily um there's no kind of ORM solution that will uh you know
transparently um map everything for you um now now primarily i think the reason for that and
certainly the reason we haven't focused on it is because the use case that we have we're not using
ORM either just because you know once you've kind of split your thing into services there's not a
very complicated mapping between between the objects and relations anymore um it's actually
kind of a simpler mapping and then you're calling lots of api so you have a an object api mapping
problem not a um object relation mapping problem if that makes sense because because like for a
service oriented architecture your your integration point is api's rather than than tables um so
versus if you have like you know a very simple application where you just have a single database
with a bunch of tables then you're trying to figure out how to piece together all those tables into
your domain model um so for that reason we haven't we haven't done any work on that uh it would be
interesting to see what people come up with um it's it's obviously going to be um a little bit more
difficult to do because a database can provide you a very inefficient query for anything
uh whereas these systems kind of only have efficient queries that's that's all there is so so if you
haven't provided the right uh structure for the lookup you actually can't you can't do it um
versus a database you would just do it and then you would notice oh such and such pages slow why
and then you would figure it out and you would put in the right index or whatever to make it faster
would it make sense to use Voldemort to store session data that the user needs to retain from
one page view to the next when you're in a clustered environment yeah yeah that's that's a perfectly
valid use case are you very familiar with uses of Voldemort outside of LinkedIn um i'm a little
bit familiar uh i usually become familiar if something doesn't work because then people email
this thing's out trying to do x why doesn't you know i expect to see why but i don't what's going on
but yeah so i mean what i've seen is basically um web companies and gaming companies seem to use
it uh they seem to be the people who have real-time problems with you know enough data to bother
looking outside of like MySQL um so it's usually fairly small cluster of machines you know maybe
half dozen dozen machines um they're they're small companies so that's that's probably funny uh so
you have an idea what are some of the larger clusters that people are building um yeah it's
probably not well outside of that range i mean for lincoln it's usually like you know maybe
ten machines or something is it is a cluster um some are some are much smaller than that so
so i think you know it's the the goal of course is to have as few as possible um so so we've gone
with a system where we have you know kind of like three or four separate clusters which are
completely isolated from each other and then a bunch of features grouped together on one cluster that
does something yeah suppose i want to move all my data out of Voldemort into a data warehouse so
we can do some offline analytics where we take advantage of the capabilities of the more rich
query language are there some tools to help with that process yeah there are um probably not as
not as many as as we would like a lot of these integration point technologies take time to
develop first you have to have very stable system then people spend time integrating it in all
different ways and then you get a good set of solutions for different problems so what we have
right now is kind of a batch streaming api that will let you stream your data into hadoop or whatever
what we don't have that i would love to have and it's kind of our next step is a good set of map
reduced jobs that work with any data type and kind of act as an input format or something to to get
you all your stuff out of your Voldemort cluster um and likewise it would be nice to have kind of
a catch-up mechanism such that you only get the updates from yesterday you don't have to kind of
re-read transfer everything um so those are those are kind of the two missing features what i'm
hearing from you in response to several of these questions is this is a fairly new technology and
all the complementary pieces aren't there yet but there's a lot of scope for obvious improvements
yeah so i think that that would be true i mean for for databases and file systems it's kind of a
decade-long project um and so so my advice to people if they're looking at this for something in their
product would be you know if you don't have a problem then you don't have a problem just stay
with whatever you've got uh because most of things which have been out for 10 years or more are
going to have a lot of things built up around them that you may not you may not realize you would be
missing with anything newer um that said if you do have a problem uh probably these systems are much
going to be much better and have a lot more infrastructure built up around them than doing it yourself
which of course then you'll have nothing uh so that's that's kind of the trade-off if you're
thinking about doing it yourself then then probably this will help you a lot and provide a lot more
built-up infrastructure if you're if you're thinking about just using MySQL then you know that's
that's a pretty good solution if that will if that will get the job done tell us about how the
Voldemort project is organized as an open source project it's loosely organized um i mean basically
what we have is we have maybe uh three or four people in lincoln who are doing a lot of work and
then we have maybe three or four people outside of lincoln who are doing a lot of work and then we
have a lot of people who seem to be kind of trying it or using it right um so people who are trying
it or using it give great feedback they say okay you know so and so just committed x and now
y doesn't work anymore please fix immediately and that's that's very useful feedback to get um
um uh the organization you know among people you know writing code is kind of like whatever uh you
know people who are not working for for lincoln are going to do whatever they they need um i think
there's pretty good consensus among all the people of what what we think is you know we would most
want um and then lincoln is kind of funding the features that that we need which you know around
monitoring and scaling and uh the ETL problems you described and other stuff like that that just
make it fit into our environment better lincoln then is the primary industry sponsor of the project
um yeah i think so and that that we're you know the we're employing the most people to work on it
for some open source products there's commercial entities that will provide support as a business
does anything like that exist for voldemort yet no no so i mean lincoln is not a database
company so you know you can definitely get a lot of help from people on the mailing list but
ultimately uh there's no kind of organization that you can uh sue if there's a problem so so that
that that you know there's no kind of 24 seven support or whatever contract that you can buy
um so that's that's also something to think about does the project get very many patches from the
community yeah so we get we get um you know kind of outside of people who are doing work all the
time i think we get a lot of just random patches that fix so and so or make such and such command
line easier to use or add support for some new compression type or whatever and those are those
are very helpful um where would people go if they want to learn more or maybe get involved uh there's
a website that's uh project dash voldemort.com and there's also a mailing list it's linked off the
website um and so you can always you can always ask you know questions on that mailing list if
you're having trouble setting something up or you're you're not seeing what you expect um there's
there's kind of a list of uh of what we thought were fun projects on the website so if you're if
you're like interested in this stuff and you want to you know take on some project and try it out
there's a list of things that were you know not too hard as to be unachievable but also pretty
interesting um and so you can check that out and see okay you know if you if you had a couple days
to to play around with it you could try out one of these projects what are some of the fun projects
you'd like to see done there's lots of them so that i mentioned the kind of hadoop integration
where we you know write the map reduce job that that um pulls all the data out um that's as you
add time down uh there's nio improvements on the client um there's uh views were one that i just
did so i need to take that off the list um so some of the fun projects we get around to you uh and then
let's see what else do we have uh there's there's a bunch of smaller improvements for um the read-only
story of gendron um implementation of kind of other uh replication mechanisms to be able to catch up
you know from a particular point like let's say you have a search engine you want to index
everything that goes in you need to have a subscription mechanism to to keep up with changes so there's
there's a there's a bunch of features like that um those are the ones that i remember up on my head
anyway do you write or blog anywhere if people would like to follow what you're thinking about
i haven't done that much recently i mean basically working on this project and we have a couple other
open source projects coming out and then i work at a startup uh as well so i have you know a bunch
of features i'm working on and keeping the website working i haven't had time to do much there is a
blog for project Voldemort so i have anything related to that i have posted pretty recently um on
different features or stuff we're working on and i think we're going to expand that as we add our
team has right now maybe three or four open source projects um and we have a couple more coming out
so we're maybe going to make it a little bit more general purpose to talk about stuff in that
area that's being done but yeah nothing nothing much outside of that i don't have a twitter account
yet so how can people reach you if they'd like to ask you anything so you can email me i'm jay.craps
at gmail um and i try to get back as soon as possible um best effort jay craps thank you very
much for being on software engineering radio thank you for having me
thanks for listening to software engineering radio software engineering radio is an educational
program brought to you by hillside europe if you want more information about the podcast and all
the other episodes visit our website at se-radio.net if you want to support us you can donate to the
se radio team via the website or you can advertise for se radio for example by clicking on the dig
reddit delicious links and the slash dot button to contact the team please send email to team at
se-radio.net or if it is specific to an episode please use the comments facility on the website
so other people can react to your comments this episode of se radio as well as all other episodes
are licensed under the creative comments 2.5 license please see the website for details thanks
to charlie krow and the pot-side music network for the music used in this show the song is called
biggest audio comments
You
