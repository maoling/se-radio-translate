**Link** https://www.se-radio.net/2019/12/episode-393-jay-kreps-on-enterprise-integration-architecture-with-a-kafka-event-log/


This is Software Engineering Radio, the podcast for professional developers on the web at se-radio.net
Se-radio is brought to you by the IEEE Computer Society, the IEEE Software magazine online at computer.org
software
For software engineering radio, this is Robert Blumen. I have with me today Jay Crapz. Jay is the co-founder and CEO of Confluent
prior to founding Confluent, he was the lead architect for data infrastructure at LinkedIn.
Jay is the author of a book on event data and stream processing and has appeared as a guest on se-radio episode
162 project Voldemort.
Jay is the author of a LinkedIn engineering blog post called The Log. What every software engineer should know about real-time
data is unifying abstraction and we will be talking about that blog post today. Jay, welcome back to software engineering radio.
Thanks so much for having me. Anything else you'd like to tell the listeners before we start the actual topic?
No, I think that's a great introduction. We're gonna work up to the idea in your blog post about an enterprise data integration
architecture organized around a log. I want to establish some building blocks first.
Let's talk about when we have a large enterprise with many systems. If you looked at that enterprise from
30,000 foot view, what does it look like?
It usually looks like a big mess. I mean at least you know in that blog post and a lot of the work
we've done at Confluent. We usually kind of draw this spaghetti mess of interconnection between
applications and data systems with you know ETL processes that kind of shunt data around in real-time
messaging and point-to-point kind of RPC communication.
Usually most people look at that big kind of spaghetti mess and they say yeah, that's right
but it's about a thousand times more complicated than that in reality. You get some idea of sizing.
So how many strands of spaghetti and how many meatballs in a big organization talking tens, hundreds, thousands?
Yeah, I mean each company is different, right?
But I think one of the key points is in some sense if you're connecting everything to every other thing,
then you know if you're going to have n things, you're going to have like n squared connections between the things in that kind of fully connected world.
And of course no organization is fully connected but they're always striving towards it.
You would always hear oh you know this needs to be integrated with that and so on and
and you know I think that kind of scalability problem is maybe at the root of a lot of the challenges that
that haven't really been solved in the prior generations of technology in this space where they were,
you know they're kind of helping you integrate thing A with thing B but you know as you do that and as you go down this path,
it's not really scalable you know across the whole company.
When you're talking about thing A needs to integrate with thing B, what do you mean by that?
You know it could be all kinds of things like you see all kinds of these kind of data copy processes
where data is getting pulled out of a relational database and put into some other
relational database or data store.
You see integration between applications, you know that could be as simple as this service calls that service
or you know it could be something you know more complex than that.
You see you know a whole set of kind of messaging systems that message cues that send off messages
and are trying to support more complicated patterns of you know routing and interconnection.
You see essentially all these kind of you know somewhat ad hoc data integrations or pipelines and technology to support it.
But if you step back and you look at a company and you say hey do we have access to what's happening in the company
in some kind of standard way that anything can plug into?
Usually the answer is you know no or not really or only partially or it's relatively complicated to get at what you want.
You use a term data integration in your blog post. Do you mean something in particular by data integration that's more specific than integration in general?
No I really just meant plugging things together. You know how do applications interconnect, how do data systems interconnect.
You know I think maybe if I step back the you know if you think about the history of software engineering and software development
and a lot of these software platforms. The problem that was focused on for a lot of history was how to build one application.
You know what's the database to support one application? What are the programming frameworks? You know what do we need to run one app?
And I think a lot of the interesting things happening in the world now are when you kind of zoom out.
Now people are like hey we don't have one application and that application doesn't exist in a vacuum and it has to exist as part of a whole kind of interconnected company.
Like in some sense we're not building one app and software. We're building like a whole company and software that needs to somehow make sense.
And when something happens there's many many things that need to to react.
Like one example I often give to make this concrete is if you think about you know a big retailer.
You know a lot of people can kind of understand retail because you understand stores and buying things.
And so it's a you know it's kind of an intuitive domain.
And you think about well you know how many different systems need to update or integrate the information if something sells.
If they sell a product to a customer.
And you know it's clear there's analytic systems and there's all kinds of coupon and discounted logic and there's pricing logic and there's a whole backend logistics for managing warehouses.
And there's you know reporting systems and there's things that look for you know fraud and bad transactions.
There's all these different systems that all have to somehow take into account that something's sold in different ways.
And so that process of kind of taking you know individual chunks of software and you know plugging it all together.
That's kind of what I mean by data integration. I don't know if it's a perfect term.
I don't just mean you know kind of copying data from one place to the other.
Is there any way to quantify in terms of either number of updates or volumes of data.
How much data are we talking about that needs to move around between applications.
Yeah you know again I think this kind of depends on the scale of the company.
And then how much of that company is really kind of modeled in software.
And so when you look at a lot of these big tech companies and you look at their usage of Kafka they actually have you know billions or hundreds of billions or even trillions of these events that are flowing through the system each day.
And so that's obviously massive scale and I think it took years and then those companies are more or less fully modeled in software.
And if you look at other companies they're often not all in software.
You know they have many things happening in the real world that have only kind of disconnected non integrated you know support in software and so they might have much less.
But if you think about how those companies would probably exist 10 years from now I do think companies are getting more and more to where there's a you know there's a complete supporting software infrastructure even for the things happening in the real world.
You know there's there's instrumentation there's knowledge of that there's support for that even the things people are doing there's associated apps and so on that help kind of track and monitor and that all needs to come together into one company that that actually works you know as as an integrated whole.
Before you went to the solution we'll be talking about based on invent log you tried other approaches to integration at LinkedIn.
What else was tried and why did it not work.
Yeah I mean the the context was and and this post is actually you know relatively old right so I think I wrote this you know maybe at the end of 2013 or beginning of 2014 right around that time frame and.
You know at that time LinkedIn had been through this journey of kind of rebuilding all the internal infrastructure for scale and there wasn't much that was available off the shelf and so when we talked before it was about a.
Key value store that we built internally and so we had all these different data systems and we were trying to be really smart about the the use of data and how that would you know feedback into the applications and customer experience and.
You know I felt like what what we were doing was was trying to solve that kind of n squared integration problem of how does this application trigger off this thing and how does this data flow into this analytic system and how do we monitor this and I feel like we were trying to solve it.
You know one one integration out of time and so we were kind of building a big team to do that but it was we were never going to finish you know we weren't we weren't even scratching the surface.
I think at one point we were like hey what do we really if we think about what happens in the business and we try to say what do we really have coverage for that you could tap into.
You know I think we said maybe we have about 10% of of everything and we were working very hard at that point so we were like okay that so that was the that was the context.
And then we thought well you know what would a solution look like and it's it's actually not hard to imagine you say well whatever happens you know somehow that data that like you know stream of things that are happening should be available and any other system should be able to tap into it.
And they should be able to react or respond or calculate something or or do something with it.
And then and then the you know the devil is of course in the details so when you say everything that could be actually a lot of data and when you say any system of course the needs of.
Different systems could be very different if you have some offline data warehouse it may read things only once a day so it may take updates very slowly if you have you know some kind of very real time application that feeds back into the customer experience it may have pretty low latency requirements and so then we were like well.
You know is this really possible can you you know can you.
You create something that actually fulfills all those demands and and do we need to like our assumption was there must be something out there in the world that solved this problem and we were just.
You know to dumb like we didn't know about it and so we we kind of went through this journey of trying these different off the shelf messaging systems are theory was maybe we could kind of.
Stitch them together in some way you know and you see this in the database world will people will take.
You know lots of my sequel databases and try and stitch it together into some kind of charted something rather and.
You know we thought we maybe we could do the same thing like will will get some of these off the shelf messaging layers and we'll try and.
You know stitch it together into some platform for this and you know it was it was kind of remarkably unsuccessful and so then we came back out and we thought well look you know we have.
Internal capability to build stuff from scratch and and we had some ideas that came out of the database world for what you really needed to do and that was kind of what led us to.
You know start working on Kafka.
We'll go more in depth into that I want to break this down into stepping stones these bits of information that are getting shared let's call them events.
What is an event give some examples.
Yeah you know it's it's one of those things that's so simple it's almost hard to explain right so an event is just something that happens and you know in that retail example it could be a sale so okay we sold something to somebody that's an event.
The you know the representation of that in a computer well it could be some snippet of Jason that says you know what was sold and where was it sold and who was it sold to and how much to cost and.
You know the data about what happened but it's you know obviously the event isn't really tied to that Jason you could represent it in some other format.
And so so if you think about it it's a type of data an event is just data and it's immutable right you know something that happened it doesn't happen.
And it happens at a particular point in time.
And so it's just a it's just a fact actually it's just something that occurred.
And you could imagine representing all kinds of different things that happen in a business that way you know I talked about sales but you know just as easily you could represent you know some change to some state about a user so you know LinkedIn maybe somebody comes and updates their user profile.
You could say okay you know my profile is now this you know where this is the contents of all the fields that would be an event like a profile update event.
And you know in if you're if you're making a ride sharing app you could say okay you know my car is currently here right that's an event it's just this you know it is true that at this point in time this car was at this point this position and it was this driver
and you know this is the ride it was part of and so on so so that's all it is it's a very different way of thinking about data you know typically we've thought about databases which are full of you know the current state of the world
right and so we think about state we think about okay you know your database tables it might have the set of products or the you know set of customers or whatever and they you know you could get updated but it's really modeling the current state of the world
and events are about you know what's happening in the world what occurred and so that could be something that's very high level about the business that occurred it could be something very low level at LinkedIn at some point you know they were monitoring even very low level things like traffic through network switches
and you know so on as events and you'll you'll see this actually if you kind of search around in you know software engineering channels for events you'll find it's actually springs up all over the place so in the world of application monitoring and observability
there's a whole movement towards events it's very much about low level mechanics of watching you know software do its job you know if you look at front end development there's a whole movement towards events which is about capturing what happening and you know having UIs that react and respond
and then of course you know in the world of microservices and applicant you know interaction between applications there's a movement towards events and so you know it's actually kind of you know coming out of the woodwork in all these different areas
and they're all talking about the same thing events are just things that happen in the world and you know it's a very different way of thinking about data but ultimately kind of a very simple concept once you understand that you're like oh you know what's the big deal
they may all be conceptually the same but if you have events which involve something like a customer bought something in the old days you'd store that in your relational database then other data like monitoring your security system detected and anomaly
maybe these things are transient low value you don't want to go to so much expense to capture them maybe you don't care if you lose them so are all these events all treated the same way as if they were all equally valuable or is there some ability to tier them
based on how much cost you're willing to incur in relation to the business value of that class of things? Yeah I mean I think the first mental jump is just understanding that there's such a thing as events and then we can think about hey how should we build the right infrastructure for it
you know I think you're a hundred percent right that it's not like this concept didn't exist before I think it just wasn't well supported and so you would see all these cases where people are trying to use a relational database for streams of events
and you know maybe the most prominent example of that is for people who know the world of data warehousing they have such a thing as fact tables and they have dimension tables
and dimension tables are kind of what you'd expect it's your you know your set of products and the set of geographies and so on that you operate in but the fact tables are actually you know the set of things that happened
you know and it's always ordered by time and it's immutable and it's very much like an event you can't subscribe to it you can't get it in real time it just gets dumped in at the end of the day
but but it's kind of an interesting thing in order to analyze a business and try and say okay what what happened in the business yesterday you basically have to reinvent this you know event concept
in a relational database and so so yeah it's it's nothing new you know I think the new thing we added was you know concretely what would the data structure for events be if you if you said hey this is an important thing
you know how should we represent it in infrastructure and is it possible to do that in a way that actually makes it easy to build around because anybody who's tried to build you know real time event streams where you know you trigger off of it and process it in real time
on top of a relational databases you know kind of come to the same conclusion that this is you know really quite hard you're kind of you're abusing a piece of infrastructure that was built for one use case for something else
you know that's that's a common thing to do but but it just gets harder and harder and harder to do well and so so that was kind of our idea was yeah you know you can't solve all problems with one piece of infrastructure but if you take this idea of
events seriously how far can it go you know what can you make something that's really built you know to do that from from the ground up
and if so hopefully it can go from you know very low latency to you know kind of long longer term you know slower storage and processing and be a basis for you know all these applications and so
we are going to get to how that infrastructure works but one more concept I want to cover if we're going to integrate many different events through data and enterprise do you need all of the apps to agree on a common schema for the events
yeah you do you know in a simple way so you know let's say that I am I'm trying to you know keep track of all the sales occurring and there could be many ways that we sell things like maybe we sell them online and maybe we sell them you know in brick and mortar stores
right so all of those people it will be much more convenient if they can all agree you know this is how we represent a sale and then we can send it all to the same event stream
and then likewise for the people who are trying to react to that or respond or tap into that or take it and store it somewhere you know if they haven't agreed with the people upstream about what the bites mean
that they're not going to be able to make any sense of it so they have to agree on what you know what the representation of a sale is and then we have to agree like how can it change over time
you know maybe we'd all love it to be the case that we just say oh this is what a sale is it can never change that's a definition but you know the reality is we work in businesses that are evolving all the time
and so the attributes of a sale what can we capture about a product how can we sell it the different things that we would record are going to change as the uses change
and so we have to agree on that how that can evolve so that things can you know software upstream can be updated and software downstream can be updated but also we don't require the whole organization to stop
and get onto the new thing with you know some downtime so there has to be some agreement around that and so in a sense it's actually I think more important than even the schemas in relational databases because typically a relational database is a kind of implementation detail of one application
so the application just kind of needs to agree on the schema with itself and even there the schema can be helpful to make sure that you're checking constraints on your data and so on but once you get to multiple applications all sharing
then it becomes that much more important that we agree on the structure of data otherwise you know you could have somebody broadcasting you know malformed things out to the rest of the world that they can't react to or handle and then you know chaos chaos
and sees what are some of those schema definition formats or languages that can be used?
Yeah at least for Kafka we've tried not to reinvent the wheel there you know there there's a bunch of these serialization layers that are common also out in the world of you know building
RPC services or restful services so you know there's things like protocol buffers and Avro and JSON and they all have different proponents of different kinds you know for Kafka and for Confluent we've tried to make you know our stuff work with as many of these as we can
so we provide out of the box support for something called Avro which is kind of a binary format with a you know JSON schema that has good support for you know the evolution of data but we're not religious about it
people tend to be the most religious about these you know kind of formats and standards and our job is just to build tools that people work with
formats have in common that they are supported by many different languages and that is probably essential because every app in the big company is not going to be written in the same language
Yeah that's a really important part of this is you want to have you know some definition that's agreed on that that any you know anybody can plug into and so you know the kind of things that are baked into the programming languages like Java serialization
you know whatever the the old school pickling and Python I mean it was really inappropriate you know it would work if you were just sending data to yourself with the same version of the software doing the reading and writing but this is something where
you know you're going to have many different applications potentially in the future or even in the present in different languages and those applications will all change independently so you can't assume a single version of everything you can't assume a single programming
language you want to have as much independence in that data as you can we're moving in these stepping stones toward this concept of the Kafka event log and your blog post you introduce the idea of an event log which is borrowed from the database world
and you're extending it to the enterprise let's start with how logs work in single node databases
yeah that's that's a good question so you know our thought process was we're trying to solve this problem of when you know when something happens out in the world how does it get integrated into all these different you know systems and applications
so if you know in my retail example something is sold and all these downstream systems need to kind of react on their their time frame and you know we were looking for where are examples where this happens
and you know the the people who are thinking about this were a set of database engineers and so one of the ideas that is kind of under the covers you know it's basically an implementation detail of databases but it it often leaks out is this concept of a commit log
and that's used to keep all the different parts of the database in sync and in a distributed database it's often used to keep these different independent nodes in sync and so the idea is you just take every update that you want to make
and that update you know maybe you're saying hey I want to update my LinkedIn profile to this new value that that might require changing multiple things right so that row has to change but different database indexes may need to update
there may be things that need to you know trigger off of that if the database supports triggers all kinds of things may happen in reaction to that and so the idea is very simple these you know commit logs are really just a file where you just append you know the change
and then the the you know taking that change and integrating it into all your data structures happens only once it's been committed and so the idea in databases if you if you want to have some notion of correctness and consistency
and fault tolerances you know in some sense the that log of committed data that is the database that's the real truth of what happened and all the tables and indexes are just representations that you could actually recreate off of that log if if you had to
right there they're kind of like a a cache of the information that's been structured for a particular type of query and so so that's the kind of basic idea of you know how a commit log is used it often becomes very important as an external
factor because people often tap into that commit log so in my sequel people use the kind of bin log to get the sequence of changes out of the database as well and so it becomes almost a you know a public API but it's kind of an accidental
public API I think in most databases where it's an internal detail that ends up getting exposed and the things that we knew from database engineering were these commit logs are the fastest part of the database so you can write
you know a series of things to disk in a in a linear order extremely fast and we also knew from distributed systems that there was a way to do this at scale across many nodes that there was you know consensus protocols and ways of committing
data that would extend beyond a single machine and and so we thought that could actually be applied to you know the enterprise at large you could actually think about your kind of whole company instead of data systems and applications
as kind of like one big data store and you would you know that you would commit to some log and then it would be applied elsewhere.
I formulated this question in terms of a single node database and now you've talked about how it can be generalized to a distributed system by which you might say multi node Cassandra cluster wherever you know it is running Cassandra.
Now it sounds like the next step was you generalize that a little bit further to the enterprise where you have these heterogeneous systems but you're still thinking about the log in the same way as if the entire enterprise was one big database.
Did I get that right.
Yeah I think that's exactly right so in a traditional single node database the log is usually just you know on the file system and then it would be used to keep replicas of that database in sync so they would just take each change that was applied on the
you know primary or master node and they would apply it on the kind of secondary follower nodes and then other applications might sink off that so you know that was the kind of typical architecture of a single node database
and then in distributed databases people came up with you know techniques for maintaining this type of this type of distributed log across multiple machines and this is figured pretty prominently in a lot of the architectures of distributed databases so things like
Amazon's Aurora Yahoo early on had a paper on the system called key nuts that worked this way I mean that was the name in the paper anyway and so it clearly generalizes to a distributed system where the individual nodes in that distributed database all work
asynchronously you know they don't all work in lockstep and so it wasn't a huge jump for us to think about like well what if those nodes are doing different things with the event.
So that would work as well. Two very important properties of these logs which you've mentioned and I'd like you to expand on why are these properties important they are immutable and append only.
You know there's even a third property which is you know kind of maybe obvious just from my description which is they're ordered and I think you actually need all those things so you need it to be the case that when something happens.
So new things can happen but that past history doesn't change in sneaky ways out from under you and that's that's really important in a database the database needs to be able to take each of these changes integrated into its tables.
If you know there's a random change is happening in the past of history where you're kind of rewriting history it's very hard to integrate that in a correct way especially if you imagine the database crashing at some instant in time like what.
What state are we really in you know or have we even made a full atomic change and so you know this idea really originally came out of the idea of how do we get you know atomicity and how do we have transactional commits and how do we handle correctness
in a database.
But it's actually arguably even more important when you get into a big diverse organization where you have different parts of the organization you know have different teams that you know talk to each other less than they should and are building
independently you know it needs to be the case that when something occurs that there's a really principled way that that propagates out to all the other systems in the organization.
We're at this point we started talking about the problem of integration the complexity of point to point integrations you simplify that you put the log at the center of the architecture.
I think we have all the pieces now where you can answer the question what what does the enterprise architecture that you're talking about look like.
Yeah well the idea is that it's based around events and that you can think of you know events representing both the kind of activity of the company but also the state changes so if I update my profile that's an event.
And so you could really think about kind of all the different types of data in an enterprise as events and then the question is well what's the what's the data structure for an event what's the abstraction that you know you can give to a software engineer that they can work off of that
and we felt this kind of commit log was really the right way to do that.
And so the idea is that you you know you commit to some kind of log and you say okay this occurred and this occurred and this occurred and again many people writing to that log.
And then you can have all all these independent readers who read off that log and apply the change and they can do that kind of on their own timetable so slower systems can do it slowly real time systems can do it you know as quickly as they can.
And they'll all get the same sequence of things they'll all get them in the same order and they can get them kind of on the timeframe that they want.
But but there has to be strong guarantees around this right if if data can somehow be lost or oh if a system goes down for a small period of time it can't be that data you know you're missing updates and all the everything gets out of sync and you have to go do some.
You know scavenger hunt to figure out what happened to your data has to be guaranteed that when something happens it will propagate out to everything that's downstream.
And we felt like that you know that that was actually really clarifying if you kind of stepped back and you just thought this way you think okay look the things that happened in a company or events.
And we need infrastructure that allows us to propagate events that kind of a you know global company scale.
Then and if we think hey the way you would build that is this kind of commit log that it really is just this distributed systems problem of how can we build that how can we make it accessible.
And then how you know what are the programming abstractions to make that really easy which is a lot of what we've done.
You know since that blog post was written was really and it confluence really focus on the kind of up the stack tools that allow you know people to build against this in a really simple way.
We did on entire show on Kafka that was 215 was about four years ago.
Listeners you want to get some introduction to a good listen to that show. So I don't want to do a whole basic intro to coffee here but how is the Kafka platform changed in the last four to five years since we last covered it.
Yeah it's it's changed in a couple ways so that the evolution of this was you know we had a pretty big vision around you know events and commit logs and how it could really be kind of the central nervous system for a company but we can't build it all at once.
And so the you know the first version that we built was very scalable but it didn't provide you know kind of replicated fault tolerance guarantees like a database would write and so it was applicable for these kind of high volume activity data like messaging
systems might have but it wasn't really applicable for kind of you know core payments transactions can't lose data and then you know then we did work to add replication to that so then we had something really right about right right right around
the time that that blog post was written that was both fault tolerant and scalable but it was still relatively low level we were giving you this kind of commit log you could go do with it as you please so it was kind of a you know a very primitive abstraction
powerful but it was up to you to figure out how to use it and of course that's that's a lot of work to take advantage of so a lot of Silicon Valley companies that just had very large engineering teams would you know build around Kafka just build it into all their different you know applications
and data systems and they would get to really large scale with it but you know kind of outside of that world it was very hard for people to kind of take advantage of the idea.
So then you know at conflict the you know kind of big investments we've made is you know kind of continuing to make that foundation better and better but also adding a layer that would manage kind of pre built off the shelf connectors so you could just plug in
you know databases if you have MySQL there's no reason you should have to build the you know 10,000 and first MySQL integration you should just be able to get the connector and use it
and then build stream processing capabilities that would allow you to build applications that react to these event streams and so we found that people were just kind of building the same stuff over and over again as they were trying to you know work
against this and we realized that in some sense you know if you think about this kind of commit log or event stream that you know with a that's a foundational abstraction but it's pretty low level so it's kind of like you know building directly on top of that can be a little bit like building
an application directly on top of the file system in terms of its data access and you know a lot of the work in recent years has been moving up the stack and trying to make that more approachable and trying to give you know a much much more like a kind of almost like a relational database
way of working with event streams and that's been this area of stream processing and things like K sequel that allow you to just apply sequel on top of data streams and materialize new data structures and data formats and things that you can act upon
some links in the show notes where people can read about some of these features.
What are your seasons architect are aspiring to become one the O'Reilly software architecture conference is designed to help you go next level.
Save 20% on your pass with code S E R 20. Register at O'Reilly essay con.com slash se radio.
One other one explore is if you're now putting this if you're making the event log the real core of your architecture where it's becoming the system of record in order to adopt it you need to ensure that it has the same degree of disaster recovery
and survivability as the enterprise database used to be which some of it means features built into the system like the ability to create backups while it's running.
But then other investments you need to make outside of the event log which is what you do with the backups where do you put them how often do you have good operational procedures around recovering them.
Have Kafka evolved a feature set that makes it worthy of being a system of record.
Yeah it's gotten better and better in that area I think what you say is exactly right there's a set of features and then also a set of operational practices that are really important for anything that's kind of a critical data store and so it's important for people who are adopting
Kafka to know you know are they using it just for transient data or are they using it as something you can kind of fall back on or rely upon.
And if the latter then definitely the degree of operational excellence has to be a lot higher.
And so it has gotten you know better and better at storing data better at scaling the number of partitions backing up something like a log is actually really easy because you can just subscribe to it and copy it elsewhere so in fact the backup features and databases are always built on top of the commit log and the database or typically are.
So that parts you know relatively relatively straightforward to do but that is a critical thing for people who are trying to rely on this you know commit log as something that you can actually restore off of without going back to source systems
and trying to reconstruct it.
There's some interesting possibilities with this architecture I would like to get into one would be if we have all our data and let's say a relational database and realize there's this new type of database called a graph database it's a really good fit to our use case.
How do we incorporate these other types of databases into the enterprise.
Yeah that's that's a great point that you raise so one of the nice things about Kafka is it's kind of independent of any of the different representations or query capabilities you might have for data.
And so if you have data feeds that are in Kafka you can easily load them into different databases and get kind of almost like a materialized view of that data.
And so you can think of you know data warehousing and analytics layers that way you could think of a search search index that way you could think of a graph data store that way.
And this is one of the core use cases people adopt something like Kafka for they typically have some kind of relational database which is serving you know live user experience type UI.
And that's actually creating the kind of core transactional data stream so maybe your your set of users. But then there's other systems like search or analytics that need a representation of that data and so Kafka kind of access this you know log of those changes that can feed out to all the system
So oftentimes a data warehouse would feed slower some kind of real time analytics layer might be faster a search index. Usually you want to be in sync you know if it's if it's meant to be alive kind of search experience, something like a graph database it's providing different types of queries
and so you know if you're a computer you can feed off of that as well. And you know that's that's definitely a core motivation for this architectures actually as the set of databases, but also even you know in the cloud a lot of the different SAS services and so on that you could kind of tap into
that set of things has grown, you know the desire to have some kind of consistent way of broadcasting that out gets you know more and more and we're seeing more of that words. You know it's not just new databases like people have integrations with things like Salesforce and you know with with rest services and with all kinds of other things that they want to react when something happens
In the article you express the idea that all of these different types of databases are just specialized indexes on the event log. What do you mean by that?
What I mean is you know in some sense if you peer inside of a lot of the distributed databases they are kind of a you know distributed log and then you know some kind of materialization of the data that's there for serving queries. And by having this log as a service and so
on right you can run lots of those systems in the way we we just described and you can you could actually compute new things off the data stream. And so you know it may be the thing that you want in your search index isn't just the simple record that you have in the database.
It may actually involve stitching together lots of bits of information that you have about the customer and then indexing that. And so you may want to do processing on top of that data stream.
And so you know that that's very much the idea and then you can think of that you know that search cluster is you know it's effectively just a kind of materialized view of your data.
You know it's just a representation that's feeding off of that commit log in database terms you can think of it as a you know in databases they have tables and they have materialized views that are calculated off the tables you can think of it very much like that it's just that
this happens to be in a separate system. That last point I want to drill a little bit down into this up to now we've been talking about a lot of examples of an event is some new information that came into the system from the outside world like
somebody customer bought something you could have stock price feeds coming in from the market a vendor updated their inventory.
These one of your systems to get that information into your enterprise. Now you're talking about you could have events that subscribe to your eternal event stream process the events and let's say classify this customer is now a higher tier of customer
or something you've been able to extract from data that came in by processing and then you publish that back out onto the event log and now other systems can react to that. Yeah that's that's exactly right so like a really simple example is in virtually any big company.
You have a bunch of bits of information about your customers right so you have maybe some primary account record but you have interactions with customer service and you have you know settings and so on and you know you have payment information and so usually one project that's really common in companies is
creating some kind of unified view of that that's used in a lot of different applications that's maybe used for analytics and so you can think of it as really just you need to perform a kind of continuous join across these different systems which have different databases
and data stores and so how can you do that how can you do a kind of continuous join across a bunch of different systems that aren't even using the same underlying data storage.
And you know I think one answer to that is yeah you treat the updates in each of those systems as a kind of stream of events.
You use stream processing to join that together and create this stream of kind of unified customer profiles and then that could be materialized in a key value store could be sent off to a data warehouse for analytics but that that feed is basically published back as you say as a log in its own right which is consumable and
subcribable. I want to get into a little bit more of the programming aspects to application needs to get its data into the event log.
How does it do that. Yeah there's there's two ways in Kafka right you can you know directly write a record and you know it's actually not hard you to say something like send and you provide a kind of key value pair and that's sent off to some topic that you would specify
and it's appended to the the commit log for that topic and that's kind of like the lowest level of interaction is you know it's it's almost like just saying right in a file system you just write something.
And so that's kind of a push mechanism. Now if you think about existing systems you may have some you know SAS service you use you may have a database those systems don't know about Kafka and so they're not actively going to push data into Kafka.
You may actually want to pull data out of those systems and push it into Kafka and that's where these connectors come in and Kafka supports something called Kafka Connect which is this distributed framework for running these connectors and managing them.
And so that's the other that's the other way to get data in and then you know of course the you know the final version of that is kind of what you described which is you may be taking some of these streams and actually doing something with it and publishing back some
you know derived stream of data back into Kafka using simple application software using one of these stream processing tools like KSQL and you know maybe that's the third way.
All of these are built on the underlying you know producer and consumer layers in Kafka which do the reads and writes.
I was writing an app today. I know my enterprise has Kafka so there's not this question how do I retrofit this app that's writing into Postgres.
So, and I want to make life easy for the programmer.
Every time I do a transaction so this app is has its own database as well let's say that's also Postgres.
Every time I write a transaction to Postgres do I have to now include some logic to write to Kafka or is there a way kind of an aspect oriented style where I could create a cross cutting concern against every database transaction that will say just ship them
off to Kafka. Yeah usually the simple I mean there's many ways to handle this and some of the things that some of the events that come out of applications may have no representation and a database and you may just write them directly.
But often the simplest way to do this is to use the connector infrastructure.
So most databases like Postgres they actually expose the commit log in some way. And so either by you know querying the tables or reading that commit log you can actually take that and publish it out into Kafka's data streams for each of the tables say.
And so that's that's the pattern that's probably most common.
And then there may be additional cleanup you want to do on that data and that can often happen via stream processing where you take the kind of original thing that was in the database and you stitch it together and just take it out.
And then you stitch it together into something that is really what you want to publish out to the organization that's in the right format for consumption.
Done all my domain modeling understand my business abstractions.
And we get the database people involved and they normalize everything into all these tiny little tables.
So I commit this one object it writes rose into six different tables.
Now you're going to look at the commit log with your connector do you have to reverse engineer that back or maybe not reverse engineer that but do almost the inverse operation to transform that back into your event log model.
Yeah, I mean I guess there's different patterns for doing this and it depends somewhat what your goals are.
If you need to have read after write consistency in the database you do typically want to kind of commit to that database first and then have the Kafka log sync off of that.
And you know that could mean either you know having a table that directly represents the event as well as the you know the more normalized representation in the database or it could mean stitching it back together off of the normalized representation.
And you know that probably depends whether you're writing the application from scratch or you're retrofitting something that already exists which of those you would do.
If on the other hand that the database doesn't have to you know have that kind of read after write consistency you can always just you know publish the event stream to Kafka and populate the database off of that.
In the first pattern you talked about then find or stand it it's more like a double right, but the rights all go to the database to take advantage of the acid property of Postgres where you'd have one table and let's say it's called Kafka log and you write something there
and then Kafka could come and pull that out but you don't have the problem of updating two different logs and trying to ensure they both get committed.
Yeah, it depends a bit what you mean by double right I mean we don't we don't usually advocate that people kind of you know write to one and then write to the other because you know obviously if your application fails in the middle, then you know the question is how do you keep those
in sync so it's usually better to write to the database and have Kafka subscribe to the database or write to Kafka and have the database subscribe to that.
If you don't care about correctness and of course you can try and do that double right but it's a bit like those you know keeping caches in sync that way where it never never quite works in the presence of failure and multiple machines and so on.
So usually picking either the database or Kafka and having you know one of those to be in front is a cleaner pattern.
You put a table in Postgres called Kafka events and you write to all your normalized tables and then you write the event out to that table and that's all one commit in Postgres and then Kafka can just look at that one table.
That's right and that would that would simplify the downstream cleanup you would need to do and you know that obviously works only if you're able to go in and change the application if it's something owned by somebody else and you just want to you know subscribe to what's in the database then you
have to stitch it back together in the stream processing side which has gotten easier and easier as the stream processing toolset has gotten better.
So being immutable. How do you deal with if bad events get written to the event log which could either be human error or you have some rogue application that has a bug and it's writing a bunch of bad events.
And now they're immutable. Are you stuck.
Yeah that's that's a great question so you know it is very hard if you have systems all reading off this in real time to handle kind of bad data is you know it's very similar to a you know the situation with the database except you may have many apps that all feed off the same thing.
And so as much as possible you know it is good to actually enforce these schemas so similar to a database right you know if you ship a version of your application that publishes corrupt data and overwrites good data with it.
You're in a bit of a bind you're going to have to go kind of rewind and you know manually truncate and figure out how to get back into a clean state.
So I'm going to go back to Kafka where yeah you can kind of rewind to when it happened but you're actually better off trying to enforce the schema correctly and making sure that valid data is you know is what is published.
And that kind of schema enforcement ends up being really important as is scales in an organization I mean we've worked with a bunch of companies that have done this and the ones that have taken kind of a best effort approach.
And then you know there's just somebody breaking everything down stream all the time because there's so many readers and so many writers that at any given point somebody has a has a bug somewhere.
What about the schema is correct but the values are not correct.
It has to be somehow cleaned up exposed how do you fix it.
It's similar to any contract you might have right so you have the same issue if you're trying to you know update individual micro services and a micro service architecture.
If you have no contract at all that can be very hard even if you have a contract the contract can't really cover everything right so you may say, oh there's some field which is URL fully normalized fully qualified URL or is it just the base of the URL right like
the person upstream and the person downstream has to agree on that and kind of keep that in sync.
If if you get something that's wrong you do have to you know either update the processing logic to handle that or go go back and clean out the log which is going to be a very kind of manual step that's hard to do.
So, I think there's the prepared material I have is there anything else on this theme or from the article or discussing that you wanted to cover that we haven't talked about.
Yeah, I think maybe one of the interesting things that we've done recently that's very closely aligned to that is around ksqlDB which really kind of ties together the ideas.
You know we talked about some of the integration stuff but there was really kind of three points like integration stream processing and the role in data systems.
So, I think it might be fun to talk about that a little bit.
Yeah, let's go ahead.
Why don't you describe what what it does what problem it solves.
Yeah, so really just last week we actually did a release of ksqlDB and this is kind of the evolution of a system called ksql that we've built to allow stream processing in SQL on top of Kafka.
And so the idea was these event streams are new and people may not know how to use them but everybody knows SQL and so if you could just use SQL on top of the event stream.
So, if you could say, hey, take the stream of updates to user profiles and join that on to the stream of activity from users and get both together and be able to count or act on that or transform it all in SQL that would be really cool.
And we've evolved that system and it's really actually I think quite exciting and if you've read this log article, it's even more exciting because it brings together the ideas about stream processing, you know, acting on real time streams of events.
It brings together the ideas on data systems that are built around logs because it turns out that, you know, if we want to join our customer profiles to the customer activity, you need some kind of stored fault tolerant representation of customer profiles.
And so what this system now allows you to do that's pretty cool is you can take these event streams, you can materialize some data set off of that.
You can use that data set to combine with other event streams and you can query it.
And so it's very much kind of a database built for working with event streams.
You know, it's not meant to replace Postgres or Oracle or whatever you would, you know, kind of have as the, you know, the backend for a crud application, but it is meant to solve this problem of, hey, I want to take a bunch of these different data streams.
I want to materialize it into some data set and build an application around that.
It aims to make that as simple as possible.
And so it includes both, you know, SQL for processing streams of data continuously as well as the ability to do lookups off of whatever the result is.
I wonder if that solves the problem we've been talking about where I have application is based on Postgres.
I'm going to commit everything to Postgres and then we have the problem of pulling it back out on the backend.
So say, I want it to be in the log.
Why don't I just write it to the log in the first place?
The only problem then is you don't have the ability to read it from a SQL like interface until a little bit of time is elapsed where it propagates down and gets picked up and added to SQL as a, as a subscribe step.
So can I then put it in the stream and if I need to read it, I can query it from SQL out of the stream now and avoid that consistency delay?
Yeah, I mean, the use case that we're targeting initially is not quite that, right?
So our theory was people aren't going to replace Postgres for their, you know, kind of crud applications or rest services.
And, you know, we're not trying to get them to do that. But if you think about the applications downstream of that of actually using event streams, those are actually relatively difficult to build and they end up being kind of, you're
gluing together some of these connectors and you're gluing together some stream processing that, you know, does something with the events and then you're eventually gluing on some, you know, kind of data store to serve results up to some application.
And we thought, if we could consolidate the set of things downstream, that would really make life much simpler and working with event streams.
And so, you know, a very simple example of that that I described was, you know, that kind of customer 360 where you're joining together data feeds about your customer across seven systems and then you want to be able to look up, hey, what's the current, you know, what's the current unified
profile for this for customer 123. And so you could do that basically just in a couple of queries and I think that kind of materialized derived cash.
I think that's a problem we see everywhere. Like people have these older applications, you know, they're running the business, you have new requirements and you need to build a new set data set that cuts across some of these.
The ability to tap into those and, you know, create a new data set off of it and then issue queries against it to kind of continually materialize a cash view of some part of the business.
That's actually really, really common. And we think really well suited to this. And it's much easier to do it in a correct way than if you're trying to, you know, do some dual right strategy that cuts across multiple applications in some, you know, transient cash, because this is directly fed off of the change log of all of those
systems. So those event streams populated. It's always kept in sync.
I see what I think you're saying now is instead of writing a application to do that, which is high programming complexity you've reduced it to. I can write a sequel query against Kafka and get the answer.
Yeah. Yeah. So, so the, the support that this provides is the ability to manage these connectors. So, so in my example, the first thing I need to do is actually capture the
stream from a number of different databases. And so maybe I would, you know, I now have sequel support to kind of declare there's almost like external tables or external streams and capture that into Kafka.
The next thing I need to do is maybe join and clean up those data feeds and create my unified customer profile. I can do that also with this kind of stream processing sequel.
And then the final thing I need to do is be able to actually query and say, Oh, I need the profile for this customer because I need to put it in my UI. And so I could actually issue those real time queries against that, that data feed.
And so, so bringing those together the ability to, you know, if you think about it in ETL terms, you're kind of extracting streams of data, you're transforming them.
And you're kind of loading them into some, you know, stored representation, and then you're actually doing live queries against them. You know, it's really that whole end to end stack that would have otherwise required, you know, you to glue together specialized systems for each part.
And once you have all the data in Kafka, that becomes the obvious place to do at least some of this processing.
Yeah, that's exactly right. So, so, you know, we felt like the investments with the connectors helped unlock and get a lot of data into Kafka.
And then, you know, the complaint people have is great. I have all these ideas for what I could do off of these streams of events, but it's, you know, in the end, it's pretty hard, right? You have to build, you know, custom Java code or Python code or whatever that does any of these applications.
It gets complicated as you think about fault tolerance and so on. And so having data layer that allows you to do that declaratively in SQL that handles the kind of more complex operations like joining or counting or adding, aggregating.
That's actually really valuable.
That wraps things up, Jay. Where can people find you on the Internet?
Probably the easiest way is on Twitter. I'm Jay Kreps, first name, last name, all together. And, you know, other than that, I, you know, I've got LinkedIn profiles and does another social media thing. So, usually pretty easy to find.
And Confluent. Where can people find that?
Confluent is Confluent.io. And so Confluent produces, you know, a whole platform around Kafka that, you know, is a lot of data.
So, you know, one of the things about Kafka that is, you know, community license, you can just go take it and use it. We produce also an enterprise software product that you can go if you're going to run this in production.
We produce a cloud service. So you can actually, you know, if you're looking at this Kafka stuff and you're like, oh, I just don't want to run one more thing.
You can get it as a service in, you know, any of the major public clouds, Google or AWS or Azure, and you kind of just pay for what you use and you get all this infrastructure as a service.
Jay Krebs, thank you very much for speaking to software engineering radio.
Thanks so much for having me.
For software engineering radio, this has been Robert Blumen. Thank you for listening.
Thanks for listening to SE Radio, an educational program brought to you by IEEE Software Magazine.
For more about the podcast, including other episodes, visit our website at se-radio.net.
To provide feedback, you can comment on each episode on the website or reach us on LinkedIn, Facebook, Twitter, or through our Slack channel at seradio.net.
You can also email us at team at se-radio.net.
This and all other episodes of SE Radio is licensed under Creative Commons license 2.5. Thanks for listening.
