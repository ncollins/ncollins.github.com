---
layout: post
title: "first thoughts on using threads in Python"
date: 2014-01-13 16:24
comments: true
categories: Python, concurrency, threads
---

One of my aims for 2014 is to improve my grasp of concurrency.
I'm hoping to write concurrent programs using a various approaches
(threads, async, channels) and take the
[Pattern-Oriented Software Architectures](https://www.coursera.org/course/posa)
Coursera class which looks into these issues in the context of
distributed systems.

With this in mind, when I decided to take my
[Github Data Analysis project](https://github.com/ncollins/github_data_analysis)
and work on [version 2](https://github.com/ncollins/github_data_analysis_v2),
the first thing I decided to do was restructure the API scraper
to handle multiple concurrent connections.

There are a few approaches for concurrent programming in Python,
and the async framework [Twisted](https://twistedmatrix.com/trac/) gets a lot
of love at Hacker School, but I decided to stick to the standard library
and use the [threading](http://docs.python.org/2/library/threading.html) module
following this [IBM tutorial](http://www.ibm.com/developerworks/aix/library/au-threadingpython/).
(Async support for the standard library is being worked on currently.)

The gist of the tutorial was to use [queues](http://docs.python.org/2/library/queue.html)
to pass data between threads using a producer-consumer model.
The main thread loads the initial URLs (users) into `download_queue`
and 30 `FetchUrlWorker` threads take the URLs from the queue and fetch the data.
The necessary data is added to `db_queue` and other URLs (repository and
contributor listings) are added to `download_queue`. The single `DbWorker` thread collects
the data from `db_queue` and inserts it in a SQLite
database. The program ends when both queues are empty.

The producer consumer model isn't too hard to understand but I had a few issues getting
it to work smoothly:

- If an error causes a task to not be indicated as complete the queue will not empty and
the program hangs. I ended up resorting to Pokémon exception handling to ensure that
`self.download_queue.task_done()` was always called.
- Ctrl-C doesn't work as expected, it refuses to close child threads in case they are
in the middle of something critical. This was a real nuisance when I was dealing with
non-empty queues and the program was hanging. I hope to experiment with catching
the `KeyboardInterrupt` exception in the main thread and setting variable that is
checked by the loop in the other threads to tell them to halt. 
- `Thread` objects are initialized in the main thread, not the child thread that they create.
I was using a `sqlite.Connection` object which can only be used in one thread by default.
Initializing it in `DbWorker.__init__` (executed in the main thread) and using it
in `DbWorker.run` (executed in the child thread) raised an exception and it took me a
while to work out what the problem was.

While it may not be my finest code I'm pretty happy with how it has turned out so far,
it runs far faster than the single threaded version and the producer-consumer model
is conceptually simple even if dealing with the edge cases requires some care. I've
been dabbling with node.js recently and I've been reading about Twisted so I should have
something to compare this approach with soon!
