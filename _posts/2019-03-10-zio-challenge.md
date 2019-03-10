---
layout:       post
title:        "The Functional Scala Concurrency Challenge"
description:  "A fun little challenge to improve your skills using FP for concurrency."
category:     articles
tags:         [fp, functional programming, type classes, scala, monads, lenses, effects, reactive, scalaz, cats, mtl, monad transformers, zio, reader, environmental effects]
---

Recently, while teaching my first-ever workshop for [ZIO](https://github.com/scalaz/scalaz-zio) with the great crew at [Scalac](https://scalac.io), we had a chance to build a purely functional circuit breaker using ZIO's `Ref`, which is a model of a mutable reference that can be updated atomically.

A circuit breaker guards access to an external service, like a database, third-party API or microservice. Once too many requests to the service fail, the circuit breaker trips, and immediately fails all requests, until the circuit breaker has a chance to reset.

Circuit breakers not only protect external services from overload (giving them a chance to recover after failure), but they help conserve local resources (such as sockets, threads, and the like) that would otherwise be wasted on a lost cost.

As opposed to retry policies, which dictate how individual requests are retried, circuit breakers share global knowledge across a system, so different fibers can act more intelligently and in a coordinated fashion.

Circuit breakers are often modeled as having three states: open, closed, and half-open. The circuit breaker logic (possibly aided by configuration parameters) is responsible for transitioning between the states based on inspecting the status of requests. 

At the ZIO workshop, exploring different possibilities for circuit breakers made me realize something: I _really_ don't like circuit breakers. I find the arbitrary nature of the number of states and the switching conditions deeply disturbing.

I think we can do better than circuit breakers, and have some fun while we're at it! So in this post, I'm going to issue a challenge for all you fans of functional programming in Scala: build a better circuit breaker!

## The Challenge

Instead of a circuit breaker, I want you to build a _tap_, which adjusts the flow of requests continuously through the tap.

The flow is adjusted based on observed failures that qualify (i.e. match some user-defined predicate).

If you want to use ZIO to implement the `Tap`, then your API should conform to the following interface:

{% highlight scala %}
/**
 * A `Tap` adjusts the flow of tasks through 
 * an external service in response to observed
 * failures in the service, always trying to 
 * maximize flow while attempting to meet the 
 * user-defined upper bound on failures.
 */
trait Tap[-E1, +E2] {
  /**
   * Sends the task through the tap. The 
   * returned task may fail immediately with a
   * default error depending on the service 
   * being guarded by the tap.
   */
  def apply[R, E >: E2 <: E1, A](
    effect: ZIO[R, E, A]): ZIO[R, E, A]
}
object Tap {
  /**
   * Creates a tap that aims for the specified 
   * maximum error rate, using the specified 
   * function to qualify errors (unqualified 
   * errors are not treated as failures for 
   * purposes of the tap), and the specified 
   * default error used for rejecting tasks
   * submitted to the tap.
   */
  def make[E1, E2](
    errBound  : Percentage,
    qualified : E1 => Boolean, 
    rejected  : => E2): UIO[Tap[E1, E2]] = ???
}
{% endhighlight %}

If you want to use Cats IO or Monix to implement `Tap`, then your API should conform to the following interface (or its polymorphic equivalent):

{% highlight scala %}
trait Tap {
  def apply[A](effect: Task[A]): Task[A]
}
object Tap {
  def make(
    errBound  : Percentage,
    qualified : Throwable => Boolean, 
    rejected  : => Throwable): Task[Tap] = ???
}
{% endhighlight %}

Your implementation of `Tap` should satisfy the following requirement:

_The tap must continuously adjust the percentage of tasks it lets through until it finds the maximum flow rate that satisfies the user-defined maximum error bound._
 
Thus, if you create a tap with a maximum error rate of _1%_, and suddenly 50% of all tasks are failing, then the tap will decrease flow until the failure rate stabilizes at 1%.

As the service is recovering, the failure rate will drop below 1%, which will cause the tap to increase flow and let more tasks through.

Once the service has fully recovered, the failure rate will hit 0% (or within some distance of that target), at which point, the tap will let all tasks through.

Your implementation must be purely functional and concurrent. Bonus points for demonstrating your knowledge of concurrency primitives, such as `Ref`, `Promise` (`Deferred`), and so forth.

## Winners

The main reason to work on this challenge is to explore solutions for concurrency in functional Scala. It's a fun little project that will take you on a grand tour of modern, purely functional effect systems in Scala.

That said, I want to give you a little extra motivation to work on this problem!

If you post your code in a Gist so the whole world can learn from your solution, then I'll both promote your solution, and buy you a drink next time we're in the same city!

Finally, if your solution is among the top 1-3 I receive over the next 2 weeks, I'll connect with you on LinkedIn and write a short, honest endorsement of your skills in functional Scala.

Read? On your marks, get set, go!