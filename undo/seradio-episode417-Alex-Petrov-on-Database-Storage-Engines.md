Link: https://www.se-radio.net/2020/07/episode-417-alex-petrov-on-database-storage-engines/



This is Software Engineering Radio, the podcast for professional developers on the web at se-radio.net.
SE Radio is brought to you by the IEEE Computer Society, the IEEE Software magazine.
Online at computer.org slash software.
Developers, are you ready?
It's time to upgrade your data platform to InterSystems IRIS.
Choose your language. Choose your tools. Choose your environment.
Collaborate, build faster and deploy more efficiently.
When you can make faster decisions, there's no telling what you'll create.
Ready, set, code.
Start coding for free. Visit intersystems.com slash try to try IRIS.
For Software Engineering Radio, this is Adam Gordon-Bell.



Alex Petrov is a data infrastructure engineer.
He is a database and storage system enthusiast.
He is a Cassandra, a core contributor, and the author of Database Internals, a new book for O'Reilly.
Alex, welcome to the podcast.
Yeah, hey, pleasure to be here.
The databases have a lot of components in them, like query optimizers, clustering, transaction management, access levels.
But I thought it would be interesting today, since you're the expert just to talk about kind of the storage layer, writing to disk, basically.
That's my favorite part.
So what is the database storage engine?
So the database storage engine is sort of something that lives in the heart of the database and the very end.
And so if you can say that you are the user and you're sending their request and probably storage is the last thing that, well, actually your request gets to.
So it's like the further distance from you to there, basically.
So in short, storage engine is something that helps you to access your data and it also communicates and is coupled somewhat tightly with many other components of the database, such as, for instance, write a headlog, or it can even be considered a part of the storage engine,
or it is coupled in a way with the transactional processor.
So yeah, it's something that helps you to store the data and, well, get it back whenever you need it.
When I think of storage engine as a concept, I think of back when I used MySQL and you could actually pick a different storage engine.
Is that common in databases that they could have interchangeable storage engines?
I wish, I wish, I wish that was true.
So there, I would say, is a concept of interchangeable storage engine in several databases.
And for example, MongoDB is probably somewhat famous for having several storage engines, or at least, you know, they migrated from one to another.
So they're using wire tiger at the moment, to my best knowledge, at least.
And before that, they were using a map search engine.
And some will, storage engines are targeting themselves, or positioning themselves as a component that can be simply dropped in.
Well, some database.
So unfortunately, I cannot say that most databases allow, you know, just dropping in and easily replacing the storage engine that they're using.
But there are some, and there are obviously use cases for that.
And one, probably most prominent example is Roxy B.
And there is a project called RockSandra, for example, that is taking Cassandra and trying to make the storage engine of Cassandra pluggable.
And unfortunately, that's not such an easy task.
And it is allegedly used in production.


I don't have any more details on that, but, well, it's an open source project and it seems like it's active and living.
But from the perspective of development, it was not, you know, just abstracting away the difficult parts and making it completely 100% pluggable.
So there are many things that I would say still not 100% compatible between, you know, the traditional Cassandra in air quotes and the rock center.
I hope that the trend continues and more folks actually try and make their storage engines pluggable and start using, you know, off the shelf, well, ready to use solutions, maybe Roxy B and wire tiger to be the best contender, so to say.
So yeah, there are examples.
I assume it's not as easy as some interface with, like, write and read and you just implement the minutes over.
Yes, I would say that that's the biggest problem.
And once again, as I said, the coupling between the database components is probably one of the biggest problems.
Yeah, we don't get to arbitrarily pick and choose because they don't.
We can't switch them out.


What about the storage engine of like an in-memory database?
How does that differ?
So when we're talking about in-memory databases, it's not like you can say that every in-memory database is the same.
So some in-memory databases store data just in memory.
So without, there is no access to this at all.
And some in-memory databases use this for some of the things.


