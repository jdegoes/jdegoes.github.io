---
layout:       post
title:        "ZIO & Cats Effect: A Match Made in Heaven"
description:  "ZIO is a compelling effect type for working with the great Cats Effect ecosystem."
category:     articles
tags:         [fp, functional programming, type classes, scala, monads, lenses, effects, reactive, scalaz, cats, mtl, monad transformers, zio, reader, environmental effects]
---

Cats Effect has become the ["Reactive Streams"](http://www.reactive-streams.org) of the functional Scala world, enabling a diverse ecosystem of libraries to work together.

Many great libraries like http4s, FS2, and Doobie are built on the Cats Effect type classes, and effect libraries like [ZIO](https://github.com/scalaz/scalaz-zio) and Monix provide instances of these type classes for their effect types. 

Although not without a few drawbacks, many of which will be [rectified in 3.0](https://github.com/typelevel/cats-effect/issues/321), the Cats Effect library is helping many open source contributors economically support the whole functional Scala ecosystem.

Application developers who use Cats Effect face a far more difficult choice: which of the major effect types they will use to build their applications.

Application developers have three major choices:

 - Cats IO, the reference implementation in Cats Effect
 - Monix, with its `Task` data type and associated reactive machinery
 - More recently, ZIO, with its `ZIO` data type and concurrent machinery

In this post, I'm going to argue that if you are building a Cats Effect application, then ZIO provides a compelling choice, one with design choices and features quite different than the Cats IO reference implementation.

Without further ado, let's take a look at my top 12 reasons why ZIO and Cats Effect are a match made in heaven!

# 1. Better MTL / Tagless-Final

MTL, which stands for _Monad Transformers Library_, is a style of programming where functions are _polymorphic_ in their effect type, expressing their requirements through _type class constraints_.

In Scala, this is often called the _tagless-final_ style (although they are not exactly the same thing), especially when the type classes have no laws.

It is well-known that it is impossible to define global instances for such classic MTL type classes as _Writer_ and _State_ for effect types like Cats IO.

The reason is that the instances of these type classes for effect types requires access to mutable state, which cannot be created globally, because the creation of mutable state is effectful.

For [performance reasons](/articles/effects-without-transformers), however, it's critical to avoid monad transformers, and provide an implementation of _Writer_ and _State_ directly atop the underlying effect type.

To accomplish this, functional Scala developers use a trick: they effectfully (but purely) create instances at the top level of their program, and then provide them downstream as local implicits:

{% highlight scala %}
Ref.make[AppState](initialAppState).flatMap(ref =>
  implicit val monadState = new MonadState[Task, AppState] {
    def get: Task[AppState] = ref.get 

    def set(s: AppState): Task[Unit] = ref.set(s).unit
  }

  myProgram
)
{% endhighlight %}

Although this trick is useful, it is also a hack. In a perfect world, all type class instances would be globally coherent (_one instance per type_)&mdash;not created locally and effectfully and then magically turned into implicit values to feed downstream methods.

A remarkable property about MTL / tagless-final is that you can _directly_ define most instances atop the ZIO data type, by using [ZIO Environment](/articles/zio-environment).

Here's one way to create a global definition of `MonadState` for the ZIO data type:

{% highlight scala %}
trait State[S]
  def state: Ref[S]
}
implicit def ZIOMonadState[S, R <: State[S], E]: MonadState[ZIO[R, E, ?], S] =
  new MonadState[ZIO[R, E, ?], S] {
    def get: ZIO[R, E, S] = ZIO.accessM(_.state.get)

    def set(s: S): ZIO[R, E, Unit] = ZIO.accessM(_.state.set(s).unit)
  }
{% endhighlight %}

This instance is now defined globally for any environment that supports at least `State[S]`.

Similarly for `FunctorListen`, otherwise known as `MonadWriter`:

{% highlight scala %}
trait Writer[W]
  def writer: Ref[W]
}
implicit def ZIOFunctorListen[W: Semigroup, R <: Writer[W], E]: FunctorListen[ZIO[R, E, ?], W] =
  new FunctorListen[ZIO[R, E, ?], W] {
    def listen[A](fa: ZIO[R, E, A]): ZIO[R, E, (A, W)] = 
      ZIO.accessM(_.state.get.flatMap(w => fa.map(a => a -> w)))

    def tell(w: W): ZIO[R, E, W] = 
      ZIO.accessM(_.state.update(_ |+| w).unit)
  }
{% endhighlight %}

And of course, we can do the same for `MonadError`:

{% highlight scala %}
implicit def ZIOMonadError[R, E]: MonadError[ZIO[R, E, ?], E] = 
  new MonadError[ZIO[R, E, ?], E]{
    def handleErrorWith[A](fa: ZIO[R, E, A])(f: E => ZIO[R, E, A]): ZIO[R, E, A] = 
      fa catchAll f

    def raiseError[A](e: E): ZIO[R, E, A] = ZIO.fail(e)
  }
{% endhighlight %}

This technique is readily applicable to other type classes, including tagless-final type classes, whose instances may require effectfully-created state (mutable, configuration, etc.), testable effectful functions (combining environmental effects with tagless-final), or anything else that is readily accessible from the environment.

So if you love using MTL-style, or find the benefits of tagless-final outweigh the costs, then using ZIO lets you easily define _global_ instances for all your favorite type classes.

No slow monad transformers, no effectfully created type class instances, no local implicits, and no hacks. Just straight up pure functional programming!

# 2. Resource-Safety for Mortals

An early defining feature of ZIO was _interruption_, the ability for the ZIO runtime to instantaneously cancel any executing effect, safely cleaning up all resources; and a course-grained version of this feature eventually made its way into Cats IO.

This feature, called _async exceptions_ in Haskell, allows composable and efficient timeouts, efficient parallel and race operations, and globally efficient computation.

While extremely powerful, interruption poses unique challenges for resource safety. 

Programmers are mostly used to mentally tracking failure in their applications, or ZIO uses the type system to help track failure. But interruption is different. An effect composed from many other effects can be interrupted at *any* boundary.

Take the following effect:

{% highlight scala %}
for {
  handle <- openFile(file)
  data   <- readFile(handle)
  _      <- closeFile(handle)
} yield data
{% endhighlight %}

Most programmers would not be surprised that if `readFile` failed, then the `closeFile` would not be executed. Fortunately, effect systems have `ensuring` (called `guarantee` in Cats Effect) that lets you add a finalizer to an effect, similar to `finally`.

So the main problem with the above effect can be fixed simply:

{% highlight scala %}
for {
  handle <- openFile(file)
  data   <- readFile(handle).ensuring(closeFile(handle))
} yield ()
{% endhighlight %}

Now the effect is _failure-proof_, in the sense that if `readFile` fails, then the file will be closed; and if `readFile` succeeds, the file will be closed; so in "all" cases, the file will be closed.

Well, not quite _all_. _Interruption_ means that the executing effect can be terminated anywhere, even _between_ the `openFile` and the `readFile`. If this happens, then the opened resource will not be closed, and a leak will result.

This pattern of acquiring and releasing a resource is so common, that ZIO introduced a `bracket` operator that made its way to Cats Effect 1.0.

The `bracket` operator is _interruption-proof_: if the acquire succeeds, then release will be called, no matter what, even if the effect that uses the resource is interrupted. Further, neither the acquire nor release can be interrupted, providing a strong guarantee of resource safety.

With `bracket`, the above example looks like this:

{% highlight scala %}
openFile(file).bracket(closeFile(_))(readFile(_))
{% endhighlight %}

Unfortunately, `bracket` only encapsulates one (particularly common) pattern of resource consumption; there are many others, especially with concurrent data structures, whose acquisition must be _interruptible_ in order to avoid a different kind of leak.

In general, when programming with interruption, there are two things we want to do:

 - Prevent interruption from happening in some region that is otherwise interruptible
 - Allow interruption to happen in some region that is otherwise uninterruptible

ZIO has facilities to make both of these very easy. For example, we can implement our own version of `bracket` using lower-level features built into ZIO:

{% highlight scala %}
ZIO.uninterruptible {
  for {
    a    <- acquire
    exit <- ZIO.interruptible(use(a)).run.flatMap(exit => release(a, e).const(exit))
    b    <- ZIO.done(exit)
  } yield b
}
{% endhighlight %}

In this code, `use(a)` is the only part that can be interrupted, and the surrounding code takes care to execute `release` in any case.

Interruptibility can be arbitrarily checked, turned off, or turned on, and only _two_ primitive operations are necessary (all others are derived from these).

This compositional, full-featured model of interruptibility allows not just a clean implementation of `bracket`, but clean implementations of other scenarios in resource handling, which carefully balance the tradeoffs inherit in interruptibility.

Cats IO chose to provide only a _single_ operation to manage interruptibility: a combinator called `uncancelable`. This makes a whole region uninterruptible. However, by itself the operation is of limited use, and can easily lead to code that wastes resources or deadlocks.

While it turns out that one can define an operator that provides more control over interruption atop Cats IO, the (quite clever!) [implementation by Fabio Labella](https://github.com/SystemFw/playground/blob/d84aebb5fc1d2ccc4328afdca7ec8e923ef5a288/src/main/scala/Playground.scala) is insanely complex and not performant.

ZIO lets anyone write interruption-friendly code, operating at a high-level, with declarative, composable operators, and doesn't force you to either choose between extreme complexity and poor performance on the one hand, and wasted resources and deadlocks on the other.

Moreover, although not discussed in this post, the newly-added [Software Transactional Memory](https://www.youtube.com/watch?list=PL8NC5lCgGs6MYG0hR_ZOhQLvtoyThURka&v=d6WWmia0BPM) in ZIO lets users declaratively write data structures and code that is automatically asynchronous, concurrent, and safely interruptible.

# 3. Guaranteed Finalizers

The `try` / `finally` construct in many programming languages provides us the robust guarantees we need to write synchronous code that doesn't leak resources. 

In particular, the construct provides the following guarantee:

 - If the `try` block begins execution, then the `finally` block will begin execution when the `try` block stops execution
 
This guarantee holds even if:

 - There are nested `try` / `finally` blocks
 - There are errors in the `try` block
 - There are errors in a nested `finally` block

ZIO's `ensuring` operation can be used exactly like `try` / `finally`:

{% highlight scala %}
val effect2 = effect.ensuring(cleanup)
{% endhighlight %}

ZIO provides the following guarantee on `effect.ensuring(finalizer)`:

 - If `effect` begins execution, then `finalizer` will begin execution when the `effect` stops execution

Like `try` / `finally`, this guarantee holds even if:

 - There are nested `ensuring` compositions
 - There are errors in `effect`
 - There are errors in any nested finalizer

Moreover, the guarantee holds even if the effect is _interrupted_ (the guarantees on `bracket` are similar, and in fact, `bracket` is implemented on `ensuring`).

The Cats IO data type chose a different, weaker guarantee. For `effect.guarantee(finalizer)`, the guarantee is weakened as follows:

 - If `effect` begins execution, then `finalizer` will begin execution when the `effect` stops execution, **unless** problematic effects are composed into `effect`

This weakening also occurs for the Cats IO implementation of `bracket`.

In order to leak resources, it is only necessary to compose, somewhere in the effect of `guarantee`, or inside the "use" effect of `bracket`, an effect similar to the following:

{% highlight scala %}
// Assume `interruptedFiber` is some fiber that is already interrupted:
val bigTrouble = interruptedFiber.join
{% endhighlight %}

When `bigTrouble` is so composed into another effect, the effect becomes _non-terminating_&mdash;neither finalizers installed with `guarantee` nor cleanup effects installed with `bracket` will be executed, leading to _resource leaks_ and _skipped_ finalization.

For example, the finalizer in the following code will never begin execution:

{% highlight scala %}
(IO.unit >> bigTrouble).guarantee(IO(println("Won't be executed!!!")))
{% endhighlight %}

Using local reasoning, it is not possible to know if an effect like `bigTrouble` is being composed somewhere in the "use" effect of bracket or inside of a finalizer.

Therefore, you cannot know if a Cats IO program will leak resources or skip finalization without global program analysis. Global program analysis is a manual, error-prone process that cannot be checked by the compiler, and which must be repeated every time any relevant part of the code changes.

ZIO has custom implementations of the Cats Effect `guarantee`, `guaranteeCase`, and `bracket` operations. The implementations use native ZIO semantics (not Cats IO semantics), which allow you to reason locally about resource safety, knowing that in all cases, finalizers _will_ be run and resources _will_ be freed.

# 4. Stable Shifting

Cats Effect has an `evalOn` method of `ContextShift`, which allows moving the execution of some code to another execution context.

This turns out to be quite handy for a number of reasons:

- Many client libraries require you to do some work in their thread pool
- UI libraries require some updates to be done on the UI thread
- Some effects need to be isolated on thread pools tailored for their specific needs

The `evalOn` operation is designed to execute an effect where it needs to be run, and then hop back to the original execution context. For example:

{% highlight scala %}
cs.evalOn(kafkaContext)(kafkaEffect)
{% endhighlight %}

_Note_: Cats IO has a related construct called `shift` that allows you to "hop" over to another context without hopping back, but in practice, this behavior is almost never desired, so the `evalOn` variation is strongly preferred.

ZIO's implementation of `evalOn` (built on the ZIO primitive `lock`) provides a guarantee necessary for local reasoning about where effects are running:

 - The effect will always execute on the specified context

Cats IO chose a different, weaker guarantee:

 - The effect will execute on the specified context **until** the first asynchronous operation or embedded shift

Using just local reasoning, it is not possible to know if an asynchronous effect (or nested shift) is being composed into the effect being shifted, because asynchronicity is not reflected in types.

Therefore, as with resource safety, knowing where a Cats IO effect will run requires global program analysis. In practice, and from my experience, users of Cats IO are quite surprised when they use `evalOn` with one context, and then later find out that most of the effect has been accidentally executed on some other context.

ZIO lets you specify where effects should run and trust that will actually happen, in all cases, regardless of how effects are composed with other effects.

# 5. Lossless Errors

Any effect type that supports concurrency, parallelism, or resource safety runs into an immediate problem with a linear error model: in general, errors don't compose. 

This holds both for `Throwable`, the fixed error type baked into Cats IO, and for polymorphic error types, which are supported by ZIO.

All the following situations can lead to multiple errors being produced:

 - A finalizer throwing an exception
 - Two (failing) effects being combined in parallel 
 - Two (failing) effects being raced
 - An interrupted effect also failing before exiting an uninterruptible section

Because errors do not compose, ZIO has a data structure called `Cause[E]`, which provides a free _semiring_ (an abstraction from abstract algebra, which you can safely ignore if you haven't heard about before!), which allows lossless composition of sequential and parallel errors for any arbitrary error type.

During all operations (including cleanup for a failed or interrupted effect), ZIO aggregates errors into the `Cause[E]` data structure, which can be accessed at any time.

As a result, ZIO never loses any errors: they can all be accessed at the value level, and then logged, inspected, or transformed, as dictated by business requirements.

Cats IO chose to embrace a lossy error model. Wherever ZIO would compose two errors using `Cause[E]`, Cats IO "throws" one error away&mdash;for example, by calling `e.printStackTrace()` on the tossed error.

For example, the finalizer error in this snippet will be "thrown away":

{% highlight scala %}
IO.raiseError(new Error("Error 1")).guarantee(IO.raiseError(new Error("Error 2")))
{% endhighlight %}

This lossy side-channel error reporting means there is no way to locally detect and respond to the full range of errors that can occur as effects are composed.

ZIO lets you use any error type you want, including `Throwable` (or more specific subtypes of `Throwable`, like `IOException` or a custom exception hierarchy), giving you the guarantee that no errors will be lost during composition.

# 6. Deadlock-Free Async

Both ZIO and Cats IO provide a constructor that allows one to take callback-based code, and lift it into an effect value.

This capability is exposed via the `Async` type class in Cats Effect:

{% highlight scala %}
val effect: Task[Data] = 
  Async[Task].async(k => 
    getDataWithCallbacks(
      onSuccess = v => k(Right(v)),
      onFailure = e => k(Left(e))
    ))
{% endhighlight %}

This creates an asynchronous effect that, when executed, will suspend until the value is available, and then resume&mdash;all transparently to the user of the effect. This property is what makes functional effect systems so pleasing for asynchronous code.

Notice that as the callback-code is being lifted into the effect, a callback function (here called `k`) is invoked. This callback function is provided with the success or error value.

When this callback function is invoked, execution of the (suspended) effect resumes.

ZIO provides the guarantee the effect will resume executing on either the runtime's default thread pool, if the effect has not been locked to a specific context, or on the specific context the effect has been locked to.

Cats IO chose to resume executing the effect on the thread invoking the callback.

The difference between these decisions is quite profound. In general, the thread that is invoking the callback does not expect the callback code to continue indefinitely; it expects a short delay before control is returned to the caller. 

ZIO provides the guarantee that control is returned to the caller immediately, which can then resume execution normally.

On the other hand, Cats IO provides no such guarantee, which means the caller thread invoking the callback may get "stuck" waiting indefinitely for control to be returned to it.

Early versions of Cats Effect concurrent data structures (`Deferred`, `Semaphore`, etc.) resumed effects that did not promptly yield control back to the caller thread. As a result, they had problems with deadlocks and unfair scheduling. While all of these problems have been identified and fixed, they have only been fixed for _Cats Effect_ concurrent data structures.

User-land code that uses a similar pattern with Cats IO will run into similar issues, and because of the nondeterminism involved, they may manfiest only occassionally, at runtime, making diagnosing and solving the issues challenging.

ZIO's model provides deadlock safety and fairness by default, and forces users to opt into the Cats IO behavior explicitly (by, for example, using `unsafeRun` on a `Promise` that is completed from the resumed asynchronous effect).

While neither choice is suitable in all cases, and while both ZIO and Cats IO provide enough flexibility to handle all cases (in different ways), the ZIO choice means worry-free use of `Async`, and pushes problematic code to `unsafeRun`, which is already a known deadlock-risk.

# 7. Precise Future Interop

Dealing with Scala's `Future` is a reality for many code bases. ZIO ships with a `fromFuture` method that provides a ready-made execution context:

{% highlight scala %}
ZIO.fromFuture(implicit ec =>
  // Create some Future using `ec`:
  ???
)
{% endhighlight %}

When this method is used to lift a `Future` into an effect, ZIO can manage where the `Future` is executed, and other methods like `evalOn` will correctly migrate the `Future` to the appropriate execution context.

Cats IO chose to accept a `Future` that has already been constructed with an external `ExecutionContext`. This means that Cats IO has no way of shifting the execution of an embedded `Future` to conform with the semantics of `evalOn` or `shift`. Moreover, it burdens the user of the API to choose an execution context for the `Future`, which means a fixed choice and separate plumbing.

Since one can always choose to ignore the provided `ExecutionContext`, the ZIO choice can be seen as a strict generalization of Cats IO capabilities, providing more seamless and precise interop with `Future` in the common case, but not preventing exceptions to the rule.

# 8. Blocking IO

As covered in [Thread Pool Best Practices with ZIO](/articles/zio-threads), server-side applications must have at least two separate thread pools for maximum efficiency:

 - A fixed thread pool for CPU / async effects
 - A dynamic, growing thread pool for blocking effects

A choice to run all effects on a fixed thread pool will eventually lead to deadlock; while a choice to run all effects on a dynamic, growing thread pool will lead to gross inefficiency.

On the JVM, ZIO provides two operators that provide direct support for blocking effects:

 - The `blocking(effect)` operator, which will shift execution of the specified effect to a blocking thread pool, which uses very good settings and can also be configured;
 - The `effectBlocking(effect)` operator, which translates side-effectful blocking code into a pure effect, whose interruption will interrupt a lot of blocking code.

If you have an effect, and you need to make sure it's executed on a blocking thread pool, then you can wrap it in `blocking`. On the other hand, if you are wrapping some side-effectful code that blocks, then you can wrap it in `effectBlocking`, and benefit from ZIO's composable, pervasive, and safe interruption (where possible).

Cats IO chose to adopt a more minimal core, and delegate such functionality to user-land code. While there are libraries that help provide the functionality of the `blocking` operator, they are based on `evalOn`, and therefore cannot actually guarantee execution on the blocking thread pool.

Power users may very well want to configure their own custom blocking thread pool (which of course, you can do with ZIO), or create more than these two thread pools (for example, a thread pool for low-latency event dispatching), but these operations provide exactly the desired semantics for the vast majority of cases.

# 9. Cost-Free Effects

Many functional Scala applications end up using one or both of the following monad transformers:

 - `ReaderT` / `Kleisli`, which adds the effect of accessing an environment
 - `EitherT`, which adds the effect of typed errors (or `OptionT`, which is a specialization of `EitherT` with `Unit` as the failure type)

The pattern is so pervasive, whole libraries have been designed around one or the other (for example, _http4s_ extensively uses `Kleisli` and `OptionT`).

Using an advanced technique called [effect rotation](/articles/rotating-effects), ZIO provides both the _reader_ and _typed error_ capabilities directly in the `ZIO` data type. 

Because not every user will need reader and typed error capabilities, ZIO also provides a variety of type / companion synonyms that cover common cases. For example, `Task[A]` provides only the core primary capability, without reader or typed errors.

This allows ZIO to provide the two most common (secondary) effects in functional applications without any runtime overhead whatsoever. In addition, supporting these effects directly in ZIO actually _reduced_ the size of its runtime, allowing simpler code and pulling non-essential functionality out of the microkernel.

Cats IO chose to provide just a primary effect. This means that users who need reader or typed errors, or just want hack-free implementations of state, writer, and other type classes, will likely find themselves using monad transformers.

ZIO can be up to 8x faster than Cats IO with an equivalent effect stack. While the impact of effect overhead on application performance will depend on a great many factors, greater performance increases the number of applications for functional Scala, and allows developers to build their applications from fine-grained effects.

# 10. Microkernel Architecture

ZIO utilizes a microkernel architecture, which pulls as much functionality as possible out of the runtime system, and into ordinary user-land code, written in pure functional Scala. Indeed, even parts of the microkernel are itself written in pure functional Scala, utilizing an even smaller core for bootstrapping.

While the original ZIO kernel was roughly 2,000 lines of code, after introducing typed errors and environment, and eliminating redundancy and improving orthogonality, the entire microkernel is now 375 SLOC, in a single file.

As the complexity of modern effect systems in Scala has grown, so has the potential for bugs. There are very few people in the world who understand how these systems work, and the potential for hidden bugs and edge cases is very high.

Personally, I am a fan of microkernel effect systems for the following reasons:

 - The smaller code can be more easily inspected for correctness
 - There are fewer places for bugs and edge-cases to hide
 - It is cheaper, faster, and safer to respond to real world feedback
 - It is easier for new contributors to help out with core maintenance

Monolithic kernels can in theory be much better optimized. However, due to the volunteer nature of open source, we only have finite resources for optimization.

Due to these constraints, often you can either micro-optimize a _part_ of a monolithic kernel, or micro-optimize the _whole_ of a microkernel. The former can give you super high-performance in a few cases, while the latter can give you great performance across a wide range of complex cases.

Of all the effect systems out there, the ZIO runtime is by far the smallest. As a reference implementation, Cats IO comes in second place, but its runtime is at least _twice_ the size of the ZIO runtime (maybe _three_ times, depending on how you count).

# 11. Beginner-Friendly

ZIO has made many decisions to increase usability for new users, without cutting corners or sacrificing principles for advanced users. For example:

 - Jargon-free naming. For example:
   - `ZIO.succeed` instead of `Applicative[F].pure`
   - `zip` instead of `Apply[F].product`
   - `ZIO.foreach` instead of `Traverse[F].traverse`
   - Etc.
 - No use of higher-kinded types or type classes (Cats, Cats Effect, and Scalaz instances are available in optional modules)
 - No implicits that have to be imported or summoned (except for `Runtime`, which must be implicit for all Cats Effect projects, due to the current design of Cats Effect); implicits are a constant source of frustration for new Cats IO users
 - No required syntax classes
 - Auto-complete-friendly naming that groups similar methods by prefix. For example:
   - `zip` / `zipPar`
   - `ZIO.foreach` / `ZIO.foreachPar`
   - `ZIO.succeed` / `ZIO.succeedLazy`
   - etc.
 - Concrete methods on concrete data types, which aids discoverability and traversability, and makes ZIO very usable in IDEs
 - Conversion from all Scala data types to the ZIO effect type
   - `ZIO.fromFuture`
   - `ZIO.fromOption`
   - `ZIO.fromEither`
   - `ZIO.fromTry`
   - etc.
 - Full, out-of-the-box type inference for all data types and methods

Anecdotally, I have seen people with no prior background in functional Scala successfully build prototypes using ZIO without any external assistance, and before there was any good documentation&mdash;unaware that by using ZIO, they were writing purely functional code.

Cats IO chose to delegate most functionality, names, and decisions around type-inference to Cats. This keeps the reference implementation small, but may increase ramp-up time for developers new to functional programming, and result in well-known usability problems around discoverability, naming, implicits, and type inference.

# 12. Batteries Included

In a small, cross-platform package, ZIO provides a highly-integrated toolbox for building principled asynchronous and concurrent applications.

This toolbox includes the following:

 - The most important concurrent data structures, including `Ref`, `Promise`, `Queue`, `Semaphore`, and a small `Stream` for file / socket / data streaming
 - [Software Transactional Memory](https://www.youtube.com/watch?list=PL8NC5lCgGs6MYG0hR_ZOhQLvtoyThURka&v=d6WWmia0BPM) (STM), which can be used to simply build composable, asynchronous, concurrent, and interruptible data structures
 - `Schedule`, which offers composable retries and repetitions
 - Tiny and testable `Clock`, `Random`, `Console`, and `System` services, which are used by nearly every application
 - Many helper methods on the effect type covering common use cases

As a reference implementation, Cats IO has none of these features. This decision makes Cats IO more lightweight, but at the cost of adding more third-party dependencies (where they are available), or having to write more user-land code.

# Summary

Cats Effect has done great things for the Scala ecosystem, providing a growing roster of libraries that all work together.

Application developers who are using Cats Effect libraries now face the difficult decision of choosing which of the major effect types to use with Cats Effect libraries: Cats IO, Monix, or ZIO.

While different people will make different choices that are uniquely suited for them, if you value some of the design decisions described in this post, then I hope you will find that together, ZIO and Cats Effect make a killer combination!