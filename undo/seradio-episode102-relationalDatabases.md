**Link** https://www.se-radio.net/2008/06/episode-102-relational-databases/


Software Engineering Radio Episode 102, Relational Databases.
This is Software Engineering Radio, the podcast for professional developers on the web at sedashradio.net.
SEO Radio brings you relevant and detailed discussions and interviews on software engineering topics every 10 days.
Thanks to our audience and the partners listed on our website for supporting the podcast.
Welcome listeners to another episode of Software Engineering Radio.
My name is Ben Kopp. Today on the microphone with me is Arnold Hase. Hi Arnold.
Hi everybody.
So our topic today are relational databases.
Arnold, why don't you explain and preview what are relational databases?
Sure. Well relational databases are basically big boxes in which you can store any kind of data in a transactional way.
There's another episode on transactions which goes into all the details of what transactional storage means.
But a relational database means you have one big container.
You can put anything you want into the container and you have queries to retrieve the data and to look at it.
They're reliable, they're robust, they're scalable, they're highly available and have all these nice non-functional properties
that are so useful in today's enterprise world and embedded in wherever.
So relational databases basically are places to store data.
Okay. And so why is that topic interesting for me or important for me as a developer or architect?
That is really a good question. I mean databases are so present in most projects nowadays that many people feel they have an understanding of them.
People think, I know what databases do. I know how to write select. I know how to write an insert.
So why should I go deeper into relational databases? I know all about them.
And actually I think this is one of the problems that people tend to think they know about relational databases.
Many applications are designed nowadays with databases as an afterthought.
You design the application logic and you have object relational mapping and then you sort of feel,
okay, I can plug in any database and it's going to work.
And basically it is not going to work really well unless you understand how databases work internally
because you run into all kinds of performance issues, scalability issues.
And so I think it's a good idea for developers and especially architects to be familiar with the persistence layer,
technology and languages and not only the middle tier languages like Java or C sharp or whatever.
I guess we should mention in the overview that schema design and writing database applications is not scope of that episode, right?
Right, yeah, very good point. Schema design is stuff enough for an entire other episode which we might or might not do at a later time.
And also designing application as a whole. It's out of scope. This is really just about databases per se.
That brings to my mind an anecdote I heard about Kent Beck who said you should not start with a relation database per se.
You should not say before you know what the application should do, I need a relation database.
You should consider the alternatives in good extreme programming manner start with the simplest possible thing which usually is file storage
and then build on that if you need something added.
This is sort of a weird approach. Oftentimes you do know that you need a relation database.
But it illustrates that there are alternatives. I mean you can always store data in files or if it's read only data read data from files
which is often a good alternative at least for some kinds of applications. And then there's other kinds of databases like object oriented databases or hierarchical databases.
So, relation databases are not the only solution. It's good to have the whole range of solutions in scope.
But relation databases are the most widely used technologies to store data.
And they're basically all over the place. You have them in huge applications that store many kind, many rows of data,
large amounts, Amazon or eBay or whatever. But you also have them in very small settings.
There are relational databases nowadays that are used in embedded devices.
So you have a whole wide range of different technologies of different products.
The underlying technology of relation databases is the same.
This brings another thing to mind. Most database vendors want to say that their database is the best, the biggest, the whatever.
So they all bring their own kinds of benchmarks.
And MySQL I think has the largest installed base which means the largest number of websites that uses MySQL which is quite probable,
quite likely because they are used by many very small online shops and sites.
Whereas DB2 or Oracle or maybe Microsoft SQL Server, I'm not up to date when it's very hard to check.
Anyway, say they have the largest amount of data or the largest number of rows or the largest amounts of money that are managed.
So you have a wide range of ways to say I'm the biggest database vendor in the world.
And so if you hear some kind of statistics that way, don't take them too seriously.
You cannot check them anyway and every vendor has his own statistics that lets him look good.
Which brings me to a very important thing here.
We're going to mention different products and different vendors.
But we are not paid by any of them and we're not going to say this is a good database, this is a bad database.
So whatever we say about them, there are many different vendors, many different products and we're not going to endorse any of them in a special way.
Before you mentioned embedded databases, so maybe you can explain what is an embedded database?
What I was talking about is a database that is small enough to run an embedded device which basically means it has very small memory footprint and very small disk footprint
and has real-time characteristics but probably will not scale very well which makes it applicable for small devices embedded devices.
And then there's another kind of databases which are known as in-memory databases.
What are their specific characteristics?
Well in-memory databases are sort of exotic in the overall landscape.
They are mainly useful for testing purposes. There are certain databases that run entirely in memory and keep all the data in memory which makes them very fast
and means that the database driver usually is the entire implementation of the database which means you can start a database, put data into it in a test without much installation overhead
which makes them very useful for development but usually not for production use.
Okay, so I guess that's all for the introduction. I think we can now start with the relational paradigm. So what is the relational paradigm?
Okay, relational databases, I mean relational sort of sounds like mathematics and it is actually.
There is a whole mathematical theory behind it that all data is stored in relations and everything is a relation which is sort of nice
has an ascetic feeling to it but it all breaks down to relational databases means everything is a table with rows and columns.
So the language to deal with relation to the databases is SQL structured query language.
There used to be all kinds of products in the 80s with all the different languages and dialects but it was standardized way back.
It's standardized quite well and this paradigm means all data is stored in rows and columns.
It is accessed by retrieving rows and columns.
Several tables can be joined together to make different tables with rows and columns and everything is a row and column.
The strength of this paradigm is that it's very simple. Everyone can understand it and the image of putting data in a table is well known to many people.
It's sort of you can catch it down on whiteboard or a piece of paper and storing data in tables and columns is so widely used nowadays
that it sort of feels like it's the only way to store data but there are other approaches especially hierarchical data which means you have one root element and from that you have the data that is contained in it.
That's basically the XML way of storing data.
XML is the modern technology that stores data in a hierarchical way. You have one root element and children, you have a tree and you have
a whole paradigm, a whole group of databases, hierarchical databases that were on vogue before relational databases came along in the 80s.
They are still used sometimes and you have object-oriented databases that are sort of an arbitrary graph.
You have an object here, an object there, an object over there and they have pointers all over the place which makes them very different but relational databases store things in tables and rows and columns.
Which means that there is no concept of a reference between different tables.
You have a table with persons, you have a table with addresses and they have something to do with each other because every address belongs to a person but there's no pointer per se but the way to match data is to use a join.
In the Person table that identifies the person which is called a primary key, you have a column in the address table which identifies an address which is called the primary key of the address table.
Then you have another column in the Person table that holds a value that is also a primary key in the address table and then you match those and say if the foreign key in one table matches the primary key in the other table,
those two should be matched together.
It's called a join.
There's special SQL syntax for that.
There's inner joins, outer joins.
We're going to go into those in a different episode on SQL.
But the key thing to remember here is you can navigate anyway.
You can start all of the place.
And the query language means the whole contents
of the database are one huge pool of data in a way like everything
is in one big global variable.
And everyone can access it.
There's no starting point.
You can start anywhere and move by joining all over the place
all through the data, which is the special thing
about relational databases.
It's different from all the other paradigms that
are sort of that have explicit links or hierarchies.
In rational database, you can start anywhere
and have any kinds of conditions and move all over the place
in any way you like, which makes it very, very powerful
for queries, for arbitrary queries.
And some of the queries that are, or if you have queries
that you use more frequently, you can store those queries
in views, which is basically a table without explicit data
storage.
It's a select query that you store with a give name.
And since the results of any select query are a table,
views look like a table.
You can access them directly.
And there are optimizations behind those.
They're called materialized views.
Some vendors have them, which means
that they actually cache the results of the query.
And some databases, even allow writing
two views, which is special and useful in some cases.
Again, everything is a table.
And the self-similarity is one of the strengths of SQL.
So these kind of queries that pull together data
from all kinds of all places of the database
and retrieve them and make them available
in the form of a table, this is sort
of the basic need of curating a relation database
and storing data in rows and columns
with using Insert Update.
But there is a whole wide range of syntax
built on top of that, like inner selects.
You can select data and have a temporary table that
is used as part of the query somewhere else.
And you can have report kind of things,
like grouping data, adding things, retrieving
the maximum, the minimum value.
Sorting data.
Sorting data, exactly.
And there's a huge range of syntax in SQL.
You have case statements.
You have if you have string concatenation, really,
really cool, powerful stuff, we're
going to go into that in this SQL episode.
But here, the key thing is everything is a table.
So all this syntax, all these special things,
are built on tables and return tables.
And as you may have noticed, I think
the relational paradigm is a really, really beautiful paradigm.
I love this paradigm because it's so simple and yet so
extremely powerful.
The select clauses, that is, the queries in SQL,
are a very expressive side effect free functional language.
For those of you who love to go into language theory,
they're sort of self-contained, self-similar.
The underlying concept is very, very simple.
And well, the downside is, as with so many simple paradigms,
the performance is intrinsically limited.
You're free to move around all over the place.
But since there is no explicit pointers,
all kinds of lookups must be done by the database engine
at runtime, which tends to make it slower.
And there's the whole issue of indexes
to deal with this, to tell the database
for which kind of queries to prepare a cache that
is maintained whenever data is changed, which
makes the lookups faster.
This is one of the things that is very important
to understand in order to write well-performing and well-designed
databases, which means that which goes way beyond the object
oriented paradigm of the middle tier.
This is something that is in the persistence tier,
and you really need to understand how the database works,
how your product works, what kind of lookups your queries do,
and then you can understand what kind of indexes can
be used to tune this.
And you need to balance this because creating index,
if you have an index, it means any time
you change the data in the tables,
the index must be updated.
So actually, an index can make the database slower.
There are special columns.
Well, first of all, every column has a data type.
Of course, they are built in data types.
Basically, they built in data types.
You have in any language.
And you expect from other languages
that is into the numbers of different precision.
You have fixed point numbers of different precision.
You have strings, and you have date and timestamp.
And that's basically it.
There is no Boolean type, which means
you have to emulate that using a number usually.
And you have special types for storing
huge amounts of data.
They're called blobs binary large objects, or C-lobs,
correct the large objects.
Strings and database are limited in length,
which means there's a maximum length, which
is important for performance optimizations
inside the database.
But you can have arbitrary size objects, blobs, and C-lobs.
It's good to have them, but they have huge negative impact
on the performance.
So you tend to avoid them unless you need them, which
is sort of cruel of most things.
There's special columns.
I mentioned them, primary keys and foreign keys.
Primary keys means you need to have one or more columns
to identify every row in a table.
They must be unique in order to have
this property of being able to provide a way to look up data.
And there are identifiers.
There is a different philosophy there.
One traditional approach was to use business values
as identifiers.
And this, well, for example, for books,
you could use the ISBN number.
Or for employees, the employee number
that is printed on their company ID card or whatever.
So you can have primary keys that are represented
in the real world, that are actually
accessible, that are visible outside the database,
that have a meaning outside the database.
And well, this is, or the social security number
in the United States is a very popular primary key
used for people.
So the thing is that the primary key
has to be unique within one database table, right?
Yes, the primary key must be unique inside every database
table.
And this leads to a problem.
If you use business values as primary keys,
because, for example, social security numbers,
they are unique, yes, but in cases of identity theft,
you may have to assign a different social security
number to the same person.
So one person can have two social security numbers
at different times.
Or ISBN numbers are supposed to be unique,
but there are exotic cases when they are not unique.
Or you might want to recycle company ID cards
so that a person who left, after one employee
left the company, let's say, after two years,
you assign the same employee number
to a different employee, because you
want to reuse the ID card.
So using these kinds of externally visible values
as primary keys is potentially problematic,
because it is very difficult to change primary keys,
to primary key values of rows.
So the preferred approach today, the modern approach,
is, and has been for quite a while,
to use synthetic primary keys, that
is to have an algorithm that calculates a unique value
for every new row that is added.
And this ID is used internally only.
It's not visible outside the database.
So you can assign it.
You can manage it.
You can use it without anyone else caring,
which means there are no external forces that
lead to the problems that I've just sketched.
So are there any special strategies
to generate the primary keys?
Yeah, there are quite a few strategies out there.
There is sequence numbers, which
is a proprietary but widely used feature
of different databases, which means the database provides
a way to access automatically incremented numbers.
There is the special case of an auto increment column, which
means the database increments or assigns a new primary key
to any row that is inserted.
Those are useful at first sight.
They have the problem that the primary key of a newly inserted
row is just inside the database.
And it's not in the application.
So the application needs to access the database
a second time to retrieve the primary key that was just
assigned.
Some databases have special means to do this in one lookup.
But anyway, they are proprietary.
Non-standard extensions of the SQL standard.
So you become very dependent on a specific database
if you use those.
Another way is to have a table in which the application itself
keeps track of primary keys.
And there are quite sophisticated algorithms out there
called Hylo, which work in a cluster, which
means they work when different applications access
the same database.
And one access to this primary key storage table
retrieves a whole bunch of range of numbers
and let kind of afterwards be assigned by the application.
There are tricky issues.
For example, what happens if different kinds of applications
access the database, they all need
to use the same strategy to assign primary key values
to newly inserted rows?
Another issue to keep in mind is if you replicate the database,
you need to make sure that both sides of the replication
have compatible, ideally identical ranges for the primary keys
so that there will not be a mismatch.
So that won't run under the problem that one database assigns
to the primary key value that was already assigned by the other one.
Those are things to keep in mind if you decide on a primary key
strategy.
OK.
So now I have primary keys.
How can I use these primary keys to make sure
that I can reference another element from another table?
You're talking about foreign keys now.
Those are just arbitrary columns that are in other tables
and want to sort of join these two tables together.
Actually, they're not really keys.
They're just normal columns.
Let's get back to this example of persons and addresses.
A typical way to do this would be to add a person ID column
to the address table and store the primary key of a person
in this person ID column of the address table
so that when you take a look at the address,
you can know which person this address belongs to.
You can do this without using the database feature of a foreign key.
And well, there are reasons to actually not use this foreign key feature
because it has a little performance impact.
Mainframe applications tend to switch this off for production.
If you use the foreign key feature of the database,
you have the additional benefit of having the database check consistency.
That is, the database ensures that the value in the foreign key column
is an actually existing value in the primary key column of the referenced table.
Which leads us nicely to constraints.
Oh, yeah, good point.
This is just one example of the constraints that databases support.
The most general kind of constraint is you can have arbitrary Boolean expressions
on the contents of a table as a constraint.
They're called check constraints and they're part of the SQL standard.
It's one of the rarely used features.
Few developers know this or use this.
And the benefit is that you have a self-documenting database schema.
And well, obviously, the database is sort of designed by contract.
The database checks at runtime if what the application gives it is valid.
This is, of course, no replacement for checking in the application as well
because the application should only attempt to insert valid data.
But the database makes sure that only valid data gets into the database.
And it's sort of a catch all, sort of like a second log on the world
to make sure data integrity is maintained.
This is what's behind these constraints.
They are enforced at runtime while the application tries to write into the database.
And if the application tries to write data into the database
that violates this constraint in exception is thrown or anywhere,
the database returns an error and, most importantly, does not insert the data
into the database or update the data.
A special case of this constraint is not null, which is probably
the constraint most developers are familiar with.
Just means that a particular column must not be null.
And there's another concept in databases, which is also not part of the standard
but it's quite widely used and useful.
That is triggers.
And it's different from constraints.
Constraints are a condition that is checked.
And if it's violated, the modification is rejected.
Whereas triggers are code that is executed in the database
and that is triggered by some kind of change.
So for example, you can have a change.
After a row was inserted, you can have some stored procedure.
We'll get to those later.
That is executed whenever data was modified.
You can perform some kinds of constraint checks there,
but they are more powerful.
The downside is they're not standardized.
I guess that's it for the relational paradigm.
Now let's have a look to performance.
What kind of performance aspects do we have in relational databases?
OK, yeah, well, that's a huge topic.
As I mentioned, the database, the relational paradigm means
that you do not have a single starting point
and you do not have predefined paths through the database.
So whenever you look up a row in a single column,
you have some kind of where clause.
So you have some kind of condition
and retrieve only those rows that are met by this condition.
Which means that naively implemented,
the database engine would have to go through every row
in the table and check the condition.
And then doing this collect all those for which
meet the condition.
This is called a full table scan.
And it's obviously really, really bad.
Really expensive.
Yeah, it's sort of the worst case in terms of performance,
because the database must read all the data of the entire table
from the disk and look at it.
So what you can do is create indexes,
which means give the database hints, or well, tell the database,
to create a cache, sort of a mapping
between certain values in the inner condition
and the rows that meet these criteria.
The simplest index is an index on a single column,
typically the primary key.
So that if you say, I want to have the row of the person table
with a person ID 5.
The database will not have to go through the entire table.
But it has a lookup table that starts with the sorted
by the primary key.
And it looks up where on the disk the row with the ID 5
is stored and the new tree is this.
There's a whole science behind much research
going into how to implement these efficiently.
The key idea behind implementations today is trees.
Ideally, databases will want to keep these indexes in memory,
which makes look up very fast.
But for big databases, it's not possible
to keep these indexes in memory.
And the second best thing is to have some kind of binary tree,
some kind of easily indexable structure on disk,
which makes it fast to look up on disk.
You can have indexes that span several columns.
Those are much more specific in their usefulness
than single column indexes because they only
helpful if you have queries that have exactly this combination
of columns in their where clause.
Can you give an example on that?
Well, let's say we want persons with an income
in the range of between $30,000 and $40,000 a year.
And we want to have those with the job title of manager.
Then we would have a select clause in which the income column
and the job title column are part of the where clause.
And the database without an index,
the database would have to scan the entire person table
to retrieve all the employees that meet criteria
in those two columns.
But if we have an index for these two columns,
then in combination, then the database
can use this combined index to retrieve only rows that
match the criteria without going through the entire table.
It's much faster.
These things can become really, really complicated
if you have inner selects and complicated joins.
And the best approach to this kind of performance tuning,
because, well, indexes our performance tuning,
is to write an application with few indexes in the beginning
and then do measurements.
Find out what kind of queries take up most of the time
and then find out why.
Finding out what kind of queries take up most of the time
is just normal profiling.
Databases have tools for that.
You can even do that in the middle here
if you want to, because in the middle,
you know what kind of queries goes out and can measure that.
But finding out what takes up the time is more tricky.
And in order to understand that database products
have tools that provide you with what
is called a query plan.
A query plan is, let me start a different point.
You can send a query to the database.
The database engine parses this query
and tries to optimize it.
Tries to turn it around this way and that
to make best use the best use it can of indexes.
There are many different ways of approaching this.
It's often a good idea to first throw away
most of the data.
If you have two conditions, one of them throws away.
Most of the data and other, that's
in most of the data.
It's good idea to first throw away most of the data
and then look at these small resulting set
and then go through that a second time.
There are all kinds of different tricks
and there's much research going to that.
And after this, this is called the optimizer.
The optimizer step and the output of the optimizer
is the query plan.
The optimizer, the query plan tells the database
to do a full table scan on this table
and use index and do this index on another table.
And so it spells out the details of how the database
accesses data.
And obviously this is very specific.
The tooling and the things that the database does
are very specific to the different products.
But it's definitely worth looking at in the documentation
of the database engine to get an understanding of this
and play around with it a little, because that's
the approach to getting efficient database
applications to tuning the database,
because only if you understand where the performance is
spent, you know which indexes are useful.
Otherwise, you'll end up with lots of indexes
and still most of them won't be used.
And you'll still have some gaps.
So analyzing the performance by looking at the query plans
is the way to go.
I also heard that you can even run into that logs with indexes.
Do you want to say something about that?
Yeah, there's a story behind that in a project I once was.
There were very many indexes on a database.
I'm not going to tell you which product it is,
because it might reflect badly on this,
and it might happen with other products as well.
But the indexes are stored in tables internally
in this database product.
And so, well, when a row was inserted or modified
in a table, all the indexes need to be updated.
And this led to a situation, an exotic situation,
but still there was where different concurrent modifications
to the table caused a deadlock in the updates
to the index tables, which meant the performance went extremely
down the hill, sometimes erratically.
We figured that out looking at the query plans
and the internal logs of the database.
So having too many indexes will degrade performance
even without deadlocks, but deadlocks in the index tables
are sort of the worst case for that.
When optimizing a database, there are two different things
you can optimize for.
Do you want to tell us something about that?
Yeah, good point.
There are basically two different kinds of things
you do with databases, many applications.
One thing is putting data into it
and occasionally retrieving data.
The other is running huge reports on it, only reading, reading,
reading, reading, and putting much load on the database
reading it.
The first thing is called OLTP, Online Transactional Processing.
And this is sort of what you think of,
if you have an interactive application,
like you log into Amazon, then the books are probably stored
in database.
I don't know, but I'd expect them to be.
And so when you do a query, some books, the books are restored
and the details are retrieved from the database
and your cart is probably stored in the database somewhere.
If you order something stored in the database,
it's sort of single row insert and very low scale retrieval.
But oftentimes in the back office,
you have very, very different kinds of things.
They will run huge reports on how many customers board
what kinds of books today to optimize their advertising
or to create a prognosis for the revenue,
that kind of thing, which does huge queries over millions
of rows in the database.
And those two require very different tuning in the database
because they want very different optimizations.
Online Transactional Processing will need few indexes.
Most of the time data is stored.
And storing the data quickly is very important.
But quick response times, deterministic quick response
times are very important.
Whereas in the spec office part,
the huge queries that's called OLAP,
online analytical processing related to data warehousing,
there you have huge queries for huge reports
that might run several minutes and will basically
bring the database server into saturation during that time,
which means this one query takes up 100%
of the resource of the database server while it is running.
You don't want to have that kind of impact
on while people are working with Amazon, want to buy books,
and then cannot buy the books because one of those reports
is running.
And you also have very different kinds of indexes
that you want there, very specialized
for those kinds of reports.
So typically you end up with two replicated databases.
One is optimized for read performance,
and another one is optimized for write performance
if you need that, right?
Yes, exactly.
That's one typical architectural approach
to have two databases, one for OLTP, one for OLAP,
and then have database replication
from the OLTP into the OLAP or the data warehouse,
typically during nighttime or low load hours,
if you have those.
So another huge topic, I guess, is security.
So what can I do in my database to ensure certain security
things?
Oh, security is, as always, a big topic.
First of all, well, the different parts of security.
One is making difficult for people
who work for your own company to spy out data.
The database administrators, people
working with the data center, because typically data
is exchanged between the application
and the database in a non-encrypted format, which
is usually not an issue.
If it's inside the data center, you
have secure network lines.
It is, however, an issue if you have,
or potentially an issue, if you have data running,
if you access the database from outside the data center,
or even over the internet, that's usually not a good idea
because the data is unencrypted and passwords
may go clear text.
For that, it is a better idea to have three-tier architecture
with an application server running
in the data center and having a secure network between the
application server and the database.
This is sort of the raw wire espionage thing, which is good
to keep in mind, but is not such a big issue in practice
most of the time.
At another level, it's important to fine-tune or to manage
who may access which data in the database, or modify what
data in the database.
You want to provide different users with different permissions
on the database.
And for that, databases come with user management of their own.
And it's quite sophisticated, usually.
The database service support users.
They have user groups, even.
And you can assign them rights, saying this user,
everyone in this group, may insert data into the person
table, or may read data from the person table.
But or you can even be more fine-grained by saying, can read
all columns in the person table except for the income column,
which is confidential.
Or some databases even have data-driven authorization by
specifying a user.
A database user can read data from the employee table
for all those rows with an income of less than $30,000
a year, or for employees whose job title is not manager.
This is quite fine-grained.
And it's good as far as it goes.
But it's not widely used nowadays for many applications.
I'll go into that right away.
There's a different model now.
And that is the model of having, well, no.
This first approach is basically having one database
user per human user of the system.
So you have administration of the users and the permissions
in the database.
And if I log into the application with my name,
Arno, and a password, these are passed through to the
database and the application logs into the database using
the credentials I provided at login.
This has the benefit that it permits traceability and
logging in the database.
And this is sort of the traditional approach for which
databases are optimized.
Another approach is to have an application server in
between and provide a technical user, or have the
database provide a technical user, a generic user, to the
application server.
And all access goes through this one generic user.
And all the permission checking is done in the
application server, which means that as long as the
application server is not compromised, there's no problem.
And the entire responsibility lies with the
application server.
The downside of this is that you have no traceability in
the database who accessed what.
All this logging is not useful anymore.
The upside is that you can have things like connection
pools in the application server, which means that
well, creating a connection to the database is an
important thing, an expensive thing, and a connection
costs lots of resources, and it takes some time to create.
So application servers create a pool of database
connections in advance and pool that whenever someone
uses, needs one, it's returned from this pool.
And when its transactions finish, basically it goes back
to the pool, which means you need fewer transactions.
You need fewer connections at the same time.
Another benefit is that you have better integrations with
single sign-on systems, and you can have, well, your
applications nowadays need to know about permissions
anyway, so it's not sufficient to have the permissions in
the database you need to have in the applications
have anyway to de-gray out menu items or whatever in the
user interface.
Not a huge problem in these kinds of systems might be
SQL injection.
Do you want to tell us something about that?
SQL injection is based on the way many applications are
implemented today by using string concatenation to create
their SQL on the fly.
One example, well, let's say someone logs in, and you
have a select statement to check authorization by things
like select asterisk from user, where user ID or by user
name equals quote, the name that was entered in the log
in dialogue quote, and password equals quote, then the
password that was entered in the log in dialogue, and then
you add another quote and send it to the database.
And if you find a row, then you know that the user is
permitted to access the system, if you don't find a row, you
know, the log in was invalid.
Now, if you enter quote or one equals one or quote, quote,
quote equals quote into the password field, then this
leads to another valid SQL statement, but this
SQL statement basically accesses any account because the
way I close, you have one equals one.
So SQL injection is based on entering SQL code into the
user interface, and if the application is programmed
insecurely by using string concatenation in an unsafe
way to create the way I close than the application can be
compromised.
There are ways around this many libraries nowadays are
intrinsically secure against that prepared statements and
Java, for example, are safe against this, but still it's
good to keep in mind.
That's all for security now.
So now let's maybe have a look for the different database
tools.
OK, well, I'll be quite brief here.
There is a wide range of database products out there, and
well, you can look at the data sheets of the vendors if you
want to compare them.
And anyway, in many companies, there is one global decision
for one database tool anyway.
The SQL standard is what is common to all of them.
They all implement the SQL standard to different degree,
but quite widely by now.
They have their specific enhancements of the SQL
standard that edge on certain special syntax.
It's used to be important.
It's not that important anymore to have these syntactic
add-ons.
But what is really, really important and is usually the
key thing to making the decision is the
administration tools around the database.
For example, user administration, table space
administration, or performance tuning support,
replication support, but also support for doing hot
backups, that is making a backup of the database
while it is still in use without compromising the
consistency of the backup.
Running all kinds of diagnostics, that kind of thing.
It's very important in enterprises, and those things
are really what differentiates commercial products from
open source products much of the time.
And this is where the commercial products have
different specialties.
So that's one important thing to look at.
Another thing is scalability.
How fast the database out of the box, which is interesting,
where it is interesting to notice that, for example,
MySQL is very fast compared to much of the bigger databases,
especially if you have read access.
So this is something to really take a look at.
But then another thing is to look at how well this
performance scales up if you have a database of a
terabyte.
Very different issue from fast performance for
small databases.
Support for clustering, having several databases that act
as a single database is another thing that goes into
this performance and cluster end and scalability thing.
And then all these operational things, like replication,
having high availability support, and service level
agreements that support these.
But then you also have special features that may or may not
be useful for your application.
Many databases now have some kind of XML query support that
is they can retrieve, they can return data as XML.
They have custom types, custom objects, they have stored
procedures.
Ah, stored procedures, yeah.
There's a lot of debate going on around stored procedures.
Some people say, some purists say they're not stuck out of
the stand that you shouldn't use them.
And anyway, if you're a Java programmer or a C-sharp
programmer, you're not very much in touch with them much
at the time.
You can do anything in Java and so C-sharp.
So what are stored procedures?
Good point.
Stored procedures are code, are procedures that are stored
inside the database in some kind of proprietary language,
PLSQL for Oracle, for example.
And this code is executed inside the database.
And stored procedures basically move program logic
into the database, which is bad for things like refactoring,
for maintainability, that kind of thing.
But it's very good for performance.
This is basically the tradeoff behind stored procedures.
And if you need them, you'll know that.
And I'm basically calling a stored procedure from an SQL
query.
Yes.
Well, the syntax is not standardized.
Different databases have different dialects, different
syntax for calling the stored procedures.
But yes, basically you call it from SQL or via an API.
Those are the two approaches that are there.
So now we heard how databases are implemented in general.
How would I make use of them in my actual system?
Yeah, well, this sort of drives the architectural issues.
It is a wide area.
I'll just start by talking about the things
that come to my mind.
First of all, there is object relational mappers.
They are basically libraries or frameworks
that make it easier to get objects and all the objects
specific way of thinking like references, collections, maps,
inheritance into a relational database that
has none of these things and retrieve data from a relational
database into these kind of object things.
We're going to have a dedicated episode on object relational
mapping.
So I'll not go into those very much here.
But if you start thinking about the system architecture
with the database, the first thing to keep in mind
really is the different roles of people
who deal with the database.
There are at least four roles, four different kinds
of people who deal with the database in a development effort.
The first role is the users of the database,
that is the programmers.
Those are usually the first that come to our mind.
The people who write the queries, who write the modification
code, who write the update, the insert statements,
call the stored procedures, who write the stored procedures,
all that kind of thing.
This is basically using the database.
It's very important, obviously, but it's not the only role.
At the other end of the spectrum is the database administrator.
He's the guy who takes care of how many disks in what kind
of rate arrays are available, how fast data in the database
grows, and well, he's the guy who orders new disk arrays
if the old disk arrays tend to become full.
He's the guy who monitors performance, all that kind of thing.
Whole different approach, whole different view
onto the database.
And it's a good idea to talk to the system,
to the database administrators, to the system
administrators, next time you develop a database system.
It'll open your eyes, and you'll get a whole different view.
And it's important to, well, it's a good idea
to have the database administrator's perspective in mind
if you write a system that uses the database.
And sort of in between, there's the database design.
That is, the people who decide what tables, what columns
there are, what they are called, what relations
there are between them.
That's the logical database design.
And there's the physical database design,
which is, again, distinct from that, which
is how are the tables spread across different table spaces.
For example, if you have a huge application in which data
runs, which all the time, many, many rows a day,
let's say, a couple of million rows every day,
and the most frequently used data
is the data of the current day.
And it may be a good idea to split the table physically,
which can be transparent to the application,
but to split the table spaces based on the time stamp value
and have the data that is of the current day in one table
and older data in another table, that kind of thing.
That's physical database design.
So those are the four roles of people who do both database.
Wouldn't I use views in that case, materialized views?
You might want to use a materialized view.
Depends on what kind of queries you have.
Splicing data on different table spaces
that are on different hard disks can be a significant speed up
in many situations.
A lot better than having a materialized view that
is in the same table space on the same disk.
Again, depending on the situation, one or the other,
maybe the better approach.
I was just sort of trying to illustrate
the range of things you can do at the physical level.
If you have huge amounts of data and want to optimize for that.
Well, you mentioned table space now.
What is the table space?
Table space is, well, it's the disk space
on which a table is stored, basically.
It's specific for different vendors,
how they manage this.
Basically, every table or, well, every row,
at least, is on one table space.
Typically, every table is in one table space.
And managing table spaces, predicting
the size of the table spaces that is needed,
and maybe even optimizing so they can be put on different disks.
So the performance of the disks is added and increases
overall performance.
That's the kind of thing you can do by dealing
with table spaces.
The new approach that most systems take, at least,
in the beginning is to have the entire schema
on one table space, and that's that.
So most systems do not really actively
do any physical database design to optimize performance.
Another thing that is related to these two
using different table spaces is distributing
the database itself using clusters.
Again, there are trade-offs there.
First of all, price, of course, but leaving that aside.
If you have a cluster, the key idea
is that the performance should go up.
But if you have a cluster, that means
that you have distributed locks because the database must still
maintain integrity across the entire cluster.
Depending on the specialties of the product,
it's important to look at the trade-offs that are involved.
But planning to cluster the database
is one way to improve performance.
It's definitely an architectural issue.
It's something to look at maybe even early on,
because it may require some effort, some rework,
and the database design.
Again, as with any optimization, you
should do that only after measuring the performance.
And obviously, it should be part of the decision-making process
as some of the databases do support clustering
and other stone.
Oh, yes, of course.
Of course.
Yeah.
I was assuming, for an enterprise application,
you were using an enterprise-scale database system anyway,
and they do support clustering.
But yes, of course, this is an important part of the decision-making.
Yes.
Another thing I mentioned that already
is the replication thing OLAP versus OLTP,
or more generally, for different kinds of users for the data,
it may be a good idea to isolate the performance
hits by replicating, especially if there
is little load during the night.
You always have that kind of thing available.
Getting closer to the code you write,
there's the decision between having a three-tier
and a two-tier application.
Three-tier is what's usually done.
So it's the modern way of doing things.
It's the way we think do things here, which
means you have a server-side application server that
runs in the data center.
And all user management authorization management
is in the application server.
And only the application server talks to the database
sort of more secure, because you can put the application
server in a demilitarized zone, have
a firewall between the applications of the database, which
makes it harder to hack the database server.
And the two-tier approach, on the other hand,
means that you have logic in the client and autologic
in stored procedures stored in the database,
and the client application talks directly to the database,
which means you have a rich client, not a web client, which
is, well, sort of against the trend.
Still, there are very important applications, or class
of applications, that use two-tier architectures.
For example, call center applications,
where fractions of a second, in response time,
are very important.
You have few users.
You have a controlled environment, that kind of thing.
Two-tier architecture can be a very good idea.
Then going even closer to, or well, even more towards,
the code, it's a good idea to know the details of the excess
library you use to access the database.
You have some kind of result set you iterate over,
but databases nowadays support scrolling in both directions,
usually using things called curses.
And you can write data to the curves that are directly,
that kind of thing is good to know what you can do.
And one final thing about the architecture
is one of my favorites, internationalization.
Why is it a topic in the database design?
It's important because using different encodings
has an impact on the number of bytes
that every character needs to be presented.
As long as you use one of the ISO character sets, ISO 8859,
one, or whatever, you're on the safe side.
You have one character, or one byte per character,
so it's sort of intuitive.
But obviously, there are limitations
with regards to internationalization, you cannot have
kerilik or even Chinese characters in there.
So if you want to go to UTF, well, if you use UTF-16,
you have two bytes per character.
And the sizing of the database columns
is typically in bytes, not in characters, very tricky.
From the database perspective, it's
sort of obvious because the database needs
to reserve some amount of memory that's in bytes.
But if you use UTF-16, you need to make all your YHR columns
twice the size.
Or if you use UTF-8, which is a variable length encoding,
for a seven-bit ASCII, you have one byte per character.
But if you have OOMLouts, you have two bytes per character,
and for more exotic characters, you
have up to six bytes per character.
So if you're planning for internationalization
application, it's important to think upfront what kinds
of characters you want to support, how do you plan to size
your database columns, and then you can,
you need to size them accordingly.
And then only after you did that, you should actually set up
the database to use the encoding you chose.
It's very tricky.
And oftentimes, it's not noticed until late in the cycle
because it's non-obvious.
Even during testing, you would need
to test the database with acrylic strings of big size,
which may or may not be a test case depending
on the technical expertise of the people
writing the test cases.
OK.
So I guess we're done, right?
Yes.
I think we basically covered most things.
There is, however, one story I would
like to share with our listeners, which
is my favorite database driver bug.
And again, I'm not going to tell you
what kind of product it was because these kinds of bugs
are present for many different products.
These products are none worse than all of the others.
But it's one of the really big players in the game.
So there was this application that
was writing on a huge time pressure.
And it started behaving weird.
So I spent two days, I think it was,
debugging the database driver until I found
that if you used Java prepared statements using
a batch update and had a database table that
had two big decimals and one Varchar.
No, it was two Varchar columns and one big decimal column,
I think.
And if you did in the batch update,
you did two updates using specific strings.
Then for the third update, the upper limit
of the big decimal column went down to less than 100,
regardless of the constraints you actually
had stored in the database.
And all these conditions actually
needed to be present for the bug to turn up.
It was really weird.
Well, after I found it, it was easy to work around it.
The reason I'm trying to, I'm telling you this is,
don't take anything for granted.
Databases are there.
Databases are very, very reliable.
They're excellent products.
And it's good to rely on them.
But if there is a problem, think out of the box,
the database may be part of the problem.
Even the database driver may be part of the problem.
And this kind of mindset has helped me in lots of projects.
OK.
So do we want to do a short wrap up?
Let me see.
Databases are cool.
Databases are here.
They're here to stay.
Databases are very, very useful.
And despite what I said, they are often the best
trade-off for a way to store data.
Why are they so predominant?
I think one of the key things is there
is a very, very well understood and well-standardized
standard for the SQL language.
They're sort of interoperable.
Well, no, not the database.
But if you know SQL, you can deal with any of the databases.
They're more or less interchangeable
from the developer's perspective.
They're very mature technology.
You have special solutions for many special problems
like stored procedures, like replication.
You have very good administration tools.
Obviously, they are very widely used,
which makes it sort of self-perpetuating
because they are widely used.
That means they are further developed.
They are well-known.
The money goes there, et cetera.
That's why they sort of keep being used.
And, well, to be honest, they are extremely well optimized
by now.
So basically, they are pretty fast,
faster than anything you can write anyway,
at least in one of the typical development projects.
Okay.
So, thanks, Arnold.
Thanks, Brent.
It's a pleasure talking to you about databases.
Bye-bye.
Thanks for listening to Software Engineering Radio.
If you want more information about the podcast
and all the other episodes,
visit our website at sedashradio.net.
If you want to support us,
you can donate to the SEO Radio team via the website
or you can advertise for SEO Radio,
for example, by clicking on the dig-raded delicious
and slash-dot buttons.
To contact the team, please send email to team
at seradio.net, or if it's specific to an episode,
please use the comments facility on the website
so other people can read and react to your comments.
This episode of Software Engineering Radio,
as well as all other episodes from license
on their Creative Commons license.
Please see the website for details.
Thanks to Charlie Krau and the Pot-Sive Music Network
for the music used in this show,
the song is called Vega's Heart Rock Shampoo.
