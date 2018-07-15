---
layout:       post
title:        "Scala Wars: FP-OOP vs FP"
description:  "Modeling effects in Scala with pure FP provides compelling advantages over FP-OOP alternatives."
category:     articles
tags:         [fp, functional programming, type classes, scala, monads, effects, reactive, scalaz, fp-oop, zio, oop]
---

I'm one of those crazy functional programming zealots and the architect of [ZIO](http://github.com/scalaz/scalaz-zio), a purely-functional effect system for Scala.

I use the purely-functional `IO` in Scala to model _all_ my effects&mdash;not out of any ideological commitment to functional programming, but because it makes my life easier and makes my programs faster, clearer and better.

In general, we functional programmers prefer to use `IO` in Scala for all of the following reasons:

1. **Uniform Reasoning**
2. **Uniform Purity**
3. **Reified Programs**
4. **Performance & Power**
5. **Flexibility**
6. **Industry Proven**

In the sections that follow, I'll explain each of these reasons.

### Uniform Reasoning

When you use immutable data structures like Scala's collections, or when you rely on pattern matching and recursion to manipulate your own immutable data structures, you are able to rely on very powerful reasoning properties of functional code:

 * **Inversion of Control**. When you see a functional expression like `list.map(f)`, you know that the `map` method cannot modify the original `list`. In functional code, the callee cannot change or do anything (only the caller), which means you don't have to program defensively and you can push decisions higher in your code base, resulting in more flexible programs with less wiring.
 * **Equational Reasoning**. When you see an expression like `list ++ list`, you know the meaning of this expression by inlining the value of `list`, and simplifying. More generally, in any functional code, you always know what an expression means through substitution and simplification. This makes it easier to understand what your programs do, and lets you refactor safely, without worrying that you're changing the behavior of your programs.
 * **Type-Based Reasoning**. When you see a type like `List[A] => (A => B) => List[B]` in functional code, you have some idea of what this function can and cannot do, merely by looking at the type. In Java, a method like `(InetAddress, InetAddress) => Boolean` could perform network IO (and in fact, `equals` on `InetAddress` does just this!), which means you need to thoroughly study implementations to understand what functions do.

