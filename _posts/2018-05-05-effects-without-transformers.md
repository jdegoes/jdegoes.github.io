---
layout:       post
title:        "No More Transformers: High-Performance Effects in Scalaz 8"
description:  "Scalaz 8 eschews monad transformers, which have proven themselves to be impractical in Scala."
category:     articles
tags:         [fp, functional programming, type classes, scala, monads, lenses, effects, reactive, scalaz, cats, mtl, monad transformers]
image:
  feature: "scalaz-8-branding.png"
---

Monad transformers just aren't practical in Scala.

They impose tremendous performance overhead that rapidly leads to CPU-bound, memory-hogging applications that give functional programming in Scala a reputation for being impractical.

Fortunately, all hope is not lost! In this article, I'll describe the alternative approach that is being championed by Scalaz 8, which offers many of the benefits of MTL, but without the high costs.

## The Trouble with Transformers

Monad transformers rely on vertical composition to layer new effects onto other effects. For example, the `EitherT` monad transformer adds the effect of error management to some base effect `F[_]`.

In the following snippet, I have defined an `OptionT` transformer that adds the effect of optionality to some base effect `F[_]`:

{% highlight scala %}
case class OptionT[F[_], A](run: F[Option[A]]) {
  def map[B](f: A => B)(implicit F: Functor[F]): OptionT[F, B] =
    OptionT(run.map(_.map(f)))

  def flatMap[B](f: A => OptionT[F, B])(implicit F: Monad[F]): OptionT[F, B] =
    OptionT(run.flatMap(
      _.fold(Option.empty[B].point[F])(
        f andThen ((fa: OptionT[F, B]) => fa.run))))
}
object OptionT {
  def point[F[_]: Applicative, A](a: A): OptionT[F, A] =
    OptionT(a.point[Option].point[F])
}
{% endhighlight %}

If you've ever written high-performance code on the JVM, even a quick glance at this definition of `OptionT` should concern you. The transformer imposes additional indirection and heap usage for every usage of `point`, `map`, and `flatMap` operation in your application.

While runtimes for languages like Haskell are reasonably efficient at executing this type of code, the JVM is not. We can see this very clearly by performing a line-by-line analysis of the transformer.

The definition of the class introduces a wrapper `OptionT`:

