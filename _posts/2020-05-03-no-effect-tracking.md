---
layout:       post
title:        "Effect Tracking Is Commercially Worthless"
description:  "Effect tracking is not a non-goal for commercial software development; the real reasons to use functional effect systems lie elsewhere"
category:     articles
tags:         [type classes, haskell, purescript, scala, cats, scalaz, mtl, tagless-final, functional programming, fp, effects, monads, io]
---

Effect tracking is not a valid reason to use functional effect systems, because effect tracking is commercially worthless.

More precisely, companies that pay their software developers to "track effects" will not obtain a return on this investment, but rather, will lose money.

In this post, I'll explain why.

## Side Effects

The whole notion of "effects" doesn't make a lot of sense outside _functional programming_.

In functional programming, we try to build our software from deterministic, pure functions. These functions are like the ones most of us learned about in high school:

<pre>
f(x) = x * x
</pre>

Such functions are said to be "free of side-effects", meaning their only effect is combining and transforming inputs to produce an output. 

Pure functions don't do anything _on the side_&mdash;they don't do anything that could be observed from outside the function, like operating system calls or mutating the heap.

## Discovering Purity

When developers first discover functional programming, they believe that it's *useless* for real world applications.

I had the same response when I first heard about programming without procedural statements, assignments, or loops.

After all, procedures that compute random numbers, call databases, and or invoke web APIs, all perform "side-effects". 

These procedures are not deterministic, pure functions that just combine and transform inputs to produce outputs. Yet they are the _cornerstone_ of building applications that solve real business problems.

Eventually, of course, I discovered that in purely functional languages like Haskell, functions that interact with the outside world all return values of a mysterious `IO` data type.

This discovery feeds a myth that's pervasive among even experienced Haskell programmers.

The myth of _effect tracking_.

## Effect Tracking

It's extraordinarily common for developers who have encountered Haskell's `IO` data type to explain it as follows:

 * Impure functions return `IO`
 * Pure functions don't return `IO` 

According to this explanation, one can determine whether or not a function is effectful merely by examining the return type. The `IO` type therefore "tracks" the presence of side-effects... hence, _effect tracking_.

Consequently, many developers now believe that Haskell uses the `IO` type to track effects, to mark them as being "impure"&mdash;that Haskell's type system is being recruited to help us ascertain purity.

There's just one _tiny_ little problem with this explanation.

It's totally and completely wrong!

## Myth Busting

In point of fact, a Haskell function that returns `IO` **is** pure: it's deterministic and entirely free of side-effects, merely transforming and combining inputs to produce an output.

For example, take the `putStrLn` function:

{% highlight haskell %}
putStrLn :: String -> IO ()
{% endhighlight %}

{% highlight scala %}
def putStrLn(line: String): IO[IOException, Unit]
{% endhighlight %}

This function takes a string, and returns an `IO` value. It does not actually _perform_ any effects, and if given the same string, it will return the 'same' `IO` value. Moreover, the value that it returns is _totally immutable_, like all values in Haskell.

These guarantees hold not just for `putStrLn`, but for all functions that return `IO` values (unless they cheat by using `unsafePerformIO` or similar).

Stated more forcefully, functions that return `IO` values are _no different_ than functions that do **not** return `IO` values: they are pure, and their type signature does not reveal the presence or absence of side-effects, because there are no side-effects at all.

Now, eventually one learns that `IO` is a data type, whose immutable values _model_ some sequence of interactions with the outside world (mutable heap, sockets, files, databases, and the like).

So, even though a function like `putStrLn` does not _perform_ any side-effects, the value it returns _describes_ a side-effect.

However, `IO` is not magical in this regard. Indeed, entirely without `IO`, we can easily construct a simple model of interaction with the outside world, like sockets:

{% highlight haskell %}
data SocketIO a = 
  ReadByte (Byte -> SocketIO a) |
  WriteByte Byte (SocketIO a)
{% endhighlight %}
{% highlight scala %}
sealed trait SocketIO[+A]
final case class ReadByte[A](withByteDo: A => SocketIO[A]) extends SocketIO[A]
final case class WriteByte[A](byte: Byte, andThenDo: SocketIO[A]) extends SocketIO[A]
{% endhighlight %}

Now one can write functions that return `SocketIO` values.

`IO` values are no less pure than `SocketIO` values&mdash;literally the only distinction between them is that, in Haskell, your main function may return an `IO` value, and if it does, the data structure will be translated by the Haskell runtime into the side-effects that it models. 

But we could easily do that ourselves for `SocketIO`, by having an interpreter that is built on `unsafePerformIO`!

## Effect Tracking?

So effect tracking&mdash;the ability to "track effects" in the type system&mdash;is a gigantic misnomer, because the `IO` type does not track side-effects. 

There are no side-effects in Haskell programs (`unsafePerformIO` and friends notwithstanding).

There are only data types, some of which model interaction with the outside world. However, even a list of bytes can model interaction with the outside world. In fact, a binary program is nothing more than a list of bytes, which models interaction with the outside world using machine code instruction sets!

That said, if we ignore the technical sense in which there is no such thing as "effect tracking" in Haskell, can we say that having "tracked effects" is a good reason to use data types like `IO`?

The answer is no, because effect tracking can be done without `IO`!

## Effect-Tracked Java™

If, by _effect tracking_, we mean that we can know, looking only at the signature of a function, whether or not the method models or performs side-effects, then allow me to introduce to you _Effect-Tracked Java™_!

In Effect-Tracked Java™ (a feasible thought experiment for now), every method is annotated with either `@Pure` or `@Impure`. For example:

{% highlight java %}
@Impure
public Config loadAppConfig() {
  ...
}

@Pure
public Config mergeConfigs(defaults: Config, priority: Config) {
  ...
}
{% endhighlight %}

Let's assume that all the "root" methods in the Java standard library, together with JNI methods, have been ascribed these annotations, which are stored in a data file accessible to our build.

This information allows a "purity" annotation processor, which we would run as part of our build process, to verify that all our annotations are correct.

For example, if we call `System.currentTimeMillis()` inside a function annotated `@Pure`,  we will get an error, and we will have to fix the error by changing the annotation to `@Impure`.

Presto, instant effect tracking, without esoteric "monads" or any of that pesky functional programming!

Further, we don't have to stop at such course-grain effect tracking. If we like, we can allow `@Impure` annotations to introduce labels, which we could mistakenly call "algebras". 

This would give us (let's call it) _Tagless-Final Effect-Tracked Java™_, which would look something like this:

{% highlight java %}
@Impure({"FileSystemAlgebra", "EnvironmentVariablesAlgebra"})
public Config loadAppConfig() {
  ...
}
{% endhighlight %}

As before, our annotation processor would enforce correctness. So if we accidentally called `System.currentTimeMillis()` inside a function annotated `@Impure()`, we would be forced to change this annotation to `@Impure({"ClockAlgebra"})`.

With these "improvements", we could tell not only which methods are pure and impure, but for the impure methods, we could tell which "algebras" they use&mdash;and all of this is tracked statically!

Have we just discovered a way to radically boost developer productivity for all Java developers everywhere?!?

Read on for my unexpected and totally surprising answer!

## Worthless Effect Tracking

Effect tracking, whether at the course or fine-grained level, is simply not commercially valuable. If it were, not only would there exist popular annotation processors for it, but the Java language itself would probably have the feature.

Instead, while the feature can be found as one of many features in the [Checker Framework](https://checkerframework.org/manual/#purity-checker), it has no commercial traction.

Moreover, early versions of PureScript used row types to provide statically-checked and fine-grained "effect tracking", but the feature was [quickly abandoned](https://github.com/purescript-deprecated/purescript-eff), as it provided no commercial value, but rather slowed down feature development without benefit.

Contrast these results with lambdas (anonymous functions), which have made their way into every programming language because of the unquestionably positive effects they have on productivity.

It's my contention that effect tracking is worthless precisely because if a developer has _any_ idea about what a function is intended to do, then they already know with a high degree of certainty whether or not the function performs side-effects.

No developer is surprised that `System.currentTimeMillis()` performs a side-effect, because they already know what the function is intended to do. Indeed, many language features and best practices (as well as IDE features!) are designed _precisely_ to give developers a better idea of what functions are supposed to do.

Given a rudimentary understanding of what a function's purpose, the purity of the function can be predicted with high probability (violations, like `java.net.URL#hashCode`, become legendary!). However, given the purity of a function, this information by itself does not convey any useful information on what the function's purpose.

Stated simply, effect tracking isn't useful in practice, because when it matters (and it doesn't always matter), we already know roughly what functions do, and therefore, whether or not they perform side-effects.

Whatever marginal benefit effect tracking would add is overwhelmed by the cost of ceremony and boilerplate.

This is doubly-true for the "fine-grained" variants.

## Why IO?

If effect tracking is commercially useless, then why use `IO` data types? 

If you're in Haskell, you don't have a choice: if you want to get useful work done, then you will use _some_ model of interaction with the outside world, and it may as well be `IO`, which is industry-proven.

If you're in Scala, however, you _do_ have a choice: you can write plain vanilla Scala code, and perform side-effects anywhere that you want. Or you can grab one of the functional effect systems like [ZIO](https://zio.dev), and create and compose values that _model_ side-effects instead.

The greatest reason to use a functional effect system like ZIO is that it makes side-effects first-class values. Values are things you can accept and return from functions. You can store them in data structures. You can write your own operators, which accept functional effects, and which transform or combine them in custom ways.

All of these abilities let you make your own control flow structures and factor out kinds of duplication you can't avoid when you solve problems using side-effecting code (take a look at some of my talks for examples).

ZIO in particular goes beyond providing first-class effects, and delivers additional valuable features, including:

* Highly-scalable fiber-based runtime
* Resource-safety across asynchronous and concurrent effects
* Declarative concurrency without locks and condition variables
* Easy and safe parallelism
* Context propagation and dependency injection
* Execution traces that work across async and concurrent boundaries
* Fiber-dumps that show the runtime graph of your application
* Powerful operators that allow you to snap together solutions to complex problems quickly
* Type-safety and user-friendliness
* Features for making user-land code fully testable

The real reason for using ZIO or other functional effect **isn't** effect tracking: it's everything else.

## Summary

Effect tracking isn't a good reason to use an `IO` like data type, in Scala or any other programming language. That's because effect tracking (which is actually a misnomer!) isn't commercially useful.

If we have a vague idea about what functions do, then we generally have a really good idea about whether they perform side-effects, and compiler-enforced effect tracking would add overhead that wouldn't pay for itself.

Instead, functional effect systems like ZIO (and Haskell's `IO` data type) let us take side-effects and make them more useful, by turning them into values, which we can transform and compose, solving complex problems with easy and type-safe combinators that simply can't exist for side-effecting statements.

Beyond this ability, ZIO in particular gives us new superpowers, like easy and safe concurrency, parallelism, resource handling, dependency injection, diagnostic and debugging information, testability, and type-safety.

In summary, don't use ZIO or IO or any functional effect system for effect tracking. 

Instead, if you decide to use functional effect systems, then use them to become a more powerful and productive programmer.