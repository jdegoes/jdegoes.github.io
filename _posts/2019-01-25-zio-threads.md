---
layout:       post
title:        "Thread Pool Best Practices with ZIO"
description:  "Thread pool management used to be hard; with ZIO, it's now free."
category:     articles
tags:         [fp, functional programming, type classes, scala, monads, lenses, effects, reactive, scalaz, cats, mtl, monad transformers, zio]
---

High-performance application's don't _block_. Blocking is when a thread synchronously waits for something to happen&mdash;for example, waits for a lock to be acquired, a result set to be returned from a database, a timer to elapse, or bytes to be read from a socket.

Instead, high-performance applications are _asynchronous_. Instead of synchronously waiting for something to happen, asynchronous applications register callbacks with handlers. Then, when something happens, the callbacks are invoked. This style of programming, which conserves threads (and their associated resources), is called _asynchronous programming_.

Unfortunately, in 2019, sometimes we _must_ block on the JVM. Some legacy APIs, and even industry-standard APIs like JDBC, block threads. Given how much valuable blocking synchronous code is out there, we often have no choice but to use it.

These days, asynchronous data types like `Future` (combined with `Promise`) or [ZIO](https://github.com/scalaz/scalaz-zio)'s `IO` make it straightforward to write asynchronous applications. [ZIO](https://github.com/scalaz/scalaz-zio) goes further (like Monix `Task`) and makes it easy to write hybrid applications that are synchronous and asynchronous.

The _real_ trouble starts when asynchronous code is combined with synchronous code.

## Tricky Thread Management

Asynchronous code executes useful work continuously until it registers its callbacks, and then it suspends until the callbacks are invoked. As soon as the callbacks are registered, the thread executing the asynchronous code is free to do more useful work for other tasks.

Blocking synchronous code, on the other hand, might execute for a _really long time_. A thread might have to wait around for tens or hundreds of milliseconds, or in extreme cases, minutes or hours&mdash;all the while, doing absolutely nothing.

Because asynchronous and blocking code behave differently, we need to be very careful about how we run the different types of code.

In all cases, we need to execute different tasks on different threads, but creating threads is slow. Instead of creating threads all the time, data types like [ZIO](https://github.com/scalaz/scalaz-zio)'s `IO` and `Future` use _thread pools_, which are collections of pre-allocated threads that wait around to execute any tasks _submitted_ to the thread pools.

CPUs only have a certain number of _cores_, each of which can execute tasks more or less independently of the other cores. If we create more threads than cores, then the operating system spends a lot of time switching between threads (_context switching_), because the hardware can't physically run them all at the same time. So _ideally_, to be maximally efficient, we would only create as many threads as there are cores, to minimize context switching.

This strategy works well for asynchronous code, because asynchronous code is constantly doing useful work until it suspends. However, it works poorly for blocking code (halting the application), because a thread pool cannot accept more work after all threads are busy&mdash;and for blocking code, they might stay busy (doing nothing!) for a very long time.

As a result, well-designed applications actually have at least _two_ thread pools:

1. One thread pool, designed for asynchronous code, has a _fixed_ number of threads, usually equal to the number of cores on the CPU.
2. Another thread pool, designed for blocking code, has a _dynamic_ number of threads (more threads will be added to the pool when necessary), which is inefficient (but what can you do!).

Keeping the right tasks on the right thread pools has proven a _big mess_, even in modern effect systems. Functional programs can pull in libraries like [Christopher Davenport](https://twitter.com/davenpcm)'s excellent [linebacker](https://github.com/ChristopherDavenport/linebacker) library.

However, in preparation for the upcoming release of [ZIO](https://github.com/scalaz/scalaz-zio) 1.0, _no_ additional libraries are necessary for proper thread management, and you gain some _killer features_ not found in any other effect system.

## The ZIO Trio

[ZIO](https://github.com/scalaz/scalaz-zio) has three functions that make thread management a breeze:

1. `IO#lock(executor)`
2. `scalaz.zio.blocking.blocking(task)`
3. `scalaz.zio.blocking.effectBlocking(task)`

Internally, two of these functions benefit from the fact that the [ZIO](https://github.com/scalaz/scalaz-zio) runtime system has two primary thread pools, which are set by the user at the level of the main function. One of these thread pools is for asynchronous code, and one is for blocking code. These pools are configurable, of course, but the built-in defaults provide excellent performance for most applications.

In order to avoid exhausting the asynchronous code, [ZIO](https://github.com/scalaz/scalaz-zio) not only provides thread management functions, but is capable of making long-running asynchronous tasks yield to other tasks, providing a tunable level of fairness, but vastly higher performance than Scala's own `Future` (which yields on every operation of every asynchronous task).

The sections that follow introduce each of the three thread management functions in some detail.

### Lock

The first method that [ZIO](https://github.com/scalaz/scalaz-zio) provides to support thread management is `IO#lock`, which allows you to run a task on a given thread pool.

The `lock` function guarantees that the specified task will be executed with a given executor, which chooses the thread pool.

{% highlight scala %}
task.lock(executor)
{% endhighlight %}

With other functional effect types, even though you can shift a task to another thread pool (via something like `shift` or `evalOn`), any asynchronous boundary in the task can shift it _off_ the thread pool. 

Because whether or not a task is composed of asynchronous boundaries is not visible in its type, it's impossible to reason locally about _where_ a task will run using the `shift` and `evalOn` primitives of other effect types. They have poorly-defined semantics.

The unique guarantee that `lock` provides is semantically well-defined and stronger than any other effect system in Scala: the specified task will _always_ execute on the given thread pool, even in the presence of asynchronous boundaries. 

The `lock` function is the missing ingredient for principled task location in purely functional effect systems. It lets you trivially verify with local reasoning that, for example, blocking effects will always be executed on a blocking thread pool; that a client request will always happen in a client library's thread pool; or that a UI task will always execute on the UI thread, even if composed with other tasks that may need to execute elsewhere.

### Blocking

The second method that [ZIO](https://github.com/scalaz/scalaz-zio) provides to support thread management is `blocking`, which allows you to lock a task on the blocking thread pool.

{% highlight scala %}
import scalaz.zio.blocking._

val blockingTask = blocking(task)
{% endhighlight %}

This function is not primitive (it's implemented in terms of `lock` and another function, which provides access to the two primary thread pools in [ZIO](https://github.com/scalaz/scalaz-zio)), but it's one of the most common methods for thread pool management in [ZIO](https://github.com/scalaz/scalaz-zio).

### Blocking

The third method that [ZIO](https://github.com/scalaz/scalaz-zio) provides to support thread management is `effectBlocking`, which allows suspending (or _importing_) a blocking effect into a pure task.

{% highlight scala %}
// def effectBlocking[A](effect: => A): IO[Throwable, A] = ...
import scalaz.zio.blocking._

val interruptibleSleep =
  effectBlocking(Thread.sleep(Long.MaxValue))
{% endhighlight %}

The `effectBlocking` function returns a task that has two important features:

1. It executes the effect on the blocking thread pool using the `blocking` combinator.
2. It is interruptible (assuming the effect is blocking!), and any interruption will delegate to Java's `Thread.interrupt`.

The following code snippet will interrupt a really long `Thread.sleep` (which is a blocking function on the JVM):

{% highlight scala %}
import scalaz.zio.blocking.effectBlocking

val interruptedSleep =
  for {
    fiber <- effectBlocking(Thread.sleep(Long.MaxValue)).fork
    _     <- fiber.interrupt
  } yield ()
{% endhighlight %}

Like `lock` and `blocking`, this feature is a first for functional effect systems in Scala. Now runaway JDBC queries, thread sleeps, remote file downloads, and other blocking effects can be trivally interrupted in a composable fashion using ordinary [ZIO](https://github.com/scalaz/scalaz-zio) interruption.

## Summary 

Proper thread management is essential for efficient applications. High-performance applications on the JVM should use at least two thread pools: a fixed one for asynchronous code, and a dynamic one for blocking code.

[ZIO](https://github.com/scalaz/scalaz-zio) bakes in these two primary thread pools so users don't need to worry about them (although, of course, they can be configured as necessary and applications can use many more than just two thread pools).

[ZIO](https://github.com/scalaz/scalaz-zio) uses its two primary thread pools to make thread management easy using three functions, which bring unique functionality to the functional effect systems in Scala:

 * The `lock` function locks a task to a given thread pool (even over asynchronous boundaries!), providing principled, well-defined semantics for locating tasks;
 * The `blocking` function locks a task to [ZIO](https://github.com/scalaz/scalaz-zio)'s blocking thread pool; 
 * The `effectBlocking` function suspends a blocking effect into an interruptible `IO`, by safely delegating to Java's own thread interruption, allowing composable interruption of any blocking code.

Thanks to no implicit execution contexts, no implicit context shifts, no weird edge-cases (like asynchronous boundaries shifting a task to another thread pool), built-in asynchronous and blocking thread pools, fairness and performance, and well-defined semantics, [ZIO](https://github.com/scalaz/scalaz-zio) makes it easier than ever to build applications that follow best practices.

Who said functional programming wasn't practical?

_Thanks to [Wiem Zine Elabidine](http://twitter.com/wiemzin) for proofreading this post._