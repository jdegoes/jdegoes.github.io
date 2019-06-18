---
layout:       post
title:        "The False Hope of Tagless-Final in Scala"
description:  "For many but not all, tagless-final in Scala is not what it's cracked up to be"
category:     articles
tags:         [type classes, haskell, purescript, scala, cats, scalaz, mtl, tagless-final, functional programming, fp]
---

Tagless-final is a technique used to [embed domain-specific languages](http://okmij.org/ftp/tagless-final/index.html) into a host language, without the use of Generalized Algebraic Data Types.

In the Haskell community, _tagless-final_ refers to a way of creating polymorphic programs that are interpreted by instantiaing them to a target data type. In the Scala community, usage of the term is closest to what Haskeller's mean by _MTL-style_, albeit, without the algebraic laws that govern the type classes in the Monad Transformers Library.

In Haskell, tagless-final is used with types of various kinds, but in Scala, the terminology has become synonymous with types of kind `* -> *`&mdash;leading to the infamous `F[_]` higher-kinded type parameter that is so pervasively associated with the phrase _tagless-final_.

In this post, I will argue that tagless-final, as practiced by the Scala community, and as embodied by the `F[_]` higher-kinded type parameter, has drawbacks so substantial, they overwhelm the benefits of the approach for the vast majority of companies doing Scala development.

I'll wrap up by providing a concrete list of recommendations for you and your team.

## Tagless-Final 101

In Scala, tagless-final involves creating an interface, or at least type class, which describes the capabilities of a generic effect `F[_]`. 

For example, we could create the following interface to describe the capabilities of a `Console`:

{% highlight scala %}
trait Console[F[_]] {
  def putStrLn(line: String): F[Unit] 
  val getStrLn: F[String]
}
{% endhighlight %}

Or, we could create the following interface to describe the persistence capabilities around `User` objects in our domain:

{% highlight scala %}
trait UserRepository[F[_]] {
  def getUserById(id: UserID): F[User]

  def getUserProfile(user: User): F[UserProfile] 

  def updateUserProfile(user: User, profile: UserProfile): F[Unit]
}
{% endhighlight %}

These interfaces, or type classes, then allow us to create functions that are _polymorphic_ in the effect type `F[_]`. For example, we could describe a program that uses the `Console` interface as follows:

{% highlight scala %}
def consoleProgram[F[_]: Console]: F[Unit] = 
  implicitly[Console[F]].putStrLn("Hello World!")
{% endhighlight %}

Combined with type classes like `Monad`, we can build entire programs using the tagless-final approach:

{% highlight scala %}
def consoleProgram[F[_]: Console: Monad]: F[String] = {
  val console = implicitly[Console[F]]

  import console._

  for {
    _     <- putStrLn("What is your name?")
    name  <- getStrLn
    _     <- putStrLn(s"Hello, $name, good to meet you!")
  } yield name
}
{% endhighlight %}

Because such programs are polymorphic in the effect type `F[_]`, this means we can _instantiate_ these programs to any concrete effect type that provides their set of required capabilities.

For example, if we are using ZIO `Task` (a type alias for `ZIO[Any, Throwable, A]`), we can instantiate our program to this concrete effect type with the following code snippet:

{% highlight scala %}
val taskProgram: Task[String] = consoleProgram[Task]
{% endhighlight %}

Typically, the _instantiation_ of a polymorphic tagless-final value to a concrete effect type is deferred as long as possible, ideally to the entry points of the application or test suite.

With this introduction, let's talk about the reasons you might want to use tagless-final...the _pitch_ for the tagless-final technique, if you will.

## The Pitch for Tagless-Final

Tagless-final has a seductive pitch that appeals to every aspiring functional programmer.

In the functional programming community, we are taught (with good reason) that monomorphic code can't be reused, and leads to more bugs. We are taught that generic code not only enables reuse, but it pushes more information into the types, where the compiler can help us verify and maintain correctness.

We are taught the _principle of least power_, which tells us that our functions should require as little as necessary to do their job.

I have helped develop, motivate, and teach these principles and others during my _Functional Scala_ workshops around the world, helping train new generations of Scala developers in the functional way of thinking and developing software.

In this context, functional programmers are _primed_ for the tagless-final pitch; I know this, because I have _given_ the tagless-final pitch, and even helped _craft_ its modern day incarnation. 

In one video, I unintentionally convinced [several companies to adopt tagless-final](https://www.youtube.com/watch?v=sxudIMiOo68), despite an explicit disclaimer stating the techniques would be overkill for many applications!

In the next few sections, I'm going to give you this pitch, and try to convince you that tagless-final is the best thing ever. Moreover, I'm going to use arguments that all have an element of truth to them!

Ready? Here we go!

### 1. Effect Type Indirection

As of this writing, there are [several mainstream effect types](/posts/zio-cats-effect), including ZIO, Monix, and Cats IO, all of which ship with [Cats Effect](https://github.com/typelevel/cats-effect) instances, and which can be used interchangeably in libraries like FS2, Doobie, and http4s.

Tagless-final lets you insulate your code from the decision of _which_ effect type to use. Rather than pick one of these concrete implementations, using tagless-final lets you write _effect type-agnostic_ code, which can be _instantiated_ to any concrete effect type that provides Cats Effect instances.

For example, our preceding console program can just as easily be instantiated to Cats IO:

{% highlight scala %}
val ioProgram = consoleProgram[cats.effect.IO]
{% endhighlight %}

With tagless-final, you can defer the decision of which effect type to use _indefinitely_ (or at least, to the edge of your program), isolating your application from changes in the evolving ecosystem.

### 2. Effect Testability

Tagless-final, because it provides a strong layer of indirection between your application, and the concrete effect type that models effects, allows you to make your code testable.

In the preceding console implementation, it would be easy to define a test implementation of the `Console` interface:

{% highlight scala %}
final case class ConsoleData(input: List[String], output: List[String])

final case class ConsoleTest[A](run: ConsoleData => (ConsoleData, A)) {
  def putStrLn(line: String): ConsoleTest[Unit] = 
    ConsoleTest(data => (data.copy(output = line :: data.output), ()))

  val getStrLn: ConsoleTest[String] =
    ConsoleTest(data => (data.copy(input = data.input.drop(1)), data.input.head))
}
{% endhighlight %}

Now, assuming we define an appropriate `Monad` instance for our data type (which is easy to do!), we can instantiate our polymorphic `consoleProgram` to the new type:

{% highlight scala %}
val testProgram = consoleProgram[ConsoleTest]
{% endhighlight %}

Tada! We can now unit test our console program with fast, deterministic unit tests, thereby reaping the full testability benefits of pure functional programming.

### 3. Effect Parametric Reasoning

_Parametric reasoning_ in statically-typed functional programming circles refers to the ability for us to reason about the implementation of a polymorphic function merely by looking at its type.

For example, there is one possible implementation of the following function:

{% highlight scala %}
def identity[A](a: A): A = ???
{% endhighlight %}

(Assuming no reflection, exceptions, or use of `null`.)

Parametric reasoning allows us to save time when we are understanding code. We can look at the types of a function, and even if we don't know the exact implementation, we can place _bounds_ on what the function can do. 

Parametric reasoning lets us more quickly and more reliably understand code bases, which is critical for safe maintenance of those code bases in response to added and changing business requirements.

Moreover, parametric reasoning can reduce the need for unit testing: whatever is guaranteed by the type, does not need to be tested by unit tests. Types prove universal properties across all values, so they are strictly more powerful than tests, which prove existential properties across a few values.

Since tagless-final is an example of (higher-kinded) parametric polymorphism, we can leverrage parametric polymorphism for effect types too.

For example, take the following code snippet:

{% highlight scala %}
def consoleProgram[F[_]: Console: Applicative]: F[String] = ???
{% endhighlight %}

 _Note there is an alternate, and, I'd argue, superior encoding of tagless-final that passes `Console` explicitly as an argument, rather than using a type class constraint; but this alternate encoding doesn't substantially change my argument, so I won't discuss it here._

Although we don't know what the implementation of this function is without looking, we know that it requires `F[_]` to support both `Console` and `Applicative`.

Because `F[_]` is only `Applicative` and not `Monad`, we know that while `consoleProgram` can have a sequential chain of console effects, no subsequent effect can depend on the runtime value of a predecessor effect (that would require `bind` from `Monad`).

We would not be surprised to learn the implementation of the function resembles this one:

{% highlight scala %}
def consoleProgram[F[_]: Console: Applicative]: F[String] = {
  val console = implicitly[Console[F]]

  import console._

  putStrLn("What is your name?") *> getStrLn
}
{% endhighlight %}

By constraining the capabilities of the data type `F[_]`, tagless-final lets us reason about our effectful programs parametrically, which lets us spend less time studying code, and less time writing unit tests.

### The Closer

Clearly, tagless-final is an incredible asset to functional Scala programmers everwhere!

Not only does it enable abstraction over effects, so programmers can change their mind about which effect type to use (without changing any code!), but the technique enables testability of effectful programs, and helps us reason about our effectful programs in new ways, which can reduce the need for unit tests.

## The Fine Print

Sold yet on tagless-final? I am!!! Well, _somewhat_, anyway. 

There's an element of truth in _every single argument_ that I've made. Although as you might suspect, a more nuanced, sophisticated look reveals some cracks in the foundation of tagless-final.

Let's take a look at all the fine print in the next few sections.

### 1. Premature Indirection

It is absolutely true that adding a layer of indirection over an effect type can reduce the cost of switching to a different effect type, assuming similar underlying models.

It is also true that adding a layer of indirection over Spark, Akka HTTP, Slick, or a database-specific dialect of SQL, might reduce the cost of switching to different technologies.

In my experience, however, the attempt to proactively add layers of indirection without a clear business mandate to do so, simply to mitigate the _possible_ cost of future change, is an example of _premature indirection_. Premature indirection rarely pays for itself.

In most cases, overall application costs would be _substantially lower_ dropping these layers of indirection, and then, if needs change, simply refactoring the code to a new technology.

The cost of refactoring from one effect type to another is not related to the cost of changing types from `IO[_]` to `Task[_]`, or swapping one set of methods for another. Rather, it is related to the _semantic_ differences between operations on these effect types. Yet a layer of indirection helps us only when semantic differences are relatively small. If they are small, then the cost of refactoring is relatively low.

A refactoring from one effect type to another only happens needs to happen once, and only if is actually necessary (which it might not be); but the cost of coding to a layer of indirection has to be maintained indefinitely, and regardless of whether or not that layer of indirection will ever be utilized.

Beyond the cost of coding to an indirection layer that may never be used, there are substantial _opportunity costs_ to premature indirection. In the case of ZIO, for example, the core effect type has hundreds of additional operations that are not available on a polymorphic `F[_]`. 

These methods, like additional methods on Monix Task, are discoverable by IDEs; their types guide users to correct solutions; error messages are concrete and actionable; and type-inference is nearly flawless.

If you _commit to not committing_, you're stuck with the weakest `F[_]`, which means that much of the power of an effect system remains inaccessible. The frustration of knowing a method is right there, but just out of reach, has prompted many to introduce custom type classes designed to access specific features of the underlying effect type.

The situation is even more exaggerated for ZIO than Monix, because ZIO features polymorphic reader and error effects, which allow parametric reasoning about the dependencies and error behavior of effects, and which are currently unsupported in Cats Effect.

In my opinion, only library authors have a compelling argument for effect type indirection. In order to maximize market share (which is critical for the adoption, retention, and growth of open source libraries), they need to support all major effect types, or risk losing market share to the libraries that do. 

While this is a compelling argument for libraries that care about market share, it is completely inapplicable to closed source applications that make up the majority of Scala software development. 

### 2. Untestable Effects

Tagless-final provides a _path_ to testability, but tagless-final programs are not _inherently_ testable. In fact, they are testable _only to the degree their tagless-final interfaces are testable_.

For example, while we can create a testable version of the `Console` interface, many other interfaces, including popular interfaces in the tagless-final community, are inherently _untestable_.

The majority of applications written in the tagless-final style make heavy use of a type class in Cats Effect called `Sync`, or its more powerful versions, including `Async`, `LiftIO`, `Concurrent`, `Effect` and `ConcurrentEffect`.

These type classes are designed to capture side-effects&mdash;for example, random number generation, API calls, and database queries. Any code that utilizes these type classes, or others like them, cannot be unit tested even in theory, because it interacts with partial, non-deterministic, and side-effecting code.

While such code can be tested with integration and system tests, _any code at all_ can be tested in integration and system tests, including the worst possible procedural code.

Ultimately, the testability of tagless-final programs requires they follow the principle of _code to an interface, not an implementation_. Yet, if code follows this principle, it can be tested even _without_ tagless-final!

Here is a monomorphic version of the `Console` interface, for example:

{% highlight scala %}
trait Console {
  def putStrLn(line: String): Task[Unit] 
  val getStrLn: Task[String]
}
{% endhighlight %}

If your program uses this interface to perform console input/output, then it can be tested, even with a monomorphic effect type. The key insight is that your program must be _written to an interface_, rather than a concrete implementation. Unfortunately, numerous widely-used tagless-final interfaces (like `Sync`, `Async`, `LiftIO`, `Concurrent`, `Effect`, and `ConcurrentEffect`) encourage you to code to an _implementation_.

Using tagless-final doesn't guarantee you will be coding to an interface, nor does tagless-final provide any inherent testing benefits. The testability of your code is completely orthogonal to its use of tagless-final, and comes down to whether or not you follow best practices&mdash;which you can do with or without tagless-final.

### 3. No Effect Polymorphic Reasoning

The most insidious claim that tagless-final makes is that it provides _effect polymorphic reasoning_.

According to this claim&mdash;which is not entirely without merit, as we will see&mdash;we can look at a polymorphic function signature, and know the effects it performs merely by examining constraints.

Previously, we looked at the following example:

{% highlight scala %}
def consoleProgram[F[_]: Console: Applicative]: F[String] = ???
{% endhighlight %}

I argued we could know something about what this function does by noting its use of `Applicative` and `Console`. In theory, these constraints provides us the ability to partially understand its behavior without examining the implementation.

Unfortunately, this argument rests on a premise that is _false_. Namely, it rests on the premise that implicit parameters somehow _constrain_ the side-effects executed or modeled by the function.

Scala does not track side-effects, however. We can perform console effects anywhere, even without the provided `Console[F]`, as shown in the following snippet:

{% highlight scala %}
def consoleProgram[F[_]: Applicative]: F[String] = {
  println("What is your name?")
  val name = scala.io.StdIn.readLine()
  Applicative[F].pure(name)
}
{% endhighlight %}

Not only can we perform any effects anywhere inside the body of the method without even so much as a compiler warning, but we can embed these effects into any `Applicative` functor, even one with a strict definition of `pure` / `point`:

{% highlight scala %}
def consoleProgram[F[_]: Applicative]: F[String] =
  Applicative[F].pure(()).map { _ =>
    val _ = println("What is your name?")
    scala.io.StdIn.readLine()
  }
{% endhighlight %}

This is legal, type-safe Scala code, and varaints of it appear pervasively in real world Scala (both with `Future` and with functional effect systems). Not even the most aggressive IntelliJ IDEA linter settings will report issues with this code snippet, except perhaps a warning about the use of higher-kinded types.

In tagless-final, "constraints"&mdash;in the form of implicit or explicit parameter lists like `Applicative[F]` or `Console[F]`&mdash;do not actually constrain, they _unconstrain_ (giving us more ways of building values). And in Scala, because effects are already _totally unconstrained_, adding more values to parameter lists doesn't change anything.

Stated more forcefully for Scala code, **effect parametric reasoning is a lie**.

Now, as highly-skilled and highly-disciplined functional programmers, we can agree amongst ourselves to reject merging such code into our projects. This requires that we inspect every line of code to ensure that side-effects are captured only in appropriate places, where _appropriate_ is defined by a social contract between us and our fellow colleagues.

A social contract, powered by good-will, skill, discipline, and indoctrination of new hires, can be quite useful. Nearly _all_ best practices are social contracts, and they undoubtedly help us write better code. But social contracts should not be confused with compile-time constraints. 

If we _assume_ a working social contract around effects in Scala, then we can assume restrictions on the effects in the preceding definition of `consoleProgram`, but these restrictions do not come from effect polymorphism (which does not exist in a language without effect tracking!), but from the social contract.

If we are going to rely on a social contract to give us reasoning benefits, then we should consider other social contracts, such as _always code to an interface, not an implementation_. 

Other social contracts can give us the exact same "reasoning benefits", but with possibly better ergonomics. Stated differently, "reasoning benefits" is not a reason to prefer tagless-final over any other approach, because those benefits come from a social contract, and lots of other contracts can give us the same benefits.

These are the hard limits on _effect parametric polymorphism_ in Scala, which&mdash;short of inventing a compiler plug-in that overlays a new, effect-tracked language onto Scala&mdash;cannot be circumvented.

### 4. Sync Bloat

Even if we ignore the fact that effect parametric polymorphism doesn't exist in Scala, and assume that a method which requires only `Applicative[F]` and `Console[F]` is _constrained_ to the operations provided by these parameters, we have a problem that I term _sync bloat_.

In Scala, the Cats Effect type class hierarchy provides numerous type classes that are _designed_ to capture side-effecting code. These type classes include `Sync`, `Async`, `LiftIO`, `Concurrent`, `Effect`, and `ConcurrentEffect`. 

A method requiring one of these type classes can literally do _anything_ it wants, without constraints, even under the false assumption that Scala has effect parametric polymorphism. These methods deny us any potential benefits to reasoning and testability, resulting in opaque blobs of side-effecting, untestable procedural code.

Nearly all tagless-final code (including some of the [best OSS functional Scala I know of](http://github.com/slamdata/quasar/search?q=Sync&unscoped_q=Sync)) makes liberal use of these type classes, freely embedding side-effects in numerous methods sprawled across the code base.

In a perfect world, perhaps programmers would create semantic type classes to represent separate concerns, and methods would require only semantic type classes to do their job. But in the real world, the vast majority of programmers using tagless-final (even high-skilled, expert-level functional programmers!) are not doing this. Instead, they're requiring type classes like `Sync` that encourage embedding arbitrary side-effects everywhere.

The pervasive phenomenon of _sync bloat_ means that even if we grant something that isn't true (that Scala has effect parametric polymorphism), we _still_ aren't gaining any benefits of effect parametricity. 

### 5. Fake Abstraction

As I teach in _Functional Scala_, abstracting is the act of discovering commonalities in the structure of different data types. By _commonality_, I mean a set of _common operations_ that satisfy _alebraic laws_.

Abstraction is the means by which we can write _principled_ polymorphic code.

Without lawful operations, there is no abstraction, only layers of indirection masquerading as abstraction.

As practiced in Scala, tagless-final encourages a form of _fake abstraction_. To see this, let's take a closer look at the `Console[F]` trait defined previously:

{% highlight scala %}
trait Console[F[_]] {
  def putStrLn(line: String): F[Unit] 
  val getStrLn: F[String]
}
{% endhighlight %}

This trait has two operations, called `putStrLn` and `getStrLn`. We can provide implementations of these two different operations for many different data types.

Unfortunately, these operations satisfy no algebraic laws&mdash;none whatsoever! This means when we are writing polymorphic code, we have no way to reason generically about `putStrLn` and `getStrLn`.

For all we know, these operations could be launching threads, creating or deleting files, running a large number of individual side-effects in sequence, and so on.

This contrasts quite dramatically with type classes like `Ord`, which we can use to build a generic sorting algorithm that will _always_ be correct if the algebraic laws of `Ord` are satisfied for the element type.

The implications of this are profound. Consider the following type signature:

{% highlight scala %}
def consoleProgram[F[_]: Console: Applicative]: F[String] = ???
{% endhighlight %}

Because `Console[F]` is lawless, we cannot reason generically about the side-effects modeled by the returned `F[Unit]`. These side-effects are unconstrained precisely because there are no algebraic laws that govern the operations provided to us by `Console[F]`.

Two different implementations of `Console[F]` could do totally different things, without breaking any laws, so we cannot actually reason _generically_ about the correctness of our code. Our code may be correct for some `Console[F]`, and incorrect for other `Console[F]`.

Keep in mind that `Console[F]` is just a toy example. A realistic (semantic) tagless-final trait would be far larger and more complex, containing numerous operations, none of which can be reasoned about generically. Any code written to such an interface will be extremely coupled to implementation details that are unspecified and unspecifiable.

_Real_ generic reasoning applies only to _real_ abstractions, like `Monoid`, `Monad`, and so forth. The moment we introduce ad hoc, unspecified, implementation-specific operations like `putStrLn` or `getStrLn`, we can no longer reason generically about the behavior of our code in any principled, useful way.

What this means is that even if we grant that Scala has effect parametric polymorphism (which it doesn't!), and even if we assume that developers won't use type classes like `Sync` (which they will!), the ad hoc nature of semantic tagless-final type classes mean we don't actually have _any useful generic reasoning_.

We can tell that `bind` is not invoked if only `Applicative[F]` is provided, but we cannot tell how many individual side-effects are chained together to make up a single operation like `putStrLn`. The limited reasoning we can do is _useless_ (a parlor trick, at best), precisely because of all the reasoning we _cannot_ do.

Does `putStrLn` print a line of text to a console? Or does it launch a concurrent `main` function with the whole application? Who knows. The types and laws don't tell you anything. 

To answer a question like this, you need to examine a specific concrete implementation of `Console[F]`. Which means adding `Console[F]` to a type signatue is at best a form of wishful documentation&mdash;a hope that whoever gives us a `Console[F]` will abide by an unspecified contract that has something to do with console input/output.

Polymorphic code that we can reason about generically is an awesome benefit of statically-typed functional programming. But the benefit requires abstractions, which must always have algebraic laws. The moment we veer into ad hoc territory (providing operations without laws), we aren't doing principled programming anymore.

## The Tagless-Final Hit List

Tagless-final in Scala is a false hope, for all the reasons we've seen:

1. **Premature Indirection**. For applications, using tagless-final to guard against the possibility of changing effect types is usually an example of _overengineering_ (premature indirection). Your application doesn't add indirection layers for Akka Streams, Slick, Future, or even the dialect of SQL you're using&mdash;because in most cases, the cost of building and maintaining that layer of indirection is _unending_. In the case of tagless-final, you also deprive yourself of additional principled operations and type-safety provided by concrete data types.
2. **Untestable Effects**. Many popular tagless-final type classes encourage capturing side-effects. These type classes destroy the ability to reason about and unit test applications built using them. In the end, testability is not a property of tagless-final code; testability requires you code to an interface, not an implementation, which you can do with or without tagless-final.
3. **No Effect Parametric Polymorphism**. Scala doesn't constrain side-effects anywhere, which means that adding more parameters to a method cannot _lessen_ constraints. If you want to _constrain_ side-effects, you need a _social contract_. In this case, reasoning benefits come from the contract, not the compiler. Yet if you are open to social contracts, requiring programmers always code to an interface provides the same reasoning benefits.
4. **Sync Bloat**. In Scala, real world tagless-final code tends not to use semantic type classes, but rather, type classes that allow the unrestricted capture of side-effects. These cargo-culted `Sync` code bases do not confer any benefits to reasoning or testability, even under the assumption of working social contracts. They combine the ceremony and boilerplate of tagless-final with the untestable, unreasonable nature of untestable procedural code.
5. **Fake Abstraction**. Reasoning about generic code requires abstractions, which come equipped with algebraic laws, which define commonality between different data types. Yet semantic tagless-final classes do not have any laws. Their operations have names and types, but it is not possible to generically reason about these operations. Any code that uses "fake abstractions" is not actually generic, but instead, is closely wedded to unspecified implementation details.

These reasons just scratch the surface. 

Tagless-final has enormous pedagogical and institutional costs, because of the huge number of concepts it requires a team to master (parametric polymorphism, higher-kinded types, higher-kinded parametric polymorphism, type classes, higher-kinded type classes, the functor hierarchy, etc.), and because of the level of ceremony and boilerplate involved (type classes, type class instances, instance summoners, syntax extensions, higher-kinded implicit parameter lists, non-inferrable types, etc.).

I've [talked about these other drawbacks before](/posts/zio-environment), so I won't repeat them here.

## Objections

Some functional programmers, when presented with these drawbacks, immediately push back with the objection that _other approaches_ (including the [reader monad](/posts/zio-envioronment)) can't constrain side-effects in Scala, either.

This is true, but also entirely beside the point.

There are _zero_ approaches to constraining effects in Scala, because Scala cannot constrain effects. This means that when comparing two different approaches to modeling effects in Scala, there is no dimension for "constraining effects". 

You can combine tagless-final with manual inspection of every line of code in an application, to ensure it satisfies the social contract that side-effects will only be executed and captured in "approved" places. This combination provides both testability and reasoning benefits.

Similarly, you can combine the reader monad with manual inspection of every line of code in an application, to ensure it satisfies a social contract that logic will be written to interfaces, not implementations. As with tagless final, this combination provides both testability and reasoning benefits.

Both tagless-final and the reader monad (and many other approaches) can indeed provide "guarantees" about where effects happen, but only when combined with working social contracts. The strength of these "guarantees" derive from the strength of the social contracts, not from the Scala compiler.

Another vague objection I have heard is that tagless-final is somehow more principled than other approaches, or that it somehow lends itself better to designing composable functional APIs. 

In reality, principled, composable, functional code has everything to do with whether your domain's operations satisfy algebraic laws. As practiced in Scala, tagless-final has absolutely no relation to principled programming. 

The use of higher-kinded parametric polymorphism may provide an illusion of "rigor", but it's just a thin veneer on unprincipled imperative code. All the effect type polymorphism in the world can't change this.

## Recommendations

In light of this analysis of tagless-final, I have some concrete recommendations:

1. **Stay the Course**. If you're in a small, stable, high-skilled team that's already using and benefiting from tagless-final due to a working social contract, then _keep using tagless-final_. Long-time functional programmers may "forget" how to program procedurally and instinctively confine themselves to a functional subset of Scala, which makes enforcement of the contract easier. Consider abandoning tagless-final if the team starts scaling.
2. **Build a Library**. If you're building an open source library that doesn't take advantage of effect-specific features, then use an existing layer of indirection (e.g. Cats Effect). Or if you want to keep dependencies small, create a tiny layer of indirection for just the features you need. This layer won't help you reason or test code, but it will help you attract the entire functional Scala market, which will improve adoption of your library.
3. **Ditch Tagless-Final**. In all other cases, I recommend picking a concrete effect type (ZIO, Cats IO, or Monix). If you decide to switch later, you'll pay a one-time cost for the (straightforward) refactor. Encourage a social contract of coding to _interfaces_, not implementations. This contract doesn't scale either, but at least many Java developers are already indoctrinated in the practice. This will give you testability, it can be done incrementally, and it can give you the same (social contract-powered) reasoning benefits as tagless-final, if employed to the same extent.
4. **Minimize Effectful Code**. Consider minimizing code that models side-effects. Look for real abstractions in your domain, which are always equipped with algebraic laws that help you generically reason about polymorphic code. Prefer declarative code instead of imperative code. Avoid monads whenver you can&mdash;they are the engine of imperative programming.

# Summary

There is no doubt about it: tagless-final has a winning sales pitch.

It promises us future-proofing to changes in concrete effect types. It promises us testability. It promises us the ability to reason about effects using parametric polymorphism.

Unfortunately, the reality of tagless-final hasn't lived up to the pitch:

 * Tagless-final does insulate us from an effect type, but that's a maintenance burden and deprives us of useful operations and type-safety.
 * Tagless-final doesn't provide us any testability, per se; it's only coding to an interface that provides us with testability, which can be done with or without tagless-final.
 * Since Scala doesn't track side-effects, we can't actually use the compiler to tell us anything about what side-effects are executed or modeled by a method, regardless of its type signature.

Beyond the failed promises of tagless-final, real world tagless-final is littered with _sync bloat_, which cannot even _claim_ an improved ability to reason about effects parametrically or unit test them. 

Worse still, since tagless-final classes have zero laws, we can't reason generically about code that uses them. True generic reasoning requires _abstractions_, defined by lawful sets of operations, and tagless-final doesn't give us abstractions, only collections of ad hoc operations with unspecified behavior.

Tagless-final, far from being a pancea to managing functional effects, imposes great pedagogical and institutional costs. Many claim the technique renders Scala code bases impenetrable and unmaintainable. While debatable, there is no question that the ramp up curve for tagless-final is significant, and the ergonomics of the technique are quite poor.

Ultimately, in my opinion, the benefits of tagless-final do not pay for the costs&mdash;at least not in most cases. 

Small, stable teams that are already using tagless-final with a working social contract should keep using tagless-final, at least if they have found the benefits outweigh the costs.

Developers of open source libraries can definitely benefit from a layer of indirection around effect types, because even though indirection can't help with reasoning or testability, it can increase addressable market share.

Finally, other teams should probably ditch tagless-final, and instead embrace the age-old best practice of coding to an interface, not an implementation. While still just a social contract, the practice is widely known, doesn't require any fancy training or background, and can be used with many other more ergonomic approaches to testability and reasoning, including traditional dependency injection, module-oriented programming, and the reader monad.

