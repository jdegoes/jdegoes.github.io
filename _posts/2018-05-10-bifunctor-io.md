---
layout:       post
title:        "Bifunctor IO: A Step Away from Dynamically-Typed Error Handling"
description:  "Too long has sloppy, imprecise error handling been the norm in effect systems. Time for precision—time for bifunctor IO!"
category:     articles
tags:         [fp, functional programming, type classes, scala, monads, lenses, effects, reactive, scalaz, cats, mtl, monad transformers]
image:
  feature: "scalaz-8-branding.png"
---

*Bifunctor IO*.

The subject is currently one of the hottest topics in the Scala functional programming community, inspiring both fear and awe in equal measure, as well as a proliferation of tickets, pull requests, and blog posts.

The occasion? Scalaz 8 IO has a new bifunctor design that challenges our assumptions about error handling in effect types, and functional programmers all over Scala's community are trying to decide if and how this design helps us write better software.

Scalaz 8's entirely new design presents a unique opportunity to re-examine every assumption about functional programming in Scala. The bifunctor design joins other innovations in the effect system (such as fine-grained interruption, resource safety, and fiber concurrency), as well as a [growing number of novel features](https://www.slideshare.net/jdegoes/scalaz-8-a-whole-new-game) intended to radically improve FP in Scala.

