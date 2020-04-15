---
layout:       post
title:        "A Brief History of ZIO"
description:  "A brief history that explains the purpose of ZIO and how it relates to similar libraries in the Scala ecosystem"
category:     articles
tags:         [type classes, haskell, purescript, scala, cats, scalaz, mtl, tagless-final, functional programming, fp, effects, monads, io, cats effect]
---

When I [started](http://degoes.net/articles/scalaz8-is-the-future) what would eventually become [ZIO](https://zio.dev), my goal was to give Scala a better asynchronous effect system, one that had some of the high-end features of Haskell's IO monad, including cancellation of running effects and easy concurrency.

While developing this effect system, I was [teaching](https://gist.github.com/jdegoes/97459c0045f373f4eaf126998d8f65dc) a lot of Scala developers functional programming professionally, covering topics like higher-kinded type classes (the functor hierarchy), monad transformers, free monads, and more. In addition, I was traveling the world speaking to Scala developers everywhere (including non-functional programmers), and listening to their feedback on functional programming, even and especially when hostile.

What I learned from my teaching experience and conversations with developers would radically change my perspective on the practicality of Haskell-style functional programming in Scala.

## Real World Feedback

While live-coding for students, I realized how painful it was to use monad transformers; and how difficult it was to justify all this machinery to do simple things with state and errors.

Together with my students, I recoiled in horror so many times at just how much type inference you had to sacrifice for higher-kinded types.

I struggled to justify to skeptical everyday Scala programmers the tangible benefits that developers get from 'effect-polymorphism' beyond just testability.

I learned, both from my experience and the experience of other developers, just how difficult it is for someone who loves functional programming to sell the benefits to someone who does not love functional programming.

Through these experiences, I came to believe that if functional Scala continued to be _Haskell on the JVM_, it would never be particularly relevant for the broader market of programmers who just want to get stuff done.

These beliefs started to influence the design of ZIO.

## ZIO Evolves

> No plan of operations extends with any certainty beyond the first contact with the main hostile force
> — Helmuth von Moltke the Elder, 1871

Confronted with the often painful and unsatisfying reality of doing Haskell in Scala&mdash;a reality that drove many a functional programmer from Scala to Haskell (notably my friends Tony Morris, Sam Halliday, Emily Pillmore, and many others)&mdash;I decided to adapt:

- Having seen countless developers struggle with `EitherT`, I thought instead of fixing ZIO's error type to be `Throwable`, I could make it polymorphic, and eliminate the poor ergonomics of `EitherT`. I was very happy with the result, because not only was it very pleasant to use, but you could see at compile-time which parts of your code can fail and which parts can't, which I found to be a huge boost to productivity and correctness.
- After a successful small-scale experiment, my friend and co-contributor [Wiem Zine Elabidine](http://twitter.com/wiemzin) and I went through and aggressively refactored ZIO to use declaration-site variance everywhere. It was a massive refactoring, and at the time, it was extremely contentious, resulting in a lot of negative feedback. Scalaz and Cats had a philosophy of _variance is bad, always avoid it_. Yet after we completed the refactoring, it immeasurably increased the pleasure of using ZIO. For the first time, everything just inferred beautifully, without type annotations. Since then, I've even become obsessed with pushing variance to maximize type inference.
 - I started changing names of data types and operators, breaking with decades old Haskell traditions. For example, I renamed `point` / `pure` to `ZIO.succeed`, among many other changes. These changes resulted in a lot of negative feedback from functional programmers, culminating most recently in a lot of negative feedback about `ZIO.foreach`. My belief is not that jargon is useless (it's essential among trained professionals), but that piggybacking on top of even imprecise domain knowledge can improve the onboarding experience for new developers.
 - I started changing documentation to emphasize practical pain points that developers face. I came to believe that no developer has a pain point called "lack of purity". But they do have pain points around asynchronous programming; they have pain points around resource safety; they have pain points around concurrency; they have pain points around testability. ZIO's current marketing focused emerged when the short pitch was changed from _a purely functional IO monad_ to _a library for async & concurrent programming_.

At the same time, ZIO-specific features like fibers and a pure (and automatic) cancellation model started influencing the design of Cats Effect, and I strongly argued for many changes in Cats Effect, most of which made the 1.0 release of the library (indeed, 1.0 was radically different because of these discussions!).

## Divergence 

Typed errors put some distance between the Cats Effect type class hierarchy and ZIO. If you wanted to reason about the error behavior of your programs statically, you could not use the Cats Effect type classes, because the type classes have no support for polymorphic errors.

In addition, a growing number of combinators on the ZIO effect type&mdash;combinators that did not exist in Cats Effect, Cats, or anywhere else&mdash;made it increasingly painful to restrict oneself to the subset of features supported by Cats Effect. 

These growing seeds of divergence led to more direct use of ZIO among early adopters.

## ZIO Environment

Early versions of ZIO only had typed errors. But in late 2018, after struggling in a class with the horrid contortions and complex machinery necessary to test functional programs using final-tagless, I had an idea to exploit contravariance to enable Scala to infer intersecting requirements in the composition of multiple effects.

I prototyped the solution in about a hundred lines of self-contained code, and in a fit of excitement, shared it with [Wiem](http://twitter.com/wiemzin), exclaiming, "This is gonna change things forever!". Unfortunately, the prototype required a third type parameter; but on the flip side, it enabled a way of testing effectful programs that inferred flawlessly and didn't require any type classes, implicits, higher-kinded types, or category theory.

Over the course of two months, [Wiem](http://twitter.com/wiemzin) and I introduced the third "environment" type parameter into the ZIO effect type, in a massive refactoring that spanned thousands of lines of code. This moment was the birth of what we know now as `ZIO`: an effect type capable of propagating errors and successes upward; and propagating dependencies downward&mdash;the two primary needs of every functional program ever written!

I introduced ZIO environment to make testing functional programs easier, which I had come to believe was the only significant, tangible and sellable benefit of tagless-final. However, I quickly discovered that the `R` type parameter, combined with primitives already in ZIO (such as `Ref`), provides a way to model state and writer effects. Thus, there was no need for "MTL" anymore, as the principal transformers (`StateT`, `ReaderT`, `WriterT`, `ExceptT`) were already totally subsumed in the ZIO effect type.

Without explicitly trying, ZIO had become an alternative to MTL and even (optionally) tagless-final: only an alternative with excellent performance (something not possible in Scala with monad transformers), perfect type inference, and minimal knowledge prerequisites. 

ZIO had introduced a different way to do functional programming in Scala. One with no jargon, no type classes, no implicits, no higher-kinded types, and no category theory.

## ZIO & Cats Effect

Despite these achievements, I did not view ZIO as a "competitor" to Cats Effect. I viewed Cats Effect as being primarily about interop between the different effect types through type classes, and I even argued that Cats IO should be a separate project to avoid conflict of interest.

I wrote about the [synergy between the projects](http://degoes.net/articles/zio-cats-effect) and encouraged people to see ZIO as the _best implementation_ for the Cats Effect type classes, because ZIO has improved semantics on Cats IO.

Some of the issues with Cats IO include the following:

- An `evalOn` implementation that does not work across async shifts and does not have proper error behavior;
- A lack of fine-grained control over interruption;
- The creation of "zombie" fibers that permanently hang any fiber that joins them, nullifying resource safety guarantees;
- Async effects that resume on the caller's thread pool;
- Etc.

Despite the synergy, I was quite concerned at the growing difference in expressive power between ZIO and the Cats Effect type classes.

On the one hand, ZIO defined an effect type with polymorphic (rather than monomorphic) errors, and with built-in polymorphic context propagation; together with improved semantics around interruption, error handling, shifting, and much more. But the Cats Effect type classes were rigidly locked to monofunctor effect types, and with increasingly dated semantics beset by confusing edge cases.

## Life After Cats Effect

To narrow the growing gap between ZIO and the Cats Effect type classes, I volunteered to help design the next version of Cats Effect. Toward this end, I sketched out a complete hierarchy and [submitted it for feedback](https://github.com/typelevel/cats-effect/issues/321#issuecomment-449030858), conservatively aiming only to support typed errors, along with fixes for core semantic issues.

Ultimately, however, I was [banned by Typelevel](http://web.archive.org/web/*/https://gist.github.com/djspiewak/39fcf30fc4480abb5096010886558792) in an elaborate and strange [press release](https://typelevel.org/blog/2019/09/05/jdg.html), allegedly for 'annoying' technical debates about the ideal design of the library, and my sketch was mostly rejected (although many elements survived in the eventual proposal for CE3).

The realization that Cats Effect would **never** catch up to ZIO, together with the massive pains of tagless-final, caused a tectonic shift in the market. Pretty soon, developers were building ZIO-specific libraries.

Many of the developers building these libraries first learned functional programming through ZIO. Mostly, they built these libraries without central coordination or planning or training. They just picked up ZIO by themselves and started building things they needed for production deployment.

Over time, it became clear to me that ZIO would have its own ecosystem, with a different focus and approach, and populated by a new and growing group of contributors from outside the historical functional Scala community.

## ZIO Today

Today, ZIO is first and foremost a library to help developers build _modern applications_.

Modern applications are asynchronous and concurrent, and they shouldn't leak resources, they shouldn't deadlock, they shouldn't have race conditions, they should be easily testable, and they should use the Scala type system to catch bugs before they happen. 

ZIO helps you quickly build very powerful and correct applications by playing to the strengths of Scala.

Yeah, ZIO happens to be purely functional. Yeah, it's an alternative to MTL. And yeah, even though it still plays well with Cats Effect libraries, it's increasingly its own thing with its own community and ecosystem.

But the big picture is about leveraging both functional programming and all the features of Scala (even the parts historically eschewed by functional programmers!) to make Scala developers really happy and really productive.

If we do that, the rest is just details.