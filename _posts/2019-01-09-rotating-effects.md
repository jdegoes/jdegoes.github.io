---
layout:       post
title:        "High-Performance Functional Programming Through Effect Rotation"
description:  "Vertical composition of effects, like monad transformers, don't perform very well. Rotate effects for higher performance."
category:     articles
tags:         [fp, functional programming, type classes, scala, monads, lenses, effects, reactive, scalaz, cats, mtl, monad transformers, zio]
---

As I covered in a [previous post](/articles/effects-without-transformers), monad transformers have poor performance properties in languages and runtimes unequipped to deal with them&mdash;including the Scala programming language and the JRE.

There's a general technique to improving performance that involves something I call _effect rotation_.

In this post, I'll talk about effect rotation and provide some examples in Scala and PureScript.

## Vertical Effect Composition

Monad transformers are an example of *vertical* effect composition. In vertical effect composition, we have layers of data stacked upon data, and layers of types stacked upon types:

{% highlight scala %}
val example : EitherT[OptionT[IO, ?], Error, Int] =
  EitherT.right(OptionT.some(IO.point(42)))
{% endhighlight %}

{% highlight haskell %}
example :: EitherT (OptionT IO) Error Number
example = pure >>> pure >>> pure $ 42
{% endhighlight %}

Vertical effect composition has some very desirable properties:

 * **Effect Introduction**. A new effect can be dynamically introduced. For example, if you have a base effect `f`, you can dynamically introduce state management by using `StateT s f`.
 * **Effect Elimination**. An existing effect can be dynamically eliminated. For example, if you have `StateT s f`, you can eliminate the outer effect to yield `f`.
 * **Information Hiding**. In vertical effect composition, you can ignore lower levels of a stack, which results in better composition and more code reuse.

Unfortunately, in too many languages and runtimes, every layer of the effect stack introduces additional overhead. In languages like Scala, the overhead is severe and renders the technique impractical for performance and memory sensitive applications.

To examine the cost of vertical effects, let's look at one the simplest of all monad transformers: `EitherT`.

## The Cost of `EitherT`

`EitherT` adds the effect of *recoverable failure* onto a base effect `f`.

We can define this transformer quite simply:

{% highlight scala %}
final case class EitherT[F[_ ], E, A](run: F[Either[E, A]])
{% endhighlight %}
{% highlight haskell %}
data EitherT e m a = EitherT (m (Either e a))
{% endhighlight %}

I won't prove this here, but when the base effect `f` is a monad, then `EitherT e f` is also a monad, for all `e`. Essentially, if you stack `EitherT` onto something that's a monad, you get _back_ something that is also a monad.

Notice the properties of this encoding:

1. **Effect Introduction**. We can introduce the effect of *recoverable errors* atop any monad `f` by lifting a `f` into `EitherT e f`.
2. **Effect Elimination**. We can eliminate the effect of *recoverable errors* by "running" the monad transformer, which gets us `f (Either e a)`.
3. **Information Hiding**. We can write polymorphic code that does not know what `f` is, but which utilizes the effect of *recoverable errors* atop the base monad.

These properties, however, come at *severe performance cost*.

### Binding Vertical Effects

It's easy to see from the structure of `EitherT` that it wraps the underlying `f (Either e a)`. Compared to `f a`, there are actually two levels of wrapping: the `Either e a`, which adds the error channel; and the `EitherT` structure itself.

However, languages like Haskell, PureScript, and Scala can (with some care, in the case of Scala) eliminate the cost of the `EitherT` structure by using &quot;newtypes&quot;. So we're left with one layer of _mandatory_ wrapping.

Monads and monad transformers allow you to build expressions using `bind` (AKA `flatMap`), whose signature is roughly:

{% highlight scala %}
def bind[A, B](fa: F[A])(f: A => F[B]): F[B]
{% endhighlight %}
{% highlight haskell %}
bind :: forall a b. f a -> (a -> f b) -> f b
{% endhighlight %}

Because `bind` is used so often for building monadic expressions, it dominates performance overhead.

An analysis of the type signature of `bind` suggests that using `EitherT` will necessarily add both wrapping and unwrapping of the `Either` data type, as well as additional function application.

You can see this clearly in the implementation of `bind` / `flatMap` for `EitherT`:

{% highlight scala %}
final case class EitherT[F[_ ], E, A](run: F[Either[E, A]]) {
  final def flatMap[B](f: A => EitherT[F, E, B])(
    implicit F: Monad[F]): EitherT[F, E, B] =
      EitherT(run.flatMap {
        case Left(e) => F.point(Left(e))
        case Right(a) => f(a).run
      })
}
{% endhighlight %}
{% highlight haskell %}
instance bindEitherT :: Monad m => Bind (EitherT e m) where
  bind (EitherT m) k =
    EitherT (m >>= either (pure <<< Left)
                          (\a -> case k a of EitherT b -> b))
{% endhighlight %}

Compared with the raw overhead for `bind` / `flatMap` on the base monad `f`, the implementation for `EitherT` adds the following:

 * Deconstruction of the `Either`.
 * Reconstruction of the `Either`.
 * Construction of at least one additional lambda (even more in the PureScript version).
 * Many additional function applications.

In an ideal world, this abstraction would be free. In the actual world, it has cost&mdash;for PureScript, and especially for Scala.

The JVM does not perform well with constant allocation of short-lived objects, nor with function application, which has to go through an additional layer of indirection (or several, if type classes are involved).