So for example, you can use this for logging, like recovery, right?
So whenever you restart the database, you can bring it at least to the latest state that it's been in before it was restarted.
Or another example is whenever you're using disk as a secondary storage, meaning that most of the data is living in memory and the rest of the data.
So it's a cold storage, it lives on disk.
So when we're talking about like in-memory databases, there are just so many things.
And in my opinion, the storage engine part of in-memory databases is probably still the thing that is taking care of the data.
And taking care of sort of disk accesses, because well, everything above it can be thought of as several levels of maybe exquisite caches, like very fast, quick and specialized caches in a way.
But ultimately, if you're accessing the disk in the end, I would say that storage engine is still something that is below the in-memory caches.
However, I'm not 100% sure if there is such strict separation of concepts or depending on an implementation, things can be titled or tied together so tightly that it's barely possible to tell those things apart.
So yeah, it really depends.
So does that mean that there could be some sort of convergence? Like if I am able to give enough memory to a Cassandra instance and use it as a key value store, does it perform like like Redis or I would say that it's very difficult to make Cassandra perform just like Redis on like if we compare like a single node of Redis and so I'm going to note of Cassandra with a bunch of memory.


But I would say that Cassandra has its own features and its own things that it can be better at.
I guess I'm imagining in my head that I could make an in-memory key value store by just having just like a hash map that I hold on to, right, and put values in and out.
Whereas like a disk-based storage engine seems much more complex. I mean, that's my understanding. Is it harder to work with a database that's storing things on disk versus one that's in memory?
I would say it's inherently harder, so much, much harder. So it's funny that you mentioned the hash table because like one of the things it's best for, it's good for, is like random access basically, right?
Whenever you have a key and you would like to get a value, so that would be the perfect example for the hash map usage.


However, if you would like to take not a single key but a range of keys, for instance, you know, give me everybody with the last name that starts with P for instance.
So it would be very difficult to respond to the same query with the hash map, or at least you would have to have either a linear scan through all the keys and values, or you would have to have some separate index structure that would sort things and prepare them in a way that would allow such a query to be ran.


However, if we're talking about the on disk structures, so things get even more complicated. So not only we're, or we're not only constrained by the fact that, well, certain data structures are best for certain things, but we are also constrained by the fact that
we are implementing the memory management by ourselves, because even if we're using the programming language with unmanaged runtime, whenever we were just using, you know, raw pointers, for instance, and we still get at least something from the system, right?
We get like virtual memory, we can access, well, we can allocate chunks of memory, we free them, and we don't really see, you know, physical representation of things most of the time, and even if we're talking about things like classes and structures, so we are
writing to fields and we rarely, you know, address things just bite by bite. And whenever we were talking about a memory, or things that are stored on disk, we have to implement our own memory management, meaning that whenever we address things that are located within the same page on the disk,
we can, you know, use like local pointers whenever we address the thing that is located in a different page on the disk, we have to use out of page pointers that, and we have to think about how we implement those because, you know, page location can change.


So, of course, I'm, you know, just throwing concepts here, but get the idea, right? So, instead of simply allocating and deallocating chunks of memory, you have to, you know, think through an entire structure, basically how to lay it out on the disk, and very little is there to help you because file system, all it gives you is an interface of, you know, seek to a specific point in the file,
and read a certain number of bytes from that file, and then, well, you can write basically to the same location in the file and that's all you get from it, right? And everything that you implement in the file, like it's header, it's like trailer, and everything in between those is pretty much like your pages, and it's certainly much more difficult than implementing
and in memory data structure. However, in memory data structures implemented right, and especially when, like, there is lots of concurrency involved, I would say that it's also not such an easy task. It's just different sets of problems, so to say.


So, a programming language is have a lot of concepts for dealing with memory, right? Like, even just, we have pointers that we can follow, but on disk you're saying, like, we have to build all these constructs ourselves, we have to build our own little pointer, you know, oh, this means jump to here.
Why do we need databases and why not just files?
In my opinion, main reasons for preferring a database over flat files is like, first one is storage efficiency.
So, files are organized in the database files are organized in the way that minimizes the storage overhead per store data record, which is not necessarily the same for the file system.
The second one is like access efficiency, so records can be located in the smallest possible number of steps, and here I mean that can also differ for your use case, you know, depending on what exactly you would like to achieve.
And the third one is update efficiency. So, records updates are performed in a way that minimizes the number of changes on disk.


