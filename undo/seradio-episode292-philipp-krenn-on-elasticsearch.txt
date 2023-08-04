**Link** https://www.se-radio.net/2017/05/se-radio-episode-292-philipp-krenn-on-elasticsearch/


This is Software Engineering Radio, the podcast for professional developers on the web at
se-radio.net.
SE Radio is brought to you by the IEEE Computer Society, the IEEE Software Magazine.
Online at computer.org slash software.
This episode of SE Radio is brought to you by BrainTree.
If you think that your payment system exists solely for the purpose of transferring money
from your customers while to yours, think again.
BrainTree.
Re-think payments.
Learn more at braintreepayments.com slash se-radio.
For Software Engineering Radio, this is Jeff Meierson.
Philip Kren works at Elastic on infrastructure and developer advocacy.
He has a degree in software and information engineering and has worked for many years
as a web developer prior to joining Elastic.
Philip, welcome to Software Engineering Radio.
Hi Jeffery, it's a pleasure to be here.
It's a pleasure to have you.
Most listeners know what a search engine is.
We all use Google to search web pages, but there are many other things that can be searched
through.
What are some applications that search is useful for?
Well, pretty much everywhere where you have lots of information you want to search through
something.
So Elasticsearch, for example, is widely used in lots of websites and applications.
One of them would be Wikipedia.
So if you search anything in the search box on Wikipedia, your search will actually go
through Elasticsearch in the background.
Or if you search on GitHub or a Stack Overflow, all of those will actually go through Elasticsearch
in the backend.
So if you don't want to rely on your search going through Google, but have it on your
own service or within your own site, then you would need some extra software to actually
do that for you.
And if we're talking from an architectural point of view, not from a specific type of
search engine, can you describe the high level components that fit together to allow users
to search through a collection of data?
Yes.
So normally I compare it to relational databases or databases in general, where I would say
that databases are very much black and white.
So you store something and then you can retrieve it.
Whereas full text search normally operates more in shades of gray.
You're not only interested in exact matches, but you want to have, you're going to search
for something like a concept.
So it's more like shades of gray.
You don't really care about if it's singular, plural.
Sometimes you don't even care if it's a noun or an adjective.
You just want to search for that concept.
And full text search normally adds some overhead in during the store part.
So what we call index, you would just index some document you have and do some additional
work for that before actually storing it so you can retrieve it afterwards easily.
So what that indexing would do is normally you would remove any formatting like HTML
tags, for example, then you could remove something like stop words, which are very common words
which appear in nearly every document and would not add much value to your documents.
Then you can do something like stemming, which basically removes the word endings or reduces
the word down to its root.
So you don't care about singular, plural or any specific flexion.
And then you could also add stuff like synonyms.
So you have, if you have multiple terms for the same meaning and you don't know what
your users want to search for, you could just add those synonyms to allow them to search
for any one of them and find the content you have.
Right.
You're talking about the process of building a index to be searched.
And the index that we're searching through is a way to look up documents.
So you often have a corpus of documents.
The Wikipedia example is perfect.
You've got the entirety of Wikipedia, it's all these articles about biology and physics
and history.
And you want to be able to map search terms to documents.
And in order to do that, you build what is called an inverted index.
And so you described it pretty well where you're building a way to map search terms to those
different documents.
So if you have an article about swimming, then the inverted index will look through the article
about swimming and it'll find things like water and chlorine and laps.
And then it maps those as the keys in the index to that article.
So all of those keys might map with some degree of accuracy to that article.
And how closely they map to that article is part of the fuzziness that you're talking
about where it's not like a relational database where you're always going to get swimming
if you look up the word water, but you might get it sometimes.
Yeah.
So what normally happens, for example, if we stick to the swimming page on Wikipedia,
we would just take that entire document, then extract the token, we call them tokens, which
is basically all the words you have there.
Then we run them through this indexing pipeline we have.
For example, swimming would probably be reduced down to swim.
So if you have swim swimming that wouldn't make a difference, you would just store the
word swim.
And then you don't even need fuzziness right away.
So as soon as somebody searches for swim, you would have a direct match.
You could add fuzziness on top of that.
So if somebody would misspell swim, even though that's picked hard on that specific
word, you could optionally add something like fuzziness there.
And of course, you could add pseudonyms for swim as well if people want to search for
something else.
So you're describing these different ways that you can add richness to what is the core
abstraction of the inverted index where search terms are mapped to specific documents that
we're looking up.
And I'm glad that we've been able to give people an explanation for this inverted index
in case they needed a reminder or an introduction to it.
So with those concepts in mind, with that being the core data structure that we're building
here, let's talk about Elasticsearch.
What is Elasticsearch?
Well, Elasticsearch is one of these full text search engines.
It's based on Apache Lucene.
That is actually what is doing all the hard work we have just discussed.
So Lucene is the thing storing the data on disk and making it searchable.
It's doing the index process.
So stop words, stemming, tokenization, all of that is Lucene's work.
And kind of around that, like a shell, we have Elasticsearch and that shell or outer
layer, the query DSL, it provides a REST interface and does the replication and distribution
of your data over multiple nodes.
So the actual search work is done by Apache Lucene.
And then a nice way to interact with that, that is provided by Elasticsearch.
What does the term Elastic refer to?
Well, I'm not even really sure.
We started off as Elasticsearch.
Actually Elasticsearch was kind of the third implementation.
The first two were called Compass1 and Compass2.
And then Shai, our CEO by now.
We named the third one not Compass3, but he called it Elasticsearch.
And after we started adding more and more products, it was not just Elasticsearch anymore.
I guess it was just shortened down to Elastic.
And yeah, that's what we have been sticking to.
And since elasticity and scaling and stuff like that is a common concern, I guess Elastic
is very fitting.
Okay.
Yeah, that was what I was hoping it's indicated because I've got a lot of questions about
scaling because of course, if we're building a search engine for something as big as Wikipedia,
or something as big as all of the logs that we're storing for our application, we're
going to need some elasticity.
So can you give a introduction to some of the elastic properties of this search index
that we're building?
So there are generally lots of moving parts.
So let's start with kind of the simple stuff.
In the background, you store your data to one index.
The index is pretty much like a database in the database world.
And that one index can be split into multiple so-called shards.
By default, those would be five shards.
And each of these shards can either live on one server or if you have multiple servers,
the data would be distributed over your multiple servers automatically.
So that is how we split up data.
And if you have more than five servers and want to use more servers for storing your
one index, you would need to change the number of shards when you create that index.
And in addition to that sharding, we also replicate the data.
So by default, we have a replica of one, basically meaning you have your five shards.
And then for each one of these shards, you would have one copy on another node.
So if one node dies and a few shards go missing with that node, you will still have those
replicated on another instance.
And that is kind of the basic concept how the distribute data and also be resilient to
failures.
And by using that approach, it's very easy to scale horizontally.
So you just add more nodes to your cluster and your shards will be automatically distributed
and replicated.
And that's basically all you need to care for.
I think this is a good opportunity to talk about sharding and replication more generally,
because this is a technique for building a resilient system that is true whether we're
building a search index or a relational database.
I know you just gave a pretty good explanation for what sharding and replication do.
But can you go more generally into what sharding and replication are just maybe given alternative
explanation for the explanation you just gave?
So I think the term sharding actually started in a computer game with it online.
I cannot recall.
But there was a game and there was some crystal and that crystal was split up into multiple
shards.
And I think every shard was one of the game worlds or something like that.
And that is where the term sharding originally comes from.
So you have one big thing and you want to divide it into multiple smaller, more manageable
pieces.
And in the example of the crystal, yeah, the crystal was split up and each one of them
contained some part of that world where in anything that stores data, the shards will
contain some part of that data you want to store in its entirety.
So the number of shards that we have is a reflection of how diverse the volume of unique
pieces of data in our database.
And we break up those shards and then we replicate them some number of times because, first of
all, the entire sharded database is too big to fit on any one machine.
And second of all, if one of your machines with a unique piece of data fails, then you
want to have a backup.
So you want to have your shards replicated onto different machines.
So we're dealing with these compute environments where machines are failing all the time.
For the average user, they're working on AWS and maybe they don't notice any failures.
But if you work at a company of any reasonable size, you start to notice failures when the
machines that you're spinning up, they'll fail occasionally.
And so since many of the biggest companies are hosting their databases on a cloud service
provider like Amazon Web Services and these computers are failing all the time, it is
important to have a replication factor, which is how many times you're replicating each
piece of data that can account for how often these machines in the data center somewhere
are failing.
What are some strategies around deciding how often you're going to replicate each shard
of data?
So maybe we should take this step back to talk about like, why would you shard like that
is one of the most common questions, like how many shards do you need?
And what is your replication factor?
So the thing about replication is they help with two things.
Firstly, they make your data more resilient to failures.
So depending on how many copies of your data you have, you can take more numbers of servers
that fail.
But secondly, in the case of Elasticsearch, the replicas can also be used to increase your
write, your read capabilities.
So you can either read from the primary shard or any of its replicas.
So if you have some high traffic website, for example, you have a shop and Black Friday
comes along and lots of people want to read more data from your shop, you could just dynamically
increase the number of replicas to scale up your reads since you have more sources to
get your data from.
On the other hand, to scale up writes, you will need more shards so that more servers
can spread that writing load evenly overall of them.
And what further kind of dictates the number of shards you want to have is either how much
data can one node store.
So how many servers you will need to store the data and be what is the write throughput
you want to achieve overall your nodes.
So that will kind of dictate your number of replicas and shards.
There are no fixed numbers.
Like often people ask like, what should those numbers be?
And our answer is always, it depends.
And one thing you should definitely avoid is just thinking, well, I don't know how many
shards I will use in the future.
I will just add 100, which is totally a random value, but you should not do that because
every shard has a specific overhead in terms of file handles and memory.
And if you have a rather small cluster and have lots and lots of shards, we always call
that oversharding.
And you will just lose a lot of resources just for sharding your data.
Also then, if you search overall your shards, you will need to combine those results from
all the shards.
So that adds quite some overhead.
It allows you to scale up very highly, but if you never have that problem to scale that
much, you're just wasting lots of resources on the number of shards.
So there is no final answer.
What is kind of the right shard or replica number?
It will always depend on your use case.
Also, what is your write load?
What is your read load?
And what growth do you anticipate for the future?
Okay, well, to give maybe an idea of at least some relative sharding and replication factors.
So Wikipedia is a giant application and there's tons of users that are using it and it's also
got a lot of updates.
It's getting dynamically updated throughout the day.
And then there's also the log management system for Wikipedia.
And I imagine that let's just assume that Wikipedia uses Elasticsearch to manage its
logs.
And I think we'll get into log management with Elasticsearch in a little more detail later
on.
But just as two examples, how would you contrast the scalability, the sharding and the replication
factor of a site like Wikipedia with the log management system of Wikipedia?
So generally, your access patterns for search and logging are a bit different.
So for search, normally you have, I would not say small, it can be in the gigabytes
or low figures of terabytes data.
But your search use case normally is kind of limited in the size of data.
But you always want to allow very quick searches on that.
So there you will probably have more service for less data, whereas for the logging use
case, you often have rather old data and you do not need queries to return that quickly.
So there you often have like the limiting factor for the log use cases often, can I ingest
all the data I'm producing?
Like if you have, I don't know, 1000 or 10,000 updates or requests per second to store,
that will dictate how many nodes you need to have and how much storage do you need depending
on how long your story or your data.
Whereas for the search use case, you will have less data, but it needs to be searchable really
quickly and probably you want to have lots of memory for that.
Whereas for the storing of log data, often you will have larger disks since it's not
that critically if a search takes like two or three seconds, which on the other hand would
be totally inexactable for the search use case.
So it's just a finding kind of the right trade off what your use case is.
And that will then dictate what kind of service you need or how many service you need.
Okay, so we jumped to scalability.
Let's scale back a little bit and talk about just the core search index again.
I probably should have gone over this more in the beginning, but I think it's okay.
I think it still fits here.
You know, we could talk about types and mappings that you can set up in Elasticsearch, but
I think people can look this information up.
Basically every type consists of a name and a mapping.
And so a blog post, for example, would have its own mapping, which defines the properties
of that type that we want to index.
So if you're indexing your blog with an Elasticsearch engine, you will give every blog post a mapping
that sets up some, it gives the search index some information about how you want results
to be returned.
And this is sort of like setting up the schema of a database.
But this doesn't answer as much about the fuzziness.
So let's talk a little bit more about the fuzziness, because as we discussed in the
beginning, there is this combination in a search engine like Elasticsearch where you've got
part relational database type of behavior where the search engine has a clear definition
for what kinds of results it's going to return.
And then you have more of a fuzziness also.
Can you describe how the relational type of properties and the fuzziness affect how a search
query might return results?
Okay, before we jump into the searching part, one point about the types and the mapping.
So firstly, mappings can either be created when you create the index, which you should
always do for production use cases, but it would be automatically generated from the
first document you insert into an index.
So if you have some attributes, Elasticsearch would try to guess the right data types from
those fields you provide, which is often working reasonably well for the prototyping phase,
but for production, you should not really rely on that.
And secondly, the types, I always say they're like an enumeration type or something like
that on one index, you can store different types of information in one index.
The only problem is Lucine, the underlying storage engine does not really know about
these types.
And we have just decided like one or two weeks ago that types will in the future go away
in Elasticsearch.
So do not rely on multiple types in one index too much anymore because they will be gone
in the foreseeable future.
Okay.
So let's go back to the original question about search.
So basically what you can do search wise is we have on the one hand, we have something
like Boolean queries where you can define exact things to match.
So you could just say this must have a specific feed or this must be in a specific range.
And then we have the full-text search capabilities, which can work on these index terms.
So you have, for example, you have the stemming from swimming to swim and it will search on
everything, anything that resembles this swim regardless of it, if it was swim, swimming,
whatever.
And on top of that, you can put more advanced search concepts like you could use engrams,
which would be like you search for specific groups of letters in terms if you don't have
an exact match on something.
It would be, for example, if you have blueberry and or let's say we have blueberries in the
future in the plural and elastic search for stemming would reduce that to blueberry with
an IIT end.
But that would never match on anything that is blue.
To search for something like that, you would need to extract groups of letters to actually
be able to match that blue since we do not do any partial matches otherwise.
We would just compare the entire token to we have.
Or another advanced search context would be we can add this fuzziness.
So we can say a few letters are either missing different or too much between what we have
stored and what you're searching.
Basically this is a Levenstein distance.
So with a Levenstein distance of one, for example, you could have a word in your index
and what you search for if that has one letter too little, one letter different or one letter
too much, that would still match.
And in the beginning we implemented it quite naively with a brute force approach, but it
has been then re-implemented with full automaton to actually be able to generate kind of a
network of all the possibilities to not rely on the brute force approach of fuzziness.
So that should make it much quicker to search for things like that.
Okay now that we've talked about the setup of our indexing and we've talked about the
elasticity components, let's talk about a read from top to bottom.
So let's say I open up Wikipedia, I do a search for how to learn this one, let's say that.
What's going to happen on the server side when that query hits the elastic search cluster?
Right, so we have the clients for different languages, we have the bindings for whatever
your language might be, Python, Ruby, Java, DotNets and lots of others.
So you would probably use that binding or you could just fire a REST request against
any of the nodes you have in your cluster.
That normally will run Robin between the nodes you have configured in your connection.
So you pick one of these nodes by randomly since you run Robin and that node will be
your client node.
Previously we got that coordinating node as well and that will coordinate your query.
So you send the request to that one node, that node then figures out, okay, this is
this specific index, the index has these charts and replicas and it will make sure that you
query hits every single chart either primary or replica and hit the right nodes where these
charts reside.
Yeah, wherever they live.
And then each node having one of these charts would actually do your search on that node
and get the partial result of that node.
And then it would send that partial result back to the coordinating node and that node
would then put together the entire result.
Actually it's even a bit more complicated than that.
In the first step all the charts would only return kind of their matches.
So if you search for the 10 best hits, each of these charts would return its own 10 best
matches but only the ID and the so called score which is basically a quality attribute.
The coordinating node would then take the 10 best results globally from all the sub results
it has and would request the documents, the nodes having the data or half by just requesting
them through the ID it has received.
So it's a two step approach, it's like a scatter gather where you just get the IDs first and
then collect the final results and then the coordinating node will send you back that
result that you have just requested.
Okay, in that example you talked about the client or coordinator node.
Is this any node in the cluster?
Are all of the nodes potentially client nodes?
So by default it could be any node but you can split up the roles of the elastic search
nodes.
So we have data nodes, these keep data and do the actual search work.
We have master nodes, these keep track of the status of the entire cluster.
We have client nodes, these don't keep any data themselves, they are just coordinating
these queries and requesting the results.
Right freshly in the latest five release we have so called in just nodes which can do
some processing and enrichment of the data when you store it and we also have tribe nodes
which can connect to multiple clusters.
And by default a node would be a master node and a data node and would just do all the work
but once you have a bigger and bigger cluster you might want to split up these different
roles of nodes to specify them a bit more and to keep your cluster in a more reliable
state.
So you would start out by splitting out the master nodes so that they just keep the state
of the cluster and don't do any other hard work to keep your cluster state quite reliable.
And then you could also split out these client nodes.
For example if you do very big aggregates that takes up a lot of memory and you do not
want to your data nodes to swap out all the data they have just fetched from disk to memory
because they need to put together some big aggregate.
You wouldn't want to pay that price to always refetch data from the disk just because some
other process just took up all your memory and expired the cache you had.
So you would split that up so your data nodes just have as much data loaded into memory
as possible whereas the client nodes they are good for coordinating your queries and
then combining results together to the final set you want to return to your application.
This podcast is supported by Atlantic.net hosting solution provider of healthcare HIPAA
PCI compliance and ADTEC.
If you're feeling the pain of having outdated technology call Atlantic.net and get the latest
security firewall intrusion detection backup disaster recovery and virtualization.
Or if you're starting a new project or not getting the results you want from your current
providers Atlantic.net can help you succeed.
The fully audited solutions of 23 years in business they won't let you down.
Visit Atlantic.net to learn more.
As I mentioned earlier these elastic search clusters are often in a data center somewhere
with dubious hardware quality on some of the machines and nodes are failing all the time
so you may have a failure in the middle of a query you may have a failure in the middle
of a write.
Actually before we talk about those failures we just discussed a read why don't you discuss
a write.
Okay so a write it will depend do you provide the AD yourself or do you let elastic search
generate the AD.
In any case any document you have will have an AD either assigned by you or generated
by elastic search.
Then the next step is whatever the client notice it will hash that ID so it will be evenly
distributed over all the shards and then after hashing the ID it knows which the write primary
chart for that data is that is part of the cluster state information so it knows this
area of the hash value is for that shard and that shard resides on that specific node.
So it will forward the data to the primary shard where the AD landed on after being
hashed and then the primary shard will check is this meaningful document can I insert that
does that not conflict with the mapping I have so for example if you have a field and
that is of the type integer and you would you try to insert a string there the node
will immediately say well I cannot process that and would give you back a client error.
But assume everything works and you can apply the document to that specific index and type.
It would write that data down in we call it trans log in postgres it would be like a
write a headlock so that data would be written down and then it would be forwarded to the
replicas like by default one you could set it to zero or more than one these would then
accept the write write it down as well and only then would the primary shard acknowledge
the write back to the client node and would then let it know okay I have applied that
write operation all done so it would return a 201 if it had created the document or a
200 if it had just updated the document.
One thing that is might be a bit surprising is that data if you do a full text search
on it is not immediately available by default so by default we have something called a refresh
so of one second so every second all the documents you have added would be added to
this inverted index actually Lucene would create a new segment for each of these one
second intervals you have and that is what is then stored on this can well this actually
searchable you can immediately retrieve a document by its ID but it's only available
through the full text search part after this refresh interval has passed and that again
is a trade off performance so you could either set this much higher than you would write
fewer segments but it takes longer until your documents are searchable or you can keep
a low value like one second or you could even disable it entirely but then you would have
lots of very small documents which will probably not perform that well but your documents are
very quickly searchable then so again it depends on how quickly you want to be able to search
something.
We've talked through both the read and the write processes now both of them involve a
lot of coordination so what happens during a machine failure in one of these it well
maybe you could describe some failure cases of read and failure cases of write you don't
have to go and I guess that's a big question but take it for what you will.
Well so depending on what kind of machine fails different things will happen if it was just
the note keeping data at some point the other notes notes will recognize that there is a
specific timeout how long they will wait until they assume the note is really dead and once
they that the note is marked as that in a cluster state the master note will make sure
that the data is replicated in the right fashion again so the note that has died if it had
any primary shots and you have any replica shots of those those replica shots would be
marked as primary shots then on another note and those shots would then be replicated to
another note to again get to your replica vector you have configured and if some replica
shots are on the server and they're gone the primary shot would make sure that it replicated
stated to another note if your master note fails there can always be one master note in
your elastic search cluster since that is the one instance that keeps track of your cluster
state the other master eligible notes would do a vote on who can become the next master
and who keeps then track of that master state so that is kind of what happens when the note
dies and then if the read or write operation goes out and it's just a failure is in flight
the operation would then just timeout is there is a timeout for every query when there is
an error just happening at that specific moment but as soon as we have something reachable
again the coordinating node would direct the traffic to the write note.
Okay well I have heard about these issues where if you're somebody who is managing an
elastic search cluster there are these statuses I guess like a red at yellow and a green status
that indicate how live a node on a cluster is and if you have things that are in a yellow
state this is equivalent to the canonical partial failure and partial failures in computer science
are typically harder to deal with than complete failures can you talk about what causes partial
failures and why those are so hard to manage in an elastic search cluster.
So the colors you mentioned like green status you have all the primary shots you have are
available and all the replicas you have configured are also available in your cluster.
The yellow state means you have all your primary shots so there is at least one copy of your
data but your replication factor is not honored at the moment so if you have a replication
factor of one and a node dies and your replica or your primary is gone you still have that
other copy so you have one copy all your data is readable and write a bullet moment but you
do not honor your replication factor and while the data is being replicated to another node
you will be stuck on that yellow state also if you have just a single node and you have
a replication factor of one it cannot replicate to that node it doesn't make sense to replicate
data on the same node so we will never do that and the data always needs to be replicated
to another node so if you have a replication factor one with a single node that cluster
will always be in the yellow state so all your queries will work but you do not have
the kind of the capabilities to have all the copies you want so you are not as resilient
as you could be and only the red state means that some data is not available so for example
if you have a replication factor of zero and a node with that one copy of the data failed
your index would be in the or your cluster would be in the red state or if you have a
replication factor of one and two nodes fail and one of them has the primary shot and the
other one has the replica shot for that specific cluster then your cluster would be in the
red state so not all your data is available anymore that is basically what these colors
mean.
Okay well so we have explored many of the scalability aspects of Elasticsearch we have
explored the basic data structure that we are scaling which is the inverted index let's
talk about some application level architectural level questions the typical application these
days uses multiple different databases to store its data and this is the polyglot persistence
model in something even in something like Wikipedia where you've been discussing Elasticsearch
as the main system of record I imagine if I click on a link for a page sometimes things
within that page are going to have to load data from a database and it's probably pulling
from a place that is not Elasticsearch because it wouldn't make sense if the query to get
the page assets is very well defined you wouldn't want to go to an Elasticsearch cluster I assume
but please tell me if I'm wrong.
So if I remember correctly, PDA is using MariaDB for the storage of its data and then for
full-text search it would add Elasticsearch something like that is a very common scenario.
Yeah okay can you talk more generally about polyglot persistence and how Elasticsearch
fits into a multi-database model?
So I think like polyglot persistence was very appealing in the beginning or specifically
for developers but for the ops people polyglot often means quite a headache because when
you develop on some feature for two weeks that is fine for you but if the ops people
have done to maintain it for the next three years they're probably less happy if you just
add random data stores.
So while I think that using multiple data stores makes perfect sense I think you should
be kind of careful not to add too many just to experiment with stuff since your ops people
will not be too happy with you in long term.
So there are these examples where you have like 5-30 data stores for one thing and I'm
never sure if that is really a good idea but of course you can do that.
So one common aspect where Elasticsearch would come into the picture is if you need some good
and scalable full text search capabilities for your product which is often an important
point for I don't know shop systems big informational sites and another point where Elasticsearch
comes in handy is if you have any logs you want to store and then be able to search them
or if you have anything around metrics and elotics those are use cases where Elasticsearch
often fits well and it can cover quite a lot of ground for that.
And there's always this elusive question like is Elasticsearch a good primary data store
and normally our answer is it's not really a primary data store some people use it as
such but we are not sure it's always the right fit for it like it always depends.
Sometimes it can work but for many use cases you don't want to put all your eggs in one
basket and have it in the full text search engine especially for resiliency reasons.
While we're improving a lot there are always like smaller issues you can run into so it's
always a trade off between SAP use powerfulness resiliency and for relational databases for
example they had I would say like 30 years kind of to bake and no of the no SQL solutions
has had any time close to that to evolve and mature.
So yes often a conservative approach for that is probably still a good approach and even
though I work for Elastic I would not say just use Elasticsearch for everything.
Be careful use it for the right approaches or problems to solve and then it will make
you very happy but it's not the hammer and nail thing you just use one tool for everything
that's probably still not a great idea.
So yeah Polyglot definitely has a place but just be careful like not to have an explosion
of data storage technologies like probably for many people something like relational
database will work well maybe a key value store for caching and keeping session information
and then Elasticsearch for full text search logging metrics.
But it will always depends.
The nice thing is there is so much technology around that you can just explore but it also
makes it much harder to find the right combination of tools I guess.
So if you're Wikipedia and you're using Elasticsearch and you're also using MariaDB is MariaDB
like a full representation of your war of the same Elasticsearch world or does the Elasticsearch
cluster or does the single point of truth have to be replicated elsewhere.
Like I guess my question is is MariaDB here in this architecture that we're discussing
I know we're not talking the exact architecture of Wikipedia but maybe a hypothetical architecture
that Wikipedia might resemble are we writing to Maria are we are we keeping MariaDB which
is a relational database are we keeping that because the query shapes that will hit that
database are different than Elasticsearch or are we keeping it because it's a backup a
reliable backup that we can use in place of Elasticsearch.
So while I'm not sure of the specific architecture of Wikipedia I don't think it's it's a backup
like I'm not sure that is that is the general use case people want to do while SQL can be
a bit hard and daunting at some points like it's still very powerful and widely used so
many developers also want to use that so I don't think you would add a relational database
just as a backup but you will often interact with that directly whereas you would then
just offload the search capabilities to Elasticsearch and probably logging in metrics and other
use cases.
So let's talk a little bit more about the logging example we touched on it a little bit earlier
but Elasticsearch is widely used as a system of record for logging and I think there are
a number of reasons for this one I can think of is that you know you the the log data is
there's so much log data a lot of it is well formed like you have your log messages that
are well formed and so you can make a recent a decent schema out of it using the Elasticsearch
models for doing schema the mappings and the types but also you can afford to lose some
log data I think so so it's not catastrophic if you're you know if you end up having this
this type of failure that you're talking about like why don't you use why wouldn't you use
Elasticsearch as the the core system of record maybe it's okay to lose some data for logging
but I shouldn't talk so much about this so maybe you could just speak a little bit about
the typical logging infrastructure that somebody is using when they're using Elasticsearch.
Yeah so first off I like I'm just careful not to give the wrong impression here but it's
not like if you use Elasticsearch that you will just lose documents every day okay like
this is not what is happening like even if you're testing stuff on the lab conditions
and you're just crashing nodes all the time losing data is actually quite hard and takes
some effort to do and if as long as you have configured your cluster and you're using an
up-to-date version since we're improving quite a lot in the resiliency as well like
data loss is not something you will experience commonly like most people who lose data or
pretty much everybody or all the cases I've seen for the last half year or so or even
longer where mostly like bad configurations and yeah there's probably nothing to do against
that but like data loss is not something that happens very commonly okay coming back to
your original question I think what was also the appeal of Elasticsearch is that it has
more of this entire stack around it because at first when you think about it full text
search and log management don't really go together that well or it just sounds like two
different problems to solve the nice thing about Elasticsearch and the so-called Elasticstack
is that it's kind of an entire ecosystem around that so we have log stash to parse an enriched
data and in the beginning it would also collect your logs now we have the beats which will
which are just like lightweight agents or shippers you can run on all your servers they
will just forward data and can either insert the data directly into Elasticsearch or have
a party parse and enriched by log stash enriching would be something like you have an IP address
and you want to get the geolocation of that IP address so you would just store the city
the country and the region of that IP address so you can easily pinpoint that afterwards
and once you have everything in Elasticsearch you can then use kibana to visualize and explore
your data so with these four open source products you kind of have an total stack to monitor
your metrics and logs and I guess that is one of the bigger appeals and on the other hand
since it's open source it's very easy and cheap to get started and thanks to the horizontal
scalability of Elasticsearch it's also very easy to scale it up so we have customers using
dozens or of nodes in one cluster or even three digit nodes in one cluster just to store lots
and lots of these logs you have so we're running low on time but just one more question is an
application perspective because I think we've touched on a lot of the lower level details
what if I was building a ride sharing platform and I wanted to search a geospaced to find
the nearest cars to a user my understanding is that Elasticsearch would be useful for this can
you explain why that is oh yeah we have we call him geonic so in in earlier versions of the
patchy Lucene you could do geocuries so you have a longitude and latitude you have a point and
then you can either just calculate the distance or do a specific geometrical figure around it and
find stuff within a region so there were lots of capabilities but performance was not that great
and at Elastic people figured out that that was not or that was an issue at some point and then
they hired one of my now colleagues Nick Knice he we call him geonic and he reimplemented the
geofeaches on Lucene and once they were in the scene we could add them to Elasticsearch and now
geolocations are very useful in Elasticsearch and also very performant especially since version
five since they were pretty much totally rewritten there so geolocations just having a point and
then finding another close point or finding all the points in one region that is totally doable
like in the query DSL we have white power query geocury capabilities what are some of the features
that are being worked on within Elasticsearch right now it's so many that is kind of hard to keep
track at the moment so big stuff coming in the next versions is again on the resiliency side
we're adding sequence numbers which should help with speeding up the replications in case of failures
and also at some point allowing cross data center replication so you cannot only have a cluster in
one location like one AWS region but could probably replicate the data over to another cluster in
another location we will get rid of types we are always trying to improve the upgrade process so
right now the upgrade process of Elasticsearch between major versions is a full cluster restart
which is for many people very painful we're also working on lots of performance improvements
both on the Lucene side and in Elasticsearch itself and the entire stack is kind of moving so in
all of the open source and also our commercial plugins there are lots of new stuff coming up we've
just had Elasticsearch on our yearly conference in San Francisco and there we already kind of got
previews of some of the cool new features coming up so kind of my personal favorites are for
Filebeat the thing that collects log files it will automatically support specific protocols now
which makes it much easier to get started so you can just collect something like syslog and
next logs Apache logs those will be automatically parsed in the right format so it will extract
something like HTTP headers or return codes IP addresses geolocations and store that in Elasticsearch
and you don't need to configure anything. OsukiBana is getting lots of new visualizations and a way to
to visually build a metric or to visualize your metrics easier Elasticsearch is improving on that
upgrade process and also adding yeah more advanced features I don't want to dive too deep into right
now Logstash is getting its persistent queue mechanism so that you probably don't need a queue in front
of it anymore which was a common use case so these persistent queues are in the works and we're
also improving the monitoring aspect of all the components so right now we monitor Elasticsearch
kibana and Logstash beat will be added soon and in one of the upcoming versions we will also collect
logs Elasticsearch for example produces in Elasticsearch itself again so you can search those easily
but you probably want to throw that into a second cluster but just to grow the stack right now it's
pretty much blocks of Legos so you have all these elements but there's some assembly required to
put them together and I think that the next steps are to make this starting experience nicer so
there are more features or more solutions coming right out of the box and less assembly for our
users required even though you will still be able to kind of configure anything you want so you're
free to do whatever you use cases it can be logging search analytics metrics the Elasticsearch is not
a one trick pony just focused on one of them but we're always trying to expand to new horizons
for example we've recently acquired a machine learning company and machine learning basically
anomaly detection that is one of the big things that will come out very soon so where can people
find you online either social media or otherwise yes so I should be very easy to find what once
you know my online nickname which is Xyara which sounds very weird and many people think it must be
something like Xena and I'm a girl but no it's not it's actually a ROT 13 of my last name which is
Kren KRE Dublin and when you rotate these letters by 13 it's XER AA that is both my Twitter handle
you can find me at xerra.net on the interwebs and I'm pretty much wherever I am I use that nickname
because nobody else does so once you have that I should be easy to find and I'm doing lots and
lots of conferences and meetups mainly in Europe so you will have a good chance to find me I think
until summer I have already 20 conferences scheduled now so yeah I will be around if you're in Europe
you should be easy it should be easy to find me and I'm always happy to hand out stickers and
answer questions there. Alright well I hope you come to the states for an American tour sometime
soon as well yeah I'm currently discussing with our marketing team if we if we could do that let's see
okay okay all right well Philip thanks a lot for coming to software engineering radio
oh it was all my pleasure thank you
thanks for listening to se radio an educational program brought to you by iJER
police software magazine for more about the podcast including other episodes visit our website
at se-radio.net to provide feedback you can comment on each episode on the website or reach us on
linkedin facebook twitter or through our slack channel at se radio dot slack dot com you can also
email us at team at se-radio.net this and all other episodes of se radio is licensed under creative
commons license 2.5 thanks for listening
