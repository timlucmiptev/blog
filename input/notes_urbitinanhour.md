# Urbit in an Hour

## nouns
* a noun is a data structure that's a tree; either it has a pair of other nouns, or it's a number
* so a noun is any bifurcating tree shape with numbers at the leaves
* the numbers can be zero or bigger, with no upper limit, which means you can treat any file of bytes as just a big number of bytes, which is in fact what we do
* at any given time, everything you care about in this computer, including your files, the states of all your apps, and even all the code the apps and the kernel are running, is just one big noun
* nock can productively be thought of as a way of embedding a programming language into this data structure
* and all that programming language allows you to do is run a function on a noun and produce another noun

* the subject is the argument to this function
* the formula is the function

* the motivation for noun as opposed to something else, btw, is basically that it's the data structure of lisp, just trimmed down to lose all the stupid extra detail

**the above is the goal**. Now we see how Nock and the Event Log get us there.

## nock

## event log
yeah, i think the big thing to keep in mind is that the event log is a massive simplification of the idea of a computer

we think of a computer as something like a mac or an iphone, but all it really needs to be is a stateful packet transceiver

it can receive an event, do some computation, update its internal memory (state), and send out some effects, either to some other computer over a network, or directly to some piece of hardware

well, i'd say most systems have never really tried to follow an event-based approach; rather, they grew organically in a different way

* the next big step is freezing the behavior of this simplified computer
* so all it ever does is run a particular function on its old state and the new event to get new state and effects
* and then that function never changes

* that function is called nock, and it's actually a Turing-complete programming language, which allows the computer to load new behaviors into itself over time in addition to just new data
* but those new behaviors, since they're just part of the state in this stateful packet transceiver, don't affect how this thing gets run
* so if you built a machine that ran nock, you could load new apps into it without needing to throw the machine away

* the subject represents all data and code in scope, which for the arvo kernel includes both the old kernel state and the new event that it's currently processing
* contains the state and the operand

* the runtime punches out the sample slot of the arvo kernel with the new event, then uses nock 9 to say "now run"
* the log is not visible to the arvo kernel
* nock doesn't even know whether it's being run with an event log

* our haskell-based interpreter actually has a dry-run mode that runs with no event log and just stores the current state in memory until you shut it down, at which point the state is lost

### writing to disk
* the final component to discuss at this layer is writing to disk
* in most systems, if a program wants to save a piece of data, it has to ask the operating system to write that data to disk for it
* a nock program never has to ask anybody to write anything to disk -- it just holds everything in memory, at least logically
* every event is written to disk before its effects are performed
* and because the only things that could have affected the current state are the events leading up to it,
* if you lose your in-memory state you can always get it back by rerunning the events in the log
* every once in a while, the runtime takes a snapshot of the current arvo state and writes it to disk, so you don't have to replay all the way from the beginning
* you can just replay from the most recent snapshot

* the second optimization is handled by the host os in practice: if your in-nock state exceeds the size of ram, the host os will spill it onto disk for you as swap space
* taking snapshots also allows you to delete old logs if you want, which we'll probably do on infrastructure ships relatively soon
* so we've now shrunk the act of programming down to just writing a function from noun to noun
* and that function itself is represented as a noun


### concept of a noun
* see above


## hoon
* this new bytecode interpreter is so much faster than the previous one that we no longer need to have jets for the hoon compiler, which compiles hoon to nock

## arvo kernel
## ames

## the pki

## jets
* jet dashboard = lookup table
* but it's not a straight lookup table
* the hard part is that it has to know which parts of the subject need to be checked for equality against what the jet expects

## nagging thoughts
I guess the nagging thought in the back of my head the whole time is "what parts of this abstraction have had to leak on order to make it work?"

* the idea that you don't know memory addresses of nouns, that leaks a little
* but you have to assume that the runtime will do structural sharing
* and write the code accordingly

* well, the compiler jets used to be a major abstraction leak
* now almost all of them have been turned off
* but some of them do a weird runtime-caching thing that's fragile and could probably violate nock as written right now

* jet-matching is more complex than it needs to be
