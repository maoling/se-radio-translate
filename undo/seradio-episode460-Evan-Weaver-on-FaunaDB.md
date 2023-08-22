**Link** https://www.se-radio.net/2021/05/episode-460-evan-weaver-on-faunadb/


This is Software Engineering Radio, the podcast for professional developers on the web at
sc-radio.net.
SC Radio is brought to you by the IEEE Computer Society, by IEEE Software Magazine.
Online at computer.org slash software.
Keeping your teams on top of the latest tech developments is a monumental challenge.
Helping them get answers to urgent problems they face daily is even harder.
That's why 66% of all Fortune 100 companies count on O'Reilly Online Learning.
At O'Reilly, your teams will get live courses, tons of resources, interactive scenarios and
sandboxes and fast answers to their pressing questions.
See what O'Reilly Online Learning can do for your teams.
Visit oReilly.com for a demo.
Hello everyone.
This is Felina for Software Engineering Radio.
Today with me on the show is Evan Weaver.
Evan is the former director of infrastructure at Twitter where he was the 15th employee.
He also masters degree in computer science and has previously worked at SC.net and SAP.
Currently he is the co-founder, CTO and co- inventor of Fauna.
Welcome to the show Evan.
Hey, thanks for having me.
So today we are going to talk about your product Fauna.
And according to your website Fauna is a flexible developer-friendly transactional databases
delivered to you as a secure web native API.
So that's a lot to cover and I want to talk about all those separate attributes.
But let's first talk about the basics.
What problem does Fauna solve?
So Fauna solves a problem that we experienced at Twitter.
So I joined Twitter whose ancient history now is 2008, the height of the Web 2.0 era in
the midst of the early stages of cloud adoption.
Like I came from CINA, I built consumer websites with Ruby on Rails primarily in MySQL.
And I went to Twitter because I wanted the product to survive.
I didn't have big dreams about building distributed systems or implementing databases or anything
like that.
But we found it was the handful like the half dozen engineers who are at the company at
the time.
There's nothing off the shelf that would let a team like ours scale a product, especially
a global soft real time product like Twitter from small to large.
And we had to invest in learning and studying and inventing new technology and new strategies
and algorithms in distributed systems just to keep the product scaling.
That didn't make a lot of sense to us.
We felt like there's nothing intrinsic in information science that says you can't have
something which is both flexible to build against in the small and scalable in the large.
There were no systems off the shelf that could do it.
In particular, there were no operational data systems that could do it.
Operational data is the most conservative part of the industry in a lot of ways because
it's the riskiest.
It's your mission critical data to user data.
It's things that are irreplaceable.
We were dealing with systems like MySQL and its primary, secondary, statement based replication.
Looked at Postgres as well.
Looked at Mongo as well.
Looked at Cassandra and invested a lot in Cassandra in the early days of the NoSQL era.
Since none of these systems could scale without these discontinuous architecture events along
the way.
We ended up re-architecting things almost on a annual basis.
That was very frustrating.
I don't think anyone works on databases out of love for existing databases.
They work on them because they're frustrated that things are not better than they are.
They see a path to eliminating that area of pain for future developers who have to work
with data.
We're no different.
After spending four years at Twitter and scaling all the core business objects and writing
a bunch of custom distributed systems, we found that Fauna did a bunch of consulting
for four years, exploring the data space.
Then eventually realized that if we didn't solve this fundamental problem, it wasn't
going to get solved.
We embarked on product development.
That's Fauna and FaunaDB.
To summarize, I think what you're saying is that the biggest problem that you found is
that existing solutions were not flexible or they weren't scalable enough.
Right, exactly.
You had to make trade-offs at every step of the way.
That means too.
It doesn't just mean things are hard for companies that are succeeding.
It means there are all kinds of companies and products and applications that just don't
get built because the incidental cost of scalability and operations and management and administration
and all this stuff is too high to justify the investment.
It's similar to the 90s.
If you wanted to have a website, you'd do a bunch of hardware and configure a bunch of
custom databases.
Fussing around with Apache was a big deal.
We don't have to do any of that stuff anymore.
We forget how much of a drag that was on productivity and industry and even having a basic HTML
page like Yahoo.com out the door was tough.
There are a few entrants in the early market of the web.
But now we're still in the same place even with Manage Cloud where you have to deal with
provisioning and the physicality of your data in order to build an application.
Especially as the world goes more global and goes more remote, none of that really makes
sense.
It's all extraneous cost.
Yeah, that makes sense.
So can you describe a typical use case of phone?
You already described, of course, the scalability part of it.
And what type of situation for developers would it make sense to consider phone eyes
and alternatives to whatever database you're currently using?
So Fauna is a serverless operational data API.
So it serves the same role as the traditional operational database, whether that's a no
seagull database like Mongo or an RDBMS like MySQL or Postgres.
All right, what Fauna does is provide you the basic building blocks of a soft real time
consumer or B2B facing service.
It could be a CRM.
It could be some other SaaS.
It could be social media.
It could be video gaming.
Anything where you need to transact across business data, user data, all the usual concerns
in the OLTP or operational data sphere.
And Fauna is designed to both offer serverless provisioning and serverless scalability so
you don't have to do any operations at all and to fit well with the rest of the serverless
stack or as augmentation to the managed cloud stack so that you can continue to build your
products, build your applications without having to do in paraditional operational overhead
or debt.
You know, another way to look at it is it's sort of serverless overall is a return and
a realization of the dream of utility computing and we used to talk about that a lot in the
90s and we tried to manage it by fussing around with Perl scripts and that kind of thing and
it didn't really work.
Well, it's back and it's back in a big way with different interfaces especially.
The POSIX interfaces for containers and that kind of thing and WebAssembly and other lighter
ways to isolate compute and we have CDNs which offer asset hosting in a truly serverless
and zero operations way.
We started this project kind of before serverless was real but the vision has always been the
same.
You know, what if you could interact with your data with the ubiquitous utility priced global
API and you never had to think about where that data lived ever again.
So the thing what you're saying is if you already have a solution that runs in the cloud, runs
on the Roku or AWS and you want to extend that with database functionality then it really
makes sense to look at fauna because it's also serverless.
Yeah, if you're building with the rest of the serverless stack and that can be jam stack,
it could be server-side compute with AWS Lambda or you have an existing managed or even on-prem
application where you don't want to make further investment in the operations of that
system.
Your typical Postgres installation for example gets to a point where people are afraid to
migrate a column or add a table because there's no performance isolation.
You know, it's risky.
So where else do you put your data?
Well, you don't really want to keep spinning up Postgres that'll keep having these problems.
The one you currently have is where you can perfectly find.
So you probably don't want to take the risk of migrating it to something brand new even
a different addition of Postgres can be risky.
But you can take something like fauna and add it into that application stack, integrate
it through web APIs and service APIs and start to get the benefit of that utility computing
that serverless data for the augmented features.
There's actually perfectly segues into the question I was going to ask next because you
also say that fauna is a flexible database.
And I think you just touched upon that on addressing the pain of adding a column to a
database that people might be worried to make changes.
I guess that's where you are different, that this flexibility means that it is easier to
make changes to your data model.
Your phone is designed to minimize operational overhead and maximize developer productivity.
That means we chose not to directly implement SQL.
Fauna is a relational database that offers transactions and indexes and views and foreign
keys and all that good stuff from the relational model.
But it doesn't at least currently expose a SQL interface.
It exposes a document relational interface over GraphQL, which is a web standard you may
be familiar with, and it's very popular, especially in the JAMstack space.
And FQL, which is our own language, which is a functional relational language.
It's essentially a list, but we try not to talk about that too loud because a lot of
people are scared of lists.
But it gives you a more safe and semantically pure way of interacting with the data, which
includes things like dealing with documents which also retain their history.
So you're not discarding data just by updating records.
You can go back and look at a particular point in time.
This type safe lets you compose queries.
It lets you store stored procedures as functions that execute business logic globally in the
database and compose it.
It essentially gives you programmable access in a modern way to your data set.
Those things are all, by the way, things I want to ask you a little bit more about.
So the stored procedure, ours, and the different types of interfaces.
We will definitely zoom into that later in the episode.
But firstly, let's talk about the flexibility a little bit more to give listeners an idea
of what FONA can mean.
Can you give an example of a case in which this flexibility really helped solve a customer
problem?
I'll give you two examples.
And I'll give you one on sort of the feature set size.
So we have a customer named ShiftX.
They have a business process modeling tool.
And one of the things they took advantage of FONA within their tool is they took advantage
of the temporality to let their own users go back in time and see how their process
models have changed.
And that's the kind of thing that in a traditional database, whether it's no SQL or SQL, you
have to build as a secondary concern in your business model.
And that gets very confusing, especially in a typical SAS application.
You end up trying to implement these partially application facing, partially database facing
concerns like by temporality, multi-tenancy, in application code.
There's a lot of security features which are missing from the SQL interface in particular.
And that all pollutes your application.
It makes it hard to iterate.
It makes it hard to sort of maintain that kind of purity of approach to the business
problem you're trying to solve.
They were able to use the temporality feature directly to expose time travel and just move
on to bigger and better things.
On the operational side, in particular, the serverless nature of FONA means that you never
have to provision any specific level of scale.
You can rely on the database to scale like an API, which is to say you don't think about
it.
Capacity is always there.
You pay by usage.
You don't pay by like scheduled time or some variation of that.
And in a use case, I just saw the other day, someone's building an application that is
used at conferences.
They're anticipating a return to in-person conferences.
And one of the challenges with conference attendance and especially connectivity is that
the Wi-Fi never works when you're actually in the room with 100 other people trying to
get on the Wi-Fi.
They're building a semi-off-line application which will work while it has partial or zero
connectivity.
And then when connectivity is restored, it uploads the data to the cloud and makes it
accessible to others.
And that means basically whenever these groups disperse, there's a huge spike of synchronization
from people who suddenly have network access.
And like you don't want to provision for that spike all the time because you're wasting
money running databases and 0% utilization while people have no connectivity all as a
group.
But at the same time, if you provision for a small amount of capacity, then you're going
to have failed queries timed out queries that application isn't going to work in these
synchronization points.
So they're taking advantage of FONAs operational flexibility to just eliminate that concern
entirely from their application process.
And instead of doing like exponential back off in this and that and all that kind of
things you'd have to usually do to work around this problem, they can just rely on FONA to
make capacity available as needed.
Great.
So I think the two things you mentioned were that you get scalability for free.
You don't have to think about it if you have certain peaks in use and that is covered automatically.
And indeed, you don't have to buy lots of servers just to cover for this peaks that might
happen very rarely.
And also the other point you mentioned that temporality, that sounds so interesting that
you have time traveled by default.
Can you talk a little bit more how that works as a developer because what I'm envisioning
is something like Select Star from customers yesterday or something.
Do you interface with a database that has time for reality built in?
That is essentially how it works.
One of the reasons we implemented FQL, FONA query language, our own language, instead of
SQL is that SQL is a business analysis language.
It was designed for people sitting at workstations writing reports.
Security was managed outside the language itself.
Performance was managed outside the language itself.
There are all these concerns which are now intrinsic to building a particular SAS and
consumer facing web applications which those interfaces make no attempt to solve.
And temporality is one of the things that we saw a lot of need for both in backing direct
user facing application features like time travel, select this document at a point in
the past, see the changes between two points that's related to the streaming capability
which I think we're going to talk about in a little bit.
But also for things like change data capture for synchronization with second systems like
analytic systems, it's very important in particular to be able to do cheap update queries to see
if anything has changed rather than having to table scan all the time because all the
records are fundamentally unpartitioned by time even though they have a time column added
to them because it's at the application codes and the database is nowhere.
It's in the application layers so the database has no awareness of it.
And you know other things like the web native security model which makes it possible to
access the database safely over the public internet and everything secured by default
unlike the opposite where you have to use firewalls and VPCs and so on to try to lock
down administrative access to the database of the granularity that lets you authenticate
users directly against the database.
It's only essentially operators who can mess with one table at a time all that kind of
stuff like what we've tried to do is basically retrace in particular the history of MySQL
and its ability to be a general purpose transactional data platform for what was at the time the
modern web that LAMP stack, Web20 and serverless operational context for the way that the stack
has changed and I think historically we get a new development stack about every 10 years
and that stack typically has only one database that really becomes the default database that's
chosen for it and I think part of this is points to some of the conservatism or the riskiness
of making that choice like people aren't choosing an operational database just for one project
they're choosing it for their company overall and even oftentimes for their career.
It's a huge investment to learn a new database to learn to operate it in particular for the
pre-serverless databases operated safely interact with the consistency model appropriately and
so on.
In the 90s we advised QL with LAMP we had SQL server with the Microsoft stacks there was
a bifurcation there because we had two totally different stacks even though they shared some
concepts.
Then in the O's we got Postgres with the ORM kind of the ORM generation with hibernate
and active record and in the 10's we got Mongo and the mean stack and we have a new stack
now the serverless stack and it needs a database so we're bringing back everything you would
expect from the traditional relational model but updated and revised and reimplemented
for serverless operations and serverless development.
So diving into that a little bit more about the options there are to be let's say the
next database for this era.
Who do you think FONA's biggest competitors are and how are you different from them?
We see DynamoDB the most and DynamoDB has done a good job in the utility computing aspect
giving you multi-tenant granular billing based on usage where it hasn't done a good job
is maintaining the flexibility of the relational model and eliminating the physicality of the
database from the application level concerns because Dynamo is an eventually consistent
system that was derived in one of these weird quirks of history it was derived from Cassandra
which itself was derived from the old DynamoDB which was an internal Amazon project but it
came full circle as a no SQL rich key value store that was designed for low latency and
consistent performance not for developer flexibility and productivity.
One has made a lot of retrofit updates to it like adding optional transactions and optional
indexes and that kind of thing but they don't compose because the model is not fundamentally
built around an architecture which is minimal to compare to FONA transactionality all the
time and ubiquitously available consistent indexes all the time and rich relational queries
that can express flexible business logic and that kind of thing.
So on the pricing side like does it scale up and down dynamically yes you know in that
sense it is competitive in the developer flexibility side it's not and we see that a lot you know
and you can compare it to Mongo I think Mongo offers more of developer flexibility but the
Mongo Atlas cloud product is not serverless and it's never going to be because the architecture
isn't there various other vendors have approached like one aspect of this problem but because
of the legacy baggage and there's so much path dependence especially in operational data
because it's so risky there's just no path for them to really deliver the full picture
without starting over from ground zero like we did.
Yeah and of course you set all those requirements from the beginning so that's why it might
be a little bit easier to actually hit that sweet spot.
Our story is a little atypical I think for Silicon Valley or for startups we were at Tech
for startup we saw this technical problem at Twitter we believed we could solve it we
watched as it you know continued to go and solve an industry and then we embarked on the
journey to solve it.
Once we had solved these core problems of basically global serverless transactional database management
we had to figure out how to take it to market.
You compared to someone you know a company like Mongo which founded product market fit
very early but then had to retrofit technology because they literally had nothing like an
m-mapped file in a JavaScript process like there's no database there there's just interface.
We went the other way we built architecture and then essentially we were too early to
market and had to wait for the serverless market to catch up and be ready for what our
original vision was and then you know we had to make tweaks along the way like adding
the GraphQL interface and so on and in particular improving the comprehensiveness of the FQL
standard library to meet the developer flexibility and productivity goals but at the end of the
day you know the serverless vision of ubiquitous utility computing has always been our vision.
Great so one thing I want to zoom in a little bit more about you say that you have transaction
nationality all the time what does that entail? Transactionality is essentially data correctness
at the highest level and databases are more or less partitioned into consistent transactional
databases or eventually consistent non-transactional databases.
One of the issues especially when we were starting out with thought I was that there's
a general idea especially push by the NoSQL vendors that having data consistency and having
scalability were fundamentally antagonistic to each other and that you couldn't have
a system which did both. If you needed to scale then scalability is the most critical
operational capability that you can have because it doesn't matter if your data is correct
if no one can get to it then you should give up consistency and correctness in favor of
scale and then try to add back in in application code whatever sort of band aids over the essentially
the corrupt state of your operational data you needed to keep the application working
and we knew that that wasn't the case based on our experience building the distributed
systems at Twitter. When we started FONA we wanted to make sure that we chose a replication
and transaction architecture which would both give us low latency at global scale and you
know literally the lowest possible latency which I can elaborate on in a minute but also never
give up the highest level of consistency which is strict serializability which basically
means you can treat the entire database cluster not just a single machine but the entire cluster
as if it was local to you as if everything is happening in a serial physical order one
transaction at a time in that database and that's a very easy model for developers to
reason about compared to eventual consistency which is essentially relativistic like different
clients different applications different users accessing the database all see different views
of the state of the world and they cannot fundamentally be reconciled in real time.
And it might also be cognitively harder for programmers to think in need of all those
different versions whereas the transaction model might be closer to what people are used
to. Yeah the transaction model is intuitive you know it's you can essentially imagine
there's a giant lock around the entire database and only one user can access it at a time
and obviously that isn't the case in fun and it can't be the case because although you
could implement it your throughput would be pretty poor but yeah we saw based on our
experience at Twitter and in our consulting days at fun that not just your average developer
but even your best developers just could not reasonably reason about the consistency of
their data in the context of their application code it's just not possible like data consistency
is the domain of formal proofs and very aggressive adversarial analysis of the systems to make
sure that the implementation matches the proof and all that kind of stuff and like that's
not how people build products there's no formal proof for Twitter tick-tock or what have you
like it doesn't make any sense it's a waste of time it would not even be possible given
the complexity of the you know the service topologies that go into these products.
Yeah so let's zoom into this topic a little bit more because it looks like you do support
replication so is FONA a distributed system? We are a distributed system so we as the FONA
provider the vendor operate FONA in multiple globally distributed regions which are hosted
on multiple cloud providers under the hood. You as a user don't have to care about that
because you just interact with an HTTPS API which gives you this facade of ubiquitous
consistent access to the data wherever you happen to be querying from and that leads
to a lot of nice properties for you because you can always guarantee that your rights
will be consistently low latency regardless of where you are in the world because there's
no primary region or cluster that you've to talk to to perform a right you can guarantee
the same things for reads with even lower latency whichever FONA region is nearest to
your application that's the region you'll be routed to to do your work.
So if I understand this correctly you have replication you are distributed system and
you even have regional distribution but as a user I don't really see that I don't have
to say I want to use this version and I don't have to think about where is my data and might
some things go out of sync because that's all covered all cared for at your side.
Right we're adding the ability to restrict the replication to pology for compliance
purposes principally because you know you don't always want all your data given various legal
regimes replicated all around the world but FONA's replication is semi-synchronous and
always consistent from the perspective of any individual group of applications which
are accessing the same data center.
So it's impossible in the read-write transaction pipeline to violate consistency and in the
read-only transaction pipeline it's impossible to violate it without deliberately going out
of your way to pass information behind the scenes that you basically disallow FONA from
all those scenes.
So in practice you get a consistent experience all the time no matter who and how is interacting
with your data set around the world.
Yeah it is a distributed system but as a user you don't have to worry about it you can just
benefit from it.
Yeah the model for you as a user is essentially the model of a single node already BMS.
Yeah like a single-fashioned database.
Right you have a rack in the closet and it has the data and you access it and the data
is always the data because it's only coming from one place like FONA restores that illusion
of the data coming from only one place to a completely globally distributed and highly
available operational context.
At O'Reilly we know your tech teams need quick answers to their most urgent questions
they need to stay on top of new tech developments they need a safe place to learn the technologies
your company adopts and they need it all 24-7.
Well they can get it all at O'Reilly.com with O'Reilly online learning your team gets
live online courses tons of resources safe interactive scenarios and sandboxes and fast
answers to their most pressing questions visit O'Reilly.com and request a demo.
So let's talk about real-time streaming because that's something that FONA also offers.
So what is this?
What is real-time streaming?
We have a feature in beta now which basically takes the temporal model and upgrades it to
an HTTP push not just a pull.
You could already pull on change sets points in time ranges in time but in particular for
backing real-time applications you want to get notified rather than query every 10 seconds
or something like that.
So FONA lets you listen on individual documents for updates from both backend services or
user applications including browsers and mobile apps that kind of thing and then we're expanding
this feature to also allow you to listen on entire collections and indexes as well.
So that means that I can implement something like an old new customer event and then I
can send an email to someone that a new customer registered.
Yeah or it's kind of topical right now.
There's a lot of E-lines happening in the world right now.
I've experienced some of them myself recently.
If you want a new gaming console you have to wait in an E-line and you sign up and then
you get notified when they're available and then you have to rush to try to grab the
reservation or for the coronavirus vaccines it's essentially the same process although
it's federal and state agencies instead of like Sony who are offering you the website.
All these kinds of experiences are really good examples for the power of something like FONA
because both you need notification when something is available.
You need the ability to order in a strictly serialized way who is in line so that you
can be fair about allocating the opportunities then you need to make sure that only one person
can buy a specific console and only one person can get a specific vaccine reservation and
that requires transactionality in particular it requires strict serializability.
If you're trying to do this an eventually consistent system you see a lot of the failure
patterns that we're experiencing today where it says oh consoles are available but they're
not you go to check out and it's gone.
Vaccine appointments are available but they're not.
You go partly through the process then you get booted out and then you can't go back in
because it wasn't reserved specifically for you.
Someone has taken over your application and it's a real mess.
That's what I really want to have transactionality.
It's very clear that this vaccine has been assigned and there are so many left in stock.
Right.
Yeah it's like I mean I think you know people say that banking is like the canonical example
for transactionality and the value of acid but it's really not because financial data
can be reconciled after the fact.
I think the best example is this kind of process basically like the ticket master process where
people are competing to hear about an opportunity and reserve it in real time and that just doesn't
work in a distributed context without a database like Fauna.
That's a great business case where it really makes sense to use a system like Fauna.
What you also support is custom business logic as atomic functions and we talked about this
I think a little bit earlier where you said something about stored procedures because
when I read custom business logic as atomic functions then I did think is that a stored
procedure?
It is a stored procedure.
Store procedures aren't cool but they're back in a big way.
We call them functions analogous to serverless functions and like I mentioned Fauna is a
programmable database like what you're submitting to the database is not only reads and writes.
You're submitting a mini program that executes in an atomic way over the state of the data
at its point in the transaction log.
That means in particular you can't do what you would do in SQL which is have an interactive
session transaction where your client chats back and forth with the database accumulating
the transaction over a period of requests.
You have to submit the entire transaction to the database so that it can be strictly
serialized as a block.
That's not really a barrier to development.
Most people don't really take advantage of interactive transactions in a meaningful way
when they're building products with RDBMSs like they use ORMs and other things like that
that abstract away the session based nature of the database anyway.
So your experience developing against Fauna is similar but it does mean that we have to
offer a very rich standard library so that you can do a lot more of your compute work
in the database co-located with the data in your transactions than you would typically
be used to doing in an RDBMS or in an Osegal system.
But that also gives us the benefit that we can then bring back stored procedures as functions
and let you basically save particular parameterized transactions as your own standard library that
you can then call and compose in your other queries to the database and that lets you
do a lot of your business modeling in the database which has the advantage of also being
secure so you can access it directly from insecure clients like mobile apps and browsers
and that kind of thing.
But lets you manage that code in a clear and consistent way.
This is another example where the temporality comes into play because the database is temporal
so are the functions so you get version control over your code in the database itself and
that kind of thing.
So really it gives you back a lot of the power that was lost when we moved from kind
of the client server architecture in the early 90s to three tier architecture with the web
where the application had to orchestrate everything and it basically used the database as the
dumbest possible transactional key value store because you kind of you couldn't scale the
compute layer of the database in particular and also there were a host of security and
code management issues involved in using stored procedures from web applications but that
doesn't mean the model is broken it just means that you know the implementation and the patterns
people were trying to achieve in their applications had diverged.
So I really want to hear more about this programmable database.
What is programmable like firstly what programming language do you write?
Do you write your FQL language?
Is this what you write to stored procedures in or do you have language bindings for JavaScript?
You do write it in FQL in the future we want to expand that but right now we're focused
on making the FQL standard library as comprehensive as possible and FQL is fundamentally a list
it's a functional language which is composable and type safe.
It's Turing complete because you can recurse we don't recommend mining Bitcoin and Fauna
but you know eventually someone's going to do it.
And then if it is this model and it goes back to you know some of the lindowork or you probably
notice like Java spaces and that kind of stuff is that you can execute computations co-located
with your data and that makes them much more efficient and also much more consistent than
when you have to query back and forth from application code and run compute incrementally
in the app while you then do reads and writes also incrementally and interleaved in the
database.
And you know that lets us then give you essentially this ubiquitous data API this data fabric
in which you can compose not just your schemas and your data models but also a lot of the
business logic around those data models too and then you can focus on the presentation
logic and application code and keep those tiers well segmented and also maintain both
performance high availability and consistency for all the business logic which is implemented
in the database itself.
So that also means that these store procedures are ran on your side so they're not run a
ran client side bet that ran on your side and also they're always run on the same machine
where the data also lives is that they are they're run in the database kernel we don't
like shell out to lambda or something like that.
When you sign up for fauna you instantaneously get a new database that you can access immediately
that the reason we can do that is that we never provision a static resource for anybody.
All the isolation is dynamic it's essentially equivalent to cooperative multi threading which
you may remember from Windows 95.
The interpreter of the query AST is inserting yield points like windows 95 at IOs and loop
boundaries which then fall back to the scheduler and that happens on a millisecond basis so
that we can interleave long running queries we can track quotas we can prioritize and deprioritize
usage across tenants with in tenants all that kind of thing and we can do that within the
stored procedures the functions too so we can maintain performance and security isolation
without having to statically allocate anything and that lets us not have to worry about running
complex computations co located with the data which then lets them obviously have zero latency
to data on the local node low latency to data that's partitioned on adjacent nodes and that
kind of thing.
So let's dive into writing queries in more depth even because what you say is that you
have a multi model interface so you support document storage relational data graph data
and also temporal data sets and this is actually something that we covered earlier in the show
in episode 353.
So can you tell us a little bit more about how those different models work and especially
how they also collaborate and when you use one when you use the other can you mix and
match within a query how does that work?
I think we as a team have a unique view on this problem so to us there are two fundamental
data models there's the document model like Mongo to normalize everything into a rich
flexible document and your transaction boundaries basically are the document boundaries you can
consistently update the document itself outside of that context all bets are off then there's
the relational model set up a schema ahead of time build indexes and force foreign keys
achieve fifth normal form if you're super into that all that kind of stuff like it's
the opposite of denormalization it's aggressively normalized it's literally in the name and use
that to both reduce redundancy which improves consistency because you're not duplicating
the same data and you might be wearing you might mess it up and also you know it works
well with storage layouts and so on of the traditional RDB mess that's all you need in
terms of fundamental data models to build almost any application and thus FONA is a
document relational database where it offers those relational links indexes views foreign
keys transactions between flexible documents that don't have rigid schema although they
are typed you know that allow you to dynamically populate data without having to go through
migrations and so on and the other models you mentioned temporality and graph they're
not stand alone models they're additional capabilities that can be added to the core
document relational model you don't have to get a graph database to do graph queries
many people who use join tables very successfully and already be a messes to model graphs understand
you know you don't have to go get some off the shelf right here temporal database to have
temporality sometimes people do don't change data into in particular logging systems like
Splunk and that becomes kind of an ad hoc temporal database but to me that's a defect
in the core system they're using not a paradigm in its own right so what we've done in FONA
is with FQL unify the document relational model in a very coherent way and then augment it
with temporality in graph querying in particular the kind of pointer traversal that you would
do in a graph data model where you don't have to have enforced foreign keys and can just
link documents willy-nilly to each other that interacts very well with the rest of FONA
standard library so that in the same application in the same query on the same data set you
can take advantage of the benefits of these models without having to think like am I
in graph land now am I in relational land now is my data even in the right data system
to run the query I want to run and I think our graph QL interface and graph QL despite
the name is not really a graph API it's a document API expresses the power of the core
FONA semantics in that it offers a view into the same data with a different syntax so
that you can keep the semantics in the data fully shared in access to how you please from
the application side so before we dive into writing queries in these different models
I want to talk a little bit more about what it means that you have unified the document
model and the relational model because to me those things sound irreconcilable how did
you unify this what does that even mean every document has an implied schema you have a
local graph structure you have keys which are nested that have values they may have
multi-value attributes like arrays they you know could have objects which themselves
nest even further and so on and so on so what we do is rather than enforcing the schema
up from on data ingest we rely on the presence of this implied schema to introspect the documents
and understand what the relations are supposed to be so like when you compose an index and
a index and FONA is actually more like a relational view it's an explicit materialization
of data from documents it can even compose data from multiple collections of documents
it can transform it it can do very view like things to express a different dimensionality
a different transformation a different composition of the data in your data set you know that
doesn't say what is the schema you know for this collection it says well I'm going to
assume that the keys the values the paths expressed in the view may exist in the documents
and as we traverse them you know if they're there I'll include them and if they're not
the document will be elided from the output of the view it gives you the ability to you
know basically take the core relational properties joins foreign keys transactions normalization
and apply them to what may not be a fully normalized data model it could be a completely
denormalized document but the relational model fundamentally can be fit into the document
record paradigm and documents can be fit into the relational query in paradigm those things
aren't intrinsically opposed yeah that makes sense it's really interesting so how does that
reflect into queries then do you have a relational accent of fql and a document based version
of the query language or can this be mixed with a one query no it's all one language
with shared semantics you know where there are differences is in the alternate interfaces
to the database so fql is kind of the central point the point that expresses all possible
semantics that fun supports you interact with fql with the application drivers that give
you a ORM like experience they give you a DSL you're not writing fql syntax but you can
also interact with the database with graph ql syntax you can use graph ql clients to access
your data through the semantics of fauna that graph ql also overlaps with or what we support
and what we want to maintain like we're not trying to be a standards compliant comprehensive
graph ql client and let you do for example service composition which is one of the big
value props of graph ql letting you compose different data sources into a single query
well we're just one data source and we're not we're not the right place for you to be
calling out to other services you may own but if that's the interface you're familiar
with then you can access fauna in this way and we intend to add other syntaxes in the
future are those the only two languages that you support or are there multiple that you
support at this point so those are the only two right now but we intend to add other languages
to the extent that they fit the fauna semantic model you know we'll implement that part of
the spec but we won't try to be comprehended we won't try to be dropping compatible with
existing other systems that kind of thing and you just briefly mentioned indexing and
I read that you do indexing on the fly what does that mean fauna indexes are similar to
relational indexes and that they're kept transactionally up to date you can add an index at any
time it asynchronously builds in the background once it's ready it's available for querying
you express your queries by explicitly referring to the index which is a current difference
from the relational model in terms of performance management like you know a lot of people are
familiar with hinting in SQL where you build an index you want to make sure it's always
used fauna is effectively and always hinted system which helps us maintain predictable
performance for your queries at scale those indexes once they're asynchronously built
are themselves serializable so when you update your documents the indexes are updated in
synchronous real time such that you don't have any divergence when you query them from
an application side something else that I want to dive into so you could say that you're
not entirely responsible right for how people use your system but what I was thinking about
when you were describing that you both support the relational model and also the document
model the schema less model how do you force developers to work properly maybe I'm a bad
developer but I think I would worry that what would happen to me is I think oh I don't need
a schema I put a bit of data here and I put a bit of data there a benefit of the traditional
relational model is also that you are forced to think of what is the data type of this
column and what is the name of this column and how does this collection go together which
sometimes slows you down but also forces you to work in a systematic way into thinking
in a systematic way like sometimes you get a database error because it says oh but this
column is of the data type integer and then you're like oh yeah yeah that's true I put
that constraint there for a reason do you know how do you people use fauna how do they balance
this freedom our goal is to maximize flexibility and let you basically opt in over time to
additional constraints as your application matures so when you're starting out you should
totally just put some data here and there and mess around and see how it goes and try
to get it working I think it's unrealistic to envision sitting down you know with a blank
editor window and saying I know what all my data models are going to be let me write
them out like you can do that once the application is mature but especially now applications are
never mature they're always changing so you always need the ability to both enforce and
maintain the invariance which you've decided are for the time being truly invariant and
still have the flexibility to augment and experiment with new aspects of the application
of the product you're building so right now the best way to kind of enforce those invariance
is through indexes which let you like I said you know transform existing documents skip
over ones that don't match expose a view which is very consistent and schema and output and
even the values of the records themselves because you can do almost anything within an
index you can call stored procedures and so on so you can really remix your data in unique
and application specific ways and if you always query through those indexes you'll only see
the consistent view of the data also the functions the stored procedures can do things like check
types and do casting and that kind of thing and air out if something doesn't match so you
can use those to walk down your rights and reads two patterns which themselves will enforce
you know proper data ingest and proper data egress we do intend to add optional typing
in the future which will let you basically do what the stored procedures do in a declarative
way instead of a procedural way and say you know I do have a column that I know what its
type is I know what its default is I want to enforce that fail or populate a specific
value for rights that don't match and so on and that will also let us get closer to the
ability to take a relational schema relational database and directly import it into fauna
with no ETL no transformation from that columnar model.
Yeah yeah was this going to say when you were describing your design philosophy that it
reminded me a bit of gradual typing in Dart where indeed you can start entirely free form
and then as your application matures you add more structure and it seems that this is something
that really fits with what you're saying as well the functions the stored procedures help
you to retrofit the structure into the unstructured data that you might have started off with right?
They do but you know they're procedural they're not type declaration so fauna is dynamically
typed but it is typed like it's not tickle right you're not just you know assuming a type
when you go to execute a function you're actually doing type checks and we can expose those types
to the developers so they can declare them and enforce them and add code to you know make
decisions when the type doesn't match and so on and merge the document model with the
dynamic typing better with the relational columnar model with static typing and let you mix and
match even in the same collection which is equivalent to a table even in the same collection or document
some enforce types some implied or derived types and some aspects of the document which aren't
typed at all. So I want to circle back to something I said all the way in the beginning of the episode
because when I was saying like what I read about fauna on your website something that you also said
was developer friendly that we haven't talked about but I think we talked about it in a sense
because I heard many things that I think are very developer friendly like the flexibility and the
possibility to switch between these different forms of data basing but I do want to ask you as well
like why do you say fauna is developer friendly can you give an example of a feature that you
specifically say well this is what we put in to make the lives of developers to make the lives of
our users easier. A good example of that is the authentication and identity system if you're
coming from an RDBMS or even something like Mongo or Cassandra you're used to administrative
security control only can somebody add and drop tables which tables they can query and so on.
We took a look at that and so we said you know this isn't how people are building applications
they're not building them for table by table access they don't care which SQL statements
the user can run they care what data they can access and also they want users to be able to
access data securely from anywhere not just from a trusted application tier and so on.
We borrowed an authentication model native to the web and support row level token based what we
call attribute based access control which lets you very granularly declare what parts of a record
which records in which collections and which scopes can be accessible to specific identities
both at the user level and the administrative level and that's also integrated with third-party
authentication providers like Gauss zero which you know let you then bring the database sort of
into the web service topology you're already using and not have it be its own island where
the application has to interpret web security policies and apply them usually without relying
on the database security model at all and force them in application code because there's such an
impedance mismatch there so like fun is a very complex system and it may sound scary from this
conversation to interact with it but all that complexity is there to prevent you from ever
having to think about it so that from your perspective you're writing clear queries in your native
application language in a DSL that makes sense that has access to the entire ecosystem of
integrated services you would expect to be there and you just don't have to care what happens beyond
that query boundary but you can rely on enforcement of the security policies pervasive through the
data tier you don't have to worry if somebody writing a mobile app implemented the same
security checks that the web app implements and all that kind of usual stuff you don't have to
worry about and sequel injection because the system is type safe and doesn't rely on string
interpolation and that kind of thing so we're really trying to basically push the hurdles to
building a secure scalable performance application down to the point that you don't think about them
anymore yeah so really your interpretation of developer friendly is that you do all the work
all the hard work is on the side of fauna and for the user it seems like a simple database system that
like we would be used to before there was distributed systems but you do enable the user to use to
benefit from the power of a distributed system right we want to give you the upside and not
the downside and i mean a lot of the downside is operational and administrative in particular
we've done a lot of work to eliminate that completely fauna is a zero operation system and that's not
to say that we don't endeavor to offer operational transparency like you need to know what's happening
in your data but you shouldn't have to be responsible for doing things in the database just to keep it
running yeah i think that concludes everything i wanted to ask you today is there anything that
we missed anything you would like to share about fauna that i haven't asked you i would just encourage
you we have drivers for almost all popular application languages you know try it out you can sign up
for free you don't need a credit card your database is instantly available try it out and let us know
what you think and where would we try it out what is the link that we can put in the show notes fauna
dot com f-a-u-n-a we'll definitely put that in the show notes is there any other place that we can
follow you are you on twitter do you have a blog that we can read i am on twitter my twitter handle
is evan just e-v-a-n oh that's easy there's a benefit of being an early employee yeah yeah it is i'm
also evan on github though so we'll put that in the show notes as well so thank you very much for
being in the show with us today thanks so much for having me keeping your teams on top of the latest
tech developments is a monumental challenge helping them get answers to urgent problems they face daily
is even harder that's why 66 percent of all fortune 100 companies count on a riley online learning
at a rally your teams will get live courses tons of resources interactive scenarios and
sandboxes and fast answers to their pressing questions see what a rally online learning can
do for your teams visit a rally dot com for a demo thanks for listening to se radio an educational
program brought to you by i-tripley software magazine for more about the podcast including other
episodes visit our website at se-radio dot net to provide feedback you can comment on each
episode on the website or reach us on linkedin facebook twitter or through our slack channel
at se radio dot slack dot com you can also email us at team at se-radio dot net
this and all other episodes of se radio is licensed under creative commons license 2.5
thanks for listening