So, you would like to avoid, you know, putting extra strain on your hardware without any need to do that.
So, in my opinion, that sort of summarizes so storage access and update efficiency is sort of three reasons for us to use file or databases as an abstraction instead of trying to implement our many database every time we're doing it.
However, from what I heard many databases, at least, you know, in the early days of computing, so to say, were implemented exactly because somebody was tired of doing same thing in every project they were starting so they just decided, okay, I just sit and implement
a general purpose tool, a database that is going to help me to, you know, retrieve the value by specific key because this is what I do very often, and it turned out to be a concept that is extremely useful for many of us today.
So, at some point, somebody's like, I'm going to write these records to this file, and I'm going to need to read them in, and they start the third time they do this, they're like, okay, I know what I need to do here, can I abstract this, and that's your database storage engine.
Yes.
And so, I assume like if I were doing that, yeah, first, there's some sort of mapping. So maybe if I have like some enum, maybe I want to just squish it into bits to keep it small.
So I'd come up with some sort of structure. So you mentioned earlier, LSM trees versus B trees. So what's a B tree?
Let me then pause you for a sec, and let's just take one more step back and discuss. What are the trees in general and why we need them, right?
Yeah. So when we were talking, for instance, about hash tables, we decided or we concluded that in order to search, not a single value by a single key, but like a range of values by a specific key, we need a structure that is different from a hash map.


And one of the data structures that we can implement in memory is a tree, a simplest way to implement a tree is a binary tree, right? So a node that has two children basically is a binary tree, right?
But whenever we're trying to, is it sorted?
So not necessarily. So binary tree only says that, okay, we have a node and two children, and they're not sorted in any way. If we would like to make it sorted, we inevitably once again, sorry, I keep repeating this word, gonna come to the concept of binary search tree.
So why we come to that is because binary tree is unbalanced, right? So you can have more elements on one side or on the other, and there is nothing that tries to, you know, maintain some sort of order in this structure.
So in order to maintain that order, we need some sort of balance tree. One of the examples of that is binary search tree. So binary search tree is a tree, which has at the root. If you look down from the root, basically, it's going to have elements less than the root


node on one side and elements that are greater than the root on the other side. So that makes it searchable, right? That means that if we progress from the root node all the way to the left, you know, go following left pointers all the way to the very bottom of the tree.
We're going to reach the minimum of the tree. So the smallest key that a stored in the street. And if we're doing the same for the right, like following right pointers from the root, we're going to reach the maximum of the tree.
And this sort of brings us to like the first usable searchable data structure that is, well, can be used in memory and, well, you can implement and use. However, if you're trying to implement binary search tree on disk, you're going to run inevitably in several problems,


because one of the things that you're going to be facing is the fact that because you have just, you know, two pointers, you are going to be constantly moving those notes and constantly reassigning those pointers.
So whatever, for instance, you have a root node and two children, and you have, you know, split the child. So you would have to, you know, reassign all the pointers that were related to everybody that was somehow connected to one another.
Like all these balancing operations are going to be in a way expensive or going to be costing.
Because you have to write, you have to rewrite it to disk.
Exactly. So that's one, one problem. And the second problem is, of course, locality. We are not necessarily going to have keys that are going to be searched together, written together in the same page, or in the same even region on the disk, or somewhere close.
So we may end up actually seeking the disk quite a lot, you know, just running around and trying to locate the things that we would like to find.
Because like you said before, on disk, like every pointer, you just have to go to a different location on the disk for every pointer follow, in theory.
Yeah, in theory, yes. Of course, you can optimize for that. And this is how we sort of progress to be trees or be plus trees.



So we just discussed, we progress from hash maps to binary trees, then to binary search trees.
And then we concluded that binary search trees are not a good fit for the disk because they have very low fan out and potentially bad locality.
And we can solve it by basically increasing the fan out and improving the locality. Right.
So that sort of means that unlike binary search trees, be trees have not just two children, but have many.
In binary search tree, we have a single key per node. And this key pretty much tells you, like, who is in the middle of this place that we're standing at?
So if we look to the right, we see, you know, all the values that are smaller than, and if we look the opposite direction, we see all the values that are larger than.
And so we are always sort of standing in the middle. So it's always sort of a separator key, right, that splits the range that we're searching into parts.
And unlike binary search trees, be trees have many keys. And so the relationship between the number of child notes and the number of keys is maintained.


So it's the same as in binary search keys. So it means that in the binary search tree, we have a single key and two pointers.
And in a B tree, we have N keys and N plus one pointers to the children. So this sort of makes a data structure that, well, first of all, whenever we're searching something, we know that we can very quickly locate the keys on this level, right,
because all the notes that are required in order for us to, well, reduce the search space are pretty much are located on this page. So we fetch or read one, a single page from the disk, and then load it up in memory and then start searching this memory segment.