If you measure the performance of `EitherT` compared to the base monad, you'll find it's at least 2x slower, with more than 2x greater heap churn&mdash;and this is for one of the simplest monad transformers imaginable, and even when the feature of *recoverable errors* is not being used (the overhead of vertical effect composition applies _whether or not_ the effects are being used; if they're in the type signature, there's overhead!).

In the absolute best case, a stack of 3 monad transformers on top of some base monad `f` would be *at least* 3 times slower, with *at least* 3x greater heap churn, and depending on the transformers involved, could be far worse, perhaps 10x slower.

Even Haskell fares poorly with deeply nested monad transformers. Haskell's baseline performance for function application is _much_ higher, so it can get away with more layers before the performance impact becomes pathological.

The solution to this problem is to _stop vertically stacking effects_, and instead, to _rotate them horizontally_.

## Effect Rotation 101

Effect rotation is a technique whereby _vertical effects_ are "rotated" to become horizontal.

Let's take the `EitherT` example, and we'll generalize later.

Instead of representing our effect as `f (Either e a)`, we push the information content of the *recoverable errors* effect onto a _new type parameter_ of `f` itself. So our representation of the effect becomes `f e a`.

With `f e a`, our concrete data type `f` can directly provide all the features of `EitherT` (the ability to introduce and recover from typed errors), as well as whatever other features are required for the effect.

It may seem we've lost something, though: how do we represent `f` values that don't have errors? Thankfully, Scala and PureScript have uninhabited types (`Nothing` and `Void`, respectively), so we can still represent `f` values without errors, simply as `f Void a` (`F[Nothing, A]`).

This lets us (quite amazingly!) preserve all the properties that matter:

1. **Effect Introduction**. To introduce an error into `f Void a`, we can use a function from `f Void a` to `f e a'` (a "fail" function, for example), thereby introducing the possibility of error into the type.
2. **Effect Elimination**. To eliminate an error from `f e a`, we can use a function from `f e a` to `f Void a'` (an error handling function, for example), thereby removing the possibility of error from the type.
3. **Information Hiding**. We can make code polymorphic in one of the type parameters (like `e`) so it doesn't care about its presence or absence, and if we have suitable type classes, we can interact with `f` solely through these type classes, hiding the concrete data type from our code and providing maximum polymorphism (even more than monad transformers).

By pushing the effect of _recoverable errors_ into the data type, we enable a "flat" but "wide" data type to provide the feature of the effect with little or no overhead. For example, the `IO` monad in [ZIO](https://github.com/scalaz/scalaz-zio) provides recoverable errors without any performance overhead, and is almost 3x faster than using `EitherT` with other effect monads like Cats IO or Monix Task.

This technique works for more than just `EitherT`.

## Generalizing Effect Rotation

For covariant effects like `EitherT`, we can replicate the `EitherT` example, and add another type parameter, using `Nothing` (in Scala) or `Void` (in PureScript) to represent the absence of the covariant effect.

For contravariant effects like `Reader` (which represents the effect of accessing context), we can add another type parameter `r`, e.g. `f r e a` (`F[R, E, A]`), and use either `Any` (in Scala) or polymorphic `r` (in PureScript) to represent the absence of the reader effect (`forall r. f r e a` / `F[Any, E, A]`).

For invariant effects like `State` (which represents the effect of state management), we can add another type parameter `s`, e.g. `f s r e a` (`F[S, R, E, A]`), and use unit `Unit` to represent the absence of the state effect (`f Unit r e a` / `F[Unit, R, E, A]`).

Similarly, if we wanted `Writer` (which is invariant and represents the effect of persistent logging), we can add yet another type parameter `w`, e.g. `f w s r e a` (`F[W, S, R, E, A]`), again using unit `Unit` to represent the absence of the writer effect (`f Unit s r e a` / `F[Unit, S, R, E, A]`).

Thus all standard effects in functional programming, including recoverable errors, writer, reader, and state, can be rotated, while still preserving our ability to introduce them, eliminate them, and write (effect) polymorphic code.

Moreover, we can use them in any combination we like. For example, if we want reader and state, but not error or writer, then our type signatures become `f Unit s r Void a` / `F[Unit, S, R, Nothing, A]`. This is the type of effects that provide state management and context, but don't provide persistent logging or recoverable errors.

Provided we're willing to pay the cost of extra type parameters, we can have modular effects (introducing and eliminating them dynamically) without sacrificing on polymorphism, all without any performance or heap overhead beyond the base effect.

This is as close as we come to having our cake and eating it too.

## Summary

Monad transformers rely on vertical composition to achieve modularity of effects. While they provide us the ability to dynamically introduce and eliminate effects (such as *recoverable errors*), they introduce massive overhead that renders the technique impractical for most applications.

Fortunately, we can take all standard effects in functional programming, and rotate them from vertical to horizontal. The technique involves adding new type parameters, baking the features of the effect into the base data type, and based on whether the effect is covariant, contravariant, or invariant, using the appropriate type to indicate the absence of the effect.

The technique of effect rotation lets us have modular effects without runtime overhead, all for the cost of a few extra characters in our type signatures.

Now as [Michael Snoyman](https://twitter.com/snoyberg) and others have discovered, not all functional programming effects are equal. Even if you need writer, state, reader, and recoverable errors, that doesn't mean you need 5 type parameters, because some of these effects are powerful enough to express the others.

In a later post, I'll discuss a minimum basis to provide all standard effects in functional programming.
