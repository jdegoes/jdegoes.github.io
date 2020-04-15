---
layout:       post
title:        "Why I'm Excited About Scalaz 8"
description:  "Scalaz 8 builds on a long-tradition of pushing the boundaries of pure functional programming in Scala."
category:     articles
tags:         [fp, functional programming, type classes, scala, monads, lenses, effects, reactive, scalaz, cats]
image:
  feature: "scalaz-8-branding.png"
---

Historically, functional programming in Scala has not been for the faint of heart.

The JVM was built for Java, which is object-oriented, not functional, and Scala *allows* functional programming but isn't a *functional-first* programming language (like Haskell or PureScript).

Some things that are easy in other programming languages, like Monad Transformers Library or profunctor optics, are downright tricky in Scala. Doing full-blown functional programming in Scala really pushes the boundaries of what is possible with the language and the JVM platform.

Thankfully, since at least 2010, the Scala ecosystem has had libraries to make functional programming in Scala easier and more practical. The most mature library with [highest adoption](https://www.jetbrains.com/research/devecosystem-2017/scala/) is [Scalaz](https://github.com/scalaz/scalaz).

Scalaz pioneered many of techniques that enable purely functional programming on the JVM, such as trampolined monadic computation. It has a rich history of delivering the most principled approach to problems facing real-world functional programmers.

These, among other reasons, are why my company, [SlamData](https://slamdata.com) chose Scalaz to power the [Quasar analytics compiler](https://github.com/quasar-analytics/quasar/), which is easily the largest purely functional Scala open source application in existence.

The current version of Scalaz 7, while the most feature-complete and widely deployed library for functional programming in Scala, is also showing signs of age. We now know better ways to model some things (like type classes and lenses), and several new abstractions have proven themselves enough to become viable for mainstream use.

Fortunately for commercial Scala developers, Scalaz 8 is under active development, and I'm happy to announce that at the request of several existing contributors, [I've joined the Scalaz 8 development team](https://github.com/orgs/scalaz/people)!

While there's much more work to do before Scalaz 8 is ready for use, I'm *very* excited about its potential. In this post, I'll share a few reasons why you might be, too!

## The Missing Standard Library

The vision for Scalaz has always been to provide a *batteries-included* environment for functional programming in Scala.

This means Scalaz provides standard building blocks that can be used to construct and describe functional solutions. It's a *standard library for FP*, if you will.

Functional programming, in the statically-typed, category-theoretic variant popularized by Haskell, relies on a tool bag of abstractions, such as monads and monoids, as well as essential data types like lists, the effect-encapsulating `IO` structure, and so on.

There's broad consensus on what's in this tool bag, and it includes all of the following (both definitions and implementations for standard types):

 * Data structures, like lists, optional data, and maps.
 * Algebraic structures, like rings and monoids.
 * CT structures, like monads and natural transformations.
 * Optics, like prisms and lenses.
 * Recursion schemes, like catamorphisms and anamorphisms.
 * A low-level effect system with concurrency primitives.

These abstractions and data types don't *solve* all problems, but they allow you to *construct* and *describe* solutions in the rigorous, precise, and powerful way that functional programming is known for.

Scalaz 8 is heading in the direction of providing all these abstractions and data types in one meticulously engineered and fully-integrated library.

You don't need to go one place for your effect system, another for your optics, and a third for your functors and monoids. All these standard abstractions and data structures will be baked-in, beautifully integrated, thoroughly tested, and ready to power functional programming awesome-ness in your applications.

Scalaz 8 is the missing standard library for functional programming in Scala.

### But Modularity!

The opposite strategy to Scalaz 8 is the *modularity strategy*, whereby what
would be a standard library is split up into dozens of libraries that each solve
one part of the overall problem. In the extreme case, for example, there could
be a `scalaz-functor` library that defines what a functor is, as well as `scalaz-functor-syntax` for providing syntax, and so forth.

The primary purpose of modularity is to enable different parts of a whole to evolve independently. For example, if you split up a large business application into different components that have few dependencies between them, then different teams who specialize on the different components can be more independent in how they work.

In practice, however, there are two arguments I find very compelling for taking
the Scalaz 8 approach:

1. Functional programming abstractions and data types are *inextricably intertwined*.
2. Users want functional abstractions to *evolve together* as a unified whole.

Trying to "modularize" abstractions and data types that are inextricably intertwined is an exercise in mind-numbing tedium. For example, we can do a high-level chunking of our FP standard library into different modules, each maintained and versioned independently:

 * **algebraic**
 * **ct**
 * **data**
 * **optics**
 * **schemes**
 * **effects**

Yet our `data` structures will permit instances and combinators that depend on `ct` and `algebraic`. Indeed, some type classes will require specific `data` structures. In theory, we can create new libraries for areas of overlap to maintain modularity:

 * **data-ct**
 * **data-algebraic**
 * ...

However, some `data` functionality will depend on both `ct` and `algebraic`, so we'll need this one too:

 * **data-ct-algebraic**

Data structures can be traversed (they are recursive or corecursive) and we can always define useful optics for them, so we'll need these libraries, too:

 * **data-optics**
 * **data-schemes**
 * **data-optics-schemes**

We're not done yet, since some of the combinators we can write in the above libraries will rely on `ct` and `algebraic`, so we'll need even more libraries for those!

In general, attempting to modularize `n` deeply intertwined abstractions and data types will require `2^n - n` additional libraries to cleanly represent instances and combinators that depend on overlapping functionality.

If we have 10 modules, for example, that means *more than 1,000 libraries*! Modern development stacks, all the way from Github to Scala to the JVM, were *never designed* for such fine-grained dependencies. Ignoring this fact imposes astronomical costs on development that far outweigh the perceived benefits.

Not all of these perceived benefits are actually benefits, either. For example, users *never* want a recent version of one part of the FP standard library (e.g. `optics`), but an older version of another part of the FP standard library (e.g. `ct`). Because of the interdependencies, such cherry-picking doesn't make sense.

Modules can actually *slow down* development and releases, and make the creation of globally consistent version sets very difficult and rare. With 1,000 modules all independently versioned and released, how difficult would it be to find a set of versions that all work with each other? You probably know the answer to that one from experience!

The alternative to this unholy mess is to recognize that the core abstractions and data types in functional programming work best when they have been *intentionally designed* as a consistent, unified, well-integrated whole that is versioned and released together, and backed by a comprehensive suite of automated tests that guarantees the pieces fit together perfectly.

That's what users want, it's what Scalaz 8 appears to be delivering, and it's what functional programming in Scala was always *meant* to be.

## Principled

Scalaz has a long history of eschewing the ad hoc over the principled. Type classes are generally required to have laws. Instances are required to comply with those laws. Hacks that have edge cases are *strongly* discouraged.

From what I have seen so far, Scalaz 8 is raising the bar on what it means to provide a *principled* library for functional programming.

This means learning from some mistakes made in earlier versions of the library, as well as mistakes made in other functional programming libraries.

Concretely, this means you can probably expect all of the following from Scalaz 8:

 * Only one unsafe method (`unsafePerformIO`) in the entire library, which will called automatically as part of any `SafeApp` (the *main* function for purely functional programs). All other functions will be pure and total.
 * No lawless type classes.
 * No dependency on dangerously unsafe data types whose methods are often partial and impure (you can still use them, of course, the library just won't *make* you).
 * No invalid instances, no matter how tempting it might be (for example, Scala's broken  `Future` will not have `Monad` or, worse, `Comonad` instances defined for it).
 * No bogus functions whose laws cannot be satisfied for all valid instances of the type class (for example, `tailRecM` will not be a function on `Monad`, because not all monads can implement it in constant stack space).

Not everyone prefers such a principled approach to functional programming in Scala. Some prefer a generous helping of unsafe convenience methods, lawless type classes, dependencies on common but dangerous Scala data types, downright invalid instances, and type class functions whose laws can't be satisfied for all instances.

The quite understandable argument for all of these things is *pragmatism*&mdash;after all, as programmers, sometimes we *must* interface with legacy code, pound square nails into round holes, break laws and throw all caution to the wind.

Personally, I prefer foundational libraries&mdash;*especially* standard libraries for FP&mdash;*not* cut any corners, no matter how tempting it might be. The principled engineering of a standard library lets you cut your *own* corners if need be, with the confidence that any erratic behavior is not caused by the standard library.

## A Teaser for `scalaz.effect`

My current contribution to Scalaz 8, which has not yet been released but is under active development, is a shiny new *effect system*.

Purely functional Scala programs need something like Haskell's `IO` monad&mdash;a data structure that allows functional programs to model interaction with external, effectful systems in a referentially transparent way.

To date, most effect systems for Scala have fallen into one of two categories: pure, but slow or inexpressive; or fast and expressive, but impure and unprincipled.

What I have wanted to do for a long time is build a solution that's fast *and* principled, and at the same time powerful and expressive enough to build high-performance, concurrent, real-world functional applications.

> If you're going to pay the cost of `flatMap` on every effect in your FP application, then you better get something amazing in return. For functional programmers, we're sold on composability and reasonability. But with scalaz.effect, my goal is to provide so much expressiveness and such strong guarantees that even non-functional programmers will want to switch to purely functional programming.

The result of my efforts is an all-new `IO` data type. Unlike prior versions of `IO` in Scalaz and competing alternatives, this type satisfies the following properties:

 * All methods on `IO` are safe and referentially transparent, and there's no implicit hackery to propagate nasty junk like executor services.
 * `IO` handles both synchronous and asynchronous computation&mdash;application code doesn't need to care, it's just an implementation detail.
 * `IO` has a super fast runtime system that executes `IO` actions in an application's main function. The main interpretive loop has a *zero-allocation* code path for left-associated binds and doesn't touch the (slow) heap any more than strictly necessary.
 * `IO` has a new fork/join concurrency framework that spawns fibers, which can be joined, killed, and supervised (!) in a lawful, deterministic, non-blocking fashion that provides strong guarantees against fiber leaks.
 * `IO` has a bracket primitive for iron-clad guarantees on resource acquisition and release, even in the presence of concurrency and fiber interruption.
 * `IO`'s semantics are compatible with Javascript, paving the way for a Scala.js implementation, which I hope to write shortly after launch of the JVM version.
 * `IO` fibers can be interrupted instantly, at any point in their computation, without any significant performance penalty, ensuring resources aren't wasted.
 * `IO` fibers utilize threads, but in the near future, they will also transparently yield to other fibers in cases of resource starvation (but only then!), allowing fibers to transparently scale well past the limits of native threads.

In addition to `IO`, I've implemented a fast `MVar` that can be used for non-blocking communication between fibers, a lawful type class hierarchy to describe the features of `IO` (including concurrency!), and I plan to implement the first real `STM` for Scala.

For nearly all modern applications, this combination of features more than pays for the cost of purely functional programming, *several times over!* When you can write massively concurrent, non-blocking, fast, and correct programs that don't leak threads or other resources, the relatively minor cost of `flatMap` becomes irrelevant for most apps.

Welcome to the next-generation of purely functional programming in Scala!

### Performance

Performance has been at the top of my mind when developing `scalaz.effect`. While the code is still under development and there are more features to add, at this point I can say Scalaz 8's effect system will be one of the fastest effect systems on the JVM.

The following benchmarks, which are run against the latest versions of Scala's built-in `Future`, Monix' `Task`, and Cats' `IO`, should whet your appetite.

#### Deep Attempt

This benchmark measures the performance of recovery from deeply nested thrown exceptions, when the distance between the throw and the catch is very large.

```
  Benchmark     (depth)  (size)   Mode  Cnt          Score         Error  Units
  -----------------------------------------------------------------------------
  Cats             1000     N/A  thrpt   25      15831.400 ±     228.361  ops/s
  Future        CRASHED
  Monix            1000     N/A  thrpt   25      12269.592 ±      52.478  ops/s
→ Scalaz           1000     N/A  thrpt   25      15954.567 ±      61.707  ops/s
```

### Broad, Deep FlatMap

This benchmark measures the performance of broad and deep binds, as generated from effectful, recursive code.

```
  Benchmark     (depth)  (size)   Mode  Cnt          Score         Error  Units
  -----------------------------------------------------------------------------
  Cats               20     N/A  thrpt   25        126.774 ±       0.333  ops/s
  Future             20     N/A  thrpt   25         15.070 ±       0.670  ops/s
  Monix              20     N/A  thrpt   25       1565.085 ±      29.217  ops/s
→ Scalaz             20     N/A  thrpt   25       1726.595 ±       5.800  ops/s
```

### Repeated Map

This benchmark measures the performance of repeated maps.

```
  Benchmark     (depth)  (size)   Mode  Cnt          Score         Error  Units
  -----------------------------------------------------------------------------
  Cats              500     N/A  thrpt   25        720.798 ±       1.847  ops/s
  Future            500     N/A  thrpt   25       5615.690 ±      82.935  ops/s
  Monix             500     N/A  thrpt   25      47047.947 ±     161.788  ops/s
→ Scalaz            500     N/A  thrpt   25      59869.712 ±     314.230  ops/s
```

### Narrow, Deep FlatMap

This benchmark measures the performance of narrow but deep binds.

```
  Benchmark     (depth)  (size)   Mode  Cnt          Score         Error  Units
  -----------------------------------------------------------------------------
  Cats              N/A   10000  thrpt   25       2058.184 ±      34.483  ops/s
  Future            N/A   10000  thrpt   25         42.750 ±       1.085  ops/s
  Monix             N/A   10000  thrpt   25       6667.112 ±      19.394  ops/s
→ Scalaz            N/A   10000  thrpt   25       7038.373 ±      18.806  ops/s
```

### Shallow Attempt

This benchmark measures the performance of recovery from shallow thrown exceptions, when the distance between the throw and the catch is very small.

```
  Benchmark     (depth)  (size)   Mode  Cnt          Score         Error  Units
  -----------------------------------------------------------------------------
  Cats             1000     N/A  thrpt   25        632.535 ±       5.784  ops/s
  Future        CRASHED
  Monix         CRASHED
→ Scalaz           1000     N/A  thrpt   25        750.508 ±      17.439  ops/s
```

## Scalaz 8: More than Great Code

A unified, batteries-included architecture, a commitment to principled design, and a (hopefully) powerful and correct effect system are just the beginning.

Many more Scalaz resources are currently in the works:

 * I'm [unveiling Scalaz 8's effect system](https://sched.co/BLvT) at [Scale By The Bay](https://sched.co/BLvT). This is one of only about 2 conferences I speak at each year, so if you're around, don't miss it!
 * [Sam Halliday](https://twitter.com/fommil) is writing a book on functional programming for Scalaz, entitled [Functional Programming in Scala for Mortals](https://leanpub.com/fp-scala-mortals). If you're confused by functional programming in Scala or by Scalaz, you'll soon have a fantastic resource to help you become a proficient at FP with the leading Scala FP library.
 * I'll be revamping my popular workshop on *Advanced Functional Programming in Scala*. The new workshop will be titled, *Mastering Functional Programming in Scala with Scalaz 8*, and will help individuals and teams quickly come up to speed on leveraging Scalaz 8 to write robust, performant, real-world software using pure FP.
 * My current plan is to try to get my company's [Matryoshka](https://github.com/slamdata/matryoshka) library folded into Scalaz 8, ensuring that Scalaz has a production-ready package for recursion schemes, which can be tightly integrated into the rest of the standard library (data structures and lenses).

If you're not already doing FP in Scala, there's never been a better time to start learning Scalaz. You'll be well-supported and in great company!

## Summary

Although Scala was never really designed for functional programming, a carefully chosen subset of the language has proven adept at real-world functional programming. What makes this practical is powerful functional programming libraries like Scalaz.

For all the language's quirks and warts, those looking for a well-paying, FP-friendly job will usually find *many opportunities* in the Scala community.

Scalaz 8 is the next saga in a long-standing tradition of innovation, building on the learning and work of countless others. It aims to be the missing standard library for functional programming in Scala, with a rigorous, principled design that serves as the foundation of correct-by-construction business applications.

The next-generation effect system that I've been working on aspires to combine the time-tested design of effect systems in Haskell and PureScript with a renewed emphasis on performance and cleanly solving messy resource and concurrency problems of the real world.

Functional programming in Scala may not be particularly easy, but Scalaz 8 is going to make it *tremendously* simpler, while simultaneously raising the bar in terms of power, precision, composability, and even performance.

Resources for learning and extending Scalaz are multiplying, making it an excellent time for any functional programming holdouts to become familiar with the technology.

These are truly exciting times for the Scala FP community!