Usually we use something like a binary search, because all the keys are sorted. And then we locate the searched key. And this key has a pointer, a child pointer that is attached to this key.
And then we follow the child pointer to the next level. And then on the next level, we do the same thing. We load an entire page. We binary search keys, find the key that, well, is pretty much matching whatever we're searching.
Once again, follow the pointer that is attached to this key, all the way to, well, the bottom of the tree to the leaf node, where we can finally find the data.
So, and once again, we have more than one key per node, and we have many children. And this sort of solves all the problems that we had with the binary search trees.
Because we can seek from disk kind of this big chunk that is our children pointers and go through them all in one pass.
I think that, so now, if I'm a database engine, so I'm using my B tree, I have less changes when a key comes in. Like, if I want to insert something new, there's less changes.


But it still has to happen. I assume we have a maximum size than any node will have in terms of children. And if I get a new element that would be sorted into the middle of a full page, then what do I do?
So, yeah, whenever we were talking about the binary search trees, we briefly mentioned, I think I even slipped and said that splits and merges, however, there are obviously no splits and merges in the binary search trees.
So, what binary search trees do in order to increase the capacity, they sort of append the node to the bottom most level, or wherever it fits basically, and then they rotate. So, a rotation step is a way to balance the tree.
And so this is how pretty much the tree is growing. Right. So, on each step, you can append the node and make the tree sort of unbalanced by making one leg of the tree slightly longer than the other ones.
So, yeah, basically what we do is we take the midpoint and leave it in this page that we were splitting. Then we take like all the elements before the midpoint and put it to like the first page that we append basically as a child to these pages which is split, which has only one element now.
And then we take the rest of the elements that were in that page and append it to the yet another page. And now we have sort of instead of having just one page we have three pages.
And of course, in order to outgrow the capacity of this tree, we will have to perform, you know, more operations because now we increase the capacity of the tree significantly and that's one of the advantages of the B plus tree is that
basically the capacity of the tree grows exponentially as the number of levels is increasing, which is the advantage. Like if I understand because that's like in a traditional spinning hard disk that's going to be like fewer seeks, right?
Because you only have to go down, say three levels or something. Exactly. Exactly. So what you usually end up doing is you end up reading maybe four, five pages depending on the depth of your tree. But basically you're going to in terms of disk seeks, you're going to have as many disk seeks as you have levels.
I mean, there are, of course, edge cases as everywhere to whatever I'm saying, but in a general case, that's true.


Yeah. So now I understand like I made my storage engine. I have some sort of file. I split my file up into pages, then I'm going to use those pages to make this tree so that I can update, write things, insert things.
We haven't talked about indexes. So how would I represent an index? Okay. Well, first of all, the question is going to be, haven't we really talked about indexes? Are you sure about that?
So maybe we were talking about indexes all the time, in fact, and thing is that there is a very fine line between indexes and tables. In fact, it's very difficult to actually find the difference.
And some of the indexes are actually just tables. It's just like tables inverted by a specific key. So if you are thinking about like a table with some sort of a primary key, it's nothing but a bunch of data that is indexed by a primary key.


So in the end, this is sort of an index. So usually we're talking about index organized tables, which people usually say just tables. And whenever we're talking about indexes, we're usually talking about something, a structure that points to the index organized table.
So even here we have, we can have potentially like several distinctions. One of the ways to implement this is going to be an index that is holding a primary key, which uniquely identifies the record.
And the other one is storing a pointer on disk basically where the record can be found. And the difference between these two approaches is as follows. In first case, whenever we're storing the key of the record that can be located, we have to perform sort of extra steps, right?
So for instance, during the read, yeah, what we have like read like whenever we approved the index and we found that okay, the primary key of the record that we would like to find this such and such, then we go to index organized table and find these record, right.
But whenever we update, we have to go back to the index and update okay and say that these keys, or these values are not indexed this way anymore. So we sort of have to keep the index and sing with the index organized table.
Because they moved right. For instance, we were indexing people by their zip code right and somebody has moved. That means that a specific person that was previously found by a specific ID has moved from one zip to the other one.


That means that an index with like zip codes or index by zip code has to be updated and the record with this person has to be removed from the his previous or their previous zip code and added to their new zip code.
So that's one of the things. And if we once again take a different approach that I just briefly mentioned, and we are storing the pointer directly to basically where the data record distort.
Obviously, we will still have to update the index in order to say for instance, if the person has moved, so we would have to move it from one index bucket to the other one.
So this has no implications in this case. If we are storing the direct disk offset, we can just bypass searching the index organized table.
That means that we can, after we found the position of the record on disk, we can just you know go ahead and read it so we don't have to search this index organized table anymore.
So in short, coming back once again to the your question about indexes and whether or not we talked about them. So indexes are sort of same, you know, so you can actually have B3 B plus tree indexes.



