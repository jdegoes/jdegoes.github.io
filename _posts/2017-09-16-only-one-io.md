---
layout:       post
title:        "There Can Be Only One...IO Monad"
description:  "Effect monads are proliferating in Scala, but unlike many other data structures, more effect monads are not necessarily a good thing."
category:     articles
tags:         [fp, functional programming, type classes, scala, monads, lenses, effects, reactive, scalaz, cats]
image:
  feature: "scalaz-8-branding.png"
---

Scala developers have more choices than ever to represent effectful computation. Broadly speaking, these choices are divided into three categories: *non-functional*, *mostly functional*, and *purely functional*.

The *non-functional* solutions focus on concurrency, and are difficult to use in functional programs because they violate referential transparency:

 * [scala.concurrent.Future](https://www.scala-lang.org/api/current/scala/concurrent/Future.html)
 * [scala.concurrent.Promise](https://www.scala-lang.org/api/current/scala/concurrent/Promise.html)
 * [com.twitter.util.Future](https://twitter.github.io/finagle/guide/Futures.html)

The *mostly functional* solutions have a core that can be used in functional programs, but they do expose features that aren't purely functional.

In order of increasing purity:

 * [monix.eval.Task](https://monix.io/docs/2x/eval/task.html)
 * [monix.eval.Coeval](https://monix.io/docs/2x/eval/coeval.html)
 * [scalaz.concurrent.Task](https://github.com/scalaz/scalaz/blob/series/7.3.x/concurrent/src/main/scala/scalaz/concurrent/Task.scala)
 * [cats.effect.IO](https://github.com/typelevel/cats-effect/blob/master/core/shared/src/main/scala/cats/effect/IO.scala)
 * [scalaz.effect.IO](https://github.com/scalaz/scalaz/blob/series/7.3.x/effect/src/main/scala/scalaz/effect/IO.scala) (7.x)

Solutions in the *purely functional* category expose only purely functional interfaces, including [Scalaz 8's IO](https://degoes.net/articles/scalaz8-is-the-future), which has no impure methods on `IO`, no impure execution contexts (implicit or otherwise), and no side-effecting combinators.

There are [lots](https://github.com/alexknvl/sio/blob/master/core/src/main/scala/sio/core/IO.scala) [of](https://github.com/aloiscochard/scato/tree/master/io/src/main/scala) [other](https://github.com/tel/scala-tk) [designs](https://github.com/ThoughtWorksInc/future.scala) in the wild at various stages of development and production usage.

While all these choices are no doubt confusing for Scala developers, I'm a big fan of competing solutions. They quickly explore the landscape of design alternatives. In the end, the "best" solutions end up winning, usually for several different and conflicting definitions of *best*.

While the competition is undoubtedly beneficial, I recently encountered what I consider to be a myth in the Scala community: that the proliferation of effect types is *necessary*, because unlike Haskell, Scala programs have needs that are *too diverse* to be met by a single `IO` type.

Today, I'm going to debunk this myth once and for all.

## One IO per Program

It's easy enough to show that for any given purely functional Scala program, it only makes sense to use a *single* effect monad.

The reason for this is that a purely functional program will necessarily be expressed as a single value of type `F[Unit]`, where `F[_]` is the effect monad of the program (such as `Future` or `Task` or `IO`).

Conventionally, we call this value `main`:

{% highlight scala %}
object MyApp extends SafeApp {
  def main: IO[Unit] = ...
}
{% endhighlight %}

Purely functional programs are expressed in terms of *composition*. Smaller fragments are composed to form larger fragments. For example, if we have some fragment `F[A]`, representing a computation producing some value `A`, and another fragment `F[B]`, then we can compose them in various ways to yield another fragment `F[C]`, producing another value.

But if we have one fragment expressed as `F[A]`, and another expressed as `G[B]`, where both `F[_]` and `G[_]` are *different* effect monads, then we have a problem: they don't compose! We can't take a `Task` and a `Future` and compose them together, for example.

In order to compose two effect monads, we'd have to convert the `F[_]` to the `G[_]`, or the `G[_]` to the `F[_]`. But if we can convert from one to the other, we have proven the second is at least as powerful as the first, *so there is no reason to use both*. We only need the more powerful effect monad.

Using two different effect monads, and then converting from one to another, just increases overhead, because many more structures will be allocated and many more virtual methods will be invoked. So not only is there *no reason* to use two effect monads in the same program, but there are (performance) reasons to *not* do so!

This is why purely functional Scala programs have only *one* base effect type.

## One IO per Ecosystem

Hopefully by now, you're convinced that any given Scala application needs only one effect monad such as `Task` or `IO`. However, you *might* still think that different applications might require *different* effect monads that provide fundamentally different feature sets.

For example, maybe Twitter's "effect monad" needs cancelation, to avoid wasting network resources, so they have to use a different `Future` than other applications.

This myth rests on the (false) premise that different types of applications require fundamentally distinct capabilities that cannot be provided in a single effect monad.

To dispel it, I will now present the *ultimate* `IO` monad, which can provide *any capability* required by *any application* whatsoever.

Brace yourself for the overwhelming torrent of Scala code:

{% highlight scala %}
sealed abstract class IO[A](val unsafePerformIO: () => A) {
  final def map[B](ab: A => B): IO[B] =
    new IO(() => ab(unsafePerformIO()))
  final def flatMap[B](afb: A => IO[B]): IO[B] =
    new IO(() => afb(unsafePerformIO()).unsafePerformIO())
  final def attempt: IO[Either[Throwable, A]] = new IO(() => {
    try Right(unsafePerformIO())
    catch {
      case t : Throwable => Left(t)
    }
  })
}
object IO {
  final def apply[A](a: => A): IO[A] = new IO(() => a)

  final def fail[A](t: Throwable): IO[A] = new IO(() => throw t)
}
{% endhighlight %}

That wasn't so bad, was it?

This `IO` monad is defined in just 17 lines of extravagantly spacious code. It can do anything that *any* other effect monad in the entire Scala ecosystem can do!

To show this, it suffices to point out two facts:

1. All other effect monads must be implemented in terms of the imperative subset of Scala programming (that is, there isn't a *different* language available to implement Scala effect monads; it's just the same old Scala!).
2. All imperative subsets of Scala code can be encoded with the above `IO` monad.

The first point is obvious. The second point can be seen by showing how a generic snippet of imperative Scala can be translated into the above `IO` monad.

Let's say we have the following snippet of imperative Scala, which consists of imperative statements `v_1 = <e_1>` to `v_n = <e_n>`, each of which do impure, effectful computation by executing an arbitrary chunk of Scala code and storing the result in a variable:

{% highlight scala %}
val v_1 = <e_1>
val v_2 = <e_2>
...
val v_n = <e_n>
<e_ret>
{% endhighlight %}

In general, subsequent expressions will depend on variables introduced by prior expressions. For example, here's a simple console program that demonstrates this sequential dependency:

{% highlight scala %}
val v_1 = scala.Console.println("What is your name?")
val v_2 = scala.io.StdIn.readLine()
val v_3 = scala.Console.println("Hello, " + v_2 + "!")
()
{% endhighlight %}

Notice how the expression defining `v_3` refers to `v_2`. Now, if some expression `<e_i>` evaluates to `Unit` (such as the first expression above), its corresponding variable assignment in statement `v_i` can be ignored.

With a little effort, you should be able to convince yourself that *all* imperative Scala code can be written in this form. In turn, any imperative snippet in this form can be translated into the above `IO` as follows:

{% highlight scala %}
IO(<e_1>).flatMap(v1 =>
  IO(<e_2>).flatMap(v2 =>
    ...
    IO(<e_n>).flatMap(v3 =>
      IO(<e_ret>)
    )))
{% endhighlight %}

Of course, Scala provides `for` comprehension syntax for code structured in this fashion (otherwise known as *do notation*), so we can write this as simply:

{% highlight scala %}
for {
  v_1 <- IO(<e_1>)
  v_2 <- IO(<e_2>)
  ...
  v_n <- IO(<e_n>)
  v_r <- IO(<e_ret>)
} yield v_r
{% endhighlight %}

This structure captures *all the effects* of the original imperative snippet, which can be recovered by running the `unsafePerformIO()` method on the final `IO` value.

Some people have claimed that, for example, a synchronous `IO` like this one doesn't provide asynchronicity, so we need a `Future` for asynchronous applications.

That's completely false, as I've demonstrated here. In fact, for this particular myth, it's useful to show a simple but powerful encoding for asynchronous effects (technically a specialization of the continuation monad `ContT`).

We can define a wrapper for asynchronous `IO` computations like so:

{% highlight scala %}
type Try[A] = Either[Throwable, A]

final case class Async[A](register: (Try[A] => IO[Unit]) => IO[Unit]) { self =>
  final def map[B](ab: A => B): Async[B] = Async[B] { callback =>
    self.register {
      case Left(e) => callback(Left(e))
      case Right(a) => callback(Right(ab(a)))
    }
  }
  final def flatMap[B](afb: A => Async[B]): Async[B] = Async[B] { callback =>
    self.register {
      case Left(e) => callback(Left(e))
      case Right(a) => afb(a).register(callback)
    }
  }
}
object Async {
  final def apply[A](a: => A): Async[A] = Async[A] { callback =>
    callback(Right(a))
  }

  final def fail[A](e: Throwable): Async[A] = Async[A] { callback =>
    callback(Left(e))
  }
}
{% endhighlight %}

There you have it, an asynchronous monad in about 20 lines of code, built on the `IO` monad previously introduced. It doesn't matter *what* features you want to add to your effect monad, they can *all* be easily expressed in terms of the above `IO`.

In other words, no effect monad, no matter who wrote it, and no matter what it does, is more expressive than the 17 LOC effect monad introduced in this post!

## Why Scalaz 8 IO?

Some may wonder if the 17 LOC monad I introduced in this post is as expressive as any other, why I am [spending additional time developing Scalaz 8 IO](https://degoes.net/articles/scalaz8-is-the-future).

The answer is simple: although the `IO` monad introduced in this post is as *expressive* as any other, expressing features common to many Scala applications would introduce many additional allocations and virtual method invocations.

The `IO` monad I'm developing for Scalaz 8 bakes in additional functionality, not because it's necessary, but because doing so *greatly* increases performance.

Most Scala applications have to constantly deal with three real world concerns ignored by toy and pedagogical effect monads:

 * **Asynchronous Computation**
 * **Concurrent Computation**
 * **Resource Management**

By providing clean, composable, and built-in semantics for dealing with these real world concerns, Scalaz 8 `IO` will provide the critical combination of performance, ease-of-use, and principled design necessary to succeed in a crowded marketplace of non-functional, semi-functional, prototype, and toy effect monads.

That's the goal, anyway. The Scala community will decide if it's successful.

## Summary

When it comes to modeling effectful computation, the Scala community has more choices than ever&mdash;from `Future` monads baked into Scala, to `Task` and `IO` monads developed by the community, with varying degrees of purity and safeness.

While these competing solutions are useful to explore the landscape of possible designs, it's critical to remember that programs only *need* one effect monad. More than that, because every effect monad is *as expressive* as any other, there's no reason why we need more than *one* effect monad for the entire Scala community.

The only reason to specialize an `IO` monad more than the toy example provided in this post is to improve performance. Real world concerns for nearly all Scala programs include asynchronicity, concurrency, and resource management, and in my opinion, the winning design in this space will provide *all three* in a performant, principled, and pure package.

In the Haskell world, there's only one `IO` monad, and there's no need for anything else. Over time, we may find that whatever effect monad design ends up winning will become Scala's one and only `IO` monad.

At the end of the day, after all, there can be only one!
