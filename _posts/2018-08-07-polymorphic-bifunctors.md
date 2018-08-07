---
layout:       post
title:        "Using ZIO with Tagless-Final"
description:  "ZIO's new bifunctor design works beautifully with tagless-final style, with or without modifications."
category:     articles
tags:         [zio, bifunctor, fp, functional programming, scala, monads, effects, reactive, scalaz, cats, tagless-final, finally tagless, mtl]
---

Since launching [ZIO](http://github.com/scalaz/scalaz-zio/) a few months ago (formerly known as the _Scalaz 8 IO monad_), one of the most common questions I get is how to use its `IO` type with [*tagless-final*](https://blog.scalac.io/exploring-tagless-final.html).

In this post, I'll explain how ZIO works with tagless-final, and when it makes sense to modify your type classes to take advantage of the new power provided by ZIO for modeling typed errors.

## ZIO vs Other Effect Types

In ZIO, a value of type `IO[E, A]` is an immutable value that describes an effectful program. The program may fail with some error of type `E`, or produce a value of type `A`.

As I've [previously argued](http://degoes.net/articles/fpoop-vs-fp), this purely functional model has many practical benefits that make it [extremely attractive](http://degoes.net/articles/fpoop-vs-fp), even for developers who don't care about functional programming.

Other popular Scala libraries like [Monix](https://github.com/monix/monix) also have effect types, but ZIO is the only one that provides [typed errors](http://degoes.net/articles/bifunctor-io). The other effect types fix the error type to `Throwable`, which means it is not possible to constrain if and how effects may fail at compile-time, leading to bugs and poor reasoning properties.

This difference in power means that the *kinds* of `IO` and other effect types differ:

 * ZIO's effect type has kind `(*, *) -> *`, which means that the type constructor for `IO` takes two type parameters: the type of errors, and the type of values.
 * Other effect types have kind `* -> *`, which means their type constructors take one type parameter: the type of values.

Ordinarily, you don't notice this difference, because ZIO is designed to have excellent type inference. Scala figures out the type parameters for both `E` and `A`.

There's one place, however, where you have to think more carefully about the difference between ZIO and older effect types: the tagless-final encoding of effects.

## Tagless-Final

In the tagless-final pattern, you describe your effects using a type class, which typically takes a single type constructor `F[_]` (with kind `* -> *`). For example, we might have a type class that describes the effect of logging:

{% highlight scala %}
trait Logging[F[_ ]] {
  def log(message: String): F[Unit]
}
{% endhighlight %}

As demonstrated in a [recent live-coding talk](https://www.youtube.com/watch?v=sxudIMiOo68&t=53s%C2%A0&app=desktop) I did for _Fun(c)tional Programming Group_, using effect type classes like these has several benefits:

1. It allows us to see what effects functions use, so we can better understand our functions without having to drill down into their implementations, and find all the functions they call (which is error-prone and takes lots of time);
2. It allows us to reuse more code. For example, we can have different instances of the `Logging` type class for different production environments. One can log remotely, one can log locally, and one can not log at all.
3. It allows us to easily mock effectful code without having to interact with the real world (so our tests can be fast and deterministic) and without having to rely on painful mocking libraries.

Yet because ZIO's effect type takes two type parameters, and traditionally, in tagless-final, our effects only take one type parameter, many developers have had the question: how can we use ZIO with tagless-final?

In the sections that follow, I'll explore the two ways to use ZIO with tagless-final, and then give you some guidance on how to choose the best method for every situation.

## Fixing the Error Type

Although we cannot make `IO` implement a type class like `Logging` directly, because it has the wrong kind, if we partially apply `IO` to its first type parameter (that is, if we _fix_ the error type), then we can create an instance for the partially applied type constructor.

In the following example, I assume we have a function `def logIo(message: String): IO[Nothing, Unit]`, which I use to implement the type class for any fixed error type `E`:

{% highlight scala %}
// NOTE: Uses the kind-projector compiler-plugin for
// easier syntax for partial type application:
implicit def IOLogging[E]: Logging[IO[E, ?]] =
  new Logging[IO[E, ?]] {
    def log(message: String): IO[E, Unit] = logIo(message)
  }
{% endhighlight %}

With this approach, we may now use the `Logging` type class with the `IO` type. Let's say we have some function `processData` that requires logging:

{% highlight scala %}
def processData[F[_ ]: Logging: Monad]: F[Unit] =
  ???
{% endhighlight %}

Then now we can call `processData` with `IO[E, ?]` for any error type `E`, because our implicit function `IOLogging` defines an entire _family_ of `Logging` instances across all type parameters `E`.

Sometimes, we can't provide instances for all error types. For example, if our logging function `logIo` can throw `IOException`, then we have to constrain the error type in our instance for the `IO` data type:

{% highlight scala %}
implicit val IOLogging: Logging[IO[IOException, ?]] =
  new Logging[IO[IOException, ?]] {
    def log(message: String): IO[IOException, Unit] =
      logIo(message)
  }
{% endhighlight %}

In this case, because we needed the ability to fail, we had to restrict the instance to a specific error type. This means we can no longer use `Logging` for just _any_ `IO` type; it has to be one that can fail with an `IOException` (or a supertype, like `Exception` or `Throwable`).

In both cases, we were able to use the same type class we would use for other effect types. We didn't need to change a thing.

## Varying the Error Type

The alternative to fixing error types is to let them vary, which requires that we change our tagless-final encoding.

Let's say we have the following type class, which allows us to execute some query locally or in a distributed fashion:

{% highlight scala %}
trait Analytics[F[_ ]] {
  def local(query: Query[A]): F[A]

  def distributed(query: Query[A]): F[A]
}
{% endhighlight %}

Let's say the local execution cannot fail, but the distributed execution can fail due to network errors. In this case, we would like to capture the distinction between these methods, so developers know by looking at the types what can happen.

To do this, we could always use an `Either`, but since the capability is baked into ZIO, there's no need to add additional runtime and cognitive overhead. Instead, we can just change the kind of our effect parameter from `* -> *` to `(*, *) -> *`.

This lets us vary the error type for each method:

{% highlight scala %}
trait Analytics[F[_,_]] {
  def local(query: Query[A]): F[Nothing, A]

  def distributed(query: Query[A]): F[NetworkError, A]
}
{% endhighlight %}

Now we can clearly see that `local` queries cannot fail, but `distributed` queries can fail due to network errors.

This type class does not limit us to ZIO, either. For the `Task` type in Monix, for example, while we cannot create an instance of `Analytics` for `Task` directly, we can create an instance for `EitherT[Task, ?, ?]` (`EitherT` is a monad transformer that adds typed errors to a base monad).

This means that when you parameterize your type classes over effects that take two type parameters, developers can choose whether to use ZIO or to use another effect monad and layer on typed errors with the `EitherT` monad transformer.

## Choices, Choices

The choice about whether to use tagless-final with `F[_]` or `F[_, _]` is pretty straightforward:

* If the effects of your type class cannot fail, or can fail with only a single type of error across all methods, then you should parameterize your type classes with `F[_]`. You can easily provide instances of this type for ZIO and other effect monads. No changes are required.
* If the effects of your type class can fail in different ways, then you should parameterize your type classes with `F[_, _]`, so you can precisely capture the different failure modes in the type class. You can easily provide instances of this type for ZIO. For other effect monads like `Task`, you can provide instances for `EitherT[Task, ?, ?]`.

It's easy to combine the two styles in one code base. For example, a data processing function that requires `F[_, _]: Analytics` can call a logging function that requires `F[_]: Logging` by simply requiring `L: Logging[F[IOException, ?]]`.

## Summary

In this post, we've seen that the effect type in ZIO allows us to precisely model if and how computations may fail. We've also seen that because of this power, the kind of ZIO's `IO` type differs from the kind of other effect types&mdash;it takes two type parameters, rather than one.

Because of this difference, many developers wonder how to use ZIO with tagless-final. Fortunately, using tagless-final with ZIO is trivial. As long as all methods produce the same error type (or no error at all), then there's no need to make any changes to the type classes.

In some cases, if we want our type classes to describe effects that can fail in different ways, we are able to improve on classic tagless-final by varying the error type. In this case, our type classes become parameterized over `F[_, _]`, allowing each method to specify its own error type. We can support other effect types by writing instances for `EitherT`.

In summary, ZIO is not only compatible with tagless-final, but also lets us take tagless-final to the next level of compile-time precision, while still preserving full compatibility with older dynamically-typed effect monads.

*If you like this blog post, don't miss John A. De Goes' [upcoming 5 day training](https://www.eventbrite.com/e/functional-scala-by-john-a-de-goes-tickets-48461417404) in Functional Scala, held remotely or on-site in the Scottish highlands.*