Let me make sure that I understand the B tree example of indexes. So I have I have my user table, and I'm going to store that as like the B plus tree on disk. Say I have an auto increment ID or something right so my tree is sorted using the ID.
And that's my primary key. So I understand that so it's easy to look up my primary key. I want to add an index on zip code. So you're saying, in fact, it's all the same thing.
You just make another B tree, but now the key is zip code. So we have it sorted by zip code. And then I think you're saying the distinction is where that pointer goes to. So the pointer could either tell me the ID, which then I have to go look up again in the other B tree.
Yes.
Or it could tell me like the exact location on disk. And then each one has advantages and disadvantages, I guess, right?
Exactly. Yes. So you just described two use cases. I can actually add the third one. So what we're going to do this time is, let's just expand a little bit on your use case. So we have, you know, first name, last name zip code and, you know, some
important, automated ID. What we can do is create an index by zip code. And then instead of storing either pointers or IDs, we're going to store an entire, like rest of the record.
So we basically say, I don't know, your zip code, and then pretty much your name, your first name, your last name, and maybe even your ID for, you know, some extra information, or just for maintenance purposes. So you can do virtually anything because it's just a matter of how you're going to be accessing
things. So the advantage of the very first approach, meaning that whenever we build a index by zip code, and are storing just an ID of the record in the main table.
The biggest advantage of that is, of course, that we don't have to duplicate the data. Right. We have the record record just once.


Yeah. The disadvantage of this specific approach is that we have to look up the table, right? So the main table, whenever after searching the index. So we sort of do two, three traversals, potentially.
In order to mitigate this second tree traversal, what we can do is we can store the pointer to this structure from index directly the offset to the disk where the record is stored. But obviously, in this case, we're coupling these two tables
to tables very tightly. Then that means that whenever one is updated, the second one just cannot be out of sync, or we should come up with some sort of a mechanism that would, you know, gracefully fall back in case this page was relocated or record was removed or something different has
happened. So, and the third approach is whenever we are duplicating the data and in favor of, you know, either doing a second look up or doing anything else. So we are duplicating the data and we don't have to do this second seek.
So, yeah, these three approaches are, in my opinion, commonly used and some of the databases even allow you to explicitly, you know, configure these things so you basically can say what sort of index you can you're creating and in which way.
And as far as I remember, like what database lets you choose as far as remember even my skill, let me dig it out for you and if we can add it to the show notes later on, I will definitely do that.
And what I wanted to mention here is, there was a blog post, I think from Uber, where they were discussing exactly the difference, exactly that difference between, you know, different databases, I think was my skill and postgres and an implication of, you know, either storing a direct pointer or, you know, referencing things
through a level of interaction. So it was not exactly the same thing, but it was in sort of similar real, I think that it was related to addressing the values through the interaction table or in the primary
call all developers. There's no telling what you can create when you upgrade your data platform to intersystems iris. Are you ready to build the applications you want, however you want them? Are you ready to develop applications faster than ever? Collaborate, build faster and deploy more efficiently.


Tomorrow's next breakthroughs are waiting for you today. Intersystems iris data platform. Ready, set code. Start coding for free. Visit intersystems.com slash try to try iris.
There seems to be attention here, which I saw in your books well, like a read versus write efficiency tension. So where if you duplicate things into your index, right, it makes writing data or updating data more expensive, but reading faster.
So yeah, there is always a tension between reading and updating and in fact, there is a conjecture, which is called to rum conjecture. It states that we have three, like a triangle of things that are interconnected.
Read amplification, write amplification and memory amplification or read update amplification and memory amplification.
And the these three things are connected in a way that if you have an improvement in two of these, you will inevitably negatively influence the third one. So it's just saying that you cannot really have all three at the same time.
So you cannot be like as efficient as you would like to be in terms of both reads and updates and memory at the same time. So you have to make a compromise. You have to choose.
And the this paper about this rum conjecture is very interesting. And it brings up several topics that probably are of use for anybody who is who has something to do with either databases or backends in general.
And I definitely recommend to read it if you haven't yet. And one of the probably most widely known or discussed examples is of course the dichotomy of LSM versus B trees, meaning that B trees are allegedly heavily read optimized and the LSM trees are heavily right optimized.


