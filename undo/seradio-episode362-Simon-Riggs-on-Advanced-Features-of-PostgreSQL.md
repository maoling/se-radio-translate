**Link**: https://www.se-radio.net/2019/04/se-radio-episode-362-simon-riggs-on-advanced-features-of-postgresql/


This is Software Engineering Radio, the podcast for professional developers on the web at
sc-radio.net.
SC Radio is brought to you by the IEEE Computer Society, the IEEE Software Magazine.
Online at computer.org slash software.
Digital Ocean is the easiest cloud platform to deploy, manage and scale applications of
any size, removing infrastructure friction and providing predictability.
So developers and their teams can deploy faster and focus on building software the customers
love.
With thousands of in-depth tutorials and an active community, we provide the support
you need.
Digital Ocean stands out of the crowd due to its simplicity and high performance with
no billing surprises.
Try Digital Ocean for free by getting a $100 infrastructure credit at dio.co slash sc-radio.
Hello, this is Simon Crossley for Software Engineering Radio.
Today I'm speaking with Simon Riggs about advanced features of Postgres database.
Simon is a professional developer and consultant with lots of experience with a variety of
database technologies.
He's a regular conference speaker on database technologies and I first met him several years
ago at a Postgres user group meeting where he's presenting some of the clustering and
replication features that's committed to the Postgres database.
Simon's continued to contribute major features to Postgres and a CTO of second quadrant
consultancy has built a team of developers and consultants delivering advanced Postgres
implementations to enterprise clients.
I hope that introduction does you justice.
Simon, welcome to Software Engineering Radio.
Thank you very much.
Postgres is widely promoted as the world's most advanced open source database and today
I'd like to talk in detail about some of those advanced features.
But first, could you please summarize the ways in which you think Postgres is more advanced
than other databases?
Sure.
Well, I think the first and foremost feature is that it follows the SQL standard extremely
closely.
Now, that may not seem like the most advanced thing, but the standard is long and complex
and it takes a lot of work to get us to match that closely.
But in general, Postgres has got broad and very deep feature support across a whole range
of different use cases.
And it's my view that Postgres is more than just a relational database and I'd like to
explain that to you if we go in that direction.
A couple of other things I'd like to mention as to why we're the most advanced, the query
planner and in general, performance is very advanced in Postgres.
For example, there are six different types of index available with Postgres, many more
than most other databases.
And Postgres also invented the partial index and expression indexes, both of which are
very useful for performance.
Postgres is also very extensible.
We pretty much invented that thought.
And that brings in a whole vast array of add-ons that you can get with Postgres that other
people need to add themselves to the core engine, whereas Postgres can just expose an
interface to allow people to add their own things.
Okay.
The last one is actually ease of use, which is a strange one because some people would
say in the past it was hard to install and that kind of thing.
But the main thing with Postgres really is it just works.
And we design things in a way that causes the least surprise for developers.
And we also go to a lot of trouble to ensure it's robust and accurate.
For example, a couple of years ago we discovered that there had been some minor, minor error
in the accuracy of one of the functions.
And I think there was sort of ten decimal places into the results.
There was some weird error.
And we spent a lot of time fixing that.
Now that kind of attention to detail is what makes Postgres the most advanced because it
takes time and it takes love and care really to make the feature set full and complete.
Yeah, that's definitely something we can appreciate.
And I know that you personally spent a lot of time doing that on the team of people.
Thanks for that, excellent list.
I think that will give us a good structure for our conversation.
So let's start at the top and let's return to some of the features of the SQL standard
which you said Postgres has good support for.
I think most of our list ends will be familiar with the select statement.
But there are lots of more advanced features, SQL, that maybe they're not that familiar
with.
What are the things that Postgres supports that developers might be unfamiliar with?
What could be useful for them in their work?
Yeah, so there's some interesting additions to the SQL language that people are just simply
not familiar with.
Obviously people have heard of things like joins and sub-selects between tables.
But we've extended the insert command which allows you to implement something that does
what's known as an up-cert in that it will attempt to do an insert.
But if the row is already there it will turn it into an update.
So that's one of the first things.
So instead of getting an unhelpful conflict error, your statement will just work.
It will just work.
And what's more important there is that we spent quite a lot of time designing it in
a way that means that it will guarantee to give you an answer.
It doesn't just sort of whirl around forever without deciding what to do.
So it's quite a subtle implementation and it took actually multiple years to get that
to work.
So that was time well spent because it saves time for the user.
Excellent.
So moving on from there, my next point of call I think would be window functions which is
something that frankly many people have never even heard of.
But a window function is something that allows you to look at the results of a query in an
ordered way.
For example, if you're looking at a time series and you want to look at the last three points
on the time series, then you need a window function.
Now a lot of people think that that's not possible in SQL and it wasn't.
In the early versions of SQL it was quite a simple language.
And SQL did more than 20 years ago receive criticism for some of the things it lacked.
What people don't realize is that those things were added to the SQL standard and Postgres
has supported them for many years.
So for example, if you want to look at a time series, you want to calculate a moving
average ever data.
If you want to look at a lag, these are all things that can be done in the SQL language
and done not just we can barely touch them.
I'm talking about implementing fully with decent optimization.
So it's been implemented for more than 10 years and it's been highly optimized.
So that's the first thing.
What are the keywords that you would use to structure a window function in your SQL statement?
So there's an over clause which allows you to calculate aggregates across a window clause.
And then you can structure things so that you can say what type of window it's going
to be, whether it's a moving window or whether it's a window that goes back to the beginning
of the series.
So you might have a cumulative total alongside the total, for example.
So you can list out a range of numbers with a cumulative total alongside them.
Or you could smooth out your data across a number of different points, as I said, to
calculate moving averages and that kind of thing.
So there's a lot of complex statistical functions built into Postgres and that is one of the
ways that we support them.
And so in your example of a time series and choosing the last three points, if my time
stamp column is a series that I'm interested in, then I would run my window over the time
stamp column and then perform some statistics or other calculations on the resulting sets.
Yes.
So the results there are actually called window aggregate functions.
And so you would, for each row in the table, you would access the sliding window and then
you'd calculate the result of the aggregate across the sliding window.
Okay.
So, yeah, please don't ask me what the syntax is because I have to look in the manual as
well, right?
But that's not the point, is whether it fully supports the standard and does so in a useful
and performant way.
And I think that, importantly as well, it's the code that a developer would have to write
client side in order to do that work and to do that work in a way that's performant and
work over large data sets, that Postgres has that power, right?
Yeah, I very much say, yeah.
Another example of how we handle that is there's a thing called a with clause that allows you
to define what's called a common table expression, so that allows you to define a fragment of
code that could be reused many times within your SQL.
Obviously, that kind of thing makes it much more sensible to write correct code because
if you have to have sections of code that are similar, you know, then that avoids mistakes.
But the interesting thing about the with sub command is that we support the with recursive
form, which is actually in the SQL standard.
And what that allows you to do is to search a network or a hierarchy.
And quite surprising to many people is the fact that it allows you to write graph queries
inside Postgres.
And most people believe that's not possible and yet it is actually completely possible
and not just possible, very well performance in that it's been fully optimized as well.
So there's some some really impressive features in there that that I use in production and
in many, many parts of the world.
So how would I need to structure my data in order to be able to run a graph query recursively
over that?
Well, the main thing that's needed is connections between the data elements.
We don't impose a fixed structure on your data before you can use a graph query, which
some other approaches do require you to have a sort of strict structuring.
So if you have the old example from years ago was the employee table where employees
have managers and their managers can have managers.
And you can write a recursive query that sort of says, you know, what's the total salary
of all the people in my department, for example, and you would start with you, go out from
there in a graph moving from employee to employee.
And it isn't something where you for each link you need to write an extra join.
You can actually do that recursively.
So it doesn't matter how deep the hierarchical the network is.
You can still search it.
So we're not limiting you to a particular sort of property graph model or something
like that.
You can use any sort of data.
So it could be XML data or JSON data that's there and you're following links between data
items.
So all of that would be fully supported.
Oh, OK.
And the common table expression is when you say it's a with clause.
Yes.
So you're thinking that it's a definition of a view within your statement.
Is that the right way of thinking about those?
Yeah.
Especially when you're doing the, so the with clause is just a sort of fragment of code
that would be reused.
But the with recursive, you have a starting point for the query and then an iterator that
moves you to the next data item.
Again, these are very advanced features of SQL that most people don't know about.
And Postgres fully supports.
So, yeah.
Excellent.
Maybe, maybe now's a good time to move on to the broad and deep feature support because
you specifically mentioned there that it's not just SQL in the database.
And that sounds something interesting that we should explore.
So what's happened over the years is that Postgres, because it's extensible, it's actually
been used as a platform for development of wider features.
For example, GIS features were added by the PostGIS project.
And that's all been done by using the extensibility features, user defined functions, user defined
data types, various types of indexing that have been added.
And as a result, the PostGIS package implements the full GIS standard for use within Postgres.
So Postgres, when you plug that in, becomes a GIS database as well.
But there are similar sort of use case packages in full text support, other document support.
And we're also now realizing that Postgres actually already has graph support.
And we're adding time series capabilities as well by the fairly recent partitioning support
that's been added.
So the functionality really doesn't just go in one particular direction.
It covers a whole range of use cases.
And that was what makes it a very useful general purpose database because if you've got a database
and it has a little bit of graph, a little bit of GIS, some text, some normal relational
features, it can accommodate all of that all at once in the one application without you
needing to result to sort of microservice architectures.
So what kind of features, what kind of functions could I perform with a GIS database?
I mean, it's really to do with viewing maps and map data.
So the PostGIS package maps the world as if it was coated onto just a flat surface.
And then you can look at things in terms of sort of bounding boxes and lines between
nodes and things like that.
So it's actually extremely complex, the support.
It goes well beyond what your average database person would imagine.
And it's actually a relative world contender now.
It's used for extremely complex GIS applications.
Basically, there's a whole industry out there of people looking at geographic factors and
they store everything in PostGIS.
So it's got a wide domain.
And very often there's not much crossover.
So can I build indexes using geometric features and make that performance?
Yeah, that's one of the great things here actually.
There's multiple types of index that you can build for that.
So there's the equivalent of archery support, which is pretty useful.
And then there's a KD tree that we can use.
So there's actually multiple styles of index which have different properties depending
on what types of things that you're looking to optimize, whether you're trying to optimize
search time or whether you're trying to minimize the size of the index or optimize for maintenance
of the index, for example.
These are all things that you can sort of select your choice of index type depending
on your use case.
Sorry, it's so complex, right?
Yes, that's the thing I'm trying to get across really.
And that's all kind of through the PostGIS extension.
That's how those features added into the database.
And as similar as that features for temporal data and for the other additional types.
Yeah, so we did actually have the full text stuff as an external package.
But some years ago that was brought directly inside the server core.
So we're now we've got full text support actually in the server core.
But the indexing that I was describing there, the index types are part of the core server,
but some of the implementation exists in the external packages.
So for example, there'll be an external package that implements a particular data type and
then searches on that data type would be would use the internal index mechanisms.
But yeah, I mean, basically it's not just one database, it's many databases because you
can build the types of support you need and to give yourself a kind of integrated platform.
So I really think of it more as less of a database and more of a data platform.
Okay, and the full text support is that will that cope with free text and structured text?
Yes, very much.
So we've got a stammer package built into the core.
There's quite advanced full text support.
You can set up dictionaries and stop word lists.
And there's quite an extensive range of features there.
By no means a trivial sort of box tick feature.
It's the full deal really.
And does that have a query interface that just slots into SQL?
Is there any special keywords that I need or have access to?
Well, a lot of it is done via functions.
The actual configuration of it allows you to set up parsers and dictionaries.
And you can have dictionaries in different languages and that kind of thing.
So it's actually a full and complex support, which also supports collations and that kind
of thing.
So yeah, it's not something that I want to delve into in massive detail on a call.
I mean, I could spend a whole call just on that topic.
And are these extensions that you're talking about, are these different from the foreign
data wrappers?
Yes.
So the foreign data wrappers is an API that allows us to write access packages to other
databases.
So for example, there's packages written that will allow you to access MySQL or Oracle or
sort of generic JDBC data sources.
And that will, what you do there is you set up a thing called a foreign table, which is
essentially a stub.
We access the stub.
The SQL then gets passed through to the remote server.
It will execute a query and then bring data back to us.
So that really allows Postgres then to become a kind of data hub across all the different
database types.
And I know that that's quite popular with MongoDB who use the Postgres technology to
give you access to their own data store.
That's really interesting.
You use the words data platform, data hook.
It's really extending the concept of a database.
Well I think that's really where we're going because time was when you put stuff in a database,
it just kind of sat there until you got it back out again.
Now there's much going to requirement for it to do something useful in the age of business
intelligence.
We need more from our data than just simply a storage mechanism.
Basically you need to depend upon those basics of storage and high availability and backup
and that kind of thing.
But working beyond that, it has to provide a wider range of functionality to fit in the
modern enterprise.
There's too much going on to just store it and then bring it back again.
It's not enough just to do that anymore.
Okay, so you have features that you can help process the data as well.
You mentioned the query planner as being advanced.
If you're handling all of these different types of data, I can imagine you're putting
some real heavy loads on the query planner being able to optimize queries in all these
different scenarios.
Yeah, I mean that's interesting point.
You might imagine that some of these things aren't fully or probably optimized.
Well, what we've also done as well as extend the database so you can add user defined data
types.
But with those user defined data types, you can also add into the database statistics
collection routines that are appropriate to those data types.
And so as a result, you can write a query that says, give me all the data in that bounding
box.
And it's all fully optimized.
And what we've done is to extend the statistics system and the optimizer to cope with it.
Okay, so as well as extending the types of data, you extend the query planner and the
optimizer to be able to handle them as well.
Yes, yes, by extending the support for new types of statistical data about those data
types.
That's really interesting.
And you've also talked about the different indexes in Haiti.
So you mentioned a couple of indexes for the GIS data types.
These are the particular types of indexes that developers might not be aware of to use in
Postgres.
Definitely.
So the basic index type that everybody will be familiar with is the tree or B3 index type.
Then people might also have heard of a hash index, which allows you to go straight to
the data without sort of navigating the full tree.
They've also extended that.
So if you've got data that is located by date or by number, you can use something called
a block range index, which is a very small index type.
So it's designed for very large data.
We also support an inverted index called a GIN index.
And there's another two index types called a GIST and an SPGIST.
And the G in all three of those last index types mentioned stands for generalized.
And there's been a lot of research in the last 10 or 20 years about generalized search
trees or generalized indexing mechanisms.
And some of this research to create these concepts was actually sponsored by the US government.
And that work has been taken and put into Postgres.
So as a result of that, we now support not just different types of index, but index mechanisms
that can be further extended themselves.
And that allows you to support multiple different kinds of structuring within an index that
allows it to work efficiently in different ways.
Gosh, I'm not even going to try to explain some of those things on a call, but the main
thing is it gives you options.
And you can use those options to make some good decisions about how to index different
types of data.
What would be a good use case then for one of these generalized index types?
OK.
So Postgres was actually the first database to implement something called nearest neighbor
indexing.
So if you, for example, were holding your mobile phone and you want to press it and say, show
me the nearest coffee shop, you might think that that is kind of an easy type of query.
But it turns out it's not because you need a specialized index type that will cope with
the fact that you're actually looking at things near you, rather than in a specific place.
And Postgres was the first database to implement that type of query via an index.
Previous to that, people used what was called bounding box techniques, and they were at
least 10 times slower.
So Postgres implemented that using an index.
It's called nearest neighbor indexing.
OK.
And is that a calculation between two points?
Exactly.
Yeah.
So it sounds quite complex, but if I showed you the query of it, you'd go, OK, yeah, that's
pretty easy to write.
So basically, we just have a column that says location.
We put the specialized kind of index on it, and then we just use a particular form of
where clause to retrieve the rows from it.
And it works using an index rather than doing some kind of full data scan.
OK.
That's interesting.
So is that your indexing with an expression?
Yes, that's great.
So you can look it up.
Basically, it's K nearest neighbor, KNN indexing allows you to look at the data points that
are the nearest to you.
And then obviously, you would put an extra filter of coffee shop on that in order to
spread things out and make that into the search that I described.
No, that's good.
Are there other uses of expressions within indexes?
Yeah.
I mean, we support expression indexes as well.
So you can build an index on the first 10 characters of a field, say, or you can index
well, any expression that you can execute.
So any function that would retrieve a result from a column can be indexed.
So it's quite advanced functionality.
And we also support partial indexing, which means you can index all of the rows that aren't
null, for example, rather than having to index all of the rows in the table.
So that can save space and improve search performance?
Absolutely, yeah.
Exactly those two things, yeah.
Digital Ocean is the easiest cloud platform to deploy, manage, and scale applications of
any size, removing infrastructure friction and providing predictability so developers
and their teams can deploy faster and focus on building software that customers love.
Digital Ocean stands out from the crowd due to its simplicity, high performance, and no
billing surprises.
Join a community of over 3.5 million developers on Digital Ocean.
Line up with a free $100 infrastructure credit at dio.co slash se radio.
It's obviously indexes are, you know, core to making it database performance.
Is there anything else that we should cover with indexes, do you think?
Well, I mean, there's many other good features, but I think some one long term guy at one
of the other vendors was asked a question on his retirement.
If he could go back and spend more time on what would he spend time on?
And he just replied indexes.
Right.
And as a result, that makes me think we've done the right thing by having the easily
the widest and most complex set of index types of any database, even beating commercial.
And that's, you know, referring back to this concept of these generalized indexes.
Yeah.
And their extensibility.
Exactly.
Yes.
We keep coming back to extensibility.
So really does seem, you know, postgres, postgres core functions have been built from
extensions over the years, pulling functionality in.
Are there other particular extensions that are interesting and relevant at the moment?
Well, I mean, there's actually what this creates is actually an ecosystem of tools for people
that want to build additional functionality in the system, but they don't want to spend
the time to bring it into the core server.
I mean, we release on what was once regarded as quite an aggressive timescale of annual
releases.
But in terms of if you're trying to build an application, you can't submit something
to postgres, then just wait for the next release because that's basically 12 to 18 months.
So if you want new functionality now on your system, you can build an extension and you
can get it within one to three months.
You can have that functionality built into your database.
So it's a way of expanding the functionality of the data platform, but it's also a way
of delivering that functionality in usable form for applications.
So delivering it quickly as well.
But as I said, what that's done is it's created a whole ecosystem of hundreds of small tools
and utilities and plugins, all of which together make postgres into a whole range of powerful
different things.
So it's not one thing.
It's anything you want it to be.
Okay.
If are these extensions difficult to write, are they complicated interfaces that I need
to understand to write well?
Postgres supports quite a range of internal IPIs that you can have access to.
A lot of the extensions out there are open source and so you can copy an existing one.
So really what you need to work out is which APIs should you be using to provide this additional
functionality and how am I going to go about using that?
Obviously, if you're building an extension that you expect thousands or millions of people
to use, then you need to go to a lot more care in the development of it than you would
do on a single function just for your usage.
So there's a whole range of emulation functions and monitoring facilities.
For example, second quadrant publishes the multi-master extension, BDR for bidirectional
application and that is available now as just an extension to the database.
So it just plugs straight in and away you go.
Okay.
So that's obviously an advanced feature data replication going beyond just handling different
types.
Postgres performs that job very well.
Maybe it just describes the concepts behind the replication.
Yeah, sure.
So we support a couple of different kinds of replication.
The first kind is physical replication where we actually take a copy of the transaction
log and we send that to another location and by doing so we're able to offer high availability.
So if you stream that in real time to another server, then you can have the choice of sending
that asynchronously, meaning you commit your transaction and then you send the data somewhere
else or you can do it synchronously where the commit waits in a queue until your data
has been sent to a secondary server and then it returns.
So you get your choice of whether you want to add latency to give you protection guarantees
or whether you want speed in that the commit will come back to you quickly but you don't
yet know whether your data is safe and Postgres actually does that in quite a clever way in
that we allow you to make that choice at the individual transaction level.
So you can have some transactions that you consider important ones being committed synchronously
and slightly less important ones.
You can have executing much faster asynchronously.
So that's the physical replication, what we then do is when we ship the data to a secondary
server, we run that server as what's called a hot standby server and you can actually
execute queries on the standby nodes and what you can do there is you can add as many standby
nodes as you want to give you a full server farm where you're able to execute thousands
and thousands of queries across the many different servers.
So that allows you to effectively have an extensible cache built of additional servers.
We also support cascading replication which allows you to have those servers connect together
so you can actually have a hierarchy of standby servers to give you a full server farm.
What we also have is something that we've introduced in the last couple of years is
called logical replication and this is where we only send the data changes, not the whole
transaction log.
So the first reason is that that saves bandwidth when you're sending it across region.
But the other thing is that the logical replication is much more flexible and we can actually
use it as some kind of real time data pump to other applications in the enterprise.
And most importantly we can send it to other versions of Postgres.
So as a result of this functionality we're able to do a full online upgrade so a near
zero time upgrade, zero downtime upgrade of your servers and obviously that's what people
are looking for now is to never take their systems down.
Okay so you can maintain availability because it's a logical replication.
You can only physically replicate the transaction log between identical versions of Postgres.
That's correct, yeah so even though it's quite an efficient replication mechanism it's not
one that works cross version and so if you want to publish data across to a remote site
you really would prefer the logical replication mechanism but it's because of its ability to
go cross version it allows you to continuously upgrade every time there's a new major version
comes out you can upgrade fairly soon to that you have to wait three years for the next
sort of window of opportunity which is what used to happen in the old days.
If I want to build a high availability system with a large amount of data in it you know
what kind of orders of magnitude of numbers in terms of database size and bandwidth what
am I kind of looking at for the capabilities that I need from my hardware?
Well I mean things are very efficient now so I would not like to sort of throw too many
numbers at you I can tell you some sizes of things but it really does depend upon the
hardware that you spend your money on as to you know to what kind of performance you're
getting out of this but there are Postgres databases out there that are above 50 terabytes
on a single node and you know it's very frequent to see databases above 10 terabytes or above
a terabyte so I would have said that most of the customers that I deal with personally
are systems above a terabyte in size and they have extremely high transaction rates I think
that's the other thing that's changed in recent years is it isn't uncommon to see transaction
rates of more than a thousand transactions a second and in some cases you'll see above
10,000 or more in many cases so it's you know the amount of data flying around now is really
quite extreme and it's difficult to get your head around the differences between sort of
low end transaction rate systems and the very highest transaction rate systems there are
people out there that are really pushing the barriers back of what's possible.
Is that where you would need the the farm of service in order to really get the optimum
use from Postgres in those scenarios?
Did we look at paralyzing queries and things like that to get those to run?
Yeah so parallel query currently with with core Postgres operates only on a single node
so if you've got sort of four or eight CPU systems then you can get some decent parallelism
going on those single nodes in terms of Postgres operating across multiple servers you'd need
to be looking at something like Postgres Excel before you could operate across multiple nodes
so it's a few years yet before core Postgres itself will will provide that functionality
but again this you know there's different versions of Postgres and different packages
that you can get that enable that type of functionality.
Okay interesting and is that replication something that's being actively developed at
the moment are there changes in future releases?
Yeah I mean we've we've just made a load of changes in Postgres 12 that relate to how
the configuration works and that there's been pretty continuous change in the replication
functionality over the years I mean the streaming replication feature went in in 2010 and then
there was a sort of big surge of work after that but actually the amount of change has
stayed fairly constant it's what I think is surprising is that or maybe it's not surprising
but because people are using it heavily in production they've got lots of small requirements
for performance changes and things like that so for example I about 18 months ago I spent
a lot of time myself working on replication latency and we removed sort of various bottlenecks
that were occurring there each of those things takes quite a lot of work but you know it
obviously it's what you might call polishing work rather than you know initial feature development
but it can still be very complex so there's lots of things still going into Postgres
it really I think that's the surprising thing when you when you call it the most advanced
relational most advanced database you would think that we would you know would be feature
complete but it's actually the opposite way around the more users we have the more demand
there is for new features right your work will never finish yeah it looks that way yeah
yeah thankfully and we talked about some very advanced you know use cases there's some very
high volumes high transaction rates but back at the beginning you mentioned you know that
Postgres is easy to use yes you know in install and use so is that is that a journey from your
from your first install to you know the multi-node server farm yeah you said can you talk us through
what that journey might look like and steps on the way well I think you know especially with
the replication stuff I know that people go will say our replication is really complex but
believe me we've simplified it a lot from the way other vendors have implemented it so for example
in order to make your application synchronous you only have to touch one parameter to change that
and the number of application parameters overall is relatively relatively small so you know what
you need to do to to get the servers to work together is you just set up a node with the software on it
and then you can launch what's called a base backup to copy the initial server yeah and then you
start it up and it will begin talking to the master node and it will bring across all of the
changes and it will just continue to work so these things you know I mean obviously we have made
it even simpler over the years but in essence we designed it from the from the very first go that
it was going to be as simple to use as possible and with with very few parameters and commands
and I think we you know we have achieved that obviously as more people add complexity
that parameters emerge but in order to build up a whole server farm you basically just point
the standby nodes at the master and then begin flowing the data across and these days that is
just a single command in order to get it to to begin the the backup and begin the
sending of the transaction log okay and if I had you know a 10 terabyte
database or something and I wanted to replicate that you know that's obviously going to take some
time even with a very high bandwidth connection now what's all monitoring would I would I need to
have in place what how would I know that I was caught up and in sync how did I was ever falling
behind yeah so I think that's one of the things that's taken longest to to build in is actually
the decent monitoring facilities but we do have the capability now to manage and monitor the
the replication lag we can measure that as a as an amount of time or as an amount of bytes we don't
actually yet have something that predicts how long it will take to do the the full backup and start
but that is something that I actually wrote the spec for just today in fact so that I never
believe a progress bar anyway well yes I mean oh you know I mean that's the that's the funny thing
I mean obviously as we're sitting here discussing it you know everybody's got the same feeling about
progress bars and not not trusting them and that's why you know we we wanted to you know if we do
this we have to you know do it properly we can't just have something that kind of kind of sort of
works when you squint okay and is there anything else that you wanted to include in sort of ease
of ease of use well you know I think robustness is ease of use you know if something breaks all the
time or or just gives you strange messages then that's very frustrating so so robustness is
and dependability means that you can hit your deadlines you can trust the software
accuracy means that you know you can always trust that we've done the right thing we haven't truncated
your data or sort of set it to null in weird circumstances or something like that and when you
when you go to use a feature you can also trust that it's been implemented in complete form and
and as a result of that you can actually use it you know with a with a good level of trust
security these days is also a big feature but I'd really call that an ease of use feature because
if something doesn't have security then you know then you have to go to some level of difficulty to
isolate it so that it can stop it from being attacked whereas with Postgres you know you've got that
level of trust there that we've you know we've done a lot for you in terms of security so and
and what kind of features would you include in security yeah so I mean we've you know we've
recently updated the authentication technology to 256-bit encryption that's pretty advanced
that's called Scram and it uses an underlying API called sazzle so what we've done there is
we've we've implemented the very latest technology but also implemented it using the API that will
allow us to to keep up to date as much as possible with that technology as it evolves okay and so
that's that's an authenticated access to to the database is that right yeah that's right we we
also implement something called host based authentication which is basically allows you to
whitelist or blacklist different addresses or IP ranges and that kind of thing so it really
allows you to control the level of security that you've you've got in the system and if somebody
for example tries to access the system with a user ID that doesn't exist we don't reply with
things like user ID does not exist because obviously that's you know information to the to the person
trying to to hack the system so we go to a lot of trouble to to make that look like it's a valid
user but you just didn't know the password for example okay yeah that makes complete sense good
and is there anything else that you would particularly like to mention we've covered a
a lot in in this conversation i'm sure there's a lot more yeah well i think the you know there's
a couple of things out there we've got very well thought out support for for time zones
good internationalization support and we also support something called serialization which is
a mode that allows you to trust that the the data in the database makes makes a lot of sense
what's called transaction isolation there are various access problems that people are
simply unaware of that databases go to a lot of trouble to to get right and if you just access
the data store that's got no no thought given to to the transaction isolation modes then it does
make it harder to use because you get strange problems with you know numbers that don't add up and
you know data that appears to have disappeared and phantom reads and that kind of thing there's
some strange anomalies so these these would be when there are concurrent modifications being
made to the database exactly so yeah the basic physics of it because if you've got you know data
is in two places if you look at the the first data location and read what it says and then you come
back and then you go to the second data location well obviously during that pause the first day
to location can have changed and you can get into a sort of strange situation where if you try and
add up the values in two separate locations you can end up with a value that never existed in the
database at the same time for example because you're looking at one and then you're looking at another
yeah and so there's quite a lot of support in the database for levels of care with that so
obviously if we're handling money for example you don't really want to be looking at your bank
accounts and have them not add up for example right so that's a pretty serious situation and
that's if you access data in a pretty simple data store then you don't get that level of coverage
and you don't get that level of protection so we go to a lot of trouble to explain the types of
sort of physics data anomaly that can exist and and what you should do about it and when when we were
talking about replication you could control how the data was replicated at the transaction level
and are there are there controls in place for different transaction isolation levels
yeah i mean so within the database you can specify the the type of transactional
isolation level you wanted at the transaction level so you can specify for the update base
if you want but you can actually request different levels depending on the type of
data manipulation that you're planning obviously making that work across nodes is significantly
harder and that is something that we're working on right now but you know it's obviously an
important thing for the future the only problem is that physics is against us there because there's
various theorems that say that you can't you can't have it both consistently and quickly yes and
yeah i've always thought of pulse graces being the database that would provide you with the
consistent answer and the accurate answer yeah and is that right or is that again something that's
tunable well it's that that is our hope by default that was always the default we would pick but we
recognize there are times when you really do need it to go faster and you might be able to prove
that your transactions are in fact correct for some reason and so you're happy to turn things to a
to you know sort of potentially less safe option for me i think it's important to have that
user control of performance v robustness and consistency i think that's quite an important
lever to say look i need it quick damn it as against look i read i need it as slow as it gets
it's just got to be correct right yeah yeah and and a whole range of options in between and now i
i mean you know managers can be sometimes you know so no i want it you know tomorrow and 100%
correct and you know it's like well you can't have both right so but you know so that's why we
want to give people a whole range of different choices for their performance and robustness and
that's the direction that we've been taking post-cousin is to is to give you the robustness by default
but the option to choose sort of how much robustness whether it's one node robust or whether that's
copied to to multiple nodes before we consider it safe or the other way which is to increase the
performance at the expense of robustness it's interesting the um and that sounds like it's
something that's been actively developed at the moment are there any other features that we should
look forward to coming in the near future well i think there's there's a couple of things that are
really worth mentioning along the way that people don't always understand i think in the early days
of relational databases when you wanted to do something like add a column to a table it would
lock the whole table for like an hour while you added the column and that was one of the reasons
why the no-seal guy's gonna say oh no like don't do that you just want adjacent blob and you can
do anything you want there and you never need to do schema upgrades and that at that time that was
a sensible thought well what we've done in the interim is we spent a long time minimizing the
lock levels that you need on on your ddl commands and as a result there are very few cases now where
we lock the whole table you can add indexes concurrently you can add columns in just a single
metadata operation that takes no time at all so there's a lot of flexibility now built into
to postgres using experience running as a you know 24 7 operational system so it's not the type of
thing where people say you know you'd better take the database down at 5 p.m. and do that now
you know we almost all of our systems are running 24 7 you know even on Christmas day right and
what I would say is that in the rush up to Christmas you'd probably be amazed at exactly what percentage
of the world's transactions actually run through a postgres database as part of the primary payment
cycle so when you're buying something for your loved ones this Christmas just think about how
many of those transactions are running through postgres and it's quite a surprising number
oh we're talking hundreds of billions of dollars annually flow through postgres databases because
the financial services industry trusts it wow okay I think on that note that's that could be a good
place to finish where can where can our listeners go to find out more about postgres and the features
that we've been talking about today well postgresql.org is the website to visit or if you look up
postgres you'll find pages on the open source project and all of the commercial companies around
it but come to conferences that's a great place to discuss your data needs there's users giving
presentations there's people listening to things and people chatting about their what they need out
of all these technologies so it's always good to come and join in the community well Simon thank
you very much for coming on the show and for software engineering radio this is Simon Crossley
thank you for listening thank you very much thanks for listening to se radio an educational
program brought to you by i3blee software magazine for more about the podcast including other episodes
visit our website at se-radio.net to provide feedback you can comment on each episode on the website
or reach us on linkedin facebook twitter or through our slack channel at se radio dot slack.com
you can also email us at team at se-radio.net this and all other episodes of se radio is licensed
under creative commons license 2.5 thanks for listening