{% highlight scala %}
case class OptionT[F[_], A](run: F[Option[A]]) {
{% endhighlight %}

Compared to the original effect `F[_]`, use of this wrapper will involve two additional allocations: an allocation for `Option`, and an allocation for `OptionT`. In the worst case (for a simple `F[_]`), this wrapper might *triple* memory consumption of your application!

The `map` function is defined as follows:

{% highlight scala %}
  def map[B](f: A => B)(implicit F: Functor[F]): OptionT[F, B] =
    OptionT(run.map(_.map(f)))
{% endhighlight %}

In the `map` of the original `F[_]`, there is a minimum of one method call. While no allocations are required, it is likely the function passed to `map` will be a lambda that closes over some local state, which means there will often be another allocation.

In the `map` function of the `OptionT` transformer, there are an additional 3 method calls (`OptionT.apply`, `run`, `map`), and an additional two allocations (`OptionT`, `_`), one for the `OptionT` wrapper and one for the anonymous function.

If the type class syntax is not free, like in the Cats library, then there will be even more allocations and method calls.

This looks bad, but the actual situation is *far worse* than it appears.

Monad transformers can be stacked on top of any monad. This means they need to require monadic constraints on `F[_]`, here represented by `implicit F: Functor[F]` on the `map` function (for this operation, we don't need to assume `Monad`, because all we need is `Functor`).

We are interacting with the base monad through a JVM interface, which has many, many implementations across our code base. Most likely, the JVM will not be able to determine which concrete class we are interacting with, and if so, the method calls will become *megamorphic*, which prevents many types of optimizations that can occur when calling methods on concrete classes.

The story for `flatMap` is similar:

{% highlight scala %}
  def flatMap[B](f: A => OptionT[F, B])(implicit F: Monad[F]): OptionT[F, B] =
    OptionT(run.flatMap(
      _.fold(Option.empty[B].point[F])(
        f andThen ((fa: OptionT[F, B]) => fa.run))))
{% endhighlight %}

This definition of `flatMap` could be optimized, but even if completely optimized, there is even more overhead than `map`.

Recently some benchmarks have compared the performance of Scalaz 8 IO (which has a bifunctor design for modeling errors) to its equivalent in `typelevel/cats`, which is roughly `EitherT[cats.effect.IO, E, A]` (modulo the additional failure case embedded into `IO`).

The difference in performance is *staggering*: Scalaz 8 IO is nearly **five times faster** than `EitherT[cats.effect.IO, E, A]`.

Actual use in production could be faster or slower than these figures suggest, depending on the overhead of the base monad and how well the JRE optimizes the code. However, most applications that use monad transformers use many layers (not just a single `EitherT` transformer), so the takeaway is clear: applications that use monad transformers will be tremendously slower and generate far more heap churn than applications that do not.

Monad transformers just aren't practical in Scala, a fact that Scalaz 8 is uniquely prepared to embrace.

## The Good Part of MTL

Monad transformers aren't all bad. In Haskell, the `mtl` library introduced *type classes* for abstracting over data types that support the same effect.

For example, `MonadState` abstracts over all data types that are capable of supporting getting and setting state (including, of course, the `StateT` monad transformer).

In fact, these days **MTL-style** does not refer to the use of monad transformers, per se, but to the use of the type classes that allow abstracting over the effects modeled by data structures.

In Scala, this style has become known as *finally tagless* for [historical reasons](http://okmij.org/ftp/tagless-final/index.html). Most programmers doing functional programming in Scala recommend and use finally tagless-style, because it offers additional flexibility (for example, mocking out services for testing).

It is not widely known in the Scala programming community that *finally tagless* does not require monad transformers. In fact, there is absolutely no connection between finally tagless and monad transformers!

In the next section, I will show how you can use *MTL-style*, without specializing to concrete monad transformers.

## M-L Without the T

Let's say our application has some state type `AppState`. Rather than use `StateT[F, S, A]`, which is a monad transformer that adds state management to some base monad `F[_]`, we will instead use the `MonadState` type class:

{% highlight scala %}
trait MonadState[F[_], S] extends Monad[F] { self =>
  def get: F[S]
  def put(s: S): F[Unit]
}
{% endhighlight %}

We can then use `MonadState` in our application like so:

{% highlight scala %}
def runApp[F[_]](implicit F: MonadState[F, AppState]): F[Unit] =
  for {
    state <- F.get
    ...
    _ <- F.put(state.copy(...))
    ...
  } yield ()
{% endhighlight %}

Notice how this type class says *absolutely nothing* about the data types `F[_]` that can support state management.

While we *can* use our function `updateState` with a `StateT` monad transformer, there is no *requirement* that we do so.

The high-performance, industrial-strength approach embraced by Scalaz 8 involves defining instances for a newtype wrapper around the `IO` effect monad.

I'll show you how to do this in the next section.

## Powered by Scalaz 8 IO™

The first step in the Scalaz 8 approach involves defining a newtype wrapper for `IO`. The point of this wrapper is to create a unique type, which will allow us to define instances of type classes like `MonadState` that are specific to the needs of our application.

Although there are more reliable ways, one simple way to define a newtype is to declare a class that extends `AnyVal` and stores a single value:

{% highlight scala %}
class MyIO[A](val run: IO[MyError, A]) extends AnyVal
{% endhighlight %}

Once we have define this newtype, then we need to create instances for all the type classes used by our application.

There are several ways to create an instance of `MonadState` for `MyIO`. Since instances are first-class values in Scala, I prefer the following way, which uses an `IORef` to manage the state:

{% highlight scala %}
def createMonadState[E, S](initial: S): IO[E, MonadState[MyIO, S]] =
  for {
    ref <- IORef(initial)
  } yield new MonadState[MyIO, S] {
    def get: MyIO[S] = MyIO(ref.read)
    def put(s: S): MyIO[Unit] = MyIO(ref.write(s))
  }
{% endhighlight %}

We can now use this function as follows inside our main function:

{% highlight scala %}
def main(args: IList[String]) =
  for {
    monadState <- createMonadState(AppState(...))
    _          <- runApp(monadState)
  } yield ()
{% endhighlight %}

Because `MyIO` is nothing more than a newtype for `IO`, there are no additional heap allocations or method calls associated with use of the data type. This means that you get as close to the raw performance of `IO` as is possible in the finally tagless style.

This approach works smoothly and efficiently for most common type classes, including `MonadReader`, `MonadWriter`, and many others.

Now you can have your cake and eat it, too: use type classes to precisely capture the minimal set of effects required by different parts of your application, and use instances of these type classes for your own newtype around `IO`, giving you both abstraction and (relatively) high-performance.

## Stack Safety

In Scala, many obvious monad transformers are stack unsafe. For example, the classic definition of `StateT` is not stack safe. It can be modified to become stack safe, but the performance of an already slow data type becomes even slower, with many more method calls and allocations.

The problem is not limited to transformers, either. It is common to use stack-safe monad transformers to implement base monads (`State`, `Writer`, `Reader`, and so on). For example, one can define a stack-safe base `State` monad by using the type alias `type State[S, A] = StateT[F, S, A]`, for some stack-safe `StateT` and trampolined monad `F`.

While this approach (seen [in the Cats library](https://github.com/typelevel/cats/blob/55e09026bd7e48f1cedf358cbe4f54666aca5460/core/src/main/scala/cats/data/package.scala#L48)) creates stack safety for the base monad (by piggybacking on the safety of the transformer and the trampolined base monad), the resulting `State` monad, which is powered by a slow `StateT` transformer (made slower by stack safety!), becomes even slower due to trampolining.

The technique presented in this post lets you eliminate base monads like `State`, bypassing the massive performance overhead entailed by older approaches to stack safety.

## Going Forward

A decade of functional programming in Scala has taught us that while the abstraction afforded by MTL is extremely powerful and useful, transformers themselves just don't work well in Scala&mdash;not today, and maybe not ever.

Fortunately, we can take advantage of MTL-style without sacrificing performance, merely by defining instances for cost-free wrappers around `IO`.

As outlined in this post, there is a small amount of ceremony associated with this style, especially if you want to use localized effects. In addition, the `AnyVal` approach to creating newtypes is fraught with problems and doesn't have the guarantees we need.

In Scalaz 8, we should be able to address these issues in a way that makes it simple for users to use the right approach.

Watch this space for more coverage of how recent innovations are making functional programming in Scala practical&mdash;suitable for mainstream use even for demanding technical requirements.