And that sounds like a good transition, I think. So what is the LSM tree.
Okay, so yeah, now we have to talk about like yet another sort of the tree. If you take B tree, and then you would like to say, let's make the pages of this B tree immutable, meaning that whatever you make a change to the page.
So we are not going to, you know, find the same location in the file and try to override it. Instead, like every page is just, you know, going to be a completely new instance of the page appended to the end of the file.
Right. If we just do that, we end up doing so called copy on right. B trees, right, because whatever we want to modify the page.
So we're creating a sort of parallel universe of pages that are on the way from the root to the leaf of the note that we just updated. And then we create this parallel reality and then we swap the pointer to the newly created root and we have this new beautiful copy and write immutable
tree. So we copy all the values like instead of changing the page, we change one value, we copy the whole thing, change one value and change pointer. Yeah.
So this may sound as something like very wasteful, but because this allows you to avoid doing something like write ahead logging. This may actually make sense for some use cases.
And there are implementations of a copy of right B trees and one of the most prominent ones in my opinion is LMDB, the storage engine that is used in open LDAP.
And there are many talks that I've seen online on this subject and there are some presentations that you can dig out and it's also a very interesting thing.
So yeah, copy on right B tree is a thing and it's very useful and it's very interesting and it's sort of a step from mutable B tree, just like with immutability like all pages are immutable.
And if we go one step further and on top of just immutability, we also add buffering because you have noticed that like we didn't have any buffers in the immutable B trees because like we don't need to buffer any data in memory.
Therefore, we don't need the right ahead log and we can, you know, just create a parallel universe of pages and write them in new locations and this can this is pretty much our log.
I just want to make sure I understand this, if we're just doing some of the rights into memory, because we want to buffer it.
We want to write it to disk in one shot so we're just going to hold it in memory.
Then you're saying, we should also write it in a log so that if we if we go down, then we can recover.
Is that the idea?
Exactly.
So, but whenever we have this buffers and because they are memory resident and not disk based, or I mean, they will eventually be this base as well.
Just with the delay for that delay, we have to store this data durably somewhere, right?
And this is going to be in the right ahead log.
So this is one of the probably most important parts of the storage engine or most of the storage engines that I had to do with.
So you were saying we were doing a copy on right B tree?
Exactly.


Yes.
So we were trying to progress towards our sem trees.
And so yeah, we just got a copy on right B trees and we added in you ability to regular B trees.
And now we're going to add also buffering on top of that.
We are ending up with a thing that is called two component L.S.A.M. trees.
This is like a first thing that is mentioned in the ounials L.S.A.M. tree paper, which if you haven't read just yet, I definitely advise you to read.
It describes a database storage system or a storage engine, you can say that splits data in a following way.
Some of the data resides on the disk and some of the data lives in memory.
And whenever a part of like we say that the memory table is full, I don't know, for instance, like larger than five megabytes.
It's an arbitrary number, of course, and we take a segment of in memory tree and we take a segment of on disk tree.
And these segments store the same range of keys, meaning that keys from like a specific, for instance, letter to a specific letter, let's say from K to Z or something like that.
And we take these segments and we merge them and then we write them down on disk.
What that means is that at any given point in time, our database resides either on the disk or in memory.
And we migrate the data slowly from memory to the disk by these gradual merges, right?
And it turned out that this design had several inefficiencies and the paper progresses to explain like why two component LSM trees are not like good enough for whatever the authors were trying to achieve.
And they describe so called multi component LSM trees.
And so the authors further say that, okay, this is not good enough. So what we're going to do is we're going to compact these tables that we just flushed.
So whenever we have more than, let's say, four tables that we have flushed from memory and written to the disk, we're going to take all of them and compact them, meaning that we are going to merge them into a single, bigger table.
And every single key that was overwritten or superseded by like its next version is just going to be discarded.
And this way we reduce the amount of memory that we're using for pretty much all the data and we can reclaim all the space that we were wasting, so to say.
Why would I use an LSM tree like versus a B tree. So the traditional response or the response that nobody was fired for so far is, you know, this like nobody got fired for ever fired for picking Oracle so it's the same in this way.
Nobody was ever fired for saying that LSM trees are good for rights and that's sort of a traditional response to that. And one of the reasons to respond this way is if you think about a B tree, whenever you have to perform an update, you have to locate the place
where you're going to perform this right, like while you're writing, right, because you're trying to optimize everything for the look up, that means that if you have to find a place before you can make an update.
And of course LSM trees, because they are like built absolutely differently, they don't have that problem and whatever we are making it right, we are just doing a right to the log and we are appending this right to the memory table.
And eventually we know that things are going to propagate from the memory table and disk after that we're going to discard this right ahead log segment, we won't need it anymore because like the keys are persisted on disk in a different place.
Yeah, so it's interesting they're almost inverted, like conceptual processes like in one you have to do the sort when you go to update, you're like sorting through and finding where to update the other one when you do the read, it's like you need to find where the data is by looking through multiple
spaces. Exactly, so this is why I was saying at some point that it's sort of a dichotomy. But I think that LSM trees have like bad rap in terms of like they are bad for reads, like because everybody says that oh LSM trees are not good for
reads, I mean they can be very good for reads, in fact they can be potentially better for reads in certain use cases if you, well first implemented right and second use like all the optimizations that are available for you so I would say that a jury is out on this one still or
not, but it tells you that LSM trees are bad for reads, you can raise your finger and say, well actually, and it sounds like it's a newer area so maybe all the optimizations haven't possibly been found.
I would say that they were rather known for a long time but everybody just assumed that if they are so good for writes, they just cannot possibly be also good for reads, you know, well there's this rum conjecture I heard about.
Yeah but the thing is that there is also like this memory overhead that is like the third component so the third overhead sort of allows us to do but you're absolutely right so it's hard to run away from rum conjecture, and we definitely still pay the price in LSM trees in order to facilitate all the things that we would like them to have by simply
having to do compaction and maintenance all the time so this is where the amplification in LSM trees is coming from.
Yeah metaphorically it makes me think of like a filing cabinet and like the B tree is like I put everything in like a folder in the right spot and the LSM is just I just shove things in there but then when I need to read, I have to like sort through and fun.
Yeah that's a very nice parallel yes so obviously it's faster just put it in you know so I was wondering about examples so what like modern databases are using what uses B trees what uses LSM.
Most of in air quotes traditional databases are using B trees B plus trees, for instance, yeah post grass scale is using B plus trees or like a variant which is called like blank trees but this is something that that is more related to the way they implement like
current splits and merges rather than anything else but structurally it's pretty much the same thing Cassandra is using LSM trees and rocks to be that we have also mentioned today is also well Sam tree based and there are different databases
which are using sort of hybrid approaches so for instance there was a paper which was called Bw trees which is a immutable B tree which is like virtually built on top of Alice look structured storage Bw trees it's extremely
interesting I definitely advise everybody to take a look at it if you have a chance and yeah my skill is using B plus trees as well. It's not like the world has completely moved from B trees to LSM trees and B trees are like completely forgotten it's not the case.
I keep seeing both names popping up in new papers and both things sort of remain relevant. One thing you know when we talked about B trees, you know the kind of an implication was that we want to minimize, you know how often we have to chase a pointer on
the disk which I think goes back to like you know physical hard drives but we have like solid state drives now. Does that kind of change the equation at all. So we if we think about when B trees were implemented or a conceive so to say we of course back then we didn't have
a B trees so we had like spin and disc right on a spin and disc you have an advantage of being able to finding a physical point on the disk and actually overriding it so it's almost like you have a pencil and an eraser and
you come to the point where you scribbled certain things and you erase them and then you you know write things in new. That means that the mutability part of B plus trees or B trees actually made sense back then because you can actually mutate things on
the page. Okay, it may be slightly slow but maybe that was not the highest priority. Whenever we're writing to SSZ, we can write a single page but we cannot simply override this page in order to override it. We have to erase it first and then write it right in new page on its place. But we cannot erase a single page. We have to erase an entire block. And that means that if you have some of the pages in this
block still relevant, still used, and some of them pretty much discarded, you have to relocate them to the new space in order to be able to erase this block. This is this process called garbage collection and during garbage collection, you will take the block relocate everything that is relevant and erase it after that.
And of course, by the time that, or if you're doing extensive writing, at some point, you're going to run out of pages and run out of blocks which are empty and you will have to start this relocation process. And if you're cycling through a bunch of pages, meaning that you keep in order to make a single bite of change, you'll keep telling controller that is abstracting all of these things, you know, garbage collection.
And everything away from you, you're telling it, could you update this page? And it doesn't tell you, hey, you know what, in order to update it, I have to do a bunch of things.
It just tells you, obediently, of course, I am going to do that. And it does, of course, it does like garbage collection. But under the hood, in fact, it's doing lots and lots of work, which can be avoided, of course.
And one of the ways it can be avoided is exactly by, first of all, using the, or trying to write pages only whenever they're full. So we are not never writing the page, which is like half empty, or we are trying our best not to write a page that wastes half of its space.
And second, we're trying not to update the page in the place. So it's better for us to make two rights, and then wait for a while and then merge them eventually to some new location, then to update, like make several updates to the same page, just like in the short span of time.
And it makes it sound that the kind of log structured merge tree, or even the copy on right process would work better with the solid state drive. Is that right?
Yeah. So one of the reasons for instance, file systems use log structure storage as well is exactly to buffer, be able to buffer things and sort of whatever you have.
Many small rights, instead of updating many small segments, like in random locations, you just buffer them all together in a single, like, larger chunk of memory and then write it. So it definitely has huge implications.
So we started off talking about your interests and database internals, and I think we covered quite a lot of detail there. What do you think the future holds for database and database storage engines?
Funny that you asked. So I just recently was doing a summary blog post of the papers of 2019 that were about pretty much databases, storage engines, indexes and everything really do that.
And one of the trends that I've noticed is everybody is trying to move away from databases that are general purpose, meaning that database doesn't try to solve like all the problems at once anymore.
And I keep seeing more specialized stores and stores that are solving like single problem, but extremely efficient efficiently. However, this, that's only one part of defining the second part is that there is a trend to have a self driving databases.
For instance, people are trying to create a storage engines that are tracking what queries you're executing against them, and then they can adapt dynamically. For instance, they can change whether they're doing like index probing or sequential scans,
they can change certain properties of the pages, they can make pages larger or smaller depending on the queries that you're writing, depending on the keys that you have, or some of the databases can even take your data and migrate it behind the scene,
turn your database like from like OLTP to OLAP basically, or something like that. So, or like turn change the storage format even. So it's just that in an era of cloud computing, I would say it's difficult to be a database vendor that forces people to know too much about their database.
And the database has to know more and more about their users and I was really impressed by that trend and how not every single but very many papers are actually talking about the same thing and trying to make it simpler for users and sort of remove some of the decisions
that users are currently making from them altogether, and maybe in a cloud setting, some of the configurables are not even available for the people at all. So, so you cannot really, like if you use like some sort of cloud storage, you cannot take an instance and restart it because you're using it
via some sort of an API so you don't even have a notion of instance anymore, and you cannot really change its tunables, you cannot change like number of threads, concurrency and so forth. They have to abstract it all for you, and they have to do it in some intelligent way.
You don't have to think about how to develop the application in a specific way and specialize yourself in the database anymore so everything is going to be just done for you. Yeah, that's, that would be exciting, definitely.
Very cool. Well, Alex, thank you for coming on the podcast. I recommend everyone check out your book. I have it over here. It's very detailed. And it's kind of like a survey of the literature, I think, on databases, I would say.
Yeah, I guess we hit some high points of the early chapters, but it covers a lot of content, so I'm sure people would love to check it out.
For software engineering radio, this is Adam Gordon Bell. Thank you for listening.
Developers, take your marks. It's time to upgrade your data platform to InterSystems Iris. It's time to deliver complex mission critical applications in the fastest route possible.
It's time to use any data from any source. It's time to embed analytics and create interactive user interfaces. So what are you waiting for?
Choose your language. Choose your tools. Choose your environment. Collaborate. Build faster and deploy more efficiently. Done and done.
Tomorrow's next breakthroughs are waiting for you today. InterSystems Iris data platform. The fastest way to build applications.
Ready? Set? Code. Start coding for free. Visit intersystems.com slash try to try Iris.
Thanks for listening to SE Radio, an educational program brought to you by IJER Police Software magazine.
For more about the podcast, including other episodes, visit our website at se-radio.net.
To provide feedback, you can comment on each episode on the website or reach us on LinkedIn, Facebook, Twitter, or through our Slack channel at se-radio.slack.com.
You can also email us at team at se-radio.net. This and all other episodes of SE Radio is licensed under Creative Commons license 2.5. Thanks for listening.
