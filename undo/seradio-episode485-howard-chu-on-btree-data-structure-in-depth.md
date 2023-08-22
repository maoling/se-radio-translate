**Link** https://www.se-radio.net/2021/11/episode-485-howard-chu-on-btree-data-structure-in-depth/


This is Software Engineering Radio, the podcast for professional developers on the web at
se-radio.net.
SE Radio is brought to you by the IEEE Computer Society, the IEEE Software Magazine.
Online at computer.org slash software.
Welcome to Software Engineering Radio.
I'm your host Gavin Henry, and today my guest is Howard Chew.
Howard Chew has been writing free and open software since the 1980s.
His work has spanned a range of computing topics including most of the Ganyuil, utilities, for
example, GCC, GDB, GMake, etc. networking protocols and tools.
Kernel and file system drivers are focused on maximizing the useful work for a system.
Howard founded Simus Corporation with five other partners in 1999 and serves as its CTO.
His current focus is database-orientated covering LDAP, protocol LMDB and other non-relational
database technologies.
Howard, welcome to Software Engineering Radio.
Is there anything I missed in your bio that you'd like to add?
I think that covers it for now, yeah.
That's quite an extensive list.
Cool.
So today's show is on a free, meaty topic called B plus 3 data structures, and they keep
coming up in other related software engineering radio shows.
We touched on them in my first show I did for Software Engineering Radio with you on LMDB.
We did it some in episode 454 with Thomas Richter on Postgres as an OLAP database.
Simon Riggs on Postgres, episode 362 and database engines, episode 415.
Someone has to try and tackle the topic and I know you're an expert in it.
So here we go.
I think because it's so complex, I'm going to split the show into three parts.
We're going to dig into what three algorithms are, B trees, lay the foundation, move on
to B trees in a bit of depth, what's involved and then probably halfway through the show,
we should have enough laid down to tackle B plus trees.
So here we go.
I'd like to start with an overview of what a tree algorithm is.
Its terms and its history so we can lay a good foundation.
Then we was on to B trees and finally onto B plus trees.
So what do we need to understand first?
Well all right so these data structures are used to quickly search for items in a large
data set and the tree structure is used because it allows you to search more quickly than
for example just a linear list layout.
And how would you describe a sort of tree algorithm?
Is that where you can search things quickly or?
Well it's called a tree just because of the logical way the data is arranged.
You start with a central root point and then it branches out two different levels to cover
the entire span of the data range.
So if you just think about a regulatory in the ground, the root would be the trunk and
you turn upside down and think about it from.
Yeah so the computer science version of a tree is upside down from what you would see
in nature.
Yes.
Okay I'll keep that image in my head.
So there's a few times coming up that you know have a stab at so we can try and break
these up.
So what's a binary search tree?
Okay so that would be the most fundamental you know the starting point for most of these
tree designs and it's based on the simple search algorithm where let's go back to our
flat list.
Let's say you have a list of 100 items and you want to find one specific item in that
list.
The slow way to find it is to just start at the beginning of the list and check every
element until you find the one you're looking for and on average it will take you one half
of the number of elements in the list to find the item you want.
So if you've got 100 items on average it will take you 50 lookups to find what you're looking
for.
Okay so we can improve that drastically using a binary search.
In a binary search you take whatever list you have and you examine the element in the
middle.
Alright so if we've got our 50 items sorry if we have 100 items we look at the 50th item
and see you know is it the one we're looking for.
And sorry I forgot to mention that this only works if the list is in sorted order.
Alright so go to the halfway point you find the 50th item you see is this what I want.
If not you see say is it higher or lower than what I want.
If it's higher than what I want then I go and divide the next interval down so from the
middle of the 50th I go down to the 25th.
Okay so at each point if I don't find what I'm looking for I can either go higher or
lower and divide the next interval in half.
So that's the essence of a binary search and the way this improves things is at every
step you eliminate at least one half of the elements from consideration.
So it'll get you to the one you're looking for much faster than a simple linear search.
Excellent I think you've answered my question of why we need a tree algorithm in binary
search tree.
So is there a particular time you would use them or you know a certain type of data or
does it matter?
It could be used with any kind of data.
So the other thing that gets us to the tree structure.
So first of all what I described as binary search is how you execute it if you have
a flat linear list right.
But you can basically encode all of those having and subdivisions by rearranging your
flat list into a tree structure right.
So at the root of the tree you put your 50th element and then at the two children of that
you put on the left half say you put the lower 50 items and right half you put the upper
50 items and then at each level below that you subdivide and keep on branching out.
And the other thing to point out is every element every node in this binary tree has
up to two children.
Yeah I was just trying to visualize that.
So you just have imagine the root is a dot and then you draw a triangle shape with dot
at the top and then two dots on the other side.
So you're trying to get another two leafs to children as it were.
If you have the dot and draw a straight line down at most you're going to split that into
two and then have two more dots split them down again.
So maybe like visualize a deck of house cards you know when you're stacking cards.
Yeah very similar yes.
You've got the two cards pointing together the top with one card base and then every
end of the card is another two points of a card isn't it.
If you're looking at that front on you'd see the root structure in the two points.
Yeah exactly.
Yeah that's good.
I just pick it down the spot.
It's a good analogy.
So you're asking why do we need them or when do we use them?
Yeah how and when is it used?
The original the generic binary tree that we just described you know these days it's
only used as a teaching tool to give you the idea of what you're trying to accomplish.
In practice the plain binary tree has a lot of management problems.
So it really isn't used in real life.
It's used in computer science classes to teach you about binary searches.
Okay well at least it helps us discuss the terms which we've touched on a root being
the base of that tree or the top of the house of cards.
And leaf is that joining a root to the next node?
To the next.
To the bottom most a terminal node and anything between a root and a leaf is generally called
a branch.
Okay okay so is that enough foundation to understand what a B tree is to think or should
we ask about it?
That covers binary trees.
So as I'm alluded to you know there are some problems when you work with just a plain binary
tree and that would lead us into discussing B trees.
Would you want to talk to some of the problems?
I know it's academic but.
Yeah well the first thing is you know it's easy to construct a binary tree if all of
your input data is already sorted when you begin.
But if you have a very naive algorithm that says for example you start with a node with
any particular value and you want to insert the next node and you could say is the new
node less than the current node or is it greater than the current node and if it's less you
make it the left child and if it's greater you make it the right child.
If you actually start with a perfectly sorted input data set then this very simple binary
tree construction algorithm will actually give you a linear linked list because every
new node will always be either greater than, okay say you have a list that's sorted in
ascending order then every new node will always be greater than the one that preceded it.
So every time you do an insert you'll always insert onto the rightmost branch and so you'll
get a degenerate tree which actually looks like a straight line which is not very useful
for the other search property so the tree.
Yeah because the left leaf is the lower side and the right leaf is the upper side so if
ever since higher than the previous one then you're only going to be popular in the right
side aren't you?
That's right yeah you'll never populate the left side.
So this is one of the problems with a plain binary tree is that it's very easy to get
into these degenerate forms that aren't balanced right and if it's not balanced then you don't
get any advantage of the binary search properties.
Yeah well you split in half and look at the higher or lower because there's never a lower.
Exactly.
So you'd be better off just going by to the original way and counting.
Yeah and so that's the most fundamental problem that motivated the development of something
like a bee tree.
And this isn't new is it?
It's been around.
Yeah it's been around at least since the 1970s.
I think the first published paper is 1971 or 72.
Okay just before we move on to what bee tree issues solve can we just revisit a root leaf
and a node?
Was that the right word?
Root, leaf and...
They're all nodes right there's a root node, there's a leaf node and there are branch
nodes in between yeah.
Yeah so a leaf is yeah you describe it again I won't get it away.
So all right so the root is the other starting point right and the leaf is a terminal point
there a leaf has no children.
Okay yeah I might more sense.
It's the end of the line right and anything that is you know interposed anything that
connects between the root and the leaf is a branch.
Okay and a node can only have two branches.
In a binary tree yes a node can only have two branches.
Okay and if it has no children it's a leaf.
Yeah a node can have up to two branches it could have zero it could have one.
And that could be a right or left depending on whether the value is higher or lower.
I think we had a good start but covering the basics and the history there.
So if we feel ready let's move on to the main types of bee tree.
So what is a balanced tree if we could revisit that?
Okay so you know we mentioned that you know if you just naively construct a binary tree
and you have data of a particular arrangement for example sorted in ascending order then
you won't have a very good tree structure but it won't be balanced and it won't give
you any search advantages.
So you know this problem was recognized pretty early on and people thought about ways to
make the tree structure self-managing self-balancing so that as you insert data to it you know it
always manages to check its own status and rebalance things as necessary.
And rebalancing in this case means actually changing the shape of the tree.
For example let's say I construct a tree with three nodes right and I'm just inserting
the values one two and three.
And if I start naively by just saying I insert node one and then I insert node two and since
two is greater than one I put it on the right child and then I insert nodes three and three
is greater than two so I put three as the right child of two.
Okay so now I have a three node tree that's a straight line right and a rebalancing operation
in this case would look at that tree and say well that's no good.
We're going to rotate it left by one position so that node number two becomes the root and
node number one becomes the left child and node number three is still the right child
of node two and then we have a perfectly balanced tree.
Yeah I'm thinking of maybe totally a help but you know any good website and they've got
one of those looks like a molecule structure when your mouth hovers over it and moves about.
Kind of like one of those where you've started with the root as one and then two three so
you've just dragged number two and put that at the top and then you've got one below it
and three reforming the tree structure.
Yeah I get that that makes sense.
So what we just described is a balancing operation and the idea is to develop an algorithm that
manages a tree structure so that it balances itself automatically so that it can do the
analysis that we just described and just automatically keep a tree balanced and the B tree is actually
only one of several ways that have come up in computing to solve this problem, to solve
the self-balancing problem and two other ones that are very common.
Again they're taught early on in computer science courses is a red black tree and an
ABL tree.
Let me just chat about those two.
Where we talk about an algorithm that rebalances the tree some practical way of that might
be there's an insert value and part of that function looks to see what is there already
and then makes some decisions and gives you an okay but behind the scenes it's reject
things.
Is that how that sort of looks?
Behind the scenes it's rebalancing?
Yeah.
So you have an API that you're using to insert a child or a value and then part of the algorithm
and the library that you're using looks everything behind the scenes and rearranges it but gives
you like a return value of one or okay or something.
Yeah, you know generally it's for example in the case of a B tree it will attempt to
do the insert first and then fix up anything that needs to be fixed up afterwards.
Okay, so it'll take the data as quick as possible and then.
I mean you know the the effect is the same because you know the caller will have to wait
for whatever cleanup or rebalancing to finish before the API returns.
Okay, just like a normal API.
Okay, since you mentioned those are the two types of trees that you're familiar with.
If you can give us a bit of information Valen, that would be great.
Well they're both improved versions of a binary tree.
For example in an AVL tree, first of all we were talking about binary trees where we inserted
a node with a simple value, right?
Node with a value one, a node with a value two and a node with a value three.
In an AVL tree we add a little extra information to each node.
We call it a balance factor.
So when you insert a value into a node or into the tree goes into a node and the node
starts off with a balance factor say of zero that says it that means it's balanced.
And if you add a left child to it it might get a balance factor of minus one that says
okay there is something on the left side.
Or if you add a child on the right side it might get a balance factor of plus one, meaning
that it's got a child on the right side.
And the algorithm that maintains the tree will go up and down examine these balance factors.
And if the balance factor ever exceeds plus one or minus one like if the balance factor
gets to two or it gets to minus two then it knows oh this tree is now too heavy on one
side and it will know that it needs to rebalance it.
That's the basic idea behind an AVL tree.
And is that attribute value something what does the AVL stand for?
As I recall they were the initials of the people who invented it.
Okay, so just the name.
A red black tree is actually named, they maintain balance factors as well but they name them
as colors red and black.
It's the same general idea.
Okay, Bora, how do I say this?
MRA search tree.
Okay, so far we've been talking about a binary search tree where every node can have two
children.
Right, and MRA tree is where any node can have.
That's just collect and children.
Sorry to interrupt.
That's just collect why it's a binary search tree because there's only one of two values.
Yes, this is more from me this episode.
So binary search tree is branch factor of two.
Any node can have up to two children.
And MRA tree is just the more generalized form where any node can have up to M children
or M is whatever arbitrary number that you wanted to assign it to.
But they have to be equal, don't they?
They have to all levels of the tree need to have up to that amount.
That's right.
Yeah.
You know this, we're going to start diverging soon from typical computer science teaching
on this as we go a little further into this.
But for now, okay, we're talking about B trees and a fixed branching factor.
And branch factor being the maximum number of children a particular node can have.
Yeah, we should probably just stick to B trees.
Okay.
So I think we touched upon logarithmic and bigger notation in our first show together.
And probably quite a few other shows we've done on software engineering radio.
We'd just like to give a summary of that because that's what this all brings us, isn't it?
That's how it's all measured.
Sure.
Well, yeah, the big O notation basically, you'll see the capital O and parentheses and some
factor inside.
And that means, you know, semantically means on the order of, right?
So if you say something is big O of log n, that means the complexity of that function
is on the order of a logarithmic value, a logarithm of the number of elements.
So if we look at our very first example of the linear search, you know, in big O notation,
that's big O of n, or the complexity of searching that element, searching that structure is
basically equal to the number of elements in the structure.
So it's a formal notation to let you know how fast or slow something's going to be?
Well, how complex something is, which usually means how fast or slow it is.
Yeah.
So the larger the number, the slower it is, and so, you know, logarithm of n is always
smaller than the value of n, and it increases much more slowly than n itself increases,
right?
And so that's a very desirable property in algorithm because it means the complexity
grows very slowly, that means as data volumes increase, the performance decreases very slowly.
You know, ideally you would want the performance to not decrease at all, but that's...
Yeah, and this is normally the graph that starts level and then curves up near the end
or the other way around.
When you see...
Starts and curves up slowly.
Yeah.
Yeah, that's right.
So B-tree is a balanced tree.
That's just a short way of it, I'm saying, isn't it?
Yeah, it is one of the self-balancing binary trees.
Yeah, except it's not necessarily binary, right?
It's an area, it's an arbitrary factor.
Okay.
And is there a proven statistic of how fast they are?
You know, if you follow that structure?
Well, it is a logarithmic algorithm, you know, it's big O of log n.
And the base of the logarithm just depends on the M, the branch factor.
If you have a binary, if you have a B-tree that has M equals 2, then it will be to log
in base 2, if you give it a branch factor of 4, then it will be O of log to base 4.
And how would you decide between a binary tree and, you know, instead of 2 nodes?
Some larger factor.
Yeah.
So that is really...
That's really a judgment call that needs to be tuned for the use case because one thing
you're trading off a couple of things here, the more branches, the more elements you can
pack to a node, the more branches you can put to a node, the fewer nodes you need.
Okay.
Yeah, that makes sense.
But you know, the larger a node becomes, you know, the more work it takes to maintain the
pointers inside.
So like if I have a node that has 100 elements, that would be slower to manage than the node
that only has 4 elements.
Yeah.
And if the...
If you're doing...
I think anyway, this is just how I'm understanding it.
If you've got a binary tree and the node's just got 2 children, and the right leaf is
the higher value on the left, but then if you go to 4 or 7 or wherever, you've got to
then order all those in the right way, doing each node.
Yeah, you have to keep all of them sorted and you have to...
You actually have to execute a binary search within that node to find what you're looking
for.
Yeah, so...
And you've got to keep that on in your head as you're thinking about it when you write
the code.
Yeah.
Generally, you're going to find, you know, in examples, in textbook examples of a bee tree
that, you know, they'll pick some fixed number and never worry about it again.
It'll be like, okay, we're going to do 8 nodes or 16, 16 leaves per node or whatever, and
then just forget about it, just set it and not worry again.
And is it something that you would come back and revisit at some point?
Well, you know, we're talking about the general bee tree here.
When we get to bee plus trees, you know, those considerations go out the window.
Okay.
So we've spoken about a bee tree.
We know we're going to move on to bee plus tree, but there's a bee minus tree.
Can you tell us a bit about that?
No, a bee minus tree is just another way of writing bee tree.
Maybe it's a typo of mine.
Okay.
We've covered logarithmic time.
So a bee tree is well suited for storage systems that read and write relatively large blocks
of data, such as disks.
Do you agree with that?
I think that's more suitable for bee plus tree.
A bee tree, the general form of a bee tree, it can be tailored for block storage.
And the idea, again, is you make your nodes the same size as your disk blocks.
Okay.
Yeah.
So this is the actual link of getting back to file systems and why it's used there.
Yes.
Okay.
That makes sense.
If the block size, which nowadays is 496 kilobytes, isn't it?
4,096 bytes.
Yeah.
And that would be your biggest leaf node.
What would that be?
That would be the size of your node.
Yeah.
Yeah.
In the bee tree, all of your nodes should be the same size.
Okay.
So let's start the big one.
So just to recap, we've gone over a binary search tree.
So that's a root node.
And then children, each leaf can have two children left and all right.
The right's generally the higher value in the left is the lower.
Everything's sort of founded upon that.
We've moved on to balanced trees.
So there's always a left and a right.
We've discussed Big O and logarithmic time and the current uses for bee trees.
Now we're going to move on to bee plus trees.
Right.
So let's start again.
What is the bee plus tree data structure used for today?
It finds a lot of use in all kinds of data management applications.
For example, in many, say, SQL databases, relational databases, the bee plus tree is almost
always used for indexing.
And in some cases, it's also used for the primary data tables.
And why is that?
Well, it's always used for indexing because it is a structured, it's an ordered structure.
Right.
So it's fast to look up elements in.
And it lets you move sequentially through the data set.
So if you want to retrieve elements in this sorted order in a particular range, it's very
well suited for that.
And for the primary data sets, it's not always used because generally bee plus tree has range
limits on the sizes of items you can put into it.
So while you might have data with relatively small keys, say, 8-byte integer keys, if you're
attaching, say, a 200 megabyte value to that key, it may not be so easy to manage in a
bee plus tree.
So you might use a different data structure to store the actual data that goes with the
keys.
Okay, what relation, say, is a simple key value store you're trying to implement?
What relationship does the bee tree structure have to the key and the value?
Is the node a key or is the key just a bit of data in that tree?
Okay.
So if we back up and look at what we talked about in bee trees, we actually were using
a much more simplified example where a node only contained a single value, like an integer,
right, 1, 2, or 3.
So in that case, in the terms that you're talking about, it would be a key only database.
Okay.
Yeah.
Because the only thing we were storing in there was a searchable value that was a key.
In the case of the bee plus tree, we have those keys, those numeric keys, but we can
also associate with them an arbitrary blob of data.
And in a bee plus tree structure, the only beliefs of the tree actually store the data
values, all of the root and the interior nodes will always store only the keys.
Okay.
So you know, the key coming in, for example, in a database search, we've got a select
and because you've got an index created for where such an equal something, then you know
that's the key.
So then you can reliably predict how quickly that's going to take.
You know, it's going to be fast.
So if you find an index, which will then recover the value, which is the row where the information
stored.
Yeah.
And again, you know, if you're using a bee plus tree only as an index, then the value
that you get back is just going to be a pointer to the data that lives somewhere else.
Right.
Yeah, I think I'm getting there.
Okay.
And again, there's the alternative where you actually use the bee plus tree to store
the primary data as well.
You know, in that case, the value that you retrieve will be, you know, the complete
data item that was associated to that key.
Yeah.
Because if you've got a quick look up and the index, you want still want a quick way
to find the actual data you're asking for as well.
So you would, you could potentially have the two bee plus trees, like you just said.
Yeah.
Okay.
We can move on to question three.
So what extent is adoption driven by some of the US hardware, like SSDs or your types
of memory for this tree structure?
Do they have to change it for where the blocks are stored or anything like that?
Or is that so transparent?
You know, it's a lot of the answer that question depends on what the operating system is doing
with the different memory structures or the, you know, the newer types of memory as you
were talking about, whether it was persistent memory or like obtain memory or some other
things like that.
You have the option of making that memory transparently usable or you have the option
of making it an explicit block device.
So it really depends on what the operating system is done.
One of the things that we didn't state explicitly here is that in the bee plus tree, again,
we're talking about nodes or nodes that are the size of disk blocks, right?
And in the bee plus tree terminology, we're going to call those pages.
So one page of a bee plus tree equals one page in a file system, one page on disk.
Yeah, I think I'm thinking back to the LMDP show, we were talking about CPU caches and
all sorts of things like that, keeping the data that can be held in cache small and nice.
It's what you're trying to do here, isn't it?
Yeah, that's why we only store keys in the interior pages and keep the actual data at
the leaf.
That means you can fit the maximum proportion of the key index in memory.
So you're not always having to go back to the file system to get the keys.
Right.
Yeah, so when you're doing key lookups, they can all be in memory and very fast.
And then when you actually get to the bottom of the tree and find the value that you want,
then you might actually have to go out to disk or some other slower storage.
And that's the idealized case, right?
But obviously, if you have a very small database, it could all be entirely in memory.
And if you have a very large database, you could wind up with portions of the interior
index also pushed outside of memory.
And when we're talking about a B plus tree data structure, obviously that's a thing.
But what does that thing look like on a computer or a hard disk?
Well, there's certainly a lot of leeway in how you construct it at the bits and bytes
level.
And conceptually, it's not much different than the B tree, right?
You have a root page and the root page can have some number of pointers in it to children.
And in this case, as I mentioned before, we no longer talk about it as an M-ary tree.
It's as many child pointers as will fit in a page.
So it's not a fixed number.
It's just you cram in as much as will fit before you have to overflow into a new page.
Okay.
I'm still asking some really basic questions here.
So is this data structure constructed from something on a file system or in a text file
or something like that?
How do you ask?
My question is coming from thinking about the data.mdb file, you know, LMDB great in
the first show we did together.
What's inside that?
What's inside?
Well, that is just a sequence of pages.
And each page has pointers to other pages.
Okay.
Yeah.
I think I know.
So in the textbooks, they will give you an example of a B plus tree where all of the
dimensions are fixed constants.
It'll say you're going to create a B plus tree where every key is eight bytes long
and there are eight keys per page.
That's it.
You're done.
In the real world, keys can be various sizes.
And because the sizes vary, that means the number of items that you can fit per page
also varies.
So this is where like a database engine like LMDB departs from what the textbook tells
you.
Because the textbooks give you a very simplified version where, you know, there are no variables
and the reality you need to accommodate, you know, keys and records of varying sizes.
Yeah, real world data.
Can you revisit and provide a simple description of a B plus tree?
Okay.
So that's a thing.
Yeah.
It's, we'll start again.
We have a root page which can have, for example, you know, a list of up to eight pointers to
children, right?
And the children will be pages as well.
So all of the pages are always the same size.
So a branch page again points to more children.
And then at the bottom of the tree, you have the leaf page, which points to actual data.
And within each page of the root or the branch pages, you have the only thing that you store
in those pages are keys, keys and a pointer to the next level down.
And the next thing that you're looking for, does that make sense?
Yeah.
I was just following it down my brain there.
And B plus tree is a replacement for B trees or they just different auctions?
Yeah, I would say it's a replacement.
You know, you find B plus trees being used everywhere and B trees not so much.
Okay.
Well, we don't really need to discuss trade-offs between them.
If everyone just uses B plus tree.
We touched on it before.
You know, the main reason is that by keeping all of the data values at the bottom leaf
pages, you get more keys in memory, which should be the smaller structure.
So that's right.
And we've touched on our GBMSS SQL databases that use them for primary keys and actually
I'm looking up the data that the keys point to as well.
Can you think of any bigger products, examples that use them or mainly just indexes?
Yeah.
Well, you know, for example, SQLite uses B plus trees for its indexes and for some of
its tables as well.
Obviously open LDAP uses LMDB.
So that's using B plus trees for indexes and for, you know, main data.
The other popular databases, Postgres, is based on B plus trees.
MariaDB or MySQL, they're actually using something called ISAM, completely different
structure.
Yeah.
That's right.
I'll see you in that a few times.
Are there any places that you think could benefit from B plus trees where you don't
see them?
I mean, I'm mainly in the database world.
I'm going to guess that, you know, the database world has been around this for long enough.
You know, they know pretty well where they can leverage it.
I would say in databases, they're already using it where it needs to be.
And have we touched upon the term height?
Let me talk about leaf nodes.
Height.
Well, height is just, you know, a description of, you know, how many levels of pages are
in the tree.
Yeah.
Yeah.
So how big the house of cards could be?
Exactly.
Yeah.
So if you have a tree that's just a single root page, then you've got a height of one.
There's nothing going on.
You know, if that root has just a bunch of leaf pages immediately under it, then it's
got a height of two.
If it's got a root page and then some branch pages and then immediately under that some
leaf pages, then you've got a height of three.
So.
Okay.
And do you have any tips on how to keep where you're working on in your head while you're
dealing with these?
Best to visualize things, write some sketches down or you just get used to them when you're
doing it.
I suppose sketching on paper when you need to, but you know, when you're using a library
to the mar, it doesn't matter.
You know, when you're using the library, all you care about is here, insert this key and
here, find this key and give me this value.
So yeah, you don't really care what the structure looks like underneath.
And you did your own B plus tree algorithm, didn't you?
For RMBP, yes.
Yeah.
You obviously know the front end and the back end to that thing.
Were there any lessons learned or tips or warnings to never do it yourself?
You know, if someone's thinking about it, what's your takeaways from actually doing the B plus
tree algorithm yourself?
First of all, as I mentioned before, what you learn in the textbooks is only a very
simplified version of what you're going to need in the real world.
So if you want to write one of these yourself, you'll be prepared for the textbooks to not
tell you everything you need to know.
And then you really need to have an extremely good reason to write this yourself because
they're at, you know, at this point, there are many libraries available and you probably
should just use one that exists off the shelf.
Yeah, I think it's the old profile and badge mark or on look at stuff as and when.
If there's an issue with something that doesn't have to trust that person or that company
or team knows what they're doing.
Right.
You know, ten years ago when I started writing LMDB, the things that we wanted it to have
obviously did not exist off the shelf.
So that prompted me to write it, but I'd say it's hard today to find a use case that's
not well covered by an existing library.
Yeah, I think people are doing new things, tend to reinvent things and forgot that there's
probably something already there that was designed 20 years ago.
They just haven't done enough research.
There's well, and there's so much tendency to just reinvent.
And you're quite strict with yourself for trusting someone else has done it better than
you.
You know, you've got a very strong reputation of speed and performance and getting the best
hour things.
Have you had to have words with yourself to not go down that route?
It's always the last resort.
Always, always I search for something that exists that, you know, that would fit my needs
before sitting down the right my own tool.
Yeah, I think that's a good advice for any of the listeners.
So I think we've touched on a couple of my questions already.
I was going to ask why B plus trees are used for indexing and table indexes?
That's because I'll try and recap.
They've got predictable times for searches and finding things and they're very efficient
at key value operations.
Yeah, they're actually optimal for searches.
You'll always get the lower bound of search times.
And okay, so with the B plus tree, is it complicated to get the data into it in the
first place?
How would you put a ton of data into a B plus tree structure?
Is it complicated?
Well, you know, it's all shown.
So yeah, it's complicated.
There you go.
It's certainly, you know, it's not as easy as the plain binary tree.
It's not as easy as some of the other self-balancing trees.
But one of the advantages, again, that it has since it uses these relatively large pages
is, you know, as you insert records into it, it has a fairly, fairly low rate of data.
Of new allocations.
But every time you allocate a new page, it's good for, you know, 20, 30, 50 elements before
that fills up a new force to allocate another page.
So it's got pretty good efficiency in that respect as far as resource constraints.
Okay.
That makes sense.
I've got a question from one of the other hosts here.
How do we handle concurrency control and data bit basis that use B plus tree?
There's a lot of different ways to handle that.
So that would be somebody to processes trying to insert data using the B plus tree library
and it hasn't quite finished, putting that data in the right place.
So that's the easy way to understand it.
So there's, this is one of those topics that, you know, like I said, it has multiple different
approaches.
Okay.
There's the very simple approach, which actually LMDB uses, which says we lock the entire tree
for one writer.
And we don't let go of it until that right is completely finished.
So in that sense, we actually have no concurrency for rights.
There's always only one writer at a time.
And as I said, you know, this is very simple and it's a decision that you've thought about
changing at any point.
No, no, that's that's a decision that was reached after quite a bit of experience with
other concurrency schemes.
And did you have to make a trade off for that or make a statement to say off in some ways,
you know, as you get to much larger databases, you might see that your overall right throughput
could be slower than optimal.
It really tends to depend on the right patterns.
Another approach that, for example, BerkeleyDB uses is to use lock coupling.
That is, as you navigate down the tree from the root to the leaf where you need to insert
data, you take locks on every page that you touch on the way down.
So then when the next right comes through, it might be trying to put it somewhere else.
It doesn't move.
If the next right coming through is going to a different part of the tree, then it's
possible for the two of them to proceed without interfering with each other.
When you traverse the tree to find the algorithm, it's trying to figure out where to put the
data in the tree.
Does it always need to go straight to that point or does it have to do a similar search
each time?
Generally, you always search from the roots for every new operation.
Okay, so there's no quick way to say this is the last place I put something.
This new one is best placed here.
Well, actually in LMDB, we do have an optimization that does that.
Yeah, we're putting in a couple of elements that sort close to each other or sort next
to each other.
LMDB will check that before it goes back to the root.
So you have a cursor that points to where you're inserting data.
When you insert a new record, it'll look at what the cursor is currently pointing at and
see if it spans the space that the new record belongs to and if it does, then it'll just
stay on the current page.
And if it doesn't, then it'll fall back to what it has to do.
It'll go back to the root and search its way down.
And do you fall back to searching the whole root to last leaf if that cursor breaks or
something happens to it?
Or do you just trust the fact that it points to the last place something was put?
Well in LMDB, since there's no other writers interfering with it, then yeah, the cursor
always points to it.
So that's another good reason to have one writer.
Exactly.
Yeah.
As I said, it simplifies so many things that, in my opinion, it wasn't worth the headaches
of maintaining all the other state needed to do multi-page locking.
Yeah, it's like having a M plus one backup for the backup for the backup, the backup.
You know, how deep do you go?
How deep will you go?
That's one of the problems with the approach in BerkeleyDB is that you can get into lock
conflicts very easily.
You know, lock conflicts are not an exceptional, unusual condition.
They're a routine occurrence in BerkeleyDB when you have multiple writers.
So every time you hit a lock conflict, one of the writers has to give up, release all
of its locks, and retry later.
So as I recall, sometimes if there was a crash, there'd been blocks left.
Yeah, that's another aspect of things.
That's to do with writing in place versus LMDB's copy on write.
That's not quite related to the locking issues.
So for concurrency management, you know, that's one approach that, and that's the most common
approach that B-trees use is the multi-page locks that Berkeley uses.
There's another method which, you know, was relatively recent.
I was reading about it back in 2009 called a B-link tree where leaf nodes contain link
pointers to their siblings.
And by maintaining these link pointers, it actually allows you to do ads and deletes
with high concurrency without as much lock overhead.
So you know, this was a really interesting mechanism when I was reading about it.
Unfortunately at the time, it was very new research, and there were no working implementations
that I could find anywhere on the web.
And you know, the research paper that I read didn't provide me enough information that
I could implement it myself.
So I kind of set it aside.
But it was a very promising paper at the time.
I think in the years between then and now, there have been at least two implementations
of it developed.
Of course, while it solves some of the locking problems that we were just talking about,
it doesn't solve the crash problem that you just mentioned, where if something crashes
in the middle, then pages will be lost.
Yeah, and what was the last place written to and who was doing more?
Yeah.
Do you think there's, you know, you just touched upon improvements that could be made for these
types of data structures.
Do you think in 2021 we've reached the predictable times for searching things and the
software algorithms, these data structures, and we're now relying on hardware?
Or do you think there'll still be some major breakthroughs?
Because I know you stay on top of all this stuff.
You know, I see a lot of research into new algorithms that are supposed to be tuned specifically
for persistent RAM or SSDs and thus and such.
I find that, you know, the research I've read so far is not making any massive breakthroughs
and I don't, you know, from that trend, I'm not expecting anything huge in the future.
There's a lot of small-scale refinements is really what it looks like, is what it amounts
to.
And what do you think the future is for this type of data access?
Well, it's going to be more of the same.
You know, today we have a multi-tiered memory hierarchy, right, where we talk about CPU
caches and then RAM and then, quote, storage, which is disks or SSDs or whatever.
In the future, we're going to have a couple different tiers of RAM.
We'll have static RAM, dynamic RAM, persistent RAM.
I would like to see further in the future, you know, a hundred percent persistent RAM
and no more plain dynamic RAM, but, you know, we're not going to get there yet.
We're going to have this intermediate step first where we have dynamic RAM coexisting
with persistent RAM.
And so that will be another layer of the memory hierarchy.
And it'll just, it's just one more level of appearance.
But that won't change the page sizes that we're dealing with.
And it doesn't have to.
It doesn't have to.
So the same algorithms could still work, but because the access speeds are faster.
It should still be predictably the same speed or?
Yeah, a generic B plus tree algorithm will still provide, you know, optimal search performance
as you layer these different memory technologies above each other.
So that really doesn't need to change.
Page sizes will increase, as you mentioned, you know, four kilobyte pages are already
a bit small for today's CPUs and disk subsystems.
16 kilobytes is more reasonable.
32k, 64k would probably be pushing it.
Yeah, 4k is a bit too small.
And is that some size you have to pick in the B plus tree structure or?
From the software side, you have to use what the hardware supports.
So some CPUs today, for example, apples and one processor are now using 16 kilobyte pages.
So that's the page size that your database reviews as well.
So that means you can have a shallower tree.
Yes.
Because you're storing more value or bigger values or pointers to the values more values
per page.
Yeah.
And which again, speed up getting things because it's not so deep.
OK.
Have you ever really been happy with your project management tool?
Most are too simple for a growing engineering team to manage everything or too complex for
anyone to want to use them without constant product.
Shortcut formerly known as clubhouse is different, though, because it's worse.
Wait, no, we mean it's better.
Shortcut is project management built specifically for software teams.
It's fast, intuitive, flexible, powerful, and many other nice positive adjectives.
Let's look at some highlights.
In-base workflows.
Individual teams can use shortcuts to default workflows or customize them to match the
way they work.
Org wide goals and roadmaps.
The work in these workflows is automatically tied into larger company goals.
It takes one click to move from a roadmap to a team's work to individual updates and
vice versa.
Tight VCS integrations.
Whether you use GitHub, GitLab, or Bitbucket, Shortcut ties directly to them so that you
can update progress from the command line.
The rest of Shortcut is just as keyboard friendly with their power bar, allowing you to do virtually
anything without touching your mouse.
Throw that thing in the trash.
Interrations planning.
Set weekly priorities and then let Shortcut run the schedule for you with accompanying
burn down charts and other reporting.
Give it a try at shortcut.com slash seradio.
Again, that's shortcut.com slash seradio.
Shortcut, again formerly known as clubhouse, because you shouldn't have to project manage
your project management.
I'd like to start wrapping up.
I think we've done a bit.
Try a good job on that.
Let's just round off with the description of a B plus tree again to grasp the little bit.
Correct me on this.
This is a knobbed off internet.
Just confirm for me.
The B plus tree copies of the keys are stored in the internal nodes.
The keys and records are stored in leaves.
In addition, a leaf node may include a pointer to the next leaf node to speed sequential access.
Are we calling the nodes pages there?
In that place, yes, the nodes are pages, yes.
Why did it make sense to use the word page when everything else uses nodes for the previous
data structures?
The word page is associated with virtual memory management and disk management.
That's what we're trying to improve here.
That's where laying the data on top of the desks.
What the?
Yeah.
SSDs.
Excellent.
So obviously, selection of the correct data structures to use is extremely important to
us, the software engineers and companies.
But if there was one thing from this journey of an episode, you would want our listeners
to remember.
What would you like that to be?
I would say most of the time for your data management needs, you're just going to need
a B plus tree.
You can ignore anything else.
Perfect.
And don't try and do it yourself.
There should be a library for it.
Yeah.
Um, there's only thing we missed that you'd like to mention.
We did cover quite a lot.
We covered a lot of rounds here, so we're good.
Excellent.
So where can people find out more about your B tree and plus?
Yeah, I get it right.
B plus tree work or what you're working on.
Obviously, we've got your Twitter account that we'll link to.
I'm sure if people are interested, they can look into the source code of LMDB and get
the head round how you've done B plus tree.
Sure.
Yeah.
And also the LMDB.tech website has the formatted documentation online.
So you can read through that there while you're browsing through the source code.
Thank you.
I'll put that in the show notes.
So this show personally has really helped me understand B trees.
Hopefully the listeners will come off a little bit better than when they started listening.
I hope they're less confused.
Yeah.
Howard, thank you for coming on the show.
It's been a real pleasure.
This is Gavin Henry for Software Engineering Radio.
Thank you for listening.
Thanks for listening to SE Radio, an educational program brought to you by Azure Police Software
magazine.
For more about the podcast, including other episodes, visit our website at se-radio.net.
To provide feedback, you can comment on each episode on the website or reach us on LinkedIn,
Facebook, Twitter, or through our Slack channel at se-radio.slack.com.
You can also email us at team at se-radio.net.
This and all other episodes of SE Radio is licensed under Creative Commons license 2.5.
Thanks for listening.
