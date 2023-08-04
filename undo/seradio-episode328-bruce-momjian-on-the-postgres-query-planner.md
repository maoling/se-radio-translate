**Link** https://www.se-radio.net/2018/06/se-radio-episode-328-bruce-momjian-on-the-postgres-query-planner/


This is Software Engineering Radio, the podcast for professional developers on the web at se-radio.net
SE Radio is brought to you by the IEEE Computer Society, the IEEE Software magazine
online at computer.org slash software
Digital Ocean is the easiest cloud platform to deploy, manage and scale applications of any size
removing infrastructure friction and providing predictability so developers and their teams can
deploy faster and focus on building software that customers love.
Digital Ocean stands out from the crowd due to its simplicity, high performance and has no billing surprises.
Join over 150,000 businesses on Digital Ocean.
Sign up with a $300 infrastructure credit at dio.co slash se-radio.
For Software Engineering Radio, this is Robert Blumen.
Today I am joined by Bruce Momgen.
Bruce is a senior database architect for Postgres.
Prior to that, he has held a number of positions relating to Postgres development and advocacy
and was an adjunct professor at Drexel University.
He lectures and presents widely on a range of Postgres topics.
You can find many of his slides at momgen.us.
Today, Bruce and I will be talking about database query planning and optimization.
Bruce, welcome to Software Engineering Radio.
Oh, great to be here, thanks.
Is there anything else you'd like our listeners to know about you that I didn't cover?
Well, I'm kind of a long timer.
I've been with Postgres for 22 years and I've been with my current employee,
my current employee, under price DB for 12 years.
I was one of the first people to help start Postgres in internet development.
Obviously, we've been through a whole lot up to this point.
We're sort of in a very golden age for Postgres and it's becoming very popular.
I stay very busy and it's an exciting time to be involved with the project.
It's great that you'll be bringing that wealth of experience in Postgres to our discussion.
In order for us to have a conversation about query planning,
I want to cover some foundational concepts and definitions starting with what is a database query?
A database query is a query that follows more or less the SQL standard as a way of requesting data from the database in a structured way.
What does the client send and what do they get back?
Well, the client is typically sending usually ASCII text, which is a mix of keywords and identifiers
and character strings and numbers.
And as a return, and it's obviously formatted based on the SQL,
it's supported by the database and what they get back is a structured result set of rows and columns.
What are the steps at a high level of the database performs in order to return the results?
So the database is first going to scan the SQL text for identifiers and keywords.
That's called Lexin. Then it's going to parse it.
Once it's parsed, it's going to be loaded into an internal structure.
The structure is going to go through something called the rewriter, which handles things like views.
Then it goes to the optimizer, which is sort of the brains of the system, which determines how to efficiently execute that query.
And finally, it goes to the executor where the actual query is executed and the result returned.
We're going to be talking mostly about the step involved in the optimizer, but do you say a little bit about the output of the parse or what type of internal structure does it create?
There are actually two different types of SQL commands, categories that postgres handles.
One is sort of a full-blown query, which has a lot of complexity to it.
It's what you're probably familiar with if you think of the key where a select query or an insert a query, an update query or a delete query.
Those are full-blown queries.
Those get loaded into something called a query structure, aptly named, and those get passed to the rewriter or the optimizer and the executor.
The second type of output of the parser is for utility commands, things like drop table, grant, revoke, create index.
Those are utility commands specific.
Those, again, are internal C structures.
Those go to a separate section of the system.
It doesn't really have an optimizer executed to it.
Those utility commands are handled in a specific way depending on the type of command it is.
Let's focus on the full-blown queries.
What are some of the things that make queries complex to evaluate?
There's a bunch of things.
First off, as I said before, the full-blown queries can be selects, but they can also be inserts or updates or deletes.
They all fall into the same category.
A delete is a select and a delete, an update is a select and an update.
They all work together.
On those type of full-blown queries typically have the ability to reference multiple tables, to join multiple tables together, to have complex where clauses that restrict what rows are actually processed.
There's group viability, which allows you to use aggregates with these.
There's a having clause.
There's sorting.
There's a limit clause for a bunch of them.
This allows you to tack on a whole bunch of modifiers to those commands to really very precisely explain exactly what type of query and what type of output you want to perform.
A single query could have one or more of these factors such as a where clause joins group by and the more of those you have, the more work there is for the backend to deliver the correct results.
There's no question about that.
We're always looking at how do you receive the command from the user and how do you return that request in the most efficient way possible.
But you're right.
The more tables you join to, the more aggregates you have, the more sorting you're doing, the more work the relational database has to do.
You mentioned the term join.
Go into more detail about what is a join.
Well, joins are probably one of the defining aspects of relational systems.
It really goes back to the foundations of relational algebra that academically represent a foundation for relational systems.
What a join is, it's really the ability to take a table or a relation depending on what term you want to use, which is a set of rows and columns.
Join it to another relation or another table that's defined in the same database.
You have the ability to not only have data in one table, but you can bring data from other tables together with that table either as a way of merging columns from the two tables together, of restricting certain rows that happen to be joined to other tables.
It's sort of the atomization of your problem space.
Instead of dumping all of your data into one big table, the relational system is really designed to break it up into pieces.
That joining process is really necessary to bring the big data back together to give the type of results and actions that the user requested.
For example, you're dealing with a retail application, you've divided your tables up to have users shipping addresses, shipping methods, but in order to bring all those back in one query, you'd have to join the user table to the address table and maybe the shipping method table all in one query.
Would that be an example of a join?
Yes, absolutely. The novice way, the non-relational way to do this would be to have one table for every aspect of the system.
You'd have one row which represents an order and you're repeating the customer name.
If the customer orders multiple items, you're repeating the customer on every row, you're repeating their address on every row.
You're repeating all sorts of things if you only had your data in one table.
That type of duplication, that type of redundancy makes reporting very difficult, management very difficult, data cleanliness very difficult, referential integrity kind of keeping a structure on a database.
It becomes very difficult.
By breaking things up as you describe, you're basically atomizing the aspects of that retail environment into entities like customer order, item, shipping address.
By atomizing those things, the system becomes much more efficient.
There's less storage required. Your data is easier to manage. Your data is easier to update because you only have the customer name in one place.
You only have their shipping address in one place and so forth.
That's the way most relational systems work and that's why the joins are so important because even though you've atomized everything, you still need to bring them together to do useful work.
A relational model in order to achieve this goal, which sounds a lot like what software engineers called don't repeat yourself, you've carefully isolated the data into these unique tables.
That creates the need to merge everything back together in order to answer many types of useful questions that cannot be answered by a single table.
The fundamental beauty, I think, of relational systems is that once things have been atomized, you can do all sorts of very interesting representations of that data.
You're not stuck with one table and one representation that you happen to have set up when you originally wrote the application.
You have the five closest shipping addresses and also give me the customer name. By atomizing the data and then having joins to bring them together, you have basically a Lego set that allows you to bring together your data in all sorts of interesting and creative ways.
I think one of the great strengths of relational systems and the reason they've been around for so long is because they have that flexibility to effectively bring together data in all sorts of computation, contortions, whatever the business need is.
When you write your application, you may have only one need for that data, but as we may know from practical experience, once the data is in the system, everyone wants that data and they want to do all sorts of interesting ways of using that data and relational systems
are just very good at that. And the joins make that type of flexibility almost seamless to the application programmers. The database is working very hard to do all sorts of very complex joins and representations of that basic data, but the user merely has to type what they want.
And the database goes and does it.
Bruce, we covered declarative programming in episode 289. This sounds like what you're talking about. The user says what they want.
And the system figures out how to do it. Is sequel a good example of a declarative programming technology?
Yeah, I would say sequel is probably the most popular declarative technology I can think of. I mean, obviously there's a lot of imperative languages, Java and C and Python and so forth, where you tell the system what to do.
But I would say SQL is one of the few declarative cases where you're actually telling the system what you want and the system figures out how to get it for you in the most efficient way. And I think that's kind of kind of a neat aspect of SQL.
Why is declarative a good solution to the problem posed by relational databases? So if you did not have the declarative ability in SQL and I do go back far enough to have used pre SQL databases.
The problem is that every time you want to look at the data in a different way, you have to write a new imperative program to do that. So if I want to look at join the data in a way that I had not done before, I'm rolling up my sleeves and I'm writing a program.
And if I want to do it a different way, I'm going to write another program. So the problem is that you don't have sort of the efficiency of being able to just write what I want and let the system figure it out.
And for some reason when you write an application, normally people aren't fiddling with the application 20 different ways.
Of course it goes through iterations but normally application stays pretty static from day to day. With SQL, there are applications that are pretty much static from day to day.
But because of business needs, you're often doing ad hoc queries all the time, getting analytics from the database, doing new ways of sort of slicing up the data and showing what's happening in your organization.
And that just really lends itself to a clarity of language where you don't really have to write a program every time you want the data in a different way.
You just have to declare what you want in a different way. And the database engine basically gets abstracts a lot of those problems away from the application programmer and handles it at the data layer, which I think is a better place to do that kind of, particularly
because you're dealing with a lot of data, that's the better place to be doing that kind of optimization.
I'm trying to imagine what this program would look like if I wrote an imperative program.
I have written programs that involve, let's say, open up this set of files, read a bunch of records, plot a field.
Now go query this API for each item corresponding to one field. And now open this CSV file.
And so is that the kind of imperative program that I would be writing if I didn't have a declarative query processing language?
Yes, absolutely. And I've written those kind of applications.
They're not really fun. They're kind of hard to do because you're dragging a lot of the data to the application.
And the joins are kind of awkward. You typically end up falling back as you're saying to some type of nested loop join, which is normally,
give me the row, go to the other table or the other file and give me all the matches and then get the next row and do the same thing.
But there's problems with that. First, you're dragging all the data to the client.
Secondly, there are other join methods that are supported in relational systems that typically aren't written in applications.
You don't see people writing merge joins or hash joins very often in applications because it's kind of hard.
And people don't really think of it. So they end up brute forcing it.
So you end up with very simplistic ways of joining the data if you don't have a relational system to do that.
But the third problem, which you might not realize is that the relational system knows how much data is in each table.
It knows how restrictive these conditions are. So it is constantly adjusting itself based on what data is currently in the system to give you the best answer.
Even if you wrote an application today and you did this type of join and let's suppose you actually were really smart and you wrote a hash join or you wrote a merge join.
Well, the problem is that that's going to do whatever your program specified for a joint type forever.
Whether the data changes or doesn't change, your application doesn't know that. It's just going to keep doing the same thing.
One of the cool things about relational systems is that they will actually adjust live based on the data that they have now.
Not necessarily the data that they had when the query was written maybe a month ago.
And that adaptability is something you almost get impossible almost to do in an application.
I think we've now established what is the problem to be solved and why you would like to have a declarative model where you hand it off to a back end and the back end will figure it out for you.
So we're kind of talking about this, but tell me what is the query planning? What is a query plan?
So as I said before that query plan is that third stage of the four stages of the relational system.
The first one is parsing, second one is rewriting, third one is the optimizer, which we also call the planner.
And that outputs a query plan and the query plan is what's fed into the executor.
So a query plan is effectively what the database software considers to be the optimal way or the optimal set of instructions to get you the data that you requested.
Okay, so a query plan consists of some set of more primitive operations that will achieve the result requested by the user.
So if I looked at a query plan, what would I see?
You would see basically elemental building blocks of instructions that the executor needs to process in order to complete that query.
So you would see a mixture perhaps of a sequential scan, an index scan, a sort, a hash join, an aggregate operation, a limit.
There's all sorts of materialized view.
I mean, there's all sorts of actions that probably a dozen or two dozen different types of actions that you would see.
And it would be set up in a tree environment where you would say do this first and then once that's done, feed that result into a join or into an aggregate or into a sort.
And then after that's done, do something else until the query was finished.
Let's go back to this retail example, suppose a query is I want to pull from a user's table and address tables, the users and the user's primary shipping address and get this all sorted by zip code.
What would a possible query plan be to execute that query?
So what that would normally be would be probably an index look up if you supplied the customer name, for example, we would do an index look up on the customer table.
We would get their identifier for that customer, the sequence number, whatever, however we uniquely identify the customer, then we would take that number and we would join it to another table.
For example, the orders table, we would see all of the open orders for that customer and then we might also take that order ID and go to the address table and find the address for that customer.
And then also we would look at the zip code and we may limit on zip code.
So for example, if we want to see all the customers that are zip code, we might do a restriction, which would say give me all the addresses in the zip code and then go see the customers that match that.
So that's an example of how you might go one way, you might go from the customer to the address table or the customer to the order table, you may go from the address table to the customer table, depending on the type of query, depending on how much data is in each table, depending on the restrictions in the where clause,
you're going to do different orders of joins, depending on the query that's supplied.
I can see if I knew the customer would make sense to start from the customer, but as you described there, if you had three tables, we know there are the combinatorial number of possibilities.
You could do starting from the customer and then orders and then addresses or any of the other combinations.
How does the query planner, what is it looking at to decide which combination of ordering is the best?
Yes, that's a great question because you normally see the plan coming out, but you don't really understand how did it get there, how did it choose the instructions that it chose.
And that's always been fascinating to me to figure out exactly why does the system do what it does.
And when you're obviously working on a proprietary database, you can't see the source code, but post it because you can, so you can actually see how it's choosing these things.
But effectively, when you're looking at the different options, every operation in the system, whether it's a limit or a sort or a join or an index scan or a sequential scan, has some type of cost associated with it.
And what the system will do is it will do a cursory look at all the tables mentioned in the query and it will look at the restrictions, the where clause restrictions that are mentioned.
And it will try and it'll actually cost out which way will probably be cheapest.
And then if it determines that one way is cheap, it'll go that way. If it thinks two ways, maybe also equally cheap, it may actually carry both options through the optimizer until at the end, it eliminates one of the two options.
So you're right, it is a factorial kind of problem, but there is a pruning process that's happening as you're running through the query.
And that pruning process prevents the system from just spending huge amounts of time considering different plans.
Okay, you're talking about costing these options out. And I do want to come back to that. First, I want to get more detail about the term scan which you've introduced.
So there's probably two or three fundamental ways of accessing data. One is to sequentially scan the entire table. We call that a sequential scan and it sounds exactly what it is.
It basically will sequentially read from the first block to the last, the entire table. And there are some queries that effectively need a sequential scan.
If you're accessing a large portion of the table to satisfy some query odds are you're going to need to read all the rows, read all the pages.
And so there are queries that will use sequential scans. There are other queries where the query is much more focused.
It's looking only for a specific customer or a specific order number. And in those cases, we're much more likely to use an index scan because we believe that the index will allow us to get at the rows we're interested in.
Much more efficiently than reading every row in the table.
An index, then it's a data structure that allows you to access, given a row more efficiently.
Right, it's basically sort of an index. Well, I guess yes, an index on top of the table that gives me rapid access to specific rows that match specific criteria.
So I might have an index on the table which gets me specific customer numbers very quickly.
Or I might have an index that gives me the, you know, customers in specific states very quickly or whatever, specific order numbers very quickly.
So you have the ability to sort of add these quick access methods on top of tables.
And you can identify, specify which columns or which pieces of data in that table you want to, you know, you want to rapidly access.
I can see, for example, if I am looking for customers and addresses and I knew the customer number or the customer name and had the ability to access that through an index,
I could very quickly get that one row and then proceed to the address table, which might have one or two addresses which match that versus if I started with addresses,
I would have to look at every single address and then try to find which ones match the customer.
So that seems pretty obvious. I would start from customer in that case.
Is that kind of what the costing process looks like?
Yeah, that's exactly what the costing process looks like. A system is going to cost out how, you know, how much, how expensive is it going to be to read every address, right?
And then it's going to also cost out how expensive is it going to be to read every customer, which is most sequential scans.
But it's also going to cost out using the index.
And because you supplied the customer name and there is an index matching that customer name, that index access to the customer table is way less expensive than the sequential scan of the customer table or the sequential scan of the of the address table.
So the system very quickly discards the idea of doing sequential scans on either of those tables, at least to get the customer and it uses the index pretty quickly.
And then it hopefully has, once it gets the customer number, there's hopefully another index on the address table, which will allow us to pull the addresses for that customer ID very quickly.
If there's no index on the customer table, then the optimizer is going to default to doing a sequential scan, which is of the address table, which effectively means it's going to read every single address looking for the matching customer ID in that table.
And we're talking about cost, then is it costing out how long it thinks it will take to read data off of some kind of storage device, or how does this cost translate into something that the user cares about?
Yeah, so the costing number is really only meaningful to the extent that it's cheaper than another costing number, so you can't really take the costing number and extrapolate exactly how many milliseconds it's going to take to execute the query.
It doesn't really work that way because you're really mixing different costs.
I mean, there's a cost for I.O. access, there's a cost for CPU usage, there's a cost for function calls, so you have different costs that kind of all kind of mash together into a single scale.
So the number that you see for the cost is really not going to be translated into any kind of meaningful, externally visible effect, except to know that an item with a cheaper cost is probably most likely going to run much faster than one with a higher cost.
But the numbers themselves don't really have a physical meaning, but the number actually, a one, for example, means I think it's the cost of a row access, but again, it's not that's not really what you're going to care about when you're trying to cost these things.
You're really going to be looking at which cost is cheaper, which cost is more expensive and you're going to do everything based on that.
I think then what reason do you have to think that that cost will translate into runtime or something that the user perceives as better?
When you're looking in the optimizer and you're looking at different plans and different ways of costing things, you don't really care what the physical representation of that number is.
You just want the plan with the cheapest cost. So if you have two ways of getting your data, for example, one's an index scale, one's a sequential scan.
If one cost is less, that's the one we're going to use. Whether that cost represents 20 milliseconds or 40 milliseconds, it doesn't matter because the only thing you look at at the end is the total cost amount.
You're basically trying to jumble a whole bunch of options of how to get the return the data to the user and you're just trying to pick the cheapest option.
It's kind of like going to a buffet and you're trying to pick the cheapest amount of food you can get for dinner.
You're just really worried about which one is cheapest and meaning of the actual number is never really an issue.
DigitalOcean is the easiest cloud platform to deploy, manage and scale applications of any size.
Removing infrastructure friction and providing predictability so developers and their teams can deploy faster and focus on building software that customers love.
DigitalOcean stands out from the crowd due to its simplicity, high performance and has no billing surprises.
Join over 150,000 businesses on DigitalOcean. Sign up with a $300 infrastructure credit at dio.co.se radio.
I know that a database like Postgres is going to run an enormous range of applications but if there is such a thing typical business applications,
how many tables are we talking about that could be in the median queries at 1, 2, 4, 12, 20?
Well, that's a good question. A lot of the OLTP which online transaction process, the typical order entry query,
it's usually going to hit 3 or 4 tables. You don't see maybe 5 or 6 would be a big on the high end of an OLTP type,
a traditional transaction query. Most of those queries are going to really be looking for a handful of rows,
you know, put an order number in, give me the order detail. Or here's a customer, give me the addresses for that customer, right?
You're really talking about only a handful of tables and you're normally going to return only a handful of rows.
When you start doing things like data analytics, then you start looking at a lot of times 15, 20 tables in some of these really big queries because your data model is just so humongous
and you've got so much data that you've atomized that usually so discreetly that you end up doing these huge amounts.
And also not only you have a lot of tables in the query, but you're also returning a huge amount of rows.
So the optimizer has to be very efficient. And in fact, when you start to do a large number of tables,
I think this is what you're alluding to, the system can get very slow because the optimizer, all of a sudden,
has so many ways of returning the data that it actually, the optimization time can actually take a lot of time.
So Postgres actually has something called a genetic query optimizer, which will randomly sample plans,
and that allows you to take a very big query that has a whole bunch of tables and get a pretty good plan without spending a huge amount of time on optimization.
I have questions here that are related. We were talking earlier about this being a combinatorial size problem.
Exactly.
I don't know the value of six combinatorial, but I could imagine it might be tractable where I could conceive of doing looking at six combinatorial different plans and picking the best one.
But I'm pretty sure when you're getting in 10 and 20 that I can't look at every one, you talked about the pruning process where you quickly eliminate a large number of plans.
So my two related questions here, how does the optimizer narrow in on a few plans?
And is there any thought given to how much time can we spend planning where we get to a point and say, I'm just going to go with what we've got rather than spending another hour planning to get you back your 50 rows?
Yeah, that's absolutely true. So up until 12 tables, Postgres is going to try every option, every possible way of returning the data, meaning all the index scan options, all the sequential scan options.
But it's going to prune it as it goes. So as it when it's looking at a single table, if it has a pretty good index that is pretty restrictive, it's going to discard the sequential scan pretty quickly.
It's not going to carry the sequential scan through all the future steps because it knows there is probably no value to that sequential scan because it has an index that's going to be very good.
If I'm asking for a given user, we can rule out, I would never want to look at every row in the user table. I'm just going to use an index.
So we're going to cost it, but we're going to throw it away before we get to the next stage.
Okay.
Because if I don't, if I carry that sequential scan along, then all of a sudden the amount of memory that I'm taking and the amount of options that I'm testing just becomes overwhelming.
Right. So we are pruning it as we're going. In fact, the pruning is one of the first optimizations I was involved in back in 1996 with the optimizer and Tom Lane, I kind of dig into how it was working and pruned it added a lot of pruning stuff.
So you're not carrying all those options through all the stages. If you did, it would be unbelievable. We slow. But even with all that pruning, once you get beyond 12 tables, even considering all of the ways of accessing the data becomes expensive.
So at 12 or more tables, we're using something called a genetic query optimizer, which is basically randomly considering different options of accessing the data.
It doesn't even consider them all anymore because even just it's much more aggressive at pruning. And particularly when you start not so much pruning different plans, but when you get to 12 tables, even the varieties of way you can order the joints becomes to you.
So what we end up doing there is we randomly try different joint orderings until we find something that's pretty good. And then we just go with it because you're right.
It doesn't make sense to spend an hour optimizing a query if you're only saving 10 seconds in execution time.
So you always have that holding over you is that every minute, every millisecond you're spending in the optimizer, you want that to yield it over a millisecond in faster query.
And when you start to get to these very large tables, that execute, that optimizer time can just be so big that you basically throw up your hands and you say, you know, we're not going to get the best plan here.
We're just going to get one that's good enough and go with it.
How does an optimizer trade off these competing goals of avoiding a bad plan, finding a good enough plan and finding the best plan?
Well, it's pretty much a binary decision. Up until up until 12 tables, we are going to test all of the possible joint options and pruning them.
And then everything over 12 tables, we're going to randomly try different options and just pick one that's the best of the ones we've randomly chosen.
So you can change that limit. I mean, you can increase or decrease.
There are a couple other things like collapsed limits, which limits how much the optimizer is going to try and collapse things.
There are a couple optimizer tunings that you can actually kind of turn it on and off.
But it's not a great, you know, it's not an exact science.
It's kind of that number 12.
We kind of came up with that number from, you know, from experimentation.
But I'm sure that there are queries where it should be 14 and there are other queries that should be 10.
And it's really hard to know when you're running. You don't know the future, right?
So you don't really know at the time you're optimizing that query how much you're really saving.
And normally you don't see queries like particularly in data warehousing.
You don't see the same query twice.
So you don't have really a track record of knowing exactly how good or bad it is.
The good news is that normally, you know, this is such an exact science that normally you're probably going to get the absolute perfect plan.
You know, maybe 70% of the time.
But you're going to get really good plans like 98% of the time.
Right?
So it's a very small number of plans that are really going to be terrible.
Even if it didn't do the perfect job odds are, you know, okay, it's going to be an extra 10 milliseconds.
But in the grand scheme of things, you probably did the best you're going to do and you just go with it.
I've looked at query plans where the optimizer is creating temporary tables.
When and why does it make sense to do that?
So sometimes the system will what we call materialize some data.
And that typically happens when you're doing some kind of data processing, like you're maybe doing an aggregate or you're doing a function call.
And then you want to use that data for some other operation, like a join or, you know, you need to sort it or something like that.
So you end up sort of creating the temporary.
So as you're running the query, you create sort of a temporary instance of an intermediate state of that data.
And then once you have the intermediate state, you can then do things like place it in a hash so you can do a hash join or maybe, you know, do another aggregate on it or sort it and then do another operation.
So you do see temporary results sets coming in.
Postgres allows you to control that if you use common table expressions.
Postgres effectively creates a temporary data set for each common table expression.
And then when you at the bottom query, when you're joining the common table expressions together, you're effectively joining these temporary results sets together and the optimizer because the temporary results sets can do all sorts of things with it that they can't necessarily do with the base table.
Because the result, the temporary table may have a lot fewer rows than the main table. It may have outputs of functions or aggregates that the main table just doesn't contain. So you need that intermediate result sometimes to efficiently return the right answer.
Is this something like in a program, an imperative program, I have a variable where I stick some result that's not the final result of my function, but it's something I calculate along the way to getting to that final result.
Yes, and you see that a lot in mathematics where you'll say X equals something and Y equals something and then we're going to use X for somewhere else and then Y will be used somewhere else and you're using these variables as intermediate results in mathematics.
But that's kind of familiar to people because those numbers are scalars, they're single numbers. What's interesting about relational systems is that these temporary results are not scalars, they're not even vectors, they're effectively matrices, rows and columns.
So they represent a dataset, not really an atomic value. And that's the only real difference between using temporary results in an aquarium and using scalar variables in mathematical proofs and stuff.
I can see why with aggregates and sorting, that would create some kind of constraint on the planning because in order to sort something you do need to have every value that goes into the sort before you can do this sort.
Does that help or constrain the optimizer when there's a sort.
Yeah, so for example, if you're taking a table and you select out 20 rows from the table and then you want to sort it. Well, what you don't want to do is to take the whole table and sort it because he only wants 20 rows.
It doesn't make sense to sort the whole thing and then pull the 20 rows out. So you're going to take the 20 rows, you're going to maybe use an index if you're lucky to pull those 20 rows out and you're going to make a copy of them in some type of temporary
location. Remember, it's not a scalar. It's a matrix. It's a row and column collection. And it's going to represent the 20 rows. And it's going to represent maybe not all the columns of the table, but only the columns that are needed for that particular query.
So we might not reference to all of the columns in the table. We may only reference a couple of them. So we're going to we're going to have fewer rows and we're probably going to have fewer columns, right.
And that's going to be basically in memory or potentially with a backing store to disk because the result set may flush the disk because it's too big to fit in memory.
And then we're going to sort that temporary result, which is much more efficient, obviously, because it has fewer columns and fewer rows.
And that that's what the result we're going to turn to the user. So that's a great example of why you would want to create a temporary result set to make sorting or aggregate processing or whatever.
Or whatever much more efficient. You were mentioning your involvement in some work to improve the optimizer by doing better pruning.
Hopefully the outcome of this work is you would make the optimizer better, but with any software development, you could also introduce bugs or make something worse.
What type of testing do you do on the optimizer to detect and prevent that you introduced regressions into the optimizer itself?
Yeah, that is something that is always it's always really haunted us.
And you know we're not alone. It pretty much haunts all of the relational systems, whether they're open source or proprietary because the optimizer is an incredibly difficult piece of software.
It's incredibly difficult to test. And you're right, if you introduce a bug, it's a major problem. You could be returning wrong results.
You could be doing things inefficiently and so forth.
What's really nice about Postgres and the reason we've been able over these 22 years to get Postgres really on par, if not better, than the proprietary databases is because we have a very close relationship with our user base.
So when somebody comes to us and says, hey, here's a query and it's not running really well, we're able to basically dig into it, figure it work with that user within a 24 hour 48 hour period, and often develop a patch that they can test to make sure that this fixes improve the problem for them.
But we also obviously have thousands of people who are looking at that patch to make sure it works. The answer is that we really don't have a great test suite. And I don't know of anybody who really does. We have a test suite, but it's hard to know because that piece of software is so complicated
that you're actually exercising all of the possible payers in that optimizer. So a lot of it falls back to the other people in the development team who are eyeballing that patch, the people who are just watching the email lists who are looking at it, and the user who's going to put that patch often in production and test it.
And because we're able to iterate so quickly with the optimizer and frankly the entire system, the quality becomes very high. And I know with proprietary databases because they don't have good contact with the users because the user interface between the users
and the optimizer developers is typically very limited. And because the release cycle is very limited, they often are terrified of changing anything in the optimizer. And it's probably good for these other databases, but they're really limited in how good they can improve it because everything has a trade off.
And unless you're in constant contact with the users, you can't really make a lot of significant changes.
I'd like to switch gears a bit and talk about this more from the SQL developer or software developer standpoint. I've used SQL and other declarative languages like puppet and Terraform.
First you think this is great. I say what I want. The system figures out. I don't need to know how it does it.
But then it turns out you kind of do need to know how it does it. To what extent should developers be looking at their query plans or QA people or DBAs before they run a query on a production system?
Right. So I think it really comes down to the quality of your optimizer. Right? Obviously if the optimizer was perfect, then you'd never need to look at it.
But of course it never will be perfect. So we just have to come with that assumption that there are going to be some cases that's true of all relational systems.
There are going to be some cases the optimizer is not going to handle well. That's just par for the course.
I know that one of the people, one of the large companies who converted from a proprietary database to Postgres had an entire team in their organization.
And their sole job was to study the query plans of that proprietary database. And I kind of talked to them early on when they were switching to Postgres.
And they wanted to know how this team should monitor Postgres. And I said, I don't know. And I'm not sure if you're going to need to do that once you switch.
And when they did switch, those people really did not have a useful purpose in the organization. They tried to train some of them, but a lot of them were resistant to learning Postgres.
And I think a lot of them left the organization. So there are some some databases, particularly proprietary ones that probably need steady monitoring of their query plans.
Postgres optimizer is pretty good. Not perfect, but pretty good. And you don't see a lot of that regular monitoring.
I will tell you, for example, that when I give a talk about the optimizer, last time I did, in fact, somebody said, how can I lock a plan?
How can I force a plan to be static so it doesn't change? And I'm like, why would you want to do that? And they're like, well, I was like, let me guess.
You're not using Postgres right now, but you're doing this locking with your current database, and you think you're going to need to do with the Postgres.
Is that correct? He said, yeah, he said, well, I said, you're probably not going to need to do that in Postgres.
So let me know if you switch to Postgres and you find you need to, and nobody's ever come to me to say they need to.
There are cases where it doesn't work, and you do need to study it. There is a command called Explain.
And if you type Explain and select or Explain an Update or Explain an Delete or Explain an Update, it'll actually show you the query plan.
And you can study it. There are some websites that allow you to analyze that query plan.
You can look for cases where the system misestimated the number of rows, for example.
Maybe it thought it was going to get 100 rows and it got 10,000 rows. That would be an indication that something is amiss, either in the statistics or in the data layout,
or maybe the function that was defined in an odd way. But there are things you can do to control the optimizer if you find that it's not executing very well.
Your response, you focus on one source of poor performance, which could be that the optimizer did a poor job.
But in an organization, a system, a lot of things could go wrong. Maybe the Postgres optimizer does a brilliant job,
but the table doesn't have the right indexes or the amount of rows in this table has grown.
And so you could end up with a poorly performing query, even though the query planter did the best it could given the information it had.
So my question is a little more on do you think development organizations should have some kind of controls or QA procedures to prevent poorly performing queries, however they originate?
That's a great point, because there are cases where we're doing a sequential scan, where an index scan would be very helpful.
There are other cases where people are creating indexes that have no use, and therefore they're slowing the system down.
So you can go both directions with indexes.
Basically, we have some tooling around that. One is PG-stat statements, which will actually show you another one is called PG Badger, which actually gives your report.
And it shows you which queries you're taking a long time, which queries are run a lot and taking a long time.
So if your system is slow, a lot of times you're going to start looking at which queries you're taking up the most amount of time.
Maybe they're taking each query taking a long amount of time, or a query may be taking a short amount of time, but it's run so many times that it's total execution across accumulated.
All of them is very high. And those are the queries you're going to want to look at first to make sure again, you're right, that the indexes that you need are in place, that the statistics are being used properly, and so forth.
That's your right. That's the kind of thing that people definitely should be monitoring to make sure that their queries are probably as good as they're going to get.
You're talking about a iterative process where people will look at these query plans, and you might notice, hey, we're doing a table scan of the user table on zip code.
We should really add an index to that table on zip code, and that will improve the performance of that query quite a lot.
I'm wondering, does the query planner or any of your tools have the ability to consider hypothetical query plans which would come about by adding indexes that don't exist and compare those to the query plans that it has, and maybe this would be a separate
report, but tell the user, hey, Bruce, you might want to add this or that index which will have a big impact on some of your queries.
Yeah, there is an extension you could install in Postgres, which will allow you to create hypothetical indexes, and then run explain, assuming those indexes actually existed.
And that allows you to kind of do that kind of brainstorming to see if the index actually would be helpful.
Now, I want to move in a little bit different direction, talk about how you might apply some of these concepts to a different problem domain.
Their massive popularity of SQL has created a huge pool of talent in this area who have SQL skills, but the choice of data sources is becoming larger, more heterogeneous all the time.
So most what we might loosely call databases have some kind of a SQL like API.
For example, there's SQL type layers on top of Hadoop, and instead of compiling down into index scans and row scans on file blocks, it generates jobs that run on a Hadoop cluster.
If you were given a problem as a problem to solve, like how would I create a good back end for a Hadoop cluster from SQL queries?
How would you approach that problem? How is it kind of similar or different to the problems you worked on it with Postgres?
You know, it's, I can give you a short answer and an odd answer.
I mean, the short answer for that particular problem domain is that our optimizer is incredibly complicated.
It has decades of sort of smarts in it.
It would be very hard to kind of from scratch.
What the optimizer is doing for Postgres is it's taking a request for data.
It's looking at the primitive building blocks it has, it's quite an index scan, different hash methods, different area, sorting limits and so forth, and having clause, at where clause.
And then it's kind of spitting out a set of building blocks that matches the request. And if you were to do that for Hadoop, it's the same problem, although you're building blocks underneath it as you stated or different.
So you'd basically have to take the same kind of process of parsing and then optimizing, but instead of spitting out Postgres capabilities, you'd spit out Hadoop commands, you know, that would, and you obviously have to optimize it for the types of things, which does well, like parallel processing.
One interesting aspect of Postgres is the fact that it supports over 100 different foreign data wrappers.
So you can create a table in Postgres, which is literally a Hadoop cluster.
And therefore, when you access that table, the Postgres foreign data wrapper effectively sends instructions to Hadoop, Hadoop returns the results to Postgres.
Postgres thinks it's a local table, and then it does all of its joins, aggregates and window functions on that Hadoop result.
And you see that a lot, actually.
So people who are mixing Hadoop data with Mongo data with Cassandra, and then maybe a call data recorder over in the corner, and then they're mixing that with relational data in Postgres, either whether to do transactional operations or analytics or serve webpages
with heterogeneous data.
So it's kind of interesting.
Postgres will effectively spit out the part of the query that relates to that specific foreign data source and send that foreign data source to the foreign data server, and then wait for the result and then combine it with other foreign data sources or native Postgres data.
This is reminding me of the earlier example where we were talking about, I have a program and it needs to find a bunch of rows based on some criteria from this batch of files that API, these emails.
And if I could just express what I want, rather than write this program, my life would be a bit simpler.
So it's a way the Postgres provides that capability to you if you're willing to write one of these wrappers.
So the interesting part about the wrapper is there's, again, over a hundred of them, and you just need to install the wrapper.
You're not having to write any commands to Hadoop.
Postgres automatically understands which part of the query that you sent us relates to that Hadoop cluster, and then sends the Hadoop specific commands to that Hadoop cluster gets the data back.
But then you're back to the Postgres optimizer, and you have all the joins, and you have all the aggregates, and you have all the other cool stuff.
And that's actually a really, really popular use for Postgres, sort of as a data broker or a sort of central data repository that can talk to all these things, but also has the smarts of a relational system to bring all that data together in a seamless way.
Yeah, I can see that being very useful. Bruce, we've covered pretty much everything I wanted to talk about.
Is there anything else that you want to cover we haven't talked about yet?
No, I really just love this topic.
The sort of understanding how that declarative SQL language is interpreted by the optimizer and generating an optimal plan has always fascinated me. It's one of the reasons I got involved.
Postgres early on having used a lot of proprietary relational databases and really curious, how does it work? How does it do what it does?
How can you influence it? How does it seem to know how much data is in every table and how restrictive things are and which indexes should be used and which indexes should not be used?
I think it's a really, really fascinating topic. And again, maybe it's just me.
Maybe it's an odd thing to find interesting, but every time I talk about it, every time I study, I find it.
It's just a new way of seeing software being used, I think, in a very powerful way.
I'm aware of Postgres MySQL is the other open source database I can think of.
I know there are people who work on these issues, commercial database vendors.
Is there a large research community around this set of topics and is there a shared body of knowledge that everyone in the optimizer community is contributing to?
You know, I would love to say yes, but I'm afraid the answer is probably no.
I've attended one IEEE conference where we talked about some of the sort of optimizer issues and database issues that are kind of common problems spaces.
But, you know, every relational system has a different set of tooling and a different set of primitives that it supports at the low level.
So a lot of the problems that we're talking about in terms of joins, in terms of query planning, I mean, these are topics that have been studied for 30 years.
And maybe some more in a lot of them.
So the body of knowledge is pretty static. There's not a whole lot of new stuff that comes out.
It's a pretty basic.
I mean, there are people who are looking at stuff like, okay, how can we do spatial better?
How can we do some, you know, some of the new data types like maybe Jason, how can we do a better job of that?
But the research community has the tendency to look at things very abstractly.
But, you know, when we're actually working on Postgres, we need a practical answer.
And a lot of the solutions we come up with are homegrown.
You know, we will read a couple papers.
We'll sort of see what people have talked about.
But when you finally get to the actual, you know, code, it's kind of a mishmash of, oh, this was made in this paper and this person in Russia had this idea and somebody over in Germany like this in Japan thinks it should be this.
And so it ends up being sort of a mishmash of research and practitioners together really sort of dictating the solution that comes out.
Digital Ocean is the easiest cloud platform to deploy, manage, and scale applications of any size.
Removing infrastructure friction and providing predictability so developers and their teams can deploy faster and focus on building software that customers love.
Digital Ocean stands out from the crowd due to its simplicity, high performance, and has no billing surprises.
Join over 150,000 businesses on Digital Ocean. Sign up with a $300 infrastructure credit at dio.co.co slash se radio.
Okay, now we've really covered everything I wanted to cover.
To go into the wrap up, we talked about your website, momjin.us. There's the Postgres website.
Are there any other personal sources or websites you'd like to direct our listeners to?
I think those are the big ones. Obviously the community website has a huge email community that's always talking about Postgres and answering problems and sort of researching new features and sort of helping people to use Postgres more efficiently.
There's a very big IRC channel. We're big on a lot of the big social media sites.
We're kind of all over now. There's a time when we were kind of small, but that's time's kind of over.
You can pretty much find us everywhere. I think those are probably the two.
There's an FAQ that might be used for some people. There's a lot of presentations on my website, including recordings of a lot of those.
If there's things that particular topics people are interested in, that might be a nice thing.
We have a lot of conferences coming up. If somebody would like to actually go and talk to other Postgres people and spend time with them and hear topics that might be interesting to them, we seem to have one or two conferences every month around the world.
So pretty much no matter where you are, there's something close to you related to Postgres. Those are also listed on the Postgres website.
Bruce Momden, thank you very much for speaking to Software Engineering Radio.
And for Software Engineering Radio, this has been Robert Blumen.
Thanks for listening to SE Radio, an educational program brought to you by IEEE Software Magazine.
For more about the podcast, including other episodes, visit our website at se-radio.net.
To provide feedback, you can comment on each episode on the website or reach us on LinkedIn, Facebook, Twitter, or through our Slack channel at seradio.slack.
You can also email us at team at se-radio.net.
This and all other episodes of SE Radio is licensed under Creative Commons License 2.5.
Thanks for listening.
