*Link* https://www.se-radio.net/2008/05/episode-99-transactions/



Software engineering radio episode 99, transactions.
This is Software Engineering Radio, the podcast for professional developers on the web at sedashradio.net.
SEO Radio brings you relevant and detailed discussions and interviews on software engineering topics every 10 days.
Thanks to our audience and the partners listed on our website for supporting the podcast.
Welcome listeners to a new episode of Software Engineering Radio.
In this episode, Bunten Anno are going to discuss transactions.
But before we get started, let me remind you that on June 15, there is the deadline for the position papers for your participation for the SEO Radio Get Together.
So if you want to participate in the Get Together October 16th and 17th in Spiber, Germany this year of course,
please make sure you'll send us these position papers, small one or two page papers that explain a certain topic that you find interesting that you want to discuss at the Get Together.
Please make sure this position paper reaches us by June 15th and by the way, please make sure ideally it's either text or PDF and you don't send us word files or ODF files or something.
Okay, thanks. Now let's get on with the episode. As I said, it's Bunten Anno talking about transactions.
So Anno, what are transactions?
Transactions are sort of a all or nothing way of dealing with persistent data usually.
The term is used quite loosely so it can refer to a business transaction as in I give you, mining you give me new computer.
But the main focus here will be on technical transactions as with the database.
I started transaction, do several things and in the end I either commit the transaction or I roll it back.
But this technical transaction thing need not be about a database.
There are all kinds of other sources of data that can participate in a transaction such as messaging, middleware, web services.
Basically, anything that has persistent data or does persistent changes.
So, well, that's basically what the transaction is. It's about doing modifications, treating them as a whole thing, one unit of work and it has a well defined beginning, a well defined end.
And all that happens in between is all or nothing.
Okay, so when talking about technical transactions, we define technical transactions by mainly for properties which are atomic, consistent, isolated and durable.
In that context you will also maybe know the acronym ACID which stands for atomic consistent, isolated and durable.
So Anno, why don't you start explaining the atomic property?
Sure, I'll do that.
This is sort of the ideal way of looking at your transaction.
It's the sort of the beautiful 10,000 foot overflight thing the way things should be. We'll get more dirty after that.
Having said that, atomic means the all or nothing part of a transaction.
It means, well, atom means it cannot be split into several parts and it basically means that if several changes to data are made or several persistent modifications are made by the transaction, they are either all made or none of them is made.
This is very important for different kinds of systems.
For example, let's look at a banking system and you do a transfer from one account to another.
Then two modifications need to be made. The certain amount is deduced from one account and it is added to another account.
And it's sort of important for the bank that either both modifications are made or none of them at all is made.
They are both changed at one unit. You either want both or none at all.
Otherwise you will in serious trouble and lose money as a bank.
So that's what the atomic part is about. Yeah, sort of the easiest to understand property of the the acid things.
The second of them is consistency.
Consistent is basically about the idea behind transactions.
Sort of you want to ensure that things remain consistent.
It's sort of vague.
And you can't always have this will, as you will see presently, but it's sort of more a vague,
a very kind of thing on the board transactions that you want to have, you want to ensure, but it's not a well-defined technical thing.
The third one is isolation.
And this is one of the sort of the trickiest part of the trickiest property of transactions.
It means that while one transaction makes modifications, the intermediate state is invisible to other transactions running at the same time.
The idea here is that read access, well, this is basically interesting for databases.
The idea here is that read access is also always part of a transaction, which is to some degree isolated from other transactions.
And, well, let's look at an insurance company as an example.
And there's one person working on a contract for one of the customers and sort of adding things over a long time,
working interactively with the application, the transaction running for two or three hours, let's say,
well, this accountant is, or whether this person is adding data to this insurance contract.
During this time, no other person or system in the insurance company must see any of the intermediate states because they're plainly inconsistent.
Or an other example where this is important is when we get back to the banking system with the money transfer between two accounts,
even if the transfer is atomic, that is either it is committed as a whole or rolled back as a whole,
you still have intermediate states.
First, you do an update statement, or, well, it's not usually an update, but anyway,
first, you modify one of the accounts to deduce the money and afterwards you modify the second account to add the money.
And let's say there is a report over all accounts because the bank wants to know how much money it manages, actually.
Then this long-running report across all accounts must never see an intermediate state of transfer from one account to another,
because then it will either count the amount twice because it was first added to the second account and then deduced from the first,
or not at all because it was first deduced from the other, then the report was running, then it is added to the second.
That's what isolation is about.
You always have, or usually have, intermediate states that are inconsistent and other transactions, other systems, other users
should not see the intermediate states always be presented with a consistent view of the system as a whole.
That's what isolation is about.
Intermediate states are isolated from being seen by other transactions.
And the fourth property is de-durable and means that the modifications are actually persistent.
A transaction is about persistent data, about persistent state that is viewable, perceivable in the long run.
That's what asset stands for, and those for property are sort of the ideal worldview of transactions the way we would like to have things, but can't always have them.
As you just mentioned, that's the ideal view on transactions. However, in the real world, we have to make some compromises here,
especially with respect to isolation.
Why do we have to make these compromises, and what are these compromises?
Okay, what we just talked about is the ideal worldview of transactions, of isolation. No transaction see anything that happens in between.
In reality, we usually don't have this.
And as we'll see, there are two things that cause this, two reasons why we don't have this or don't want to have it, actually.
The first is isolation has a price in terms of processing power and storage space, sort of tradeoff between the two.
And the other thing is you pay your price in terms of locking.
That is, no two transactions do the same thing at the same time. One has to wait until the other is finished.
That's the two things that limit, those two things are the price. You have to pay for having isolation, having high isolation levels.
To understand this, let's imagine we want to write a transactional database system.
That's sort of a mind experiment I like to do in order to understand things.
Let's say we want to implement transactional data modification with ideal isolation.
Then there are sort of two naive approaches we could do. One approach would be that no two transactions takes place at the same time.
So if I have a transaction running and you want to start a transaction at the same time, you are blocked until I do commit a my transaction.
Transactions are a lot about concurrency. And this is one easy way to have isolation because then, of course, if your transaction starts after mine is finished,
you will always see a consistent entirely isolated view of the world from my transaction.
This is obviously not a very good approach because it doesn't scale. It has no concurrency at all.
And for enterprise systems, it just doesn't work. You want concurrency.
So a somewhat, well, a still naive approach that is just a little bit less naive maybe is to copy the entire state of the database at the beginning of every transaction.
Sort of as you do with a version control system, you do a branch for each transaction, do the modifications in the branch and then merge them back when you commit.
This sort of works, but it does have significant overhead in terms of storage.
Beginning a new transaction would be very expensive operation, copying data or updating internal data structures.
So this is sort of the problem, the fix you are in if you want to have isolation.
If you want ideal isolation, that's expensive, but for many systems having less than ideal isolation is fine.
The systems don't require absolute isolation.
So for relational databases, the standard describes four levels of transaction isolation. We look at them presently.
The details of the implementations are vendor specific and the technology behind them is the same, but the names that are used are a little different between the vendors, but, well, the standard has standard terminology and you'll find them mapping there.
But the key thing to see here is that you can set the level of isolation separately for every transaction.
So you can have one transaction with very high isolation because this is a really key thing with high concurrency and many transactions would potentially conflict and then you have many other transactions with reduced isolation.
So you can think about isolation on a per transaction basis.
So let's look at the four levels of isolation that the standard describes. The first level in order of increasing isolation would be read, uncommitted, which is basically no transaction isolation at all.
One transaction modifies data, other transactions see the modifications at the same time. This is called a dirty read.
You see modifications, intermediate inconsistent modifications done by other transactions. And this sounds like a very, very bad idea, but actually for quite a lot of systems, it's okay.
For example, if you have web shops, low scale, rather small web shops that are mostly read only databases because the product data is stored there.
All these retransactions on the tables that are very rarely updated, it's perfectly fine to have no transaction isolation there because these transactions don't do any modifications that could conflict.
So you can avoid the overhead of having high isolation.
Or if you have transactions that work on strictly separate parts of the database, strictly separate rows, and your application logic ensures this, it's perfectly okay to have read uncommitted transaction isolation level.
Well, but this is not really isolation.
So the next level would be read, committed, which means one transaction sees only those.
Those rows in the database that another transaction has already committed. No updates to existing rows are visible.
This means you have no dirty reads. Changes become visible only after an update is committed.
But there are other limitations. You still don't have ideal transaction isolation.
You have something that's called a non-repeatable read. You read one row with the database in the beginning of a row your transaction, read the same row later on and get something else because another transaction committed it.
Let's look at this from the perspective of the insurance system.
Let's say I work on an insurance contract with a long running transaction.
And while you're in one transaction working with it, I commit my changes. And then while you are still the same transaction, see different values for the rows that you read at the beginning of your transaction.
For many applications, this is perfectly okay to have this kind of non-repeatable read, but still it is not ideal isolation because something that I committed is visible while your transaction is running.
So you cannot rely on seeing the same state of the database throughout your transaction. The next level of isolation deals with this. You have repeatable read, which means that the database remembers for each row of data that is read by you what the value at the beginning of your transaction was.
And it always returns that even if another transaction committed the data in between. So if I commit my changes to this contract and you do a select on the data, well, you do select before I do my commit, and then you select on the same data.
After I commit, you still receive the data.
Because you had at the beginning of your transaction, my changes are not visible, although I committed them. So you have no dirty reads and no non-repeatable reads, but there's still some limitation to the isolation.
That is, if I add new rows to the database and you do a select statement and return a whole lot of data, my changes become visible. So with the example of the insurance contract, let's say this insurance contract consists of several building blocks that make up the insurance policy, the insurance policy as a whole.
And I add building blocks in my transaction. You do a select with a where clause for this contract.
Then you receive at the beginning of the transaction just the building blocks that were there before my commit. If you do the same select afterwards, you get the additional building blocks for the policy that I added.
You don't get any modifications to the ones you had at the beginning, but you get additional rows. This is what is called a phantom read.
And if you want no phantom reads, you want to avoid them, you have what is called serializable transaction isolation, which sort of is the idea that things are the same as if you executed the transaction one after the other.
This is a very, very expensive isolation level. It's very expensive to do. And you don't want to have this for all your transactions.
But for things like, well, assigning primary keys, you often have a table that has a counter for assigning new primary keys.
And access to such a table is usually done with serializable isolation because there's high concurrency on this table.
And you want to be really, really sure that no two transactions modify the table at the same time.
As a matter of fact, serializable transaction isolation is not really serializable.
It means that it's mostly as if the transactions were done one after the other, they're still tied to a situation where it makes a difference if you do them on or for the other or just with serializable isolation.
And that is, if you read data for the first time, you receive that data at the time, at the commit state of the time when you read it.
Oh, that's somewhat technical. Let's go to the insurance contract example again.
Let's say you read, I'm working on the contract and the person having the contract, the data, and you have your transaction and you read the person's personal data,
like name and address and phone number contract data.
Then I do my commit with modifications both to the person and the contracts.
And after that, you read the contract for the first time.
Then you receive the contract data after the first time when you first read it.
So you have the personal data of the person in a version before my commit and the contract data in a version after my commit.
And for some very paranoid cases, this is a problem because it's not really the same as if you did them one after the other.
Well, one way of dealing with that, if you really need this, is if you have some sort of global things that actually,
if you need this, you can really serialize things in a physical way, or you can address this by historicizing data and have bi-temporal databases and that kind of thing.
But as for the technical support of the database, even serializable isolation is not really serializable.
And, well, maybe that's enough for looking at the dirty details of isolation. I don't want to bore the listeners.
The key thing to remember is isolation is important.
Isolation is expensive, and it's important to think what level of isolation do you need and how much are you willing to pay for it in terms of concurrency?
So now we had a look in the deep dirty details of isolation.
How do the database systems solve the problem internally?
This is a good question, and it actually does have some impact on programming, so I'll say a little about it.
For readcommitted, that is, this basic isolation that only committed data is visible, you have a thing called rollback segments.
Well, most databases do it that way.
That is, they sort of do a branch of the data and write modifications of one transaction to a separate part of the disk
and have filter all access of this transaction to the database.
Whenever this transaction reads data, it filters it to sort of merge, virtually merges the data between the rollback segment
and the actual table space.
And other transactions directly access the main table space without seeing this rollback segment, which is sort of a separate part of the disk.
And only when the commit is done, the data is actually merged, so commit is the expensive operation that copies and merges data
from the rollback segment to the main table space and then checks whether another transaction modified data in the meantime
that was modified by this transaction as well.
So if you have concurrent modifications of data, the commit fails because the merge of the two versions of data
from the rollback segment of the main table space fails.
This is obviously, this obviously solves the problem of isolation at the roll level of, well, you get read committed isolation,
but it is an expensive concept because it, well, you sort of copy all the data when you commit.
It means that you need bigger rollback segments, the bigger your transactions get.
So especially very big transactions become expensive with this rollback segment approach.
So this sort of the reason why it's a good idea to keep transactions small to commit cheap and to avoid the merge conflicts,
the concurrent modification exceptions.
Going beyond that, databases use the concept of locks that is they, one transaction modifies a row.
It acquires a lock, the different levels of lock, read locks, write locks are not going to the dirty details here,
and they're different for different vendors.
But the key thing to remember or to see is that the database marks different rows as having been read or modified
by one transaction.
So if another transaction wants to modify them, it is blocked.
And the higher the isolation level, the higher the locking level is.
Basically, for serializable isolation, every, if you do select to one table, all entries of that table are locked with an update lock,
sort of like that.
There are optimizations behind that, but that's what it makes it important because if you want to have the same result set,
returned by subsequent selects, it means that no other transaction must be permitted to modify the data in the meantime.
So most databases solve these higher isolation levels by serializing access using locks.
And if one transaction has a lock and another wants to acquire that locks, it has to wait until the first transaction is finished.
So higher isolation level reduce concurrency, which is the important thing to remember with regard to these locks.
The details, as I said, are vendor specific.
The concepts are a little different.
And also the granularity of the locks is different.
Technically or analogically, it would be a good idea to have locks at the row level.
If you lock one row of a table, you have a lock only to that.
But there are database systems that only lock pages, it's usually called pages, of data that is a group of rows,
because at the granularity, which they use to manage their internal spaces,
so you might have surprising reduction of concurrency using these locks, even if different transactions only access different data.
There's no overlap at the row level, you may have overlap at the page level, because data is on the same page and the page is locked.
The things to remember here are that higher isolation reduces concurrency because databases lock, data lock rows,
sometimes pages sometimes even hold tables, and then other transactions wanting to access the same data must wait until the first transaction is finished.
You can have deadlocks if one transaction locks part A of the database and then wants to access part B.
Well, another transaction first locked part B of the database then wants to access part A,
as with, well, this is actually a concurrency thing, was talked about in the concurrency episodes we did.
The thing here is if you start locking, you can have all kinds of deadlocks and breaking of deadlocks by timeouts and that kind of resolution policies,
but locking reduces concurrency by serializing access, which reduces throughput and is a problem for concurrency,
is a problem for concurrent access to the database.
Again, the key thing is keep transactions small and short to avoid these kinds of problems.
So I heard that, for example, Postgres and Fireword as well as database implementations use a different approach on locking.
Do you want to say something about that?
Yeah, I can do that.
Postgres QL, for example, uses a history-based approach,
which means that every transaction has a global unique number,
and Postgres QL remembers the global database state
through all tables while the transaction is running, and returns the data for one version number while this transaction is running.
So if other transactions commit data after that, they sort of append it to a history.
And this gives perfect isolation, but it is somewhat experimental still,
and anyway, it does have performance impacts as well.
So it is a different approach, but it's not a solution to all the problems here,
but yes, it is a different approach with different trade-offs.
So if you want to go into this and it's about performance,
you should definitely look at the manuals of the database systems you're working with.
All the database systems vary in the way they implement these, and Postgres QL has, well, it's optional there as well,
but it has an alternative approach that has some benefits for significant benefits for some scenarios.
As far as I remember, there are approaches called MYCC, which stands for Multi-version Concurrency Control, right?
Yeah, right.
So now we talked about logs on the database level, but on the application level we have logs as well
and think about optimistic locking, pessimistic locking.
How do they relate to that?
Okay, yeah, well, it's sort of the same term for different things,
as is so often with widely used terms such as transaction.
The problem with locking, or the problem these application level logs solve is concurrent modifications of data at the application level.
Let's say we have a web application, and it's this insurance application we used to talk about,
and I have one, well, I open a case, I open a contract and work on that,
and you do the same thing at the same time, shortly after me.
And then I have my modifications, and then I commit them to the database,
and as it is a web system, the technical database transaction is closed after the read,
it's not kept open while I'm working in a browser.
So there's one database transaction for reading the data and presenting it,
and another one for writing the data to the database.
So I write my data to the database, and you do the same thing.
You commit your data, you write your data to the database in a second transaction.
Overwriting my data without even noticing that you overrode my data.
Last one wins.
This is what happens if you have short database transactions, but long running work intervals
in user interface, for example.
So in other words, the higher assimilation level on database level wouldn't help me here, right?
Yeah, exactly. It wouldn't help you at all because the database transactions are kept short and small
as we, well, as I suggested.
It's a good idea to improve concurrency to have short database transactions,
and then the database level locking doesn't help you to avoid these kind of overwriting conflicts.
You need a higher level solution of the same problem once again.
So on the application level, I'm using optimistic or pessimistic logs here.
Right, those are the two usual approaches that are used.
Actually, most of the time, optimistic locking is the better approach.
It's sort of, if you don't want to think about it or don't think about it, just use optimistic locking,
which basically means there's a counter in every row in the database,
and when the application writes data to the row, it checks that this counter still has the same value as it had
when the data was read and then increments it by one.
So in the example, when I write my data to the database, well, when I read the data from the database,
I say this counter has the value five for the contract.
You read the data, the counter also has the value five, and then I commit my data,
and the application, when it's accessed, the database checks that the counter still has the value five,
which it does, so it writes, mine changes and increments the counter,
the application wants to write your data, it checks if the counter still has the value five,
but actually it's incremented to six in the database, so you get an error message saying that you lose your
modifications because someone else modified the data in between.
This is not really ideal, especially from your perspective, because you lose your work,
but at least it's explicit.
No modifications are lost on the way implicitly, so you know that someone else did that
somewhat of the applications at the same time, but maybe you get the information, who did it,
if that's part of what is stored in the database, so you can call me and we can talk about how it happened
that we both were working on the same contract at the same time.
So there's no implicit loss of data using optimistic glocken, which is often a good idea.
It's inexpensive, it has no performance overhead at all, and in most systems,
it's a rare case, it happens very rarely, that people modify the same data at the same time,
so it works quite well in these.
Actually it's quite interesting how you can implement this without having any performance impact.
You can add a check to the where clause of the update statements and then modify the counter by one,
so you have something like update contract, set counter to six, so counter equals six,
and all the other fields of the row, in the where clause you have where counter equals five,
and then the database driver tells you how many rows were affected,
and if one row was affected, that's great, because it means the counter still was five,
but if no row was affected, it means that the counter was incremented in the meantime,
so you know without having any additional SQL statements, you need to issue,
you know that there was an optimistic glocken problem.
It's a very, very cheap approach, and that's the main good thing about it.
So the opposite approach would be pessimistic glocken, which means that I have to explicitly acquire a permission to modify data.
Yes, exactly. The idea of pessimistic glocken is that the application prevents two people
modifying the same row, so if I start working on this contract,
pessimistic glocken would be that there is a flag in the row in the database,
and when I read, when I sort of open the case, when I open the contract,
this contract would be marked as it is in work, and the application would prevent any other user from working on this contract at the same time.
So we can both open it, but you would have a read-only view, sort of the same like if you open a Microsoft Word document.
The first person opening this document from a file share has commodified, and the second person gets a notification.
This is a read-only version of the document.
Is that okay with you? So you still want to open read-only. That's about this pessimistic glocken thing.
Persistent data in the database saying this row, this contract is in work, and the application makes sure that no other person can work on it.
This is great for really, really, really preventing two people modifying the same data at the same time.
But it is significant overhand because this requires application logic to deal with this.
It's hard to get really right and take care of all the details because you sort of have things like,
for web applications, I open the contract and my browser, I close my browser without saving my changes.
And then, well, the case just remains locked, so you need to have some sort of timeout maybe if that's what you want.
Or you need logic to allow some other person to break the lock to say, I know this guy is on holiday,
and he forgot to release the lock before he went on holiday, so there must be some way to release the lock explicitly without having been the one to open the contract to have acquired the lock.
You need all kind of logic around it. It's very much work to implement, and the benefit and practice is often very small.
So it's good to know about it, but it's not a very commonly useful thing to do.
Actually, this is sort of the long-running pessimistic lock thing.
If you want to have a lock over several technical transactions, which is what you usually have in web applications because you don't want to keep database transactions open for one user of several HTTP requests.
You can also have some kind of pessimistic locking for the duration of a single database transaction.
If you have a rich client application, you might open it to a transaction when the data is read, and then close the transaction when it's written back again half an hour later.
In that case, you can use things like select for update or use these technical locks of the database to implement pessimistic locking, which reduces the overhead significantly because the database takes care of all these timeouts and implicit lock release things for you.
So now I know about optimistic and pessimistic locking, but when do I use which strategy?
Okay, well, as I said, optimistic locking works well if there's very unlikely that there's a conflict anyway.
So most of the time, people sort of talk to each other before opening the insurance contracts in the application.
It's a rare case that two people are working on the same contract at the same time, so there's no problem there.
Optimistic locking is very good for that, and pessimistic locking is a good choice if you have huge modifications, where it's a really, really bad thing if someone loses them changes because someone else made a change in the meantime,
or can also be part of sort of a workflow-like thing.
If a contract is handed from one person to another, then there's always one person who is responsible for this contract, and scenarios like that, pessimistic locking can be a good idea.
Well, the main thing to remember is don't ignore locking.
If you ignore locking, it just means that the last one to write wins, which is usually a bad thing, and if you're in doubt, use optimistic locking.
Use pessimistic locking if you see significant benefits for your application, for your application logic, from pessimistic locking.
Actually, there's something about the names.
The names are sort of weird.
They both solve the problem of preventing concurrent modifications, but the names stem from the attitude behind them.
Optimistic locking means, well, I want to be really sure that no concurrent modification occurs, but I'm sort of optimistic that it's a rare case that two people modify the same data at the same time.
So I'm optimistic that it's a rare thing to happen anyway, so this is just, well, it's just okay to notify people that one person lost their work, but it's a rare thing I'm optimistic about.
So, optimistic means I do these heavy-weight precautions because I expect the worst to happen, and if I'm not really sure, no, if I don't really ensure that no concurrent modification can occur, then people will always do the worst.
That's called as thought of the attitude behind pessimistic locking.
It's a little like working with version control systems.
CVS has the optimistic lock approach. Everyone can modify the same things and commit, and CVS tries to merge modifications if two people modify the same thing, and, well, sometimes you do get a merge conflict, but that's okay.
This is sort of the optimistic approach.
The pessimistic approach is sort of explicitly checking out and locking files from version control systems, other version control systems support this.
And this is sort of the difference between optimistic and pessimistic locking.
Yeah, clear case is one example for that, right?
Yeah, well, most others offer both, but clear case is one of those that offers locks.
So, in the first part we talked about database transactions, now we talked about application-level locks.
They sort of solve the same problem, but yeah, maybe you can say something about that.
Yeah, sure. As I said at the beginning, the term transaction is used for different things, really.
First we talked about the strict technical transaction concept, and these application-level locks go more towards the business transaction thing.
And they solve the same problem of preventing concurrent modifications, of isolating changes from one another, of, but it's a technical decision with, well, valid trade-offs.
Either way, how long you want to have your database transactions open?
For call center applications with rich clients, it's common practice to have dedicated database transactions for each rich client and keep them open all day long.
And that's fine. Then you can use the pessimistic or can use database mechanisms, database transaction mechanisms for locking.
For web applications with high concurrency requirements and indeterminate large numbers of users,
you want to have very short technical transactions, and then you sort of need to add this layer of application that are locking on top of them.
Okay. So they basically solve the same problem, but with different trade-offs, because, well, depending on how long you want to keep your database transactions open.
Before you said, ideally, I close them as soon as possible and keep the transactions as short as possible.
But in some cases, as you just mentioned, this is not possible, which leads us to long-running transactions.
So what do I have to keep in mind when I have to use long-running transactions?
Well, actually, long-running transactions is, again, one of those things that can mean several things.
Long-running transactions can mean, as in the example of the call center application, that you have technical transactions that are open for a long time.
But more often, the term long-running transaction refers to long-running logical business transactions.
The longest one I heard about is the process that a new medicine has to go through from its invention until it's approved by the authorities.
This can be several years. So this is sort of one transaction, and if it's not...
If this new drug doesn't receive permissions to be used, this sort of row back logically, but until it's actually committed and validated, it can be several years.
And you obviously don't want to have a technical transaction that's open for such a long time.
Originally, the database transaction was invented to reflect the business transaction.
And then SQL was the user interface to the application.
We've gotten a long way since then, so we can have long-running transactions, which cannot be reflected by the technical database transactions as well.
Obviously, if you have several technical transactions from the database perspective, you don't have acid properties over this long term.
If they are not atomic from the database perspective and they're not isolated from the database perspective because you can...
Well, other technical transactions can see changes there. So it's absolutely application logic to implement this.
One typical approach to this is to implement a state machine that one case, for example, the drug application or the insurance contract application goes through from initial through completely
entered into the database, next state might be approved and next state might be approved by the boss and then actually signed and valid, something like that.
With different transactions there between these states.
And then it's up to the application logic to deal with these states. And you might, for example, implement the isolation by having the applications in one database.
And only when they're finally approved and the contract is signed, they're copied into a production database.
And only then they are visible to the booking system that takes care of the financial parts of the contracts.
Or you can have... Well, the simplest case would be that you have a flag if the contract is completely complete and signed or still preliminary, something like that.
The key point is it's up to the application logic to deal with this.
And if somewhere along the way one of the steps fails, this is just a transition in the state machine.
So, for example, to the state rejected of the contract, which means the application, the business transaction, the business case, the business transaction of going...
of bringing the application to the signing state fails. It's rejected.
But it is not a rollback at the database level, but rather you have something like an inverse transaction.
You sort of undo the modifications. You have made preliminarily and then bring the system into a new consistent state with a rejected contract.
It's good to think about... Well, it's good to think about this at two levels. One of them is actually the transactional part.
And if it fails, you undo things you did before. And the other is to see this sort of as a workflow with a state machine with transitions between the different states and actions you need to take when you do the transitions.
Isolation can be an issue because sometimes it's very important that one user, that only the user working on the contract, can see the details.
If the payment of the person working on the contract depends on the things he's entering there, it's important that other people, coworkers, are not able to see this.
So it can be an important issue to prevent others from seeing preliminary modifications, but that's up to the application logic.
Another context in which this concept appears is actually in long-running workflows, and this is what happens now in service-oriented architectures, where some different systems co-operate to create some results.
And there are basically you don't even have technical transactions. You don't have the infrastructure for technical transactions spanning all kinds of different systems.
So one system deals with the application. The next one checks the consistency against the some backend system.
The next system notifies some technical infrastructure, that's for a telephone company. And if this third step fails because some of the data does not really match or cannot be mapped to the technical infrastructure, the whole thing must be rolled back.
But you can do a technical rollback, so you sort of need to do in what's called an inverse transaction or compensation transaction, you sort of in a new technical transaction, you undo the modifications beforehand.
It's the same thing that banking systems do when a transfer fails.
If you do a transfer from one account to another, first a new row is entered to your account history saying the money is deduced and it's transferred to this other account.
And then later, maybe you gave a wrong account number to transfer to, so you transfer fails.
So a new row is added to your account balance sheet stating that the transfer failed and the money is again added to your balance.
But those are two different rows. It's not a technical rollback of the transfer, but it's an inverse or compensation transaction.
This is a good thing to keep in mind for long running transactions that spend longer time or spend more things that can be done in a single technical transaction.
So now back to technical transactions. So far we looked at transactions with regard to a single stateful server or database.
But what if I have several servers or databases involved?
You're probably talking about distributed transactions to face commit xa compliance, that kind of thing.
It's sort of popular to have these and it's one of the buzzwords that all application servers need to have.
The idea is if you have several databases participating in your transaction, you want to do modifications to one database and to another database or to a database and a messaging infrastructure or a local database and a web service or whatever.
You want to commit the data only if you want to have atomicity across several systems. That's typically the driver behind it.
You want to see?
Well, yeah, consistency. But atomicity in the term of either the changes to both databases are committed or none is committed.
And this consistency is sort of the idea behind it. It's the biggest part of asset.
So yeah, this was about to keep several databases, several resources consistent to sort of merge several local technical transactions into one.
Well, then either approach would be if you're finished doing things, so you write data to one of the databases, write data to the second database, commit the modifications or the first database, commit modifications on the second database.
And often enough, this is great. It works fine. But if the second commit fails, the second database says, no, I don't want to commit this.
For whatever reason, you have already committed data to the first database and you have lost your atomicity and your consistency across the different systems.
Actually, there's two different kinds of scenarios against which you can guard here. One is that one of the systems participating refuses to commit for, well, let's say there's a locking issue there.
It notices that it cannot commit because some of the modifications conflict with other modifications that were made.
The other is that some of the processes maybe might be killed. So let's say you committed the changes to the first database and then someone kills your process in a hard way, kill minus nine on Unix or something like that.
So that your process is just gone and didn't get the chance to commit modifications to the second database or someone pulled the network cable before you were able to commit changes to the second database and that way you lose them.
Those are the scenarios that sort of drive the fear towards two-phase commit.
Actually, there is a standard there. It's the XA protocol for two-phase commit.
The first prerequisite is that all participating systems support this protocol. They are all XA compliant.
Relations and databases usually do. Messaging systems often do. Web services basically never do.
And so many mainframe systems don't have two-phase commit support. So this is the prerequisite in order to be able to have these XA two-phase commit things.
All participating systems must support the standard. The second thing is you need a transaction monitor that is you need some process, some system that coordinates all the different databases and messaging systems
and all the other systems that participate in the distributed transaction.
Application servers often have a transaction monitor built in and there are standalone transaction monitors as well.
And if these prerequisites are present, what happens basically is that your application code tells the transaction monitor, I want to commit.
And then the transaction monitor asks all participating systems to do a pre-commit, that is to check all prerequisites and have them be really sure that they can commit,
that they won't have any objections against committing. Check all their constraints and everything, but do not actually do the commit.
And only if all the participating systems vote, yes, I want to commit and I'm willing to commit.
Only then in a second step, that's where the name two-phase commit comes from. In a second step, the transaction monitor tells all participating systems to actually do the commit.
And this guards well against the second system says, oh, I can't really commit problem because they all voted, yes, I will commit.
But there's no guarantee that it actually goes through in case you pull the network cable at the wrong time.
Actually, you can mathematically prove that it's not possible to have any protocol that really, really guarantees consistency in all kinds of evil hardware failure scenarios.
So it's basically a guard against the late notification or the too late notification that one system says, I cannot commit, but it's not a perfect guard against hardware failures.
It's good to keep that in mind.
And actually, two-phase commit is very rarely needed. A far more common approach is to have some sort of orchestration engine that talks to all the different kinds of participating systems, especially in web service where this is very widely used, but it's useful in other scenarios as well.
So you have one central part that sort of talks to all the others. And if one of the systems says, oh, I cannot commit, then an undue transaction is done and the modifications are undone.
It's sort of what we talked about in the long-running transaction part.
This often works very well, and two-phase commit is far overrated. It's very rarely really useful.
It's very expensive to have not only in monetary terms, but in terms of resource use as well.
So it's good to know about it when you need it, but it's often the better approach to think about the compensation transactions you can do if some later step fails.
Okay. I guess that wraps the episode. Let's do a summary.
Okay. Transactions are an important concept. Ideally, transactions have the acid properties atomic, that is all or nothing.
Consistent, meaning this is what's behind it. You want to have consistent data.
Isolated, meaning while one transaction is running, other transactions, other people cannot see intermediate states.
And durable. The data is actually stored in a persistent way at the end of the transaction.
Long-running transactions, especially if you have high isolation, are expensive and cause concurrency issues because isolation is often implemented using some kind of locking,
which means that if there's a lot of conflict, other transactions will have to just wait until one transaction is finished.
The rule of thumb here is reduce isolation as far as your application can deal with and keep transactions short and small in order to avoid the overhead because large transactions tend to be expensive.
Typical database systems support four levels of isolation. The first one is read uncommitted. The next one is read committed, repeatable read and finally serializable.
Yeah, exactly. If you keep your transactions short and small, you often read data in one transaction and write data in another transaction after it's been modified, especially in web systems.
And that means that your application needs to take care of ensuring consistency. Optimistic locking is the best approach to use there because it doesn't add any overhead and often times, most of the time, it's just sufficient.
If you have long-running transactions, the application logic needs to go further and ensure isolation, maybe even read isolation so that other users may not be able to see intermediate changes.
The data goes through different states as an estate machine with transitions between them.
That's up to the application logic to actually deal with these isolation things.
And if you have something like a failure where you would roll back a technical transaction, this means that you do it compensation transaction or inverse transaction, sort of adding another modification that undoes the changes, the effect of the changes you don't want to have anymore.
And with regard to two-phase commit and XA compliance, short thing is remember that they are there, but then forget about it. Most of the time, you don't really need them.
Well, that's about the way I'd summarize what we talked about here in transactions.
Okay, thanks Arno.
Thank you, Band. And thanks everybody for listening.
Bye-bye.
Thanks for listening to Software Engineering Radio. If you want more information about the podcast and all the other episodes, visit our website at ses-radio.net.
If you want to support us, you can donate to the SEO Radio team via the website, or you can advertise for SEO Radio, for example, by clicking on the dig-raded delicious and slash-dot buttons.
To contact the team, please send email to team at ses-radio.net, or if it's specific to an episode, please use the comments facility on the website so other people can read and react to your comments.
This episode of Software Engineering Radio, as well as all other episodes from license on their Creative Commons license, please see the website for details.
Thanks to Charlie Krau and the Potsai Music Network for the music used in this show, the song is called Vegas Heartrock Shuffle.
