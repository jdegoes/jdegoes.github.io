---
layout:       post
title:        "Smooth and Easy Testability for Functional Effect Systems"
description:  "Tagless-final has given us testable functional effects, but at great cost. Now ZIO provides an easier way."
category:     articles
tags:         [fp, functional programming, type classes, scala, monads, lenses, effects, reactive, scalaz, cats, mtl, monad transformers, zio]
---

**See my accompanying talk, [The Death of Finally Tagless](https://skillsmatter.com/skillscasts/13247-scala-matters), which was released today and covers ZIO Environment.**

Today's functional effect systems for Scala, such as the [ZIO library](https://github.com/scalaz/scalaz-zio) that I work on, Scala are _incredibly_ powerful.

They provide an effect data type that unifies synchronous, asynchronous, concurrent, and resource effects, and support automatic error propagation across these boundaries.

They're way faster and more powerful than Scala's `Future`, well-documented, reasonably easy to use, and sometimes come equipped with concurrent data structures, a fiber-based concurrency model, and compositional interruption and timeouts for efficient global computation.

They're also purely functional, and showcase the power of pure functional programming to solve modern business problems. 

Unfortunately, there's a dark secret to these functional effect systems: out of the box, they don't live up to the full promise of functional programming. Despite being referentially transparent, they're not really testable.

## Untestable Effects

Functional programming ordinarily gives us the incredible ability to easily test our software.

The reason for this is quite simple: in functional programming, all a function does is map its input to some output. Functions are total (they return an output for every input), deterministic (they return the same output for the same input), and free of side effects (they only compute the return value, and don't interact with the outside world).

Surprisingly to many, these properties also hold for functions that returns functional _effects_. A functional effect, it turns out, is just an immutable data structure that _describes_ an effect, without actually executing it. 

Functional programs construct and compose these data structures together using operations like `map` and `flatMap`, resulting in a data structure that _models_ the entire effectful application. Then in the application's main function, the data structure is translated, step-by-step, into the effectful operations that it describes.

The simplest way to build a functional effect is to _describe_ an effect by using a data structure to store a _thunk_ (a `Function0` in Scala's terminology) that holds an arbitrary hunk of effectful Scala code.

Here's a data type called `IO` which does exactly this:

{ % highlight scala % }
class IO[+A](val unsafeInterpret: () => A) { s =>
  def map[B](f: A => B) = flatMap(f.andThen(IO.effect(_)))
  def flatMap[B](f: A => IO[B]): IO[B] =
    IO.effect(f(s.unsafeInterpret()).unsafeInterpret())
}
object IO {
  def effect[A](eff: => A) = new IO(() => eff)
}
{ % endhighlight % }

Now we can construct pure functions that return functional effects (_models_ of effects) quite simply:

{ % highlight scala % }
def putStrLn(line: String): IO[Unit] = 
  IO.effect(println(line))

val getStrLn: IO[String] = 
  IO.effect(scala.io.StdIn.readLine())
{ % endhighlight % }

These functions are total, deterministic, and free of side effects, because they don't _do_ anything effectful, they merely build a data structure that _describes_ effectful operations.

Using `map` and `flatMap`, we can build describes of whole effectful programs. For example, the following `IO` program asks the user for some input and prints it back out to them:

{ % highlight scala % }
val program: IO[String] = 
  for {
    _    <- putStrLn("Good morning, what's your name?")
    name <- getStrLn
    _    <- putStrLn(s"Great to meet you, $name")
  } yield name
{ % endhighlight % }

Now if you evaluate `program` in the Scala REPL, you'll find that it doesn't actually _do_ anything except construct an `IO` value, which is itself an immutable data structure.

However, you can (non-functionally) interpret this program to the effects that it describes by calling the `unsafeInterpret()` function:

{ % highlight scala % }
program.unsafeInterpret()
{ % endhighlight % }

In this way, while we can't avoid doing something "non-functional" forever, we can at least make the vast majority of our code purely functional, and benefit from increased power of abstraction, refactoring, and testability.

Well, _in theory_. There's a big problem with _testability_. 

In our tests, we need to call functions and verify their outputs match our expectations. Unfortunately, `IO` values, like the above `program` value, cannot be compared to other `IO` values. The reason is that they embed arbitrary hunks of Scala code inside them (functions), and Scala functions cannot be compared for equality.

Although Scala functions do have `equals` and `hashCode`, like all ojbects, these do not have meaningful implementations; they are not based on what the function does, but rather, based on the _reference_ of the constructed object.

An easy way to see this is comparing the values of two `putStrLn` values constructed with the same text output:

{ % highlight scala % }
> putStrLn("Hello") == putStrLn("Hello")
: false 
{ % endhighlight % }

Even though both of these `IO` values represent the _same_ program, Scala cannot know that, because functions cannot be sensibly compared for equality. This is not just a limitation of Scala, but rather a fundamental limitation of computation: in Turing complete languages, we cannot know for sure if two functions are equal, even if we look at their implementations.

This means that while functional effect systems _do_ provide us lots of concrete, tangible benefits (asynchronicity, concurrency, resource-safety, etc.), and while they _do_ give us increased powers of abstraction and refactoring, they don't make it any easier to test effectful code.

In part to solve this problem (and in part to gain a benefit called _parametric reasoning_), some Scala functional programmers have used _tagless-final_, a technique popularized in Haskell.

## Tagless-Final 101

In tagless-final, we often use _type classes_ to model effects (although it's _possible_ to use records, this approach seems not very popular in Scala).

So instead of interacting with `putStrLn` and `getStrLn` directly, we define a type class to describe console capabilities. The type class is _parameterized_ over the effect type:

{ % highlight scala % }
trait Console[F[_]] {
  def putStrLn(line: String): F[Unit]

  val getStrLn: F[String]
}
object Console {
  def apply[F[_]](implicit F: Console[F]): Console[F] = F
}
{ % endhighlight % }

Then we can define instances of this type class for `IO`:

{ % highlight scala % }
implicit val ConsoleIO: Console[IO] = new Console[IO] {
  def putStrLn(line: String): IO[Unit] = 
    IO.effect(println(line))

  val getStrLn: IO[String] = 
    IO.effect(scala.io.StdIn.readLine())
}
{ % endhighlight % }

Now we write programs that are _polymorphic_ in the effect type, which express which capabilities they require from the effect by using type class constraints (commonly modeled using context bounds, which desugar to implicit parameter lists):

{ % highlight scala % }
def program[F[_]: Console: Monad]: F[String] = 
  for {
    _    <- Console[F].putStrLn("Good morning, what's your name?")
    name <- Console[F].getStrLn
    _    <- Console[F].putStrLn(s"Great to meet you, $name")
  } yield name
{ % endhighlight % }

Since this program is _polymorphic_ in the effect type, you can _instantiate_ it to any concrete data type (such as `IO`) that supports its required capabilities. For example:

{ % highlight scala % }
val programIO: IO[String] = program[IO]
{ % endhighlight % }

(This assumes a sutiable instance of some `Monad` type class has been defined for `IO`, which is required because Scala's `for` comprehension desugars to `map` and `flatMap`.)

Once all this machinery is in place, it becomes fairly straightforward to define a data type just for testing:

{ % highlight scala % }
case class TestData(input: List[String], output: List[String])
case class TestIO[A](run: TestData => (TestData, A)) { s =>
  def map[B](f: A => B): TestIO[B] = flatMap(TestIO.value(_))
  def flatMap[B](f: A => TestIO[B]): TestIO[B] = 
    TestIO(d => 
      (s run d) match { case (d, a) => f(a) run d })
}
object TestIO {
  def value[A](a: => A): TestIO[A] = TestIO(d => (d, a))
}
{ % endhighlight % }

With this test data type, you can define an instance of the `Console` type class that simply pulls lines of input from the test data, and writes lines of output to the test data (left as an exercise for the reader!).

Once you define this and the `Monad` instance, you can instantiate the polymorphic program to the test effect:

{ % highlight scala % }
val programTest: TestIO[String] = program[TestIO]
{ % endhighlight % }

Finally, at long last testability has been regained: you can write fast, deterministic unit tests that thoroughly test your application logic. Your CI builds will complete quickly and you can refactor with confidence.

Unfortunately, this benefit comes at considerable cost.

## The Dark Side of Tagless-Final

The tagless-final approach is robust, and many people are quite happy using the technique to build production business applications. However, the technique suffers from a number of drawbacks, each explored in the sections that follow.

### Massive Ramp-Up

As demonstrated in this article, the tagless-final technique is not for the faint of heart. It requires advanced knowledge of the Scala programming language, functional programming, and how we model some functional constructs in Scala.

In particular, to competently use tagless-final in all common scenarios, you will have to understand:

1. Functional Effects.
2. Parametric Polymorphism.
3. Higher-kinded Types.
4. Type Classes & their Scala encoding.
5. Type Class Instances & their Scala encoding.
6. Partial Type Application (AKA _Type Lambdas_).
7. The Monad Hierarchy.

These are not topics that one co-worker can casually introduce to another co-worker over a lunch break. It's not possible to sneak tagless-final into a code base. Some combination of training and / or mentorship are required.

### Type Class Abuse

Although you don't have to use type classes for tagless-final (indeed, the earliest encoding used ML, but type classes were used in the seminal _Finally Tagless_ paper), it's overwhelmingly common to do so in the Scala community.

The reason is that type classes give you nicer syntax and help you thread the (many) constraints throughout your application. 

Unfortunately, this is an abuse of the concept of a type class. A type class, fundamentally, is an _abstraction_. It lets us talk about the ways in which data types are similar, by describing those similarities with _algebraic laws_. 

These algebraic laws let us write generic code across many different data types that share a mathematically-precise definition of _similar structure_, making our functional code principled in a way that ad hoc polymorphism is not.

Tagless-final type classes do not, in general, have algebraic laws. Most have no laws at all. This represents a serious abuse of the construct of a type class and an impediment for teaching type classes to Scala developers.

### Big Bang

If we wish to use tagless-final to test a method deep inside our code base, a method which uses `Future` or maybe `IO`, then we cannot make a small series of rote changes.

Instead, we have to perform a "big bang" style refactoring, which involves a commitment to tagless-final and a lot of work to obtain testability for a single method.

_Big bang_ refactoring can improve a code base, but it is often at odds with the needs of shipping software. It's friendlier to the business if we can make changes incrementally and pay only for what we need today.

### Tedious Repetition

Constraints on type classes are propagated with implicit parameter lists. Context bounds provide a more compact syntax for implicit parameter lists, but when a method that's polymorphic in an effect requires a lot of different type classes, it can still be unwieldly:

{ % highlight scala % }
ef genFeed[F[_]: Monad: 
  Logging: UserDatabase:         
  ProfileDatabase: RedisCache: 
  GeoIPService: AuthService: 
  SessionManager: Localization:   
  Config: EventQueue: Concurrent:   
  Async: MetricsManager]: F[Feed] = ???
{ % endhighlight % }

Unfortunately, if you are following functional programming best practices, and pushing dependencies to the edges, requiring as little as possible from every method, then you will find yourself engaging in tedious repetition of similar lists of context bounds:

{ % highlight scala % }
def cacheFeed[F[_]: Monad: 
  Logging: UserDatabase:         
  ProfileDatabase: RedisCache:  
  Config: EventQueue: Concurrent:   
  Async: MetricsManager](feed: Feed): F[Unit] = ???
{ % endhighlight % }

Some developers try to work around this tedium by creating "module" classes that declare the same set of dependencies for every method inside the module&mdash;even if many methods require less than the full set of constraints across all methods.

This technique makes it easier to deal with the tedium, but at the cost of overly constraining methods and making weakening so-called _parametric reasoning_.

### Stubborn Repetition

Not only is there a lot of repetition in tagless-final programs, but this repetition proves stubborn to abstraction.

Ideally, if we have two methods with the same set of type class constraints, we'd like to be able to create _something_ to represent that set of constraints, and then use it to remove the duplication across the two methods:

{ % highlight scala % }
def method1[F[_]: AllConstraints] = ???

def method2[F[_]: AllConstraints] = ???
{ % endhighlight % }

Unfortunately, Scala does not have any mechanism to abstract across duplicated parameter lists. 

So not only is the repetition quite tedious, but it's unavoidable, due to limitations in the Scala programming language.

### Completely Uninferrable

One of the reasons writing Haskell or PureScript is so exceedingly pleasant is the universal and flawless type inference. Scala has enough type inference to make it a joy compared to Java, but many types in Scala cannot be infered (full inference for a higher-kinded type system in the presence of subtyping is still research-grade).

We would love to be able to take advantage of type inference for tagless-final programs, writing the equivalent of:

{ % highlight scala % }
def genFeed = ...
{ % endhighlight % }

Unfortunately, the type class constraints cannot be inferred, even in theory, because they are not actually type parameters, but values in an implicit parameter list (in a language in which anything can be implicit), and asking any compiler to infer arbitrary implicit parameter lists is unreasonable.

Not only do we have to type out the full list of constraints every time, but if we get the constraints wrong, the error messages will fail with non-obvious "implicit not found" or "method not found" errors.

The lack of full type inference for tagless-final programs makes writing them an exercise in discipline and self-control, and raises the knowledge and skill barrier for becoming proficient in writing programs in this style.

### Fake Parametric Guarantees

An often-touted benefit of tagless-final is that we it provides us _parametric reasoning_.

This claim is not without merit. For example, if we look at the following method signature, we should be able to tell from its type that it works with any effect that provides `Monad`, and is therefore free of effects (it may only use `Monad` operations, such as `map`, `flatMap`, and `ap`):

{ % highlight scala % }
def innocent[F[_]: Monad]: F[Unit] = ???
{ % endhighlight % }

However, since Scala does not restrict procedural effects, this means that we can embed them anywhere, even in supposedly pure code like this:

{ % highlight scala % }
def innocent[F[_]: Monad]: F[Unit] = {
  println("What guarantees?")

  Monad[F].point(())
}
{ % endhighlight % }

Worse still, it is trivial to write a helper method that can embed _any_ effect into any `Applicative`:

{ % highlight scala % }
def effect[F[_]: Applicative](a: => A): F[A] = 
  Applicative[F].point(()).map(_ => a)
{ % endhighlight % }

We may not use this helper method to further contaminate the original definition of `innocent`:

{ % highlight scala % }
def innocent[F[_]: Monad]: F[Unit] = {
  println("What guarantees?")

  effect(System.exit(42))
}
{ % endhighlight % }

In a large code base, whatever _can_ happen, _will_ happen. 

As a testament to this fact, it is common practice among new users of effect systems to accidentally embed effects inside the functions they pass to `map` and `flatMap` (in fact, some effect monads _encourage_ this anti-pattern), as well as embed effects inside lazy versions of the `Monad` `point` operation.

The benefits of _parametric reasoning_ do apply to tagless-final programs, but only "up to discipline". Yet, a lot of other techniques with fewer drawbacks _also_ provide reasoning benefits _up to discipline_.

### Summary of Tagless-Final

Tagless-final does have benefits and experienced functional programmers have deployed many production-worthy applications using the technique.

However, looking at all these drawbacks, it's hard to recommend tagless-final for most Scala shops. 

In my opinion, the technique will never go mainstream, and because of all the machinery and ceremony involved, encouraging tagless-final may push more people away from functional programming in Scala than it lures in.

As I have long argued, some techniques that work well in other programming languages (like monad transformers in Haskell), simply don't work well in Scala. We can't make them work, and nor do we need to, because we can find other techniques that give us similar benefits without the costs.

In the next section, I present one such technique that I believe is exceptionally well-suited for Scala.

## Discarding the Extraneous

If testability is our primary concern, then it's possible we can take a page from Java. If we want to write testable code in Java, then we use interfaces, and we provide different implementations for live and test scenarios.

In the case of our preceding example, we can create a simple Scala trait to represent console capabilities:

{ % highlight scala % }
trait Console {
  def putStrLn(line: String): IO[Unit]

  val getStrLn: IO[Unit]
}
{ % endhighlight % }

This is just an ordinary interface. The only difference is that the methods return functional effects. They don't actually _do_, they only _describe_.

It's easy to teach this to Scala developers, because they probably have used interfaces in Scala and whatever programming languages they knew before Scala.

Now our program, which requires console capabilities, can simply accept `Console` as a parameter:

{ % highlight scala % }
def program(c: Console): IO[Unit] = 
  for {
    _    <- c.println("Good morning, " +
                      "What is your name?")
    name <- c.readLine
    _    <- c.println(s"Good to meet you, $name!")
  } yield ()
{ % endhighlight % }

We can provide either test or production instances of `Console`, ensuring we can reliably test our program.

This technique works reasonably well for tiny programs, but most programs will require more than one service. If you try to scale this technique up, it becomes quite unpleasant:

{ % highlight scala % }
def program(s1: Service1, s2: Service2,
            s3: Service3, … sn: ServiceN) =
  for {
    a <- foo(s1, s9, s3)("localhost", 42)
    b <- bar(sn, s19, s3)(a, 1024)
    ...
 } yield z
{ % endhighlight % }

The pain results from us having to thread `n` services into our methods, and then manually pass subsets of these services into all of the methods that we call.

Indeed, this is the pain that _dependency injection_ was invented to solve. It should not be surprising if we take a more object-oriented approach to solving the testability problem, we will end up in dependency injection territory.

Fortunately, using the _module pattern_, we can at least make steps toward something usable.

### The Module Pattern

The module pattern involves placing our services inside a module trait to provide easier composition. Sometimes this pattern can be identified by the `HasXYZ` naming convention&mdash;for example, `HasConsole`.

To use this pattern, you first define a module, which contains a single field with the appropriate service type:

{ % highlight scala % }
trait HasConsole {
  def console: ConsoleService
}
{ % endhighlight % }

Then you define the service type as normal:

{ % highlight scala % }
trait ConsoleService {
  def putStrLn(line: String): IO[Unit]

  val getStrLn: IO[Unit]
}
{ % endhighlight % }

Now personally, to avoid extraneous typing, I prefer to choose a simplified naming convention and organizational style: I use a shortname for the module, and put the service definition inside the companion object of the module.

For example:

{ % highlight scala % }
trait Console {
  def console: Console.Service
}
object Console {
  trait Service {
    def putStrLn(line: String): IO[Unit]

    val getStrLn: IO[Unit]
  }
}
{ % endhighlight % }

In any case, with the module pattern, we are now able to take advantage of _intersection types_ to compose multiple modules into a single module.

### Module Composition

Scala 3 has first-class support for intersection types. But in the meantime, we can use the `with` operator, which provides pseudo-intersection types.

The `with` operator enables us to create a type that must satisfy multiple requirements. In our case, we can use it to create a module that contains many services.

{ % highlight scala % }
def program(s: Module1 with Module2 ... with ModuleN) =
  for {
    a <- foo(s)("localhost", 42)
    b <- bar(s)(a, 1024)
    ...
  } yield z
{ % endhighlight % }

Notice the _dramatic_ reduction in the amount of work necessary to thread services throughout our application.

Our method now takes a single service parameter, which, using intersection types, bundles together all of its module dependencies into a single module. 

Further, when we pass service dependencies down the stack, we can simply pass the bundle, because due to Scala's support for subtyping, we are free to pass methods _more_ than they require (this is called _contravariance_).

While a satisfying improvement over the predecessor, there's still the painful passing of a single parameter all the way from the top of our application to the bottom.

Fortunately, functional programming provides an extremely simple and elegant solution to this problem.

### The Reader Monad

The `Reader` monad is a monadic data structure that can be used to automated passing an environment from one level in the application down to lower levels.

Every level of the application has access to the environment, and they can even do local modifications (the environment can vary at each level of the application, if so desired).

The `Reader` monad is explained elsewhere, so I won't go into depth on how it works, but I'll present a simple reference implementation:

{ % highlight scala % }
case class Reader[-R, +A](provide: R => A) { self =>
  def map[B](f: A => B) = flatMap(a => Reader.point(f(a)))
  def flatMap[R1 <: R, B](f: A => Reader[R1, B]) =
    Reader[R, B](r => f(self.provide(r)).provide(r))
}
object Reader {
  def point[A](a: => A): Reader[Any, A] = Reader(_ => a)
  def environment[R]: Reader[R, R] = Reader(identity)
  def access[R, A](f: R => A): Reader[R, A] = 
    environment[R].map(f)
}
{ % endhighlight % }

A `Reader[R, A]` is an effect that requires environment `R` and produces a value of type `A`. 

So, for example, a `Reader[Config, String]` is an effect that requires a `Config` and produces a value of type `String`. To extract the `String` from the `Reader`, you first have to _provide_ the `Config` that it requires:

{ % highlight scala % }
case class Config(serverName: String, port: Int)

val serverName: Reader[Config, String] = 
  Reader.access[Config](_.serverName)

val name = serverName.provide(Config("localhost", 43))
{ % endhighlight % }

Now a `Reader[Any, A]` means that the effect can work with _any_ environment. This is equivalent to saying it requires _no_ environment. You can extract values from these type of effects by supplying any value at all (for example, unit):

{ % highlight scala % }
val tempFile: Reader[Any, String] = 
  Reader.point("/tmp/tempfile.dat")

val file = tempFile.provide(())
{ % endhighlight % }

Note that the `Reader` definition above only models _reader_ effects, not effects like input / output, which were modeled by our previous `IO` data type.

However, if we ignore that fact, then if one squints hard enough, one can see a path forward to a final simplification of the module pattern, which uses the `Reader` monad to pass modules:

{ % highlight scala % }
def program: Reader[Module1 with ... with ModuleN, String] =
  for {
    a <- foo("localhost", 42)
    b <- bar(a, 1024)
    ...
  } yield z
{ % endhighlight % }

This step is not so far away. There is a `Reader` monad transformer that can add the reader effect to any base monad, including the `IO` monad. However, not only are monad transformers very slow in Scala (adding 2-4x overhead per layer), but they have clumsy ergonomics and bad type inference.

So instead, using a technique called [effect rotation](/posts/effect-rotation), we can bake the reader effect into the base effect monad, yielding a data type that is high-performance, and, if we are _very thoughtful_ in the design of the data type, opening the door to _delightful_ ergonomics and _flawless_ type inference.

This is the approach taken by _ZIO Environment_, a new feature in ZIO and quite possibly the most defining feature of the impending 1.0 release of the ZIO library.

## ZIO Environment

ZIO Environment uses a functional effect data type with three type parameters:

{ % highlight scala % }
ZIO[R, E, A]
{ % endhighlight % }

The interpretation of these type parameters is as follows:

 - `R`&mdash;This is the type of the environment required to run the effect, which can range from a bundle of modules, to just some configuration details, to `Any` (indicating no requirement).
 - `E`&mdash;This is the type of error the effect may fail with, which can range from `Throwable`, to a custom data type (which may or may not extend `Throwable` / `Exception`), to `Nothing` (indicating the effect cannot fail).
 - `A`&mdash;This is the type of value the effect may succeed with, which can be anything, but if the effect runs forever (or runs until error), it could also be `Nothing`.

Not everyone may be comfortable using the full ZIO data type, so the library defines three type synonyms for common cases:

{ % highlight scala % }
type UIO[+A] = ZIO[Any, Nothing, A]
type Task[+A] = ZIO[Any, Throwable, A]
type IO[+E, +A] = ZIO[Any, E, A]
{ % endhighlight % }

The meaning of these types is as follows:

 - `UIO`&mdash;Unexceptional effect, which doesn't require any specific environment and cannot fail.
 - `Task`&mdash;An effect that doesn't require any specific environment and can fail with any `Throwable`.
 - `IO`&mdash;An effect that can fail with an `E`.

 All of these type aliases have companion objects, which can be used to construct values of these types. For example, `Task.succeed(42)` constructs a `Task[Int]`, which of course is really a `ZIO[Any, Throwable, Int]`.

 This hierarchy of power allows users to start with `Task` and possibly `UIO` (any type they handle errors, they'll get something that has type `UIO`), and then gradually migrate to either `IO` or `ZIO`, or maybe their own type alias that uses offers a combination of types suited to their application.

 In this post, I won't talk about the `E` and `A` parameters, since you can find [previous material](/posts/bifunctor-io) on these, and [ZIO itself](https://github.com/scalaz/scalaz-zio), including Scaladoc and the microsite, have extensive documentation on failure and success values.

 Rather, I'll focus on a few methods that help you use the new `R` type parameter, and then we'll take a look at how we can use these methods to solve the testability problem.

 ### Core Environment

ZIO Environment just adds two new primitive functions (and then a couple helpers based on these):

{ % highlight scala % }
sealed trait ZIO[-R, +E, +A] {
  ...
  def provide(environment: R): ZIO[Any, E, A] = ...
  ...
}
object ZIO {
  ...
  def accessM[R, E, A](f: R => ZIO[R, E, A]): ZIO[R, E, A] =  
    ...
  def access[R, E, A](f: R => A): ZIO[R, Nothing, A] =
    accessM(ZIO.succeed(_))
  def environment[R]: ZIO[R, Nothing, R] = access(identity)
}
{ % endhighlight % }

The core functions are `ZIO#provide`, which allows you to "feed" an `R` to an effect that requires an `R`, to eliminate its requirement; by changing the environment type parameter to `Any`); and `ZIO.accessM`, which allows you to effectfully access part of the environment.

The helper functions are `ZIO.access`, which allows you to (non-effectfully) access part of the environment, and `ZIO.environment`, which gives you the whole environment.

To really understand how the core methods can help us solve the testability problem, let's revisit the console example, this time using ZIO Environment.

### Console Environment

To make our console program testable, we're going to start out defining a module and associated service class. We've seen these before, and there are no substantial changes this time around:

{ % highlight scala % }
import scalaz.zio._

trait Console { def console: Console.Service }
object Console {
  trait Service {
    def println(line: String): UIO[Unit]
    val readLine: IO[IOException, String]
  }
  trait Live extends Console.Service {
    import scala.io.StdIn.readLine

    def println(line: String) = 
      UIO.effectTotal(scala.io.StdIn.println(line))
    val readLine = 
      IO.effect(readLine()).refineOrDie(JustIOException)
  }
  object Live extends Live
}
{ % endhighlight % }

Note that in the console service, the `println` function returns a `UIO[Unit]` (because it cannot fail), while the `readLine` function returns an `IO[IOException, String]`, because it might fail because of an `IOException`. 

If you wanted to be less precise, but also eliminate the need to think about the types, you could just use `Task` everywhere, which is more familiar to Scala developers who have used `Future` and don't yet think about typed errors.

The `Console` companion object holds an implementation of the `Live` version, while a test implementation of the `Console.Service` interface could live inside a test package.

Notice how there are no polymorphic types, no higher-kinded types, no type classes, no type class instances, no implicits, and no monads. This is literally just an interface and implementation, where the methods return functional effects.

The next step is to define a few helper functions, to make using the module easier. This step isn't necessary, but it's convenient, so I'll show the technique:

{ % highlight scala % }
package object console {
  def println(line: String): ZIO[Console, Nothing, Unit] =
    ZIO.accessM(_.console println line)

  val readLine: ZIO[Console, IOException, String] =
    ZIO.accessM(_.console.readLine)
}
{ % endhighlight % }

This package object, which I called `console`, defines `println` and `readLine` functions that return functional effects. These functional effects are defined by using `ZIO.accessM`, which gives us access to any set of modules we want. In this case, we just need the `Console` module, which is reflected in the return types.

Using these helper functions, we can now build our purely functional ZIO program:

{ % highlight scala % }
import console._

val program: ZIO[Console, IOException, String] =
  for {
    _    <- println("Good morning, what is your name?")
    name <- readLine
    _    <- println(s"Good to meet you, $name!")
  } yield name
{ % endhighlight % }

Again notice the simplicy of this definition. Without any of the final tagless machinery, a basic understanding of functional effects and `for` comprehensions is all that's necessary to write code like this.

Now when we need to unsafely interpret this data structure into the effect that it represents, we will generally first provide its required environment using the `ZIO#provide` method. Since this effect only requires `Console`, and since we have already written an implementation in `Console.Live`, we can easily provide our program its production environment:

{ % highlight scala % }
val programLive: IO[IOException, String] = 
  program.provide(Console.Live)
{ % endhighlight % }

Notice the use of the type synonym `IO[IOException, String]`, which of course expands to `ZIO[Any, IOException, String]`, indicating our effect no longer requires any specific environment.

We're now ready to run our program, which we can do with the default runtime system in ZIO:

{ % highlight scala % }
DefaultRuntime.unsafeRun(programLive)
{ % endhighlight % }

It's nearly as easy to test our program. All we have to do is construct an implementation of the `Console.Service` interface for testing:

{ % highlight scala % }
object TestConsole extends Console {
  val console: Console.Servie = ...
}
{ % endhighlight % }

Now we can run the same program using our test service:

{ % highlight scala % }
val programTest = program.provide(TestConsole)
DefaultRuntime.unsafeRun(programTest)
{ % endhighlight % }

That's all there is to ZIO Environment! With just two primitives (`provide` and `accessM`), and an additional type parameter, we're able to completely solve the testability problem in a way that requires only a tiny fraction of the knowledge and skills of tagless-final.

But it gets even better than this!

### Delightful Functional Effects

Tagless-final had a number of drawbacks beyond just a massive ramp up curve. In the next few sections, we'll look at how ZIO Environment stacks up against tagless-final.

#### Composable

Like tagless-final, ZIO Environment is composable: we can compose requirements horizontally, using the `with` operator for type intersection:

{ % highlight scala % }
trait Console { def console: Console.Service }
trait Logging { def logging: Logging.Service }
trait Persistence { def persistence: Persistence.Service }
...

val program: ZIO[Console with Logging with Persistence,
                 ProgramError, Unit] = ...
{ % endhighlight % }

#### Performant

Like tagless-final, but to an even greater extent (because all ZIO methods are monomorphic), ZIO Environment is high-performance.

If you tried to emulate the ZIO Environment technique in another effect type, using the `ReaderT` monad transformer, then you would suffer as much as a 4x performance penalty. If you tried to emulate both ZIO Typed Errors as well, using an `EitherT` monad transformer, you could suffer as much as an 8x performance penalty.

Thanks to effect rotation, ZIO gives you the benefits of the reader and either monad transformers, without any of the cost, and with far better ergonomics and type inference.

#### Fully Inferable

As an experienced functional programmer who has used and generally likes tagless-final, I personally find one of the most compelling benefits of ZIO Environment to be _type inference_.

Thanks to a careful design and appropriate use of variance, ZIO is fully inferable (!). In fact, as far as I'm aware, it's the _only_ approach to testable functional effects in Scala that's demonstrated to be fully inferable.

This means if you use many different modules, you can call functions from all the modules, and the Scala compiler will infer the proper environment.

As an example:

{ % highlight scala % }
val program =
  for {
    _    <- putStrLn("Good morning, what is your name?")
    name <- getStrLn
    _    <- savePreferences(name)
    _    <- log.debug("Saved $name to configuration")
    _    <- putStrLn(s"Good to meet you, $name!")
  } yield ()
{ % endhighlight % }

In this case, Scala will infer the environment to be `Console with Persistence with Logging`.

Not only can Scala infer the type, but if you give an explicit type annotation, but it's incorrect, the hints that Scala provides will eventually lead you to the correct type signature.

Even if you believe in providing top-level type signatures, being able to infer local signatures, have your IDE insert the top-level signatures, or just ask Scala for the corret type (by intentionally inserting the wrong type) is a tremendous benefit to productivity and makes working with ZIO effects an extremely pleasant experience.

#### Concise

With full inference, ZIO can be extremely concise. However, inference is actually not necessary for concision.

Because ZIO Environment uses a type parameter, and because Scala has type aliases, this means we can eliminate all duplication in method signatures:

{ % highlight scala % }
trait Console { def console: Console.Service }
trait Logging { def logging: Logging.Service }
trait Persistence { def persistence: Persistence.Service }
...

type ProgramEnv = Console with Logging with Persistence

val program1: ZIO[ProgramEnv, AppError, Unit] = ...

val program2: ZIO[ProgramEnv, AppError, String] = ...
{ % endhighlight % }

In fact, if we want to create a custom effect type, with our own environment and error type, it's easy to do that too:

{ % highlight scala % }
type Program[A] = ZIO[Console with Logging with Persistence, 
                      AppError, A]

val program1: Program[Unit] = ...
val program2: Program[String] = ... 
{ % endhighlight % }

Type synonyms like this, especially when combined with associated companion objects, can make it possible for beginners to rapidly become productive in a large code base. Further, they enable beginners and experts alike to avoid repeating themselves, which makes code maintenance easier, less costly, and more predictable.

#### Modular

With ZIO Environment, there is no need to build up a monolothic environment. Rather, individual layers of the application can supply local environments to lower layers. 

An example of this technique is shown below:

{ % highlight scala % }
def fn1: ZIO[R1, E, A] = {
  def fn2: ZIO[R2, E, B] = ...

  val localEnvironment: R2 = ...
  val v1 = fn2.provide(localEnvironment)
  ...
}
val globalEnvironment: R1 = ...
val v2 = fn1.provide(globalEnvironment)
...
{ % endhighlight % }

This is only one technique to provide vertical modularity. Over time, other techniques may emerge.

Achieving modularity with tagless-final is possible, but quite difficult and hacky, relying on creating local type class instances. In comparison, modularity is something that other approaches like `Free` and MTL excel at.

#### Incremental

Unlike tagless-final, a code base that uses no abstraction, which is merely using some base functional effect like `Task`, can be modified every-so-slightly to allow testability of something deep in the stack.

For example, let's say we have a database call deeply embedded everywhere inside our program:

{ % highlight scala % }
// Deeply nested code:
def myCode: Task[Unit] = …
  for {
    ...
    result <- database.query(q)
    ...
  } yield ()
{ % endhighlight % }

We would like to be able to test application logic without connecting to a real database, because that will slow our tests down and may fail for unrelated reasons.

In order to do this, we need merely refactor the `database.query` function to require a `Database` module. Then with simple introduction of a type synonym, we can leave the code unchanged:

{ % highlight scala % }
type TaskDB[A] = ZIO[Database, Throwable, A]
...
def myCodeV2: TaskDB[Unit] = …
  for {
    ...
    result <- database.query(q)
    ...
  } yield ()
{ % endhighlight % }

All of the other code can stay exactly the same as it is. The only change we needed to make was the type synonym (which we could have called `Task`, if we didn't want to update the type signatures at all), and the single method we wanted to make testable.

In an ideal world, everything would always be 100% testable; and if we needed to make legacy code testable, we would have the resources necessary to make _all_ the effects testable. 

Yet in the real world, we often don't have the time or luxury of making our greenfield code _fully_ testable from day one; _or_ of doing giant refactorings to legacy code.

ZIO Environment lets us make _pinpoint_ changes, and pay for _only_ the cost of testing what we _need_ to test today. As a result, it helps us deal with real world code bases and meet the needs of the business.

## Summary

Functional effects can be enormously beneficial to solving modern business problems. Yet as we've seen in this approach, because of the way functional effects are implemented, we don't gain all the beneifts of pure functional code.

While functional effects give us the ability to abstract over our programs and to refactor them without changing their meaning, we can't easily test functional effects, because we don't have a way to compare two effects for equality.

Solutions like tagless-final help us re-introduce testability into our functional applications (along with other benefits, like parametric reasoning). However, they come with a massive ramp up curve, they don't integrate well into Scala, and their ergonomics, boilerplate, and ceremony can be unpleasant and further alienating to developers.

The new approach pioneered in ZIO Environment allows us to regain testability, but without any additional ramp up time(beyond the ramp up required for functional effects). It's friendly to beginning functional programmers, and unlike tagless-final, the new approach is fully inferable, modular, and can be used incrementally, just where we need it.

For the first time, it feels like Scala has an idiomatic solution for testable functional effects. Something that's fast, fully inferable, with a low barrier to entry.

If you'd like to give it a try, head over to the [ZIO project page](https://github.com/scalaz/scalaz-zio), where you will find the [ZIO microsite](https://scalaz.github.io/scalaz-zio/)and the [Gitter chatroom](https://gitter.im/scalaz/scalaz-zio). 

As of today, the first release candidate (RC) for ZIO 1.0 has been published, which means a (nearly) stable API and a focus on documentation, polish, and performance. It's my hope that ZIO 1.0 will be released sometime in March, and that the 1.x line will enjoy at least a full year of backward-compatible tweaks, fine-tunings, and enhancements to the microsite.

If you're still using `Future` and on the fence about a functional effect system, now's the perfect time to jump in and give ZIO (or one of the other functional effect systems) a try. You might just find you can't live without one!

**P.S.** A huge thanks to [Wiem Zine Elabidine](https://twitter.com/wiemzin) for her work on ZIO Environment, and to [Itamar Ravid](https://twitter.com/itrvd), [Regis Kuckaertz](https://twitter.com/regiskuckaertz), and [Kai](https://twitter.com/kaidaxofficial) for their early feedback on the ZIO Environment project, and to [SkillsMatter](https://twitter.com/skillsmatter) for the opportunity to present this work at Scala Matters, London.