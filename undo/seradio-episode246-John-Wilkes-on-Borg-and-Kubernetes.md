**Link** https://www.se-radio.net/2016/01/se-radio-show-246-john-wilkes-on-borg-and-kubernetes/

This is Software Engineering Radio, the podcast for professional developers on the web at
sce-radio.net.
SC Radio brings you relevant and detailed discussions of software engineering topics
at least once a month.
SC Radio was brought to you by IEEE Software Magazine, online at computer.org slash software.
Hello this is Charles Anderson for Software Engineering Radio.
Today we are talking to John Wilkes about Google's board system.
John has been at Google since 2008 where he was working on cluster management and infrastructure
services.
Before that he spent a long time at HP Labs becoming an HP and ACM Fellow in 2002.
He is interested in many aspects of distributed systems but our current thing has been technologies
that allow systems to manage themselves.
Is there anything else you'd like to add John?
No that's great, thanks I just hit my 7 year anniversary Google and still having a blast
here.
Oh, most excellent.
So at a very high level what is board and what does it do at Google?
Great well let me first start out by saying we have to officially say the system that
is internally known as Borg to keep the lawyers happy but now we've done that once we can
carry on.
Borg is the system we use to manage our internal clusters of machines.
Sort of what is primary purpose is to serve our internal users and the folk would write
things like Gmail or Google Docs and give them access to the resources they need to do their
work which is what they need to do and not be able to do stuff and support us in the
outside world.
So Borg is basically a scheduling system.
Give it some work it decides which machines to run that work on and picks up the pieces
when one of those machines needs to be taken out for an operating system upgrade or we
discover that things don't quite fit the way they thought they would fit.
Okay yeah we'll talk a bit later about the schedule.
Not the benefits of a whiteboard draw a picture or something.
Can you please tell us about the players in the Borg landscape?
Sure so let's start on the physical side.
So I like to think of you know we have a fleet of machines that is scattered across the globe.
Those machines live in data centers.
Data centers typically contain one or more buildings.
A building will typically contain one or more clusters.
And a cluster is a set of machines with a high speed network between them.
Think order of magnitude 10,000 dishes, a medium sized cluster.
And there's about those machines live in racks etc.
Just in sort of the standard way that you would expect a large scale data center to
be built.
On the software side what we do is we take a cluster and we divide it into one or more
cells and a cell is a group of machines that are managed as a unit by one instance of
Borg.
So typically each cluster has one large cell carved out of it.
Sometimes there's a few additional ones for testing purposes and occasionally you'll take
one cluster and sort of subdivided into a large number of cells.
But most large clusters have a large cell as part of them.
A cell is managed by Borg which has a couple of different components.
One is Borgmaster which is the central component that looks after the state of the whole cell.
And then also on each machine in that cell we run an agent we call the Borglet which is
responsible for keeping tabs on the various tasks and containers and processes and things
running locally.
I've used the word task here.
Task is just a Linux container running some application and we group those into an entity
we call a job which is just a collection of basically identical tasks.
When you submit a piece of work to Borg you say here is a job containing 400 copies of
this task.
And these could you schedule it and place it and keep it up and those kinds of things.
Okay, yeah that's a pretty good coverage there.
So you touched on a couple of examples of programs that run under Borg.
What programs do run?
You mentioned maybe like something like Gmail.
Is it just things that run at scale?
Things that are actually pretty much everything that we do run the Borgl.
There's a few special case exceptions.
But to a first order think of everything that we do runs on top of Borgl.
So every single externally facing application that you can see almost all of those will
be running on top of Borgl.
And then of course you can imagine there's a bunch of behind the scenes stuff analytics,
you know, MapReduce jobs, data analysis pipelines, monitoring those all run on top of Borgl as
well.
So it's almost sort of the operating system of Google.
Exactly.
Okay, since our audience is primarily software developers I want to talk a little bit about
the programmers perspective.
How does a programmer writing an application to run under Borgl?
Does Borgl impose any limits or anything like that?
Is it a special paradigm or anything?
So the idea we have a Borgl is that it's sort of universal underpinnings for pretty much
whatever you would like to do.
The assumption is that you're going to be running a Linux task on a sort of production
version of Linux.
We have our own internal production versions that we sort of develop and enhance over time.
But that's basically the assumption.
If you can run in a Linux container you can run on top of Borgl.
We don't particularly constrain the programming language.
We support particular ones internally but there's actually a wide variety of languages that
get used by different people.
The language you write your programming, you build a binary using our build system.
You might have heard of that.
That sort of thing called externally.
It's called Basil, internally call it Blaze.
The output of that is a binary that can be run on top of Borgl.
We have lots of libraries that get to make things easier.
If you do certain things in a particular way.
There are conventions and protocols we use internally.
If you do it this way it will be easier but you can actually write stuff your own way
if you really want to do that.
That describes the program.
Then you also have to tell Borgl, I want to run that program.
To do that we provide a little RPC interface to Borgl where you say it's a bit my job.
Most people don't use the RPC interface directly.
They go through a couple of different systems.
The most common is a thing called the Borg configuration language, BCL, which is a language
where you basically get to describe a protocol description.
You provide a set of properties about your job, a key value pairs.
BCL is a language that lets you write those things.
In a straightforward simple case you can write a description for a job in a dozen lines or
less.
But you also have a programming language available to you as Lambda functions built in.
Some people can and do right dynamic stuff based on in a way you land in a cell what
the properties are of the computation you're trying to do.
So all compute values that get ended into the job descriptions.
There are other people who prefer to do it in a programming language like Python.
We have a wide variety of different approaches where programmers can provide the configuration
data.
MapReduce jobs, for example, you describe a MapReduce job and the MapReduce master goes
off and generates behind the scenes job requests to the rest of Borg on your behalf.
But the real thing to take away from this is as a programmer you get to write programs
pretty much the way you like and then you get to run them at whatever scale you think
is appropriate.
A story I often tell when I'm describing Borg to the outside world based on the Borg paper
is that at one point we decided we needed to rerun all the experiments we've done which
was doing some performance evaluations and comparisons.
So we pressed the rerun all the experiments button and went away for lunch.
When we came back I asked the question great how what did that take?
How many things were we running at once?
The answer was we were built up to a maximum of about 200,000 CPU cores allocated to our
experiment and nobody cared.
That was the kind of thing that will make possible because those machines were available
people were currently using them so we were able to jump in and take advantage of them.
That's the kind of productivity that we didn't have to worry about that stuff that we as
board developers are trying to make available to our internal users.
How goals to help them focus on what it is they want to do and not worry about stuff
like can they get resources in the right place at the right time?
Well yeah I remember the number of the 200,000 cores from the paper and that is impressive
to if you will almost by accident happen to be using 200,000 cores and like you say not
not even causing a big problem for anyone.
With the board configuration language or the API for it is that a static thing?
Do I say here I want a thousand instances of this or is it a dynamic thing once my job
is running?
If I see that you know Christmas is coming and everyone is hitting the system and I need
to double my number of instances running can I do that?
Absolutely so the way to think of it is you submit a description of what you want to
be true which is a job description like I want a thousand instances now you send that
to the board and it says great thank you got it and then it will start working on making
that happen.
You can come along later and update that job description.
You can change the number of tasks, you can change parameters on the tasks, you can change
priorities, you can change the sizes of some of the things, you can change the binary they're
running.
Those are just updates and you can do them some of them are non disruptive they just sort
of get pushed out and no changes happen some of them, for example if you add extra tasks
that's the existing ones stay up you just add extra tasks the job list gets bigger.
Some of them are disruptive and you actually have to restart things for example if you
push out a new binary then yes the old ones have to stop at some point and pick up the
new copy but there's a range of different behaviors and those things are documented
but yes the system will respond to changes that you make as you make them and the standard
answer will be great thank you got your request I'm working on it and sometime later the system
will ideally have responded to and made true the request that you asked of it.
And presumably also if you're pushing a new binary and it does require restarting it
can do something like a rolling restart if that's a correct.
We have ways of saying I want you to start at this end and work your way up and if you
discover that's not working well you can you can say no hold on well let's go back so
if we have a sort of present state and a future state and an update actually says sort of
like a almost a lightweight transaction who says stop moving towards the future state
and if you decide you like it then you can basically commit it to say okay great make
the future state be the one and if you decide you don't like it you can say no let's go
back to the previous state and sort of undo the changes and they will get reversed they
will go back to the previous state and you can pick up and carry on.
So we use that often for sort of rolling out a new version of stuff so we can carry it
and watch it happen and make sure that we haven't accidentally broken something it's
really useful to be able to go back to a previous state that you knew that worked.
Right yeah the canary release kind of thing was my next question actually.
Okay if I'm going to write an application for BORG and I'm going to distribute it across
hundreds or thousands of machines and I don't really have I assume I'm not picking and choosing
individual machines BORG is for me correct how do I debug something like that what facilities
does BORG provide to help me get a handle on what's going on with my program what's
working what's not working. Great question we provide a whole bunch of different a couple
of different things right monitoring there's a lot of monitoring that takes place sort
of keep an eye on what your job is doing most applications but almost all applications
will link in sort of by default a little web server which publishes information about
things like RBC requests that it makes or are made to it so you can sort of go poke
at it and retrieve information from it about what's going on how fast it is going on how
long it takes to do how many of them are happening in different kinds so that information is
gathered up and made available through monitoring systems those monitoring systems provide dashboards
who so you can look and see how many of my tasks are up which ones are healthy which
ones failed why did they fail let me go find all the logs all sort of standard error and
RBC logs if necessary you can drill down into those things and we provide a sort of a UI
which is based around the web of course and you can sort of start at the top of the fleet
and drill your down really way down from the fleet as a whole to an individual cell to
job that you're running or a set of jobs a particular user to a particular job to a
particular task in a particular job and go actually open up the error logs for that task
well you can have a view which says let me collect all the error logs across all the
tasks of the job and so we provide that monitoring information little graphs of how much CPUs
being used across time how much memory has been allocated how much being used all those
things are part of the system we provide our users to try and help them understand what
it is their jobs are doing excellent and I can imagine that the volume of that log data
would be pretty substantial and maybe there's other board tasks for you know doing the equivalent
of a correct through my logs if I'm trying to track something down across a thousand machines
exactly so the monitoring jobs are actually things that run on top of Borg they sit there
high priority so they get access to resources when they need them and they push out dashboards
though dashboards have query engines built into them so there's a very sophisticated
set of monitoring stuff that's been built on top of the Borg ecosystem so yes that's
one of the ways we make life easier for our developers in addition to programmers board
also another class of users of Borg would be the site reliability engineers who are ensuring
that my program or working with Borg to make sure that my program is running and what not
what what kind of features does Borg provide for them well first thing to note is that the
SRE teams actually get access to the same information as the developers do and vice versa so they
can both set stuff and try and debug things together if they need to they tend to build
additional dashboards on top of the underlying data sources that both Borg and the monitoring
systems that run on top of it provide to put things like SLOs and alerting rules and triggers
and you know paging people with things are going really bad or just pointing out that they should
pay attention if not so yes the same kind of visibility we have a model that a lot of the stuff is
visible to anybody who's qualified and permitted to get access to the monitoring information about
what's happening so that makes it actually quite possible to go for anybody to go in and they
know they specialize in building tools on top of the underlying ecosystem parts to do things like
rolling out and to push out a new version and to make sure it's working right and to test it and
probe it every now and again to make sure that it's actually all up for example storage system right
you want to make sure you can access all of your bits well the way you have to do that is you have
to go touch some fraction of all of your bits to make sure the storage system as a whole is up and
running so they build things like probing systems that run as jobs on top of Borg so they are both
customers of Borg and sort of mentors of things that running on top of it and then of course we all
have we also have the Borg SRE team which are spectacularly good at keeping the thing up and
running and they are I like to describe and I think as operators with PhDs it's sort of one
way to think of these incredibly capable people who understand interrelationships and interactions at
very large scale between complicated systems that are in dynamic flux all the time and they're
spectacular at what they do and you know we provide what we can in terms of tools to help them understand
what's going on okay so for the programmer it gives me the ability to write my program and have it run
at scale for the SREs gives them the ability to ensure that my program is running at scale and I
suppose also it's a scaling function for SREs in terms of allowing fewer of them to do more
work in terms of managing tens of thousands of machines and what.
Exactly and I think one thing I want to stress about the SRE team is that they don't operate
systems but they also program and do engineering around automating a whole bunch of the stuff that
you know if you do it twice if they have in sake that's automated because people make mistakes
if we let them do it so they're in the process of building layers of software and tooling on top
of the underlying systems that make both their life and the life of their internal customers better.
Right and you mentioned that they're building on top of of Borg also so everyone's playing with the
same toolbox and everyone has a vested interest in making it the best it can be.
Exactly. So let's kind of move into a little more details about what Borg is doing. At the highest
level what are some of the objectives it's trying to mean. I guess we've sort of touched on that a
little bit at least from the SRE angle but in terms of scaling and whatnot. Yeah so I think
the highest level Borg's goal is to minimize the amount of time that our developers and the SRE
teams are supporting them spend thinking about infrastructure right they have better things to
spend their time on and do development for so we'll do that for them we'll worry about infrastructure
we'll worry about provisioning enough resources in the right places with the right kind we'll
worry about coping with failures on their behalf or restart stuff if it falls over and pick it up
again and at the same time we try to do this in an efficient as a fashion as we can right once
something stays up then we come back and ask the question great how can we make it more efficient
so how basically how do we act as good custodians of Google's investments in hardware machines and
networking and stuff like that to make it maximally effective in delivering end results to our end
users by trying to make sure we don't have stuff that's lying around idle when we can we do
make other things with it sure sure efficiency in a word there yeah so I tend to say availability
first efficiency second right right an unavailable efficient system is pretty useless exactly so
one of the functions of of Borg is is scheduling and I believe you mentioned this before what exactly
is it scheduling yeah unit of scheduling for Borg is a task or a container now we actually run
tasks in container so from Bull's perspective they're basically the same thing so this is a Linux
C group so it has a capacity in terms of memory in a matter CPU cycles a matter of disk space a
matter of disk traffic and a few other things as well and it has these sort of multi-dimensional
rectangles if you want to think of that way that it is trying to pack into machines which also have
multi-dimensional properties about how much memory they have how much CPU they have and stuff like
that so those are the units of placement scheduling now we collect to make things easier we provide
collections of tasks or containers so a collection of tasks is called a job we also have high level
collections we call Alex which basically our containers our set of containers and they just
reserved space right so they can put aside some memory and some CPU cycles and then you can run a
job or a task within one of those alloc instances we can talk later about how that's useful but it
makes it easier to provide things like high resiliency log management on top of a complicated system
and what Borg does is it schedules those outer level containers whether they're an alloc instance
or a task that's running by itself that's what our top level task we call it and it packs those
onto machines in an expeditious fashion in a way that makes sure that things like availability
considerations are taken into account okay so the the efficiency of that packing packing the most
items you know given number of machines is sort of the overall efficiency and then
you mentioned availability so that we don't pack everything all on one machine in one rack or
something like that right exactly so for example by default Borg will spread job the task of a job
across multiple racks because that increases your resiliency in the face of let's say a top of racks
which breaking right and then presumably there are other kind of placement things that one could
just declare in the configuration language oh indeed yes okay so last time I looked there was
something over 200 parameters you could specify in a description for a Borg job or a Borg alloc
there's lots of things you can constrain some of these are placement constraints you can say
things like don't have more than two of these tasks on one rack you can say things like I need
to be running on a system which has this version of the operating system because I need parameters it
has I need to be running on this particular kind of hardware because I'm relying on some features
the hardware has or I need to avoid a bug that was present there's a whole raft of these things
and there's lots of control over sort of fine graying placement of CPU threads onto CPU calls
which gets passed out of all good and then eventually to the kernel and so on so forth
and each of these things has evolved over time and we've built up a pretty broad library of stuff
that gives people who really need it very fine graying control of exactly what happens and people
who don't need it they actually don't have to specify most of those things the system will just
do something plausible even if you don't say very much and the tasks are running in containers and
those containers are maybe not Docker containers exactly but they're built from the same primitives
right exactly same basic idea right it's a Linux C group which is a resource it owns a chunk of
resources in particular dimensions like a mount a CPU mount a memory we also put them in a little
churuk jail so they can't sort of break out and see bits of the machine they shouldn't have access to
you know various things like that so yes it's it's the docker container sort of notion is the
external manifestation of this we have been using that same technology inside google for about a
decade now right yeah I think I've heard recently that a lot of those primitives were developed either
in part or full by Google and pushed into the other stream per hour yeah exactly that's good for us
and it's good for the community we like to help to contribute in that way okay so in a cell
where are the various scheduling decisions being made sure so I mentioned the boardmaster earlier
which is actually central overseer for all the stuff that happens inside the cell and that's also
the place where we do the placement of scheduling decisions you submit a request to the boardmaster
and it makes a choice about where things run you can also go to the boardmaster and get access to
which jobs are running where so for some of those user interfaces I described I'll provide it data
that was the background for them by the boardmaster so it's a central point this is different than
some other systems which sort of make the scheduling the thing that gets distributed and done across
all the individual jobs we've chosen to centralize it and actually surprisingly it has worked quite
well occasionally people ask us you know when you're going to hit a skating limitation and the answer
is we come close to it every now and again and then we do something to make the thing go faster so
the scheduling limitation sort of evaporates for a while so we haven't hit a hard limit yet
clearly we will one day but we so far we haven't run into that particular problem
okay and then at an even higher level how does a programmer or an sra go about choosing which cell
to run something in is that a geographic type decision or something like that great question
actually let me first inject a reason why we have these multiple cells the first is to give people
control over things like geographical diversity another is to give them control over sort of
planned outages we will do maintenance on on a cluster for example or a whole cell every now and
again and we want to make sure that we don't take down you know both copies of your application if
you put them in different cells so we publish schedules that allow people to say i think
this pair of cells they will basically they will not be taken down for planned maintenance
simultaneously and the third reason is just to avoid error propagation right an awful lot of
outages in large scale systems are caused by human error and we want to put boundaries on the
scope of the error that a human can cause and a cell is a great boundary for that right so you
can probably you can take it out how the whole cell but you won't take down the one next to it
there's a story one my sra colleagues actually was was trying to drain a cell to do some maintenance
work when you type the command to do this impressed enter and then went and checked the logs and
nothing happened so like any smart person he does exactly same thing again and the same results
are repeated and then he looks more closely what he done and what he discovered was the hit
mistyped the cell name and a taken down a cell just not the one he meant so those are kind of
errors that you actually want to bound in scope so that's one reason that cells are actually very
very isolated from one another they don't depend on one another for almost anything
so they can be brought up and brought down again independently one another and you know errors have
some sort of bounded scope now having that we've been said you now have a placement choice if you
are an application developer you might start by saying well where are my external customers
or where is my data for example your latency sensitive app you might want to have instances
close to where the majority of your customers are if you're doing a large scale data mining
analytics you want to who might want to be co-located fairly close to where your data lives so that's
sort of the first level thing and from that you look at you know which cells are available in that
region let's say on that part of the part of the globe and then you go down and look and say which
ones have space available in them and then you go down and say great let me actually fire the
request off to that particular cell and how it do work and then inside the cell board we'll find
machines on which to do stuff now we actually provide some tooling to help if you don't really
need to know in detail which cell things are running in and you've got a batch job you could
actually submit your batch job to a sort of global scheduler will make these choices on your behalf
it'll look around and say you know where is your data or maybe just where three resources and it
will make the cell placement choice for you so we do uh automation of things that occur at
level above books again think of this as a base building block on which you can there a whole
bunch of other things and then go down the other direction when you eventually get down to a particular
machine running a particular task the ball that on the kernel collaborate between them to make
sure that those tasks are being placed in a sensible way across all the different CPU calls and
normal memory allocation stuff is done in a smart fashion so it goes on the ecosystem covers the
whole range from global placement down to the individual threads run on individual CPU calls
okay and the board master is running is the master of one cell right correct okay you mentioned
before about in the context of log collection the concept of prioritization how our tasks
characterize or prioritize that the paper mentions production and non-production
i would think user facing and non-user facing come into that uh indeed yes so we we actually
have a fairly fine grain priority system you know small integers zero is low priority
and goes up to 400 each i think most production jobs are run at one we have a good production
priority band all the jobs in that band have about the same priority and the basic rule of thumb is
a high priority job can preempt a low priority job if we need the resources for high priority
job it will run it will even kick out low priority jobs so typically we'll run user facing jobs
but especially latency sensitive ones that have to respond to external requests for web service
or something like that we'll run those at production priorities and if there's a batch job that was
in the way well the batch job is run at low priority and it will get kicked off and that's fine because
there's probably some other machine in which you could run on so the idea is that we're actually
trying to make sure that we do the stuff that is high priority which is after what high priority
means right this is a business decision and then if we need resources to run other stuff they will
fit in around the stuff that actually has to be up all the time so sometimes this means that some
batch jobs will get starved resources because you know we're busy serving front-end user requests
and absolutely that's fine that's a very conscious choice about how we prioritize things
okay and just to make sure i understood this correctly lower priority jobs could be running
and they can be terminated preempted and for will at some point in time maybe
start them up again somewhere else or at some other time exactly basically they'll get put on the
so premissions actually happen at the level of individual tasks or or alux so individual
containers rather than whole jobs and you know when the task gets preempted it'll get put on a queue
let's see we can find another space somewhere else for it and as soon as we can find space we'll run it
and then it's up to the program or we develop that to then ensure that the task can be restarted
and i don't know pick up where it left off or just start from the beginning again something like that
yes so it's both you have to think about if i've got a whole job right if some fraction of it
has disappeared because it's being preempted can i keep going for an individual task you have
choices you can sort of pick up where you left off if you're stateless that's easy you could choose
to try and take a checkpoint every now and again to go and roll back to that you could maybe hope
that you will be tapped on the shoulder we will try to tap you on the shoulder and say hey we're
about to preempt you if we can we have time we'll do that if we're permitted to we don't always
aren't always able to hit for example if the machine is running out of memory we will just kill stuff
right now there's no there's no chance to tell to notify them but often probably majority of the
time if you request a notification you'll get it and you'll have a few seconds to do something about
it like maybe dump some stuff out of memory if that's useful so there are different ways of
recovering and you know depends on the cost of picking up and carrying on from where you left
off as to how which of those techniques you use and all of them happens but the fundamental
principle is if your system is going to be up you have to worry about parts of it being down
all the time right especially at that scale and google has published a number
of of papers and whatnot about failure is not an option it's it's just reality it's just normal
yes so for example we publish it in we internally we publish a what we call the eviction budget which is
for every thousand tasks running for a week roughly how many of them you should expect to get
preempted and you have to expect that as normal right and then possibly then depending on the
nature of your your application even be prepared to deal with admin as well if it's something
especially important in user facing yeah so the model is that things can break at any moment
whether they break because you know we need to take the operating system down to reboot it for
a upgrade or the machine fell over or the network stop working it really doesn't matter if you're
prepared for things disappearing at any moment then you could also cope probably more gracefully
with the things the way you have some notice but you have to assume that won't happen because at
this scale it is always happening in the background right okay um the paper presents a whole lot of
data a lot of numbers graphs etc and whatnot and it's not really practical for us to dive into
those too much here but were there any surprising or counterintuitive results that you found in
your experiments things where you were expecting one sort of outcome and you were surprised to see
what actually happens in some ways yes the first sort of surprising meta result to me was that
nobody had asked some of the questions that I asked that we got answers for in the paper so for
example we run both sort of front end user facing tasks and back end batch job tasks on the same
machines and the premise had been that that saved resources we didn't have to buy so many
machines and I asked the question great how many machines is that safe we didn't know so that was
sort of surprising in some sense and now we have some quantification and we're pleased to see the
results positive the other thing that sort of happened along the way was just getting appreciation
for just how complicated this system is and its behaviors we spent a lot of time running an
experiment getting an answer looking at it and going that's not right and eventually we ended up
where we possibly wherever we could we ended up doing evaluations in two different ways so we'd
build a sort of simulation model and actually try and act using the real code and then we also
build some sort of other path to the same data maybe using traffic from sort of loganels and
log systems and sort of see if we could cross-correlated results to make sure that they were releasing
the same ballpark and that actually found quite a few bugs in the simulation work that we were doing
and occasionally in analysis of the traces so a lot of it is just getting a really good handle on
making sure we understood how the system was behaving and what it was doing when we were
tweaking in particular ways to explore different behaviors that it didn't normally do on a day-to-day
basis it's a very complicated system it has a lot of moving parts in it but I'm pretty comfortable
when we were done that we had we had good answers but they were mostly roughly what we expected
there is this policy it's there for this reason now we know just how much of that reason it provides
but we knew it was going to be positive we just weren't quite sure how big the magnitude was
perhaps a lot of kind of quantifying whether it become sort of standard operational procedures
or that perhaps it evolved from somebody's sort of that feeling about how things should work
and some experience running things so more typically you know before we make a change we'll actually
do some internal studies and performance analyses but those are typically done at a point in time
across a set of workloads which is a subset of the whole one thing that was slightly surprising
in sort of the review process for the paper was a few reviewers asked us so can you explain why
you have you know why this cell is different than that cell and the answer we came back with after
one think about it a bit was actually no and we don't want to right because we try we need to
write software in the war geeker system that is able to cope with whatever workload it is that
our interluses throughout it so if we end up trying to understand its behavior by specialising
on particular workloads we're probably ending up with a losing gain so we need to make sure
that this means we build a pretty capable of coping with a wide range of different use patterns
because they're going to change over time and as people move in and out of the cell that we
different as people evolve their applications to what they're doing what they're trying to accomplish
is going to be changing as well and we can't be too tightly tied to a particular workload
indeed yeah that makes perfect sense going back to the boardmaster we were discussing it as if it's
a single entity but it's not because that would be a single point of failure right
exactly yes so indeed it in fact is replicated we typically will run five Borgmaster instances
we elect one of them as a master we run they have a data store which we keep all the state that they
need to keep we call it a checkpoint it's built on double Paxos we use the Paxos algorithms to make
sure that before replies that back to a user we have got a persistent copy in state with that state
on disk and even if that boardmaster then disappears and then another one gets elected then we will
be able to pick up where we left off we collaborate with Chubby to store and a pointer to which is
a current boardmaster we use that for the lock management so yes that the boardmaster itself is
designed to be a failure tolerant system another example of that kind of behavior is that even if
the boardmaster itself all of the copies disappear which occasionally happens pretty rare but occasionally
all the work that was currently running in a scale the cell keeps going there's no reason why
it should stop the tasks are running on the machines the board will just be doing it stuff
if it can't talk to the boardmaster for a while they'll just keep doing whatever it was last told
to do so the system is designed up with multiple levels of resiliency to try and keep going even
when the world of entities going pear shaped right so it can continue on even if you can't
tell it to make any changes yet because the or any changes at the moment because the boardmaster
is unavailable those changes exactly then how are the boardmaster the five boardmaster processes
placed or scheduled in in the cell is that a recursive thing where board says it itself
oh so when I said almost everything this is an example where we actually place those replicas
manually so we choose machines you know for the right properties when you bring up a cell
and you know the process is the rate of which these things change is relatively small
manual intervention is just fine we can operate for a while with a subset of the five who prefer
not to but we can if we need to so yes we'll do a manual selection of the five boardmaster
instances where they run the thing take off from them okay and then how is the board itself tested
so we view the boardmaster as just another distributed replicated failure tolerant application
so all of the standard tricks you throw at a distributed replicated failure don't application
get thrown at Borg we run it through a bunch of unit tests we inject artificial failures we
do integration tests we throw results of nasty corner cases out it to make sure it will survive
all of those every time we do have a bug we do have bugs occasionally we will write a test case
that will trigger that bug if it ever shows up again so we make sure we don't do regression
and accidentally reintroduce them by mistake and once we got to pass that stage then we'll actually
run it in what we call a test cell actually bring it up we'll throw workload addict we have sort of
synthetic workloads which some of which are derived from real life production stuff that happens elsewhere
we'll run that for a while we'll do a sort of soak test in a testing cell if that's successful
then we'll push it out to what we call a quality cell where it's actually running real life
production workloads and we're just sort of monitoring and seeing how it does
occasionally we discover things because real life stuff is different than anything you might
have captured in a test or in a standard benchmark we'll catch a few more things there and then
we start a sort of cell by cell processor rolling it out across the rest of the fleet again think
of this as a canary right we don't trust it until it's proven to work so we'll roll it out a couple
of cells look at it for a while if that's good we'll roll it out a few more if fails start working
well then we'll start speeding up the process of rolling it out so eventually we'll end up
having pushing it pushed it everywhere but at every stage we make sure that if we decide we
don't like what it's doing we can roll back go back the previous version we'll take a snapshot of
the state before we push out a new version so we can always revert to it so we can always recover
from various things that might go wrong and by the time you've finished doing the canary rollout
it's probably time to begin the release process again even before that yes
sometimes there's more than one rollout going off at once it's easier if there's only one
but sometimes you know we're trying to actually decrease the time between releases you know fast
release with sort of smaller changes in them are probably a good thing so we're in the
process of making that happen much more quickly than it used to in the lessons learned section
of the paper there's a discussion of Kubernetes and Kubernetes is an open source cluster management
system developed by google what are some of the biggest things that you learned
in working with org that are reflected in Kubernetes great so yes Kubernetes actually came out many
of the same people who developed work so quite a few of the world developers are now Kubernetes
developers we tried where we could to echo the things that we found helpful in the Borg world
and to try to avoid the what we viewed as mistakes that in retrospect we wish we hadn't done in the
Borg world and do them in a different way in Kubernetes hopefully better so things we liked right
containers are definitely a great way of organizing applications they make developers able to focus
on their applications they don't have to worry about things like operating systems and patching
bring them up today keeping the virus systems and probably deployed all that sort of stuff
it's a much higher level abstraction much closer to where application programmers typically work
so that was a good thing the second is this idea of a declarative specification I want this to be
true and then what we call reconciliation which is comparing the external world with the statement
about what you want to be true for example I want to have a hundred copies of this task running somewhere
you put that into a loop and it says well how many are running oh zero let me fix that let me place
a hundred off the monkey running great do that go back how many of you want to have a hundred
tasks running great how many running 98 let me fix that plus you'll be two that's will be somewhere
else so that is a very resilient process in the place of failures because you can always sort of
pick up where you left off and carry on because you have a state and you compare it with the outside
world and that difference tells you what you should do differently I mentioned these Alex
allocation units so in the Kubernetes world they come as pods or pod in Kubernetes is just one or
more containers that get scheduled together onto a machine we use them internally for things like
building a very complicated web service and then sticking into it into the same machine a little
tiny super lightweight super resilient easy to debug absolutely guaranteed to stay up thing that
will extract logs from the complicated web service and push them out to a split safe place on the
distributed system so even if the web service is going wrong in a horrible way the log saving
system is going to keep running and pushing stuff out and you want those things to be co-located
because you don't want to have any complications and shipping stuff across networks and things
like that so that idea turns out to be useful for those sorts of different things but the simplest
example and we replicated that in the pods in Kubernetes and then a couple of examples that
things that didn't work quite so well the first of this notion of a job as being a
a vector of tasks where a task index in a job is a small integer on the third task in this job
that's actually in nuisance because why was that third one dies and you need to add another one
you have to reuse that slot and you get sort of head banging complications about how do you
account which ones do up and do reuse them or what have you so we got rid of that in Kubernetes
and we went to sort of unique jobite it's unique um pod IDs rather than a vector in a slot
now the other thing we did was we generalised a notion of job from being just one single
notion of a collection of things that is glued together into a very general notion which is
a combination of applying labels to individual pods or whatever or containers or what have you
together with what we call a label selector which is basically a query it says find me all the
pods with the set the following set of properties for example front end production version that's
been up for more than a month just give me list of all those things and then maybe do something
on them so operations in board work on jobs operations in Kubernetes work on sets of pods chosen by
a label selector so it's not more flexible so you can have sort of then diagram like overlaps and
you have multiple degrees of freedom and they can be application and devops specified sets rather
than stuff that's hardwired into the system some things we liked is the idea of being able to have
sort of smaller miniature services Borg is a bit of a monolithic ecosystem even the Borg
master is doesn't offer those stuff as one chunk so we've taken some of the functionality in Borg
and some broken out into multiple separate microservices so the idea of keeping the number of replicas
of a pod fixed to some target you've given it is a separate microservice in Kubernetes world
whereas in Borg it's all buried in part of the scheduler and then two more thoughts one is
about services so we built a support for jobs inside Borg but we didn't really do the next level
of support which was for groups of jobs that are collaborating to provide one thing and Kubernetes
has a much better set of facilities to do that but one thing we did take away from Borg and we've
applied to Kubernetes as well is you know actually scale is not that frightening if we can service
like I said a median size sale is 10,000 machines and these machines have quite a lot of causes
in these days and we have ones which are much larger we can see this in that the scale of
Kubernetes is targeting don't worry central layer solutions work just fine so those are
things we took forward from Kubernetes and applied them and then one thing that's obviously different
is that Kubernetes has been developed completely in the open univer open source world right so
everything that's been done is out there we have I think 400 contributors at the last count
and lots of people participating in them this is just great yeah I wanted to touch on that a bit
you know there have been over the years there have been a number of papers that have come out of
Google that have described various internal tools that have then spawned open source clones of those
tools by developers outside of Google for example MapReduce or BigTable or Chubby all have kind of
open source clones that were developed by people outside of Google although Google makes a lot of
open source contributions we mentioned the whole underpinnings of containers and one ah
as just one tiny example Kubernetes seems like the first time that Google is doing their own
open source clone if you will or next version is there a particular reason for that
I think actually you touched on it earlier right we've we've seen people take our ideas and
and run with them and come up with systems that are although very useful to people they're not
quite what we had available internally they don't necessarily always include the lessons that we
learned the hard way from from building the first systems and people will relearn them but sometimes
that takes a while so what we thought here was that we could actually jump start that process by
making contributing a bunch of our expertise from from our experience and sort of make that
available to people in the outside world in a way that we couldn't do we can't just open source
our internal systems they're just so interconnected with the rest of our ecosystem internally but
this is a way where we could sort of give people a leg up on making these things usable and available
we know the first people to be doing stuff in this space right there's plenty of people who
have been doing things in the VM world we have uh for like mesos and and other other people
who have done systems like this but this sort of our salon on that stuff you know it's a way for
us to expose the lessons that we have found helpful in making our developers more productive we think
that's a good thing to bring to the community and you know it turns out we also have a cloud
business and this is a way for us to provide perhaps a a way of encouraging people to be more
aware of that and take advantage of it where it makes sense so i could use Kubernetes in
if i were running on the Google a number of containers on the Google cloud yes in fact a
couple of different ways right you can run Kubernetes in a house on bare metal you can do it yourself
just with raw machines you can run it on Google Compute Engine and various other people's
virtual memories so virtual machine suppliers and we also provide what we call a hosted version
of Kubernetes Google Container Engine where we run all of the sort of mastery stuff for you and you
just say hey great great me one of these things and then i'm going to submit pods to it to get them
scheduled so we just sort of take even one level further of support because we run the version of
Kubernetes on your behalf and manage it for you so all of those things are available to you which
ever makes the most sense is Kubernetes a second system as defined by Fred Brooks
yeah i think we got that out of our system with omega which is probably the second system that
i actually gave talks about omega describing warning people this is a prototype and there are
some flavors of it which are a little bit system-second system you like we learned a lot from
omega in fact many of those ideas are now being folded into bawak right so bawak now supports
multiple different schedulers which is an idea that we've loaded with omega and tested out and
demonstrated it worked pretty well and there's a few other things like that some of the other
ideas that came out of mega actually went directly to Kubernetes so it had a very beneficial effect
in terms of idea generation so we're now realizing in which parts work well which parts don't most
of the things that don't work were not because they didn't work it was questions about what the
right trade-off is to make in terms of adoption and we have you may have many many users who are
already built interfaces on top of the old system and then ask them to move to new one to adapt
to be harder than we had anticipated so we're trying to avoid some of those mistakes again
let's learn from the things that we've done in the mega universe and try to bring the Kubernetes
so Kubernetes is a much more granular system it has sort of microservices
partly that was an omega thing but we've actually made them much more explicit
we're trying to provide API so you can bring your own when you need to we're trying to make it
possible and encourage people to try that so we're trying very hard to bring forward the things
that actually have incredible value and leave behind the stuff that we're just more experimental
ideas that aren't necessary to get the ideas out we can come back to them later and add them in
once we got the basic framework right but we don't need them to get the basis system up and go in
okay so omega is the cursed second system and Kubernetes is the third system where everything's
right you can try counting that way that there were predecessors to Borg that would suggest
that Borg was itself a second system but let's not go down that path too far okay fair enough and
we're recording here in the fall of 2015 Kubernetes is production ready at this point in time and
released indeed so we pushed out of version one which was the firmware one where we started
providing version control over the APIs that came out in July and as we're recording this we're
in the process pushing out version 1.1. Excellent you mentioned MASSOS in terms of other cluster
managers that are out there we had an episode of the podcast number 243 on MASSOS how does Kubernetes
compare with MASSOS or work with it so let me first start out by saying MASSOS was being developed at
a roughly similar time to omega it was being done by a lot of people from the University of
California and Berkeley so we actually collaborated with shared ideas in particular we would have
them come down talk to us about their latest ideas and MASSOS we'd go great have you thought
about the following issues and basically share problem statements with them until their heads
exploded and they went back to Berkeley to think about them some more so there's a lot of overlap
in ideas and cross-fertilization which is a wonderful thing from the research side on the
practical side MASSOS committees in my mind actually complement each other very nicely if you have a
greenfield site where you're you like the container models or the docker way of doing business and
you want to get something up and running you don't have anything you could just run Kubernetes on
Maymetal and it's great and we'll do a bunch of stuff but if you were in a world where you actually
already have multiple different kinds of applications and they're sharing resources using MASSOS that's
also fine because MASSOS supports Kubernetes as one of its frameworks so you can actually have
Kubernetes running on MASSOS on premises you can then have Kubernetes applications running
either on premises or metal or MASSOS or in the cloud so that way you actually get the best of
both worlds with a single API where you write with sort of right the Kubernetes stuff you get
that where however it's being supported in the next level down so MASSOS as of today it's more
sophisticated in a few aspects but for example it's scheduling has been out for
longer than Kubernetes Kubernetes is very young MASSOS has been around for quite a few years
longer as a result it's made advances in some areas and if you want to benefit from those
you can do so by running on top of MASSOS framework a goal frankly is to try and help Kubernetes
catch up but in the meantime they work together extremely nicely and I think in the long run
there may always be applications where you know you want to have something like a Spark ecosystem
at the same time as you have a Kubernetes ecosystem and you want to to share the same resources in
which case a MASSOS-like solution is a pretty good way to go. Okay so if I had my fleet of machines
and I'm running them with MASSOS and maybe I've got a dupe in there or a spark and now I want to
do some other things that don't fit into that model and I want to run them over under Kubernetes I can
just put in Kubernetes as another tenant framework in my MASSOS cluster. Exactly and our goal is to
try and as Kubernetes evolves and as more and more people start contributing to it when we'd like
to try and move some of those other frameworks to be hosted on top of Kubernetes so if you have
you don't have a MASSOS environment you can still run things like Spark or Hadoop there's no reason
why they couldn't be moved onto that world they just run containers after all so that we think
they could fit in because for long term plans. So I could run MASSOS on my Kubernetes and then
have duped on my MASSOS on my MASSOS. I suppose. But try it so let me know how it goes.
Okay so kind of winding things up here are there any items that we didn't cover that you think our
listeners would like to know about? Almost certainly yes but I would point people to the wallpaper
there's actually quite a lot of information in there people occasionally accuse me of putting
too much stuff in there so it's sometimes but that's fine I mean I'd rather have that accusation
than not having enough so there are many topics in there we didn't get a chance to talk about
but we'll also give you a chance to look at some numbers and some of the evaluations we did
and then there's a bit more in depth comparison than we had time for here today between what we
learned about World Kubernetes and how we bring those ideas forwards so yeah probably that's a
good place to put a good resource to look at. Right we'll certainly have a link to the
paper in the show notes are there any links for your presence online that you would like us to
include or how can people get in touch with you or follow you? They're welcome to have access to
my website I'll make sure you have the link and put that in the show notes too that has a list of
my papers and it's got things like my email address and contact information. Like I said we'll have
links to all of that in the show notes for the episode. John I wanted to thank you for being
on the show. Thank you very much I really enjoyed this. I'd also like to thank Jeff Morrison and
Sven Johan, fellow host on the show who started the ball rolling on this episode. This is Charles
Anderson for Software Engineer in Radio. Thank you for listening to SE Radio an educational
program brought to you by IEEE Software Magazine. For more information about the podcast including
other episodes visit our website at se-radio.net. To provide feedback you can write comments on
each episode on the website or write a review on iTunes. Mention our messages on Twitter at
se-radio or search for the Software Engineering Radio group on LinkedIn Google Plus or Facebook.
You can also email us at team at se-radio.net. This and all other episodes of se-radio is
licensed under the Creative Commons 2.5 license. Thanks again for your support.