These reasoning properties only hold for functional code. You have to use different reasoning properties for non-functional code, which involve examining the implementation of methods (to see what they do, since the types don't tell you!) and simulating their stateful execution in your head.

Humans aren't particularly good computers, and so lots of us don't like to study implementations and simulate stateful execution in our head.

If you mix functional code and non-functional code, then you have to constantly change the way you reason about the code, depending on whether you are in a functional section, or a non-functional section. The process of constantly switching how you reason about the code is mentally exhausting and error-prone.

Contrast this with `IO`, which is just an ordinary immutable value, exactly like `List`. All methods on `IO` return new `IO` values derived from the original. Because `IO` is an immutable data structure, you can model effects while still benefiting from all the reasoning properties of functional code.

The functional approach gives you a powerful tools to understand and safely change your code, and it eliminates the need to switch back and forth between functional and non-functional reasoning.

### Uniform Purity

It's well-known that functional code mixes poorly with non-functional code. For example, if you're using a functional data structure like `Stream`, and you map over the `Stream`, it doesn't actually do anything. If you place a `println` in the `map` function, you're not going to see anything.

It's not obvious from the types alone where you can insert side-effects in higher-order functional code, and have them do what you want them to do, when you want them to do it.

`IO` lets you use effects anywhere and gives you the tools to reason about them at compile-time. If you map over a `Stream[A]` and convert each `A` to `IO[B]`, then you end up with a `Stream[IO[B]]`. This type tells you it's now a stream of effects. In order to do anything, you have to pull those effects out and combine them together. One way to do that is to call `IO.sequence`, which transforms `Stream[IO[B]]` into `IO[Stream[B]]`.

There are other ways, but no matter how you choose to transform the stream of effects into an effect, the types tell you what you must do in order to compose the effect together with the `IO` of your main program. You can't do something that doesn't make sense, because the compiler won't let you.

With `IO`, you never have to worry mixing pure and impure code, because everything is uniformly pure.

### Reified Programs

In languages like Scala, procedural programs are not first-class citizens. You can't pass a bunch of statements around from one function to another. You can't store them inside data structures. You can't combine them in an expression-oriented way.

Sometimes this limitation cripples the functionality of your programs. For example, an undo/redo manager requires the ability to model effects as values, so you can store them in a stack; user-interfaces are usefully modeled as infinite trees, where different user actions correspond to different paths down the tree; and so on.

Other times, the lack of first-class programs means poor expressivity. Expressions, like `1 + 2 * 2`, provide enormous power in a compact package, allowing us to easily build larger expressions from smaller ones. Yet, because procedural statements are not first-class values, we don't have this same power with our programs.

`IO` cleanly solves these issues by turning our programs into first-class values. We can pass these values around, we can store them in data structures, and we can combine them in an expression-oriented way to yield other programs.

I'll show you a few examples with [ZIO](http://github.com/scalaz/scalaz-zio) to demonstrate the power of this approach.

Suppose we want to define a combinator which retries a program until it succeeds. It's as simple as this:

```
def retry[E, A](io: IO[E, A]): IO[E, A] = io orElse retry(io)
```

Or suppose we want to define a combinator to run an action forever:

```
def forever[E, A](io: IO[E, A]): IO[E, Nothing] = io *> forever(io)
```

Or suppose we want to continuously take values from a queue and upload them to S3 (logging failures), in a separate thread:

```
queue.take.flatMap(uploadToS3 orElse logFailure).forever.fork
```

Or suppose we want to spin up 1000 threads to load test a web server, adding a random delay before each worker starts, and timing out the whole load test after 60 seconds, cleanly shutting down each worker:

```
IO.parAll(List.fill(1000)(randomTime.flatMap(IO.sleep) *> doLoadTest))).timeout(60.seconds)
```

The fact that we can define and use combinators on our programs allows us to solve very complex problems with very little code. These solutions don't have distracting clutter or irrelevant details; their structure cleanly reflects their intended semantics.

### Performance & Power

The final very attractive property of an effect monad like [ZIO](http://github.com/scalaz/scalaz-zio) (and similar ones like Monix `Task`) is that it can provide us with a level of performance and power that cannot be obtained elsewhere.

For example, in a variety of benchmarks, ZIO is 100x or more faster than Scala's own `Future`. `Future` is not functional code, and it has all the drawbacks of dysfunctional code, including a different reasoning model and poor interop with pure code.

That's not all, though. Programs written using ZIO can be maximally "lazy", without effort, thanks to a feature of ZIO called _interruption_. _Interruption_ allows the runtime system for ZIO to interrupt any "thread" immediately and safely release resources.

This happens automatically in many places. For example, if a web response depends on some key piece of information that's not available, then all the other computations running in parallel will be gracefully shut down, reducing latency, network bandwidth, and CPU overhead. Similarly, if you race a bunch of computations, then as soon as one of them succeeds, the others will be terminated immediately and shut down gracefully.

ZIO models both synchronous and asynchronous effects in a uniform way. If you have some `IO[A]`, it could represent an arbitrary composition of both synchronous and asynchronous effects. But as a user, you don't have to care about that. If you don't use `IO`, then you need to structure your code quite differently, as a combination of both statements and callbacks&mdash;every bit of code is painfully aware of whether it's synchronous or asynchronous.

ZIO gives you other superpowers, too, like letting you supervise child "threads" spawned by a parent (so you can take action if they crash), and the ability to automatically shut down child "threads" when the parent "thread" finishes. These aren't threads in the JVM sense&mdash;they're super lightweight _fibers_, which are a user-land implementation of green threads, so you can have hundreds of thousands of them running at a time.

All of these capabilities rely on the functional nature of ZIO. Because `IO` is just an ordinary immutable value, your main function needs to call the runtime system to interpret the data structure. This interpreter adds very powerful features that you can't get from other approaches.

Even if you hate functional programming, the benefits of ZIO are so compelling, you might be tempted to use it anyway.

### Flexibility

Although I have mentioned using `IO` data types, and you'll hear functional programmers talk about writing `IO` programs, it's more common these days to define type classes to represent bundles of effects, and to make programs polymorphic in the effect type, so long as they provide the required capabilities.

This pattern, sometimes called _final tagless_ or _mtl style_, is very powerful. It lets you easily plug-in new implementations of type classes without changing any code. For example, you could have multiple implementations of a key-value store.

While there are many uses for this functionality, a powerful use is testing systems thoroughly without having to rely on error-prone, type-unsafe mocking frameworks built on byte code rewriting and custom class-loading.

### Industry Proven

Other proposed solutions for modeling effects in Scala are interesting research projects&mdash;worthy of exploration in labs, but they have not been proven in industry.

For example, a proposal to use implicit function types for effect capabilities (`CanIO`) has a variety of drawbacks that render it Dead On Arrival:

 * **Strictly Synchronous**. Implicit function types are not powerful enough to model asynchronous effects, generators, or other effects that require continuations. If you combine them with continuations, then you now have two effect systems, when you really only need one (continuations are actually powerful enough to implement everything else!). The stitched-together creation adds complexity, confusion, and cognitive and runtime overhead.
 * **Stack Suicide**. Functional programming relies on recursion to perform (possibly infinite) iteration, and while `IO` types in Scala are built for unbounded, safe recursion, implicit function types will stack overflow on recursion, making them unsuitable for general-purpose programming.
 * **Escaped Effects**. Implicit function types require a linear type system in order to guarantee that no capabilities are leaked. Because Scala does not have linear typing, it cannot guarantee that capabilities won't leak. Monadic approaches are much simpler and can't leak.
 * **Dysfunctional Drawbacks**. Implicit function types encourage you to write non-functional code, and therefore have all the drawbacks of dysfunctional code&mdash;you cannot reason about them in the same way, they mix poorly with pure code, they don't reify effects as values, and so on.

Contrast this with monadic approaches based on `IO`, which have had 30 years of active development and production use. They have proven so successful, you can now find effect monads implemented in PureScript, Kotlin, Scala, Javascript, Java, C#, F#, and numerous other programming languages.

If you want to get work done and write functional code everywhere, then monads are the only general-purpose, industry-proven technique available.

### Summary

In summary, functional programmers like me don't use `IO` for ideological reasons. Instead, we use `IO` for very practical, real-world benefits.

A lot of these benefits are about making it easier for us to refactor our programs and to know what our programs mean and what they do, without having to study their implementations and simulate their runtime behavior in our head.

Other benefits include uniform purity, so we don't have to worry about mixing pure and impure code; and the ability to turn our programs into first-class values, so we can pass them around, store them in data structures, and combine them in expressions&mdash;giving us insane levels of expressivity.

Beyond all this, there is a strong business case for using an `IO` effect system like [ZIO](http://github.com/scalaz/scalaz-zio). ZIO includes features like super fast performance, a uniform interface for synchronous and asynchronous effects, lazy evaluation (_interruption_) that safely eliminates wasted resources (memory, network, CPU), parallelism, concurrency, scalability, and so much more.

Unlike implicit function types and other academic curiosities, monads are battle-tested and industry proven. We know how they work, how they compose, and how they perform, and they are becoming pervasive across the functional programming community, regardless of language.

Maybe these reasons aren't compelling enough to get you to use `IO`. But they should _at least_ be compelling enough to get you to _try_ `IO`.

After all, the growing armies of developers using `IO` to solve everyday problems can't all be crazy! (Or can we?)