Fibers and interruption were hotly debated when I first introduced them at Scala IO and [Scale by the Bay](https://www.youtube.com/watch?v=wi_vLNULh9Y), but now [even competitive libraries](http://github.com/typelevel/cats-effect) have moved in the direction of Scalaz 8 IO.

What of bifunctor IO? Is this design going to change the way we manage errors in Scala effect types? Or will it be an interesting footnote in the history of functional programming in Scala?

As a contributor, I'm certainly biased, but I do believe bifunctor IO, which introduces typed errors into the effect type, is here to stay, and that it's going to radically improve our ability to reason about our software. Bifunctor IO helps us better reflect the properties of our software through types, which is what drove many of us to statically-typed functional programming.

In this post, I'm going to give you an overview of the design and debunk some common myths. You can then decide for yourself!

## What is IO?

In many libraries, including Scalaz 7.2 and prior, `IO[A]` (or `Task[A]`) is an immutable data type that models an effectful program that runs forever, fails with a `Throwable`, or computes an `A`.

Like everything else in purely functional programming, `IO` is referentially-transparent&mdash;and all of its methods, excepting `unsafePerformIO` or its equivalents, are total, deterministic, and free of side effects.

You use `IO` values to build other `IO` values, in the same way you can use `String` values to build other `String` values. This style of programming is very similar to programming with immutable lists, maps, and other data structures common in functional programming, except instead of modeling a list, or a map, or something else, it models input and output operations (so-called *effects*).

Inside your main function, you "run" an `IO` by performing its individual operations. Typically, this "run" function is baked into effect types, which makes it trivial to call, as in `unsafePerformIO(myProgram)`. This translation from a model to actual effects is not functional programming&mdash;but for languages like Scala which have no purely functional `main`, it's a necessary evil.

I mentioned earlier that `IO[A]` can fail with a `Throwable` or produce an `A`. This bifurcation should remind you of `Either[E, A]`, and indeed, `IO` data types are merely effectful versions of `Either[E, A]`, where the `E` type is either untyped or loosely typed.

Effect types do not throw exceptions, of course. Their notion of failure is exactly like the notion of "failure" with `Either[E, A]`&mdash;failure is a value and merely short-circuits the remainder of the computation unless the failure is handled. This behaves like an exception, but it's not an exception, it's just an ordinary value (semantically, anyway, if not always in the implementation).

All `IO` ship with equivalents of the following two error management functions:

{% highlight scala %}
object IO {
  // A function defined on the `IO` companion object:
  def fail[A](t: Throwable): IO[A] = ???
}
trait IO[A] {
  ...
  // A method defined on the `IO` data type:
  def attempt: IO[Throwable \/ A] = ???
  ...
}
{% endhighlight %}

where `type \/[A, B] = Either[A, B]`.

These two functions allow you to produce a value that describes a failed computation (`fail`), as well as handle a value that describes a failed computation (`attempt`). They satisfy the obvious properties (e.g. `IO.fail(t).attempt === IO.point(\/.left(t))`, where `IO.point` lifts a pure `A` into `IO[A]`).

`IO` is often how we model effects in purely functional programs. Because it can represent any type of effect and has reasonably high performance, it tends to be the foundation that FP programs are built on. While FP programs may use other effect types, those types usually end up translated into `IO`.

Lots of programming languages have `IO`-like types, including of course Haskell (from which the Scala effect types are inspired!), PureScript, TypeScript, Kotlin, and many other languages.

Interestingly, every `IO[A]` data type in all of these languages has a bifurcated model of computation: either the effect fails with some (dynamically-typed) error, or it succeeds with some value of type `A`.

## What's Bifunctor IO?

I came up with a bifunctor design for IO in 2017, and recently upgraded Scalaz 8 IO to support the new design. In this design, `IO[E, A]` is an immutable data type that models an effectful program that runs forever, fails with an `E`, or computes an `A`.

`IO[E, A]` is a "bifunctor" because you can define an instance of the type class `Bifunctor` (defined [here](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Bifunctor.scala)), which lets you map over not just the value (`A => A2`), but also over the error (`E => E2`).

This bifunctor capability of the data type is essential, as it allows you to "lift" programs that produce one class of errors into programs that produce another (typically larger) class of errors.

In bifunctor IO, the two core error management functions are nearly unchanged, but become significantly more general:

{% highlight scala %}
object IO {
  def fail[E, A](e: E): IO[E, A] = ???
}
trait IO[E, A] {
  ...
  def attempt[E2]: IO[E2, E \/ A] = ???
  ...
}
{% endhighlight %}

Instead of being forced to fail an `IO` with a poorly typed `Throwable`, you can create `IO` values that model failure *for any error type at all*, including strings, numbers, ADTs, or even type-level sets.

Similarly, the `attempt` function becomes polymorphic in the new error type, allowing you to choose any type of error for `E2`. This represents a compile-time guarantee that the attempted `IO` cannot fail, because there is no way it can summon a value of an arbitrary type `E2` into existence. You can easily choose this to be `Nothing` or equivalent (`Void` in Scalaz, Haskell, PureScript, and others).

The power of this approach can be demonstrated in the following snippet:

{% highlight scala %}
def right[A](v: Void \/ A): A =
  v.fold(_.absurd[A])(identity)

val io: IO[E, A] = ???

val attempt1: IO[Void, E \/ A] = io.attempt[Void]
val attempt2: IO[Void, Void \/ (E \/ A)] = attempt1.attempt[Void]

val attempt3: IO[Void, E \/ A] = attempt2.map(right)
{% endhighlight %}

This snippet demonstrates at compile-time the intuitive fact that if you handle an error from an effect, then if you try to handle it again, you will never see the failure case, because you've already handled it. The fact that you can express the compile-time guarantee that *repeated attempts cannot fail* is quite remarkable and powerful.

While error management changes only in minor ways, the resulting ramifications to error model are profound. Before I explore them, however, we must talk about the different types of errors.

## Types of Errors

There are two classes of errors in applications:

 1. **Recoverable Errors**. Recoverable errors refer to the errors we expect to happen occasionally. For example, when we perform an HTTP request, we expect that sometimes, it will not succeed because the server is down or there are network issues. Because we expect these errors to happen, we want to build our code in a way that acknowledges the failure scenarios and has graceful fallbacks at some level of the application (not necessarily locally).
 2. **Non-Recoverable Errors**. Non-recoverable errors refer to the errors we do not expect to happen. For example, if a user has configured the JRE with 10k of memory for the stack, then most ordinary (non-recursive) code will stack overflow, producing a `StackOverflowError`. This is not a normal error we expect to happen, nor is there a sane way to recover from it.

 In Java, the convention is to use `Exception` for recoverable errors, and `Error` for non-recoverable errors. Both of these are subclasses of `Throwable`, which is the only error type supported by the JVM.

In languages like Erlang and C, recoverable errors are modeled with ordinary values, and non-recoverable errors do not have an explicit representation and are managed outside application logic.

The distinction between failure scenarios that we expect to happen and failure scenarios that we do not expect to happen is extremely important.

Failure scenarios that we expect to happen can be recovered from in application code. In statically-typed functional programming, we have the ability to guarantee at compile-time that expected failures are handled, or at least tracked, so we know the consequences of interacting with such code.

Failure scenarios that we do *not* expect to happen cannot be sanely recovered from, and the reason is precisely this: we did *not* expect these errors, so we do not know *where* they will occur, *what* these errors will be, or whether it makes sense to continue, and if so, what remedial actions permit a well-defined recovery.

Attempting to recover from an unexpected error puts your program into an undefined state: it is undefined precisely because you do not know where the error occurs, which means you don't know the state of the application or its external environment; and you do not know what the error is, which means you don't know if and how to recover.

Non-recoverable errors are almost always defects in a program, but can sometimes happen for reasons outside the control of the programmer (such as running out of memory). A C program may overwrite some arbitrary piece of memory (defect), a Java program may try to allocate more memory than reserved for the heap (catastrophic), or an Erlang actor may make a false assumption about data (defect).

The distinction between recoverable and non-recoverable errors fully informs the design of the Scalaz 8 `IO` error model. In the next section, I'll introduce this error model and talk about its implications.

## A Principled Error Model

![Scalaz 8 Error Model](/images/scalaz-8-error-model.png)

In the Scalaz 8 `IO` error model, the distinction between recoverable and non-recoverable errors is explicit, and forms the foundation of the error model.

Recoverable errors are statically-typed, and described by the `E` type parameter in `IO[E, A]`.

Effects of type `IO[E, A]` can fail for any `E`, and can be recovered from to yield `IO[E2, E \/ A]`, for any type `E2`, including uninhabited types like `Nothing` and Scalaz's `Void` type.

These strong, compile-time guarantees mean the type system tells you *if* and *how* an effect can fail.

Non-recoverable errors are loosely typed, and include the full range of runtime errors on the JVM, which is any type that extends `Throwable`.

Non-recoverable errors are, as their name suggests, not recoverable, which means that like in C or Erlang. If one of these errors occurs, the fiber running the effect will be terminated.

The error model implies that aside from non-termination, there are exactly three possible outcomes from running an effect `IO[E, A]`:

 * The effect fails due to an unhandled `E` created with `IO.fail`.
 * The effect is terminated due to a non-recoverable `Throwable`.
 * The effect computes an `A`.

We'll explore some of the standout features of the error model in the sections that follow.

### Feature 1: No Dynamically-Typed Errors

The most distinguishing feature of the error model is that it allows us to move away from dynamically-typed errors.

Programming with `IO[A]` provides no compile-time guarantees about failure. From the type alone, you must assume that all effects may fail for any reason at any time, even if you've already handled them or they can only fail in specific ways.

With `IO[E, A]`, you know all possible (recoverable) failure scenarios. If you handle some failure scenarios but not others, you can express this in the type. If the effect may truly fail for any reason at any time, then that, too, will be reflected in the type.

For one practical application among many, suppose that you want your REST API to return errors in the JSON format. In this case, you can simply express the requirement in the type:

{% highlight scala %}
def serve(handler: HTTPRequest => IO[Void, JSON])
{% endhighlight %}

The type here demands that the handler you pass to the `serve` function handle all its own errors and encode them into JSON. Now you don't have to worry about code forgetting to handle some error and then not knowing how to transcribe that into the format documented by your API.

As another example, suppose we have an evaluator in our spreadsheet application for some domain-specific scripting language. In this case, we want to know precisely how evaluation may fail, because failure is expected and we will handle different failures in different ways.

In this case, we can model the evaluator as follows:

{% highlight scala %}
sealed trait EvaluationError
case class ParseError(line: Int, expected: List[Symbol]) extends EvaluationError
case class TypeError(expected: Type, actual: Type) extends EvaluationError
case class RuntimeError(function: String) extends EvaluationError

def evaluate(script: String): IO[EvaluationError, Unit] = ???
{% endhighlight %}

Both statically-typed values *and* statically-typed errors let us use the compiler to prove properties of our programs. In the case of statically-typed errors, we can prove that effects only fail in some ways, or that they don't fail at all.

Further, we can vary this in different parts of our program, because we have the flexibility to vary the type of error.

### Feature 2: No Lost Errors

Most programming languages have lossy error models. By lossy, I mean it is possible for exceptions to be swallowed.

In the following snippet, two errors will be swallowed:

{% highlight scala %}
try {
  try {
    try throw new Error("e1")
    finally throw new Error("e2")
  } finally throw new Error("e3")
} catch {
  case e : Error => println(e)
}
{% endhighlight %}

Most Scala (or Java) developers don't know which error will be caught and printed in the outermost `catch` block, but one fact is clear from the structure: at most one exception will be reported, meaning the other two will be lost forever.

Scalaz 8 IO dramatically improves on this situation by _guaranteeing_ that no errors are ever lost, whether they are recoverable errors modeled with `E`, or non-recoverable errors that result from defects or catastrophic runtime errors.

This guarantee is provided with a *hierarchy of supervisors*. A *supervisor* is just a function `Throwable => IO[Void, Unit]`, which will be passed all unhandled errors.

The root fiber, which runs the application `main` function, has a default supervisor, which can be overridden. The default supervisor prints the stack trace of the error to the console.

Other supervisors are specified by the application code when forking an effect. For example, if you want to run some work in a separate fiber, you call `fork0` and can specify the supervisor for the fiber:

{% highlight scala %}
for {
  fiber <- work.fork0(t => logError(t))
  ...
} yield ()
{% endhighlight %}

In this example, the supervisor merely logs the error to a file.

Supervisors are hierarchical. If you don't specify a supervisor when forking an effect, then it will inherit the supervisor of the fiber that performed the forking.

The hierarchy provides a modular way of dealing with unhandled errors. Lower levels of an application can decide how to supervise their children, or simply delegate to their parent.

### Feature 3: No Resource Leaks

Scalaz 8 `IO` has a built-in primitive called `bracket`, which is a more structured version of `try` / `finally`.

The `bracket` primitive allows you to acquire a resource in one effect, use the resource in another effect, and finally, close the resource in a different effect.

The usage of this primitive is quite simple:

{% highlight scala %}
openFile("data.csv").bracket(closeFile(_)) { handle =>
  // Use resource in here...
}
{% endhighlight %}

Resources will be *always* be released, whether the cause is due to an unhandled `E`, a defect in the code, fiber interruption, or a catastrophic external error.

Resource safety lets you program bullet-proof, long-lived applications that are resilient to failures, whether expected or unexpected.

### Feature 4: No Changes Needed

A remarkable property of the Scalaz 8 `IO` bifunctor design is that it generalizes over both the _existing_ dynamically-typed error effect types and so-called _non-exceptional_ effect types.

Existing effect types may fail at any time, for any reason. To model this type of effect type, we can use the following type alias:

{% highlight scala %}
type Task[A] = IO[Throwable, A]
{% endhighlight %}

With one exception, there is _no difference_ between this definition of `Task` and other dynamically-typed error effects. This is the preferred migration option if you are moving to Scalaz 8 from Scalaz 7 `IO` or `Task`, Monix `Task`, or cats-effect `IO`.

The only difference relates to how _defects_ are handled. Older error models conflate recoverable errors with non-recoverable errors, which "works" because they fix the error type to `Throwable`. These types encourage users to write code like `IO.point(42).map(_ => throw Error("Surprise!")).attempt`.

In Scalaz 8 `IO`, defects in your code cannot be "caught", similar to Erlang, C, and many other real-world programming environments. Since we use values to represent failure in functional programming (in the case of `IO`, values constructed with `IO.fail`), throwing exceptions from your pure code is considered a defect, and the fiber running the effect will be terminated.

The cause for the termination will, of course, be passed to the fiber's supervisor, so it can be logged or otherwise made visible. Or, depending on the context, you might decide to restart a fiber that fails due to a defect, similar to supervision in actor systems.

Aside from this one difference, users who prefer only ever using `Throwable`, or who wish to introduce typed areas selectively and gradually, can simply use `IO[Throwable, A]` pervasively. No changes to your error handling are necessary.

### Feature 5: No Errors

As discussed previously, `IO` can also model effects that cannot fail. The following snippet creates a type synonym to do just this:

{% highlight scala %}
type Unexceptional[A] = IO[Void, A]
{% endhighlight %}

Effects of type `Unexceptional[A]` cannot (recoverably) fail, meaning if they fail, it's because of a non-recoverable defect in your code or because of a catastrophic error like `OutOfMemoryError`.

`IO[E, A]` generalizes over both non-exceptional effect types and dynamically-typed effect types. In other words, the bifunctor design of Scalaz 8 IO is _strictly more expressive_ than all other effect types. Other effect types fall out elegantly as special cases.

Developers who complain about the bifunctor design are really complaining because they have _gained_ the ability to be more precise, not because they are _required_ to be more precise. This makes no sense at all&mdash;it's like complaining about optional type annotations.

The alternative to the bifunctor design is to introduce special-case types for non-exceptional or strongly-typed effect types. For example, cats-effect will probably get `UIO`, which is a non-exceptional effect type, and may at some point get a bifunctor `BIO`. Such proliferation of effect types (with massive redundancy in the implementations) is unnecessary with the bifunctor design, because a single generalized type `IO[E, A]` is capable of modeling everything.

## Debunking Myths of Bifunctor IO

The bifunctor design for the Scalaz 8 `IO` effect type is so new, there's not much documentation on the subject. As a result, the design has been subject to a lot of speculation.

In the next sections, I want to take some time to debunk what I perceive to be myths around the bifunctor design. Arguably, some of these myths are really _strong biases_ toward dynamically-typed errors, but most of you who read this blog prefer statically-typed functional programming, and will probably share my own bias in these regards.

### Myth 1: Composition Destroys Specific Error Types

This myth states that composing two effects `IO[E1, A]` and `IO[E2, B]` will result in an error type that erases the distinction between `E1` and `E2`. In other words, composing effects leads to a loss of precise error types.

This myth is false because users themselves define how error types compose. For example, I can easily combine two effects `IO[E1, A]` and `IO[E2, B]` with different errors in a lossless way using a disjunction type, such as `E1 \/ E2`.

This technique is demonstrated in the following snippet:

{% highlight scala %}
val io: IO[E1 \/ E2, (A, B)) =
  for {
    a <- op1.leftMap(\/.left)
    b <- op2.leftMap(\/.right)
  } yield (a, b)
{% endhighlight %}

In general, error types compose in a lossless way using sum types. In some cases, you may want a built-in sum type like `\/` (or `Either`), as shown in the above snippet. In other cases, it is more convenient to define a custom sum type, such as the previously introduced `EvaluationError`:

{% highlight scala %}
sealed trait EvaluationError
case class ParseError(line: Int, expected: List[Symbol]) extends EvaluationError
case class TypeError(expected: Type, actual: Type) extends EvaluationError
case class RuntimeError(function: String) extends EvaluationError
def evaluate(script: String): IO[EvaluationError, Unit]
{% endhighlight %}

Others still may prefer to use type-level sets to represent errors, because this makes it very easy to add and subtract errors, while still providing the ability to handle some or all errors.

The [Mitigation library](https://twitter.com/propensive/status/993422307388424192) by [Jon Pretty](http://twitter.com/propensive), based on Totalitarian, provides a convenient and powerful solution for managing sets of errors. Look for `scalaz-mitigation` in the near future! (Bug Jon and me about it until we do something!)

### Myth 2: You Don't Recover From Errors Often

This myth states that typed errors are not useful because most code doesn't recover from most types of errors.

This is only true when recoverable errors are conflated with non-recoverable errors. If an error is not expected, and therefore cannot be sanely recovered from, then it will neither appear in any type signature, nor affect the development experience.

For example, take the following snippet of code:

{% highlight scala %}
for {
  r1 <- op1
  r2 <- op2
  r3 <- op3
} yield r1 + r2 + r3
{% endhighlight %}

If `op1`, `op2`, and `op3` do not fail in recoverable ways, then they will have type `IO[Void, A]` (for some type `A`), which means they compose with effects of any other error type, and we know from looking at the type signature that these operations do not fail.

### Myth 3: The Error Type is an Encapsulation Leak

This myth states that having statically typed errors leaks implementation details. This is not true at all&mdash;in fact, poor API design leaks implementation details.

Let's take the following API:

{% highlight scala %}
def read(id: String): IO[FileNotFoundException, ByteVector]
{% endhighlight %}

This API states `read` can fail with `FileNotFoundException`. However, we may want to abstract over different ways of reading a resource (in-memory versus over HTTP, for example). If we do want to abstract over different sources, then `FileNotFoundException` is a poor way to represent failure. Instead, we should design the API as follows:

{% highlight scala %}
def read(id: String): IO[ResourceNotFound, ByteVector]
{% endhighlight %}

Or possibly even more simply:

{% highlight scala %}
def read(id: String): IO[Void, Option[ByteVector]]
{% endhighlight %}

Whether or not an effect may fail in a recoverable way is part of an API's public contract, and should be reflected clearly in the types. That you can make bad design choices here just means API design requires skill, which is true even without typed errors.

For example, take the following API:

{% highlight scala %}
def resolve(path: String): Task[File]
{% endhighlight %}

This API resolves a `path` to a `File` resource. However, `File` is constrained to represent local files, and could easily be generalized to `URI` or something similar. If we need to abstract over different resource locations, then `File` is a poor choice.

No one would suggest that because it is possible to return `File`, and that `File` leaks implementation details, we should therefore use `AnyRef` for all return values so we can let implementations vary.

In the same way, just because it is possible to leak implementation details in the type of an error channel, doesn't mean we should use dynamically-typed errors.

There is no connection whatsoever between typed errors and the leaking of implementation details. They are completely orthogonal.

### Myth 4: It Pushes Complexity to the User

This myth states that using typed errors pushes "complexity" to the user. The myth conflates *inherent complexity* with *incidental complexity*.

*Inherent complexity* is that complexity that is inherent to the task of properly modeling a domain. For example, if we wish to precisely model the domain of JSON values, then we must have a data type that supports boolean, null, numbers, arrays, and objects. Further, our number type must support arbitrary precision.

{% highlight scala %}
sealed trait JSON
case object JNull extends JSON
case class JBool(value: Boolean) extends JSON
case class JNum(value: BigDecimal) extends JSON
case class JArr(value: List[JSON]) extends JSON
case class JObj(value: Map[String, JSON]) extends JSON
{% endhighlight %}

This is complexity, but it is inherent complexity because it is inherent to the problem of properly modeling JSON values.

*Incidental complexity* is complexity that arises not because of the problem domain, but because of implementation decisions that could have been otherwise without sacrificing correctness. For example, dependency injection frameworks have much *incidental complexity*&mdash;they are a very complex way of passing arguments to functions.

Precisely modeling data types always comes with some cost due to *inherent complexity*. Making illegal states unrepresentable requires design work. If we want to precisely describe *if* and *how* our effects may fail, then this, too, may require work, but this complexity is inherent to precision in error handling.

A design like Scalaz 8 `IO` lets you choose how precise you want to be. It has the flexibility to describe infallible effects or effects that fail in only pre-defined ways, but you only need to use this flexibility when you decide the precision adds enough value.

Other designs remove the choice from you, forcing you into a dynamically-typed error model that offers no ability to make illegal (error) states unrepresentable.

### Myth 5: The Bifunctor IO Doesn't Reflect the Runtime

This myth states that any effect type should "reflect the runtime", which is vague enough to warrant [sourcing the claim](https://twitter.com/djspiewak/status/983805298526699520):

> "IO needs to reflect and describe the capabilities of the runtime, for good or for bad. All it takes is an 'innocent' throw to turn it all into a lie, and you can't prevent that."

This is an argument to catch defects, such as exceptions thrown from pure code, so users can attempt recovery. It's also an argument to fix the failure type to `Throwable`, because this is what the JVM does.

The argument is incoherent, as can be seen by examining the implications of "reflecting the runtime".

The JVM does not support parametric polymorphism, which means the `A` in `IO[A]` is erased at runtime to the equivalent of `IO[AnyRef]`. Because of this type erasure and the lack of type safety in Scala, this means having a value of type `IO[A]` is no guarantee the `IO` value will actually compute an `A`.

To "reflect the runtime" in the `IO` data type, it would be necessary to avoid using parametric polymorphism entirely, because this feature is not supported by the JVM, and it's impossible to enforce that `IO[A]` actually produces anything of type `A`. After all, it just takes one innocent, accidental coercion to turn `IO[A]` into a lie, and you can't prevent that!

In addition, the JVM uses `null` for optionality, not `Option` or `Maybe`. Reflecting the runtime would mean using `null` to represent optionality, and expecting `null` everywhere in the interface to `IO`.

Doing functional programming, [even in languages like Haskell](http://www.cse.chalmers.se/~nad/publications/danielsson-et-al-popl2006.pdf), requires a set of assumptions. In Scala, these assumptions include no runtime reflection, no coercion, no exceptions, no soundness bugs, no null, and so forth.

Without these assumptions (often called the [Scalazzi subset](http://www.lihaoyi.com/post/StrategicScalaStylePracticalTypeSafety.html#scalazzi-scala) of Scala, after Scalaz), you would program fearfully, constantly checking the runtime class of every value, testing to make sure no values are null, obsessively try/catching, and fearing that a function like `def identity[A](a: A): A` may generate a novel `A` through reflection.

Principled software requires sane assumptions, and pragmatic software requires we know when those assumptions go wrong. The Scalaz 8 error model provides both in a single package, because even though the design assumes you are working in Scalazzi, if you don't, your defects will be captured and reported to supervisors.

### Myth 6: Bifunctor IO Doesn't Let You Use Impure Code

This myth states that because the bifunctor design doesn't "automatically catch errors", you can't use the effect type with Java code or impure Scala code.

This is not true at all. All code, including exception throwing, impure code, can be trivially imported into `IO`. In fact, there are three utility functions provided to do just this:

 * `def syncThrowable[A](eff: => A): IO[Throwable, A]` &mdash; This function imports code that may throw `Throwable` values, properly translating them into `IO.fail` values.
 * `def syncException[A](eff: => A): IO[Exception, A]` &mdash; This function imports code that may throw `Exception` values, properly translating them into `IO.fail` values.
 * `def syncCatch[E, A](eff: => A)(pf: PartialFunction[Throwable, E]): IO[E, A]` &mdash; This function imports code that may throw a user-defined range of errors, properly translating them into `IO.fail` values.

These functions let you safely interact with dysfunctional code, which performs side-effects and throws exceptions. You can import this code into pure values, which can interface with your functional code.

Wrapping unsafe code with `Try`, to safely convert exceptions into values, is something that many Scala developers are already doing.

### Myth 7: Bifunctor IO Is Just EitherT

This myth states that bifunctor IO is unnecessary, since we can already achieve the same benefits by stacking the `EitherT` monad transformer on top of ordinary `IO`.

There are two flaws in this argument:

1. For older effect types `F[_]` (such as Monix `Task`, cats-effect `IO`, and Scalaz 7.x `IO` / `Task`), `EitherT[IO, E, ?]` has two error channels, overlapping instances for `MonadError` (with one having an error type fixed to `Throwable`), two conflicting ways of failing, and two conflicting ways of recovering from failures. Contrast this to `IO[E, A]`, which has a single error channel. In truth, Scalaz 8 bifunctor IO is functionally equivalent to `EitherT[UIO, E, A]` for some infallible effect type `UIO`.
2. Although Scalaz 8 IO could be redesigned to be infallible, this would mean as much as a *5x performance penalty* for recapturing error management using an `EitherT` monad transformer. As discussed in my [last blog post](http://degoes.net/articles/effects-without-transformers), monad transformers are not practical in Scala.

These points mean that Scalaz 8 IO is bringing something genuinely new to the table&mdash;a clean, typed, single error channel, and high-performance, in a single package that generalizes over all existing effect types.

## Final Words

Like other changes in Scalaz 8, including the fiber concurrency model, fine-grained interruption, and resource safety, the bifunctor design of `IO` requires rethinking assumptions.

This new model doesn't _require_ you to be more precise, since you're free to stick with `Throwable` for all your errors. But it does _empower_ you to describe failure scenarios precisely, and vary *if* and *how* effects fail at different levels of your application.

Benefits of the error model are many, and include being able to model infallible effects (non-exceptional `IO`), use effects the way you're familiar with (using `Throwable` everywhere) so you can gradually incorporate typed errors where useful, guarantee no resource leaks (even if your program generates non-recoverable errors), never lose any errors (unlike other error models, including the ones baked into Java and Scala), and have strong, compile-time guarantees about if and how your effects can fail.

While there are myths out there about the new bifunctor design, this is expected since there is a scarcity of documentation. Without good documentation, there will always be questions about how a new design works or what the tradeoffs are. Hopefully this blog post succeeds in making the bifunctor design more accessible.

If I've convinced you the bifunctor design has value, vote with your feet: support projects like Scalaz 8 IO, encourage contributions like Luka Jacobowitz's [bifunctor IO](https://github.com/LukaJCB/cats-bio), and contribute issues and pull requests to your favorite effect type.

Scalaz 8 IO may be the first one to adopt this design, but from the looks of things, it's not the last. I greatly look forward to seeing others adopt and improve upon the design, to the benefit of the entire Scala functional programming community.
