---
layout:       post
title:        "Scala Resurrection"
description:  "Scala's market may not grow any larger unless we take action to address the challenges the language faces"
category:     articles
tags:         [scala, scala3, tooling, ides, functional programming, llvm, native, wasm, compiler, zio, ziverge]
---

As a long-time user and one of the earliest evangelists of the Scala programming language, and as CEO of a [tech company](https://ziverge.com) that has [heavily invested](https://zio.dev) in Scala open source, I've been [acutely interested](https://www.youtube.com/watch?v=v8IQ-X2HkGE) in growing Scala adoption within industry.

Scala 3 has the most powerful static type system and metaprogramming facilities in any major programming language. The language's strength in designing robust internal domain-specific languages like Spark has opened new doors into [concurrent programming](https://zio.dev), [distributed programming](https://github.com/zio/zio-flow), [database access](https://github.com/zio/zio-quill), and many other domains.

Yet, not all is sunshine and roses in the world of Scala:

 - The Scala 3 revision of the language has proven every bit as challenging as I [anticipated](https://www.youtube.com/watch?v=v8IQ-X2HkGE) it would be.
  - The tooling around Scala is worse now than in the 2.x days, and poor compared to even much newer languages like Go, Kotlin, and Rust. 
  - Critical platform and tooling projects in the Scala ecosystem remain underfunded and underdeveloped.
  - Scala has completely intertwined its fate with the JVM, which is under constant threat from more modern alternatives.
  - Lightbend has effectively close sourced the entire Akka ecosystem.
  
These factors, combined with the scarcity and high cost of senior Scala talent, have pushed many companies to other languages like Go, Kotlin, Java, and Rust, depending on their use cases.

While there are too many commercial users of Scala for the language to "disappear", it is not clear that the Scala programming language will grow adoption over time. The [ZIO community](https://zio.dev) continues to grow, but upon close inspection, the majority of this growth is occurring within the _existing_ Scala market, and it is therefore not a proxy for the commercial success of Scala.

In light of this uncertain future, I believe that the time for action has come. As passionate commercial Scala developers, evangelists, and OSS contributors, we have seen how quickly the language helps us build reliable systems at scale. We know that Scala has the potential to take cloud-native programming to a new level of robustness, performance, and productivity.

In the ashes of Scala 2 and the Lightbend OSS ecosystem, we need a rebirth for Scala. A **Scala resurrection**, so to speak, which can help the language and ecosystem soar to new heights.

In this post, I will discuss the issues standing in the way of Scala's growth in industry, and share my thoughts about how we can all help.

## Language

With the introduction of Scala 3, the language has new life.

Scala 3 simplifies the Scala language in many important ways, such as splitting different use cases for implicits into special-purposed language features. The compiler itself has a more modern and maintainable architecture than its predecessor, which allows improvements and bug fixes to be made more quickly and more reliably.

Scala 3 is powerful, and while by no means perfect, it packs significant improvements in the areas of type-safety, metaprogramming, and abstraction, compared to all mainstream programming languages. Moreover, the language learns from mistakes made in Scala 2, and responds to some (but not all) feedback from commercial Scala developers.

Despite the promises, there remain challenges with Scala 3:

1. **Academic bias**. Scala has a history of accreting academically novel features that do not help or which actively hurt commercial adoption, and I believe this process is ongoing with Scala 3.
2. **Language instability**. The Scala 3 language is essentially a different language than Scala 2, and the changes are of such size and scope that upgrading large code bases will take years, if it happens at all.
3. **Unproven compiler**. The Scala 3 compiler is still beset by many bugs that have proven troublesome to work around, and which hint at the growing complexity of the compiler code base.

Continued pain and attrition due to the Scala 3 upheaval is inevitable, but not fatal. With language and ecosystem stability, combined with tangible library benefits from upgrading to Scala 3, I believe we will eventually convince most serious commercial users.

However, I know that millions of euros are currently being invested into Caprese, which is a capability-based effect system built on capture tracking that could be a new language unto itself. If Caprese is transformed into a hypothetical Scala 4, it would upend not just the language, but the entire OSS ecosystem and all tooling, making the syntactical changes between Scala 2 and 3 seem tame by comparison.

The looming threat of a future radically different Scala 4 is the single greatest existential threat to Scala.

## Tooling

In the old days, a text editor and `make` were considered sufficient "tooling", but as programming languages have matured, so has their tooling, and expectations from developers are vastly higher today than at any point in history.

Every successful commercial programming language must include four key tools:

1. A package manager and online registry.
2. A build tool.
3. An integrated development environment (IDE).
4. An interactive mode (REPL).

A programming language might be able to acquire mainstream status with one deficiency. But achieving mainstream status with multiple deficiencies is extremely unlikely, given how far developer tooling has come.

Let's look at how Scala stacks up in each area.

### Package Manager

Despite the existence of [Coursier](https://github.com/coursier/coursier), Scala essentially has no native package manager or online registry. Instead, owing to its roots on the JVM, Scala relies heavily on infrastructure common to the platform.

In the JVM, _repositories_ hold build artifacts and dependencies. The structure of these repositories and the protocol by which they are accessed ultimately derives from _Apache Maven_, a project management tool built around objects like artifacts, versions, groups, and projects.

With Maven, there does not exist a _single_ online repository of all public packages. Rather, there are _many_ repositories, such as Maven Central, Sonatype Releases, JCenter, JBoss, and so forth. In addition, anyone can create and maintain their own repository, although build tools will not be able to fetch projects from custom repositories without additional configuration.

Relying on existing JVM infrastructure offers some advantages:

 * The tooling around Maven repositories is incredibly rich, including commercial offerings for hosting private repositories, usage analytics, multiple client libraries in every JVM language for accessing repositories, support for Maven in every build tool, and so forth.
 * Java is a common port-of-entry for developers learning Scala. 

On the other hand, piggybacking on Maven comes with some serious drawbacks:

 * Maven was not designed for Scala versioning and multi-platform support.
 * Publishing new projects on Maven is unreasonably complex and involved.
 * Maven is built on technologies of yesteryear, such as XML.
 * There is no one-stop-shop (_global online registry_) for all open source dependencies.

Modern package managers such as Cargo or NPM are tailor-made for their respective languages. Publishing and consuming dependencies is simple, in many cases a _one-liner_ that's baked into standard tooling. JSON and REST APIs are supported natively. And there is a definitive online registry of open source projects that is pre-wired into all developer tooling.

Maven may have been an advantage in earlier, simpler times, but in this day and age, it's a significant liability&mdash;a dinosaur from another era, which pushes people away from Scala.

Overall, package management in Scala gets a D, barely passing and that, based solely on Maven's maturity and its familiarity to Java developers.

Grade: D

### Build Tool

Except for an IDE or text editor, there is no tool a developer spends more time with than a build tool: the tool responsible for running compilers, downloading dependencies, and assembling artifacts that can actually be executed or deployed.

A good build tool can keep developers happy and productive, all the way up from simple "Hello World" applications to complex, large-scale applications that are split across dozens and dozens of projects, with hundreds of dependencies and very custom build steps.

The de facto build tool for Scala is _Simple Build Tool_, better known by the acronym _SBT_. For well more than a decade, SBT has gotten the job done, providing a way to build small and large Scala projects, both simple and complex.

To its credit, SBT did not shy away from complex builds, unlike many other attempts at creating build tools for Scala. Furthermore, its wide adoption is a testament not merely to its first-mover advantage, but to its features, documentation, build model, and extensive ecosystem of plugins. I and many other Scala developers owe a huge debt of gratitude and appreciation for the countless developer years poured into this tool, both by individual contributors who generously donated their free time, as well as by companies.

Unfortunately, SBT falls far short of the standards for modern build tools. Indeed, SBT falls so far short that numerous frustrated developers over the past decade have attempted to write better build tools for Scala (Mill, CBT, Seed, Fury, etc.).

At a high-level, SBT suffers from the following major drawbacks (among others):

 - **Extremely complex**. SBT has a high ramp up curve. When faced with a new SBT project, a developer starts with a terrifying empty Scala file, and must know what sequence of characters to type in order to create a "Hello World" project (there are workarounds, but the fact that these workarounds exists is proof of the massive developer-experience problem).
 - **Neither code nor data**. Data-based build tools, such as Ant, are purely declarative, and can be analyzed, optimized, turned into documentation, and easily generated and consumed by other tools; code-based build tools allow developers to write ordinary code to build their projects. SBT is neither data-based nor Scala-based, instead choosing a weird Scala DSL that has none of the advantages of either approach.
 - **Legacy monolith**. SBT was written over more than a decade, resulting in layers of aging code, an overly complicated architecture, and completely different, incompatible styles of Scala, making it more difficult to attract contributors.
 - **Poor Scala integration**. Despite being written in Scala and targeting Scala builds, SBT has no deep understanding of Scala versioning or cross-platforming, or the Scala compiler itself, and leaves many of the hard problems of building complex Scala applications to developers.
 - **Inscrutable failures**. Diagnosing problems with SBT can be extremely frustrating. To this day, there are times when SBT will fail to build a project, leading to the loss of countless developer hours. I myself once walked away from fixing a build problem after spending 40 hours without progress.

In my opinion, the main reason developers choose SBT to build Scala projects because developers choose SBT to build Scala projects. For all these drawbacks, Scala's build tool gets an F.

Grade: F

### IDE

An Integrated Development Environment (IDE), once a luxury, is now standard for all modern mainstream programming languages. With the most popular programming languages, developers have a choice of multiple IDEs&mdash;some commercial, some open source; some desktop-based, some cloud-based.

Thanks to type- or receiver-directed auto-complete, inline code documentation, inline errors and warnings, intelligent code refactoring, go-to-definition, and many more features, IDEs greatly increase the productivity and joy of developers, making many developers refuse to go back to editing code files with primitive text editors (whose only tool is "copy and paste").

In recent history, Scala developers have had a choice of three IDE-like experiences:

 - _IntelliJ IDEA_, with the Scala language plug-in
 - _Visual Studio Code_, with _Scala Metals_
 - _Emacs_ with _Ensime_

Only one of these would claim to be an IDE (IntelliJ), and only the first two have roughly the right feature set for an IDE.

Before evaluating these solutions, a few words of praise are in order.

JetBrains deserves kudos for building the finest Java IDE I have ever used (I happily paid for licenses in my Java days). I'm also quite grateful that JetBrains has invested in so many language plug-ins. Similarly, I'm grateful to the Scala Metals team for bringing me the first solution that I felt like _could become_ the IDE that I wanted for Scala.

Many talented and smart developers poured much laudable effort into both of these solutions.

Yet both IntelliJ IDEA for Scala and Scala Metals suffer drawbacks so serious, they are more prototypes than products, and the distance between them and something that delivers the experience of IntelliJ IDEA for Java is _growing_, not _shrinking_.

For quite plausible-sounding reasons, IntelliJ IDEA made a gigantic mistake when they decided to implement their own syntax, type, and kind-checking for the Scala plug-in. This same decision for Java resulted in an amazing IDE that could function incrementally and in the presence of type errors (it's quite useful to refactor code even when it doesn't fully compile).

However, Java has a very simple type system that is static and well-defined. Scala's type system is extremely complex and defined only by the implementation, which itself has numerous quirks, edge cases, and of course bugs. Moreover, the semantics, quirks, edge cases, and bugs all vary somewhat between minor versions of the Scala compiler, and are completely different between major versions. Attempting to emulate Scala's type system means that IDEA and Scalac often do not agree on whether code compiles, and if it does compile, what the error message is.

Rather than correct this mistake by using the Scala compiler itself for errors and warnings, IntelliJ IDEA has doubled-down on the mistake, time and time again. Perhaps their next-generation editor _Fleet_ is an acknowledgement both of the fact that JVM desktop applications are truly dead, and duplicating independent syntax, type, and kind-checkers for every language is not viable.

The architectural issues with the IntelliJ IDEA Scala manifest themselves in ways that frustrate Scala developers: code that compiles with `scalac` does not compile with IntelliJ IDEA. Code that does not compile with `scalac` compiles just fine with IntelliJ IDEA. One never knows if one's code is truly compiling without running the build tool, which defeats the primary purpose of an IDE.

Beyond these frustrating issues with simple compilation, the Scala plug-in for IntelliJ IDEA simply is not up to the same standards as their Java IDE. Refactoring support is barely present, and doesn't always work. With so many resources spent merely trying to keep the plug-in's model of Scala consistent with all the editions of the two Scala compilers, it is hardly surprising the team can't implement other IDE features.

As a result, while many commercial Scala developers still use IntelliJ IDEA (because, for all its drawbacks, it is significantly more stable and robust than Scala Metals), it is not an acceptable development experience, and it is no where in the same league as modern IDEs for languages such as Java, Kotlin, C#, Go, TypeScript, or even Python.

My own frustrations with IntelliJ IDEA led me to try Scala Metals, and I was blown away by the early experience of using the tool: with just a few clicks, it would compile my project and start reporting errors in VS Code, which were powered directly by the real Scala compiler.

Yet having used Scala Metals for a few years now, I know well that even though Metals demos well in controlled environments, the experience of using Metals in production is extremely frustrating. Quite simply, Metals is an [extremely buggy piece of software](https://github.com/scalameta/metals/issues/4837), which frequently gets into a state where it completely stops working.

In fact, so common is the experience of Metals breaking that some developers have made scripts to blow away all of its directories, kill its JVM processes, and restart Metals in hopes the incantation will work again for some (probably short!) period of time.

Beyond the painful experience of using Metals due to its longstanding instability, Metals has chosen an architecture which is fundamentally flawed. Although Metals does not have its own syntax, type, and kind-checker (like IntelliJ IDEA for Scala), Metals invented its own "Build Server Protocol" and a separate build engine called Bloop for understandable but ultimately technically unjustifiable reasons.

So not only does Scala Metals deliver a very poor experience, but it's built on a suboptimal architecture&mdash;a foundation that is far too large, and which will continue to accrete technical debt at an unsustainable rate.

In short, there are no IDEs for Scala right now with a modern experience suitable for commercial software development, nor any obvious path to a commercial-grade IDE. Consequently, Scala's grade for IDE support is a definite F.

Grade: F

### REPL

The REPL, which stands for Read-Eval-Print-Loop, allows developers to explore a language's syntax, semantics, and libraries, and to solve simple programming problems in an interactive way that maximizes feedback and encourages iteration and experimentation. While often targeted at new developers, REPLs are often used by seniors.

To its credit, Scala had a REPL before it was cool for statically-typed languages to include one. The REPL grew increasingly outdated over the years, failing to keep up with modern REPLs (especially those of dynamically-typed programming languages).

Then [Ammonite](https://ammonite.io/) came along, and added numerous features, such as syntax highlighting, multi-line editing, dependency loading, and much more, giving Scala a much more impressive REPL story, albeit one laden with edge cases in the wake of Scala 3.

Overall, because Scala does have a modern REPL in Ammonite, but because Ammonite does not entirely support Scala 3 and the project is not integrated into the Scala compiler, Scala's REPL support gets a grade of B.

Grade: B

### Tooling Summary

Scala's average rating is a D, well below what I would consider the _bare minimum_ for a mainstream programming language. Moreover, this grade represents a lowering from the peak of the Scala 2 lifecycle, where tooling and in particular IDE support reached an all-time high.

It is worth pointing out that while these are my own ratings, they are [consistent with the views of the broader Scala community](https://scala-lang.org/blog/2022/12/14/scala-developer-survey-results-2022.html), at least as collected by the non-random (and therefore certainly biased) 2022 Scala survey.

The state of tooling represents the most immediate and greatest challenge to working with the Scala programming language today.

## Ecosystems

Scala has multiple open source ecosystems, which target specific pain points in building backend applications. In the sections that follow, I overview the strengths and weaknesses of each, with an eye toward how these ecosystems affect Scala adoption.

### Akka

The Akka ecosystem originally intended to bring Erlang-style actors to the JVM. Through excellent marketing, and partially by riding the early Scala hype train, Akka managed to find itself over-hyped and over-deployed, with many companies later needing to rip out the technology (quite simply, actors are a terrible solution for local concurrency).

As Akka matured, however, and valid use cases became clear (distributed, persistent actors and an async foundation for local streaming), the ecosystem found its niche and its stride, and was successfully adopted by many large companies in mission-critical use cases.

These days, however, Lightbend, the company behind Akka, decided to monetize the framework directly, by changing from an open source license to the _Business Source License_, which had proven monetizable for databases and other developer infrastructure.

Although there is an open source fork of Akka called Pekko, it is not well-maintained and stands on questionable legal foundations. Thus, every company I have talked to about Akka has told me they are ripping it all out. Apache Flink, which depended on Akka, also plans to rip it out, with no plans to depend on Pekko.

Akka is _still_ a powerful framework for building distributed and streaming applications. However, I do not expect Akka to help grow the Scala ecosystem, and in fact, I expect the change in licensing to push some companies away from Scala.

Thus, with a Scala-centric lens, Akka gets a grade F, solely for its negative effects on Scala adoption.

Grade: F

### ZIO

The ZIO ecosystem has brought a fresh and innovative fusion of practicality, ergonomics, and functional programming to the domain of building scalable, bulletproof cloud applications.

Ported to multiple programming languages, and even inspiring a few companies to "Go Scala", ZIO has become widely adopted on its own merits, while providing a partial but increasingly viable alternative to Akka, Spring, and other modern application development frameworks.

The ZIO community actively creates many new open source contributors and speakers each year, mentors existing open source contributors, and creates and maintains integrations to other Scala ecosystems, without any politics and with old school professionalism.

Despite all the good that the ZIO ecosystem is doing for Scala, however, the ZIO ecosystem has multiple weaknesses:

 - **Incomplete stack**. As of the date of this writing, the ZIO ecosystem does not have a complete stack for backend development, its major weaknesses lying in HTTP (there is no 1.0 version of _ZIO HTTP_), persistence (only _ZIO Quill_ is production-grade, and not everyone likes LINQ-style database access), and gRPC (there is a great library but performance needs to be optimized).
 - **Early-grade projects**. The ZIO ecosystem contains a number of projects that are still early-stage (_ZIO JDBC_, for example), and which require continued investment in order to become production-grade.
 - **Limited documentation**. While the core ZIO project is quite well documented at this point, many critical ecosystem projects are not documented at the same level. In some cases, there is only a page or two of documentation.

Given these drawbacks, together with the positive aspects, the ZIO ecosystem receives a grade of B. 

Grade: B

### TypeLevel

The TypeLevel community, which started life as a political fork of the much older Scalaz (the first port of Haskell libraries to Scala), attempts to bring category-theoretic functional programming to commercial software development in Scala.

Being the oldest still viable example of "functional programming in Scala", the TypeLevel ecosystem has slowly developed a full-stack solution for the backend, including solutions for persistence, HTTP, logging, and caching. With much better documentation than Scalaz ever had, TypeLevel has also made Haskell-style functional programming accessible to more Scala developers than ever, showing commercial software developers the practical benefits of referential transparency and algebraic abstractions.

While having a positive impact on adoption of functional Scala within industry, the TypeLevel ecosystem has weaknesses of its own, including:

  - **Developer-experience**. Numerous developers have shared feedback on the challenges of using some TypeLevel libraries such as http4s and Cats Effect, particularly for those new to Scala and functional programming. Some new TypeLevel contributors have pushed for improvements (successfully, in the case of Cats Effect documentation, for example), but generally such feedback is not welcomed or labeled FUD, and leads to responses like "Get Educated".
  - **Community challenges**. TypeLevel has had its share of community challenges, partly due to its controversial founder [Travis Brown](/articles/travis-brown-abuser), a known abuser, cyber-stalker, and harasser who was forced out of the Scala community, but also due to the actions of a tiny minority of TypeLevel members, the personal friends of Travis Brown&mdash;actions like supporting Travis Brown, deleting reports of CoC violations, engaging in elitist putdowns of OSS work deemed "insufficiently pure", smearing the character of other OSS contributors, and all-around unprofessional behavior toward ZIO, ZIO integrations, and ZIO contributors.

In light of both the positive impact but also TypeLevel's challenges around usability and community, the ecosystem receives a grade of B.

Grade: B

### Other

Though Scala is home to many open source libraries, most of these are not part of any ecosystem. The two partial exceptions are SoftwareMill's range of libraries, including Tapir and sttp, and Li Haioa's Python-esque libraries that focus on getting things done with a minimum of ceremony.

There is no doubt many of these libraries are high quality, especially in the aforementioned two ecosystems. Where they are present, weaknesses of libraries across the greater Scala ecosystem include:

 - **Duplicated effort**. For its relatively small size, Scala has a disproportionately high number of libraries all trying to solve any given problem. Given that Scala is not very opinionated, but that OSS contributors very often **are** opinionated, perhaps this is not surprising.
 - **Abandoned libraries**. Due to their origin as one-person passion projects, many older Scala libraries are abandoned or in a state of disrepair, without being officially deprecated or transitioned to more motivated maintainers.
 - **Relatively unambitious**. There are few concerted efforts to build large-scale open source that could rival the size and scope of projects like Flink, Spring, or even Akka (with some [exceptions](https://github.com/JohnSnowLabs)).

Overall, Scala gets a B in this category, an above average score. For its size, Scala has a wonderful third-party OSS ecosystem.

Grade: B

### Ecosystem Summary

Scala's modern OSS ecosystem suffered a blow when Lightbend took the Akka ecosystem proprietary. Akka provided a comprehensive toolkit for building resilient, distributed, cloud-native applications, and showed the world that Scala offerings could compete with the very best of what other languages offered.

Despite these tumultuous changes, the Scala open source ecosystem remains vibrant and powerful&mdash;definitely playing in a league well beyond what the size of the Scala community would suggest.

However, despite being above average, there is ample room for improvement, in ZIO, TypeLevel, and other ecosystems. With improvement in a few core areas, Scala could unquestionably have one of the finest open source ecosystems of any programming language.

## Platform

We live in a multi-platform age. Our applications must run locally on different desktop operating systems, remotely on different server operating systems, on different virtual machines, and sometimes even in browsers or on the edge.

In this section, I will overview Scala's support for targeting multiple platforms.

### JVM

The JVM was the first platform that Scala supported, and to this day, it remains the only officially supported platform for Scala, and the only one that is baked into the compiler and broadly assumed by ecosystem tooling and libraries.

Despite the JVM being the first-class platform for Scala, there are still some major deficiencies in Scala's support for the JVM:

 - **One-way interop**. It is virtually impossible for Java or Kotlin code to call into Scala code. The fact that other languages cannot consume Scala libraries diminishes the interop argument for Scala-on-JVM.
 - **Lackluster optimizer**. Scala is a higher-level language than Java. Rather than trying to minimize the cost of abstraction, the Scala compiler generates dumb code that executes more slowly in the presence of more abstraction.
 - **Lagging support**. Scala's support for newer versions of Java, as well as new JDK or JVM features, lags behind. When the JVM gets value objects, for example, it is quite likely Scala support will severely lag; or as another example, the Scala compiler itself has been known to crash on modern JVMs.
 
Because Scala does support the JVM natively and directly, but also because its support is not best-in-class, Scala's support for the JVM platform gets a grade of C (average).

Grade: C

### JS

Supporting the Javascript platform for Scala has always puzzled me somewhat. Currently, Javascript is used primarily for web front-ends, and most Scala developers have neither interest nor skills in frontend web development.

Most web frontend developers use Javascript or TypeScript. They do not want to learn a backend programming language to write frontend code.

That said, it's quite nice to use the same language for both frontend and backend (if you are familiar with both), and doing so can often reduce code duplication. In addition, providing Scala support for the Javascript platform allows backend engineers to create and maintain applications that they would ordinarily have no interest in creating or maintaining.

The quality of the Scala.js platform is high. Sporting a small subset of the JDK, one can compile real world code on Scala.js, and get high-quality Javascript output. Method overloading and other Scala features are mostly emulated in the way you would want.

That said, however, Scala.js has a number of weaknesses:

 - **Standalone target**. Scala.js is not integrated into the compiler, but must be added as a compiler plug-in, with library dependencies.
 - **Complex build**. Setting up a project for cross-building is difficult (with quite painful developer experience), complicates the build, and significantly slows down building and testing.
 - **Incomplete JDK support**. The JDK is so vast that inevitably some class is needed which is not emulated in Scala.js.

Considering both the strengths and weaknesses of Scala's Javascript platform support, Scala.js gets a grade of B.

Grade: B

### LLVM

Scala's LLVM support is known as _Scala Native_, because the end result of using LLVM is _native_ platform-specific executables that run without bytecode-based interpretation or just-in-time compilation.

Scala Native is an attempt to wean Scala from its long-standing addiction to the JVM, providing a viable alternative platform for backend applications. JVM applications suffer from high-memory usage, GC pauses, and lengthy application startup and warmup times, all of which prove problematic with many classes of applications (command-line applications, cloud-native applications, etc.).

Scala Native is an enormous undertaking, because LLVM is much lower-level than the JVM or Javascript platforms. As with Scala.js, Scala Native must provide an emulation of some JDK constructs for virtually any program at all to run. But more than that, Scala Native must provide memory management, multithreading, and many other functions that we take for granted on the JVM platform.

A number of highly talented engineers have poured their own personal time and effort into Scala Native. In addition, the project has seen some modest investment by EPFL and the Scala Center.

Despite all of this investment, however, Scala Native is _no where near_ to a suitable replacement for Scala JVM. The vast majority of Scala applications are not even close to _compiling_, let alone executing reliably and performantly, on Scala Native. 

As much as I see the promise of viable alternatives to the JVM, Scala's LLVM support receives a grade of D, simply because it is not there yet, and requires significant further investment to become viable for most JVM-based Scala applications.

Grade: D

### WASM

WASM has emerged as a possible successor to the JVM specification, one built for a cloud-native era, in which sandboxing is the norm, and low-latency, low-memory consumption cloud apps rule the skies.

In theory, there is a way to translate from LLVM IR to WASM through Emscripten. In practice, this is so far away from anything usable for production that Scala gets an F for WASM support.

Grade: F

### Platform Summary

Scala's platform strengths reside squarely with the JVM, which is unsurprising given the origin of the language. What is increasingly surprising, however, is that Scala remains an essentially JVM-only language.

Scala.js is an excellent attempt to bring Scala to the front-end via Javascript, albeit with a limited market and challenges around JDK support. Scala Native (LLVM), on the other hand, while being a remarkable achievement, is such a vast undertaking that it's too early, too limited, and too immature to benefit the typical Scala application. The LLVM pathway to WASM (if you want to call it that!) is merely a hello-world toy.

If you believe the JVM is, without question, the future platform for cloud- and edge-native applications, then Scala's existing platform support may be everything you want and need. For those hoping for native applications or a more modern take on a cross-platform virtual machine, Scala's current cross-platform support leaves a lot to be desired.

## Scala Resurrection

The difference between a hater and a lover is that a hater shouts (destructive) criticism to tear down, while a lover offers (constructive) criticism to build up.

To go as far as I believe the language can go, Scala needs building up. Not from just one individual, such as myself, or even Martin Odersky. Rather, *Scala needs all our help*, as we work together to make the language as commercially successful as it deserves to be.

We need a coordinated campaign to rectify known weaknesses, and shore up emerging strengths. Such a campaign will not be easy. It will require discipline, collaboration, and sacrifice, acting with a sense of common purpose for the greater good of all.

In the sections that follow, I'm going to present what I think needs to happen, as well as a few ideas on how it could happen. If you disagree or have your own suggestions, then please post below!

### Industry Language

In my opinion, mainstream adoption of Scala requires a commitment to language stability. There can not be more "Scala 3" editions, which re-imagine the semantics and syntax of the core language, break virtually every library ever developed, reset tooling to ground zero, and require a combination of automated and manual code rewrites.

It is absolutely true that Scala 3 is not a perfect language. It is also true that Java is not a perfect language. Yet Java remains backward compatible with the very first Java program ever written, which allows the ecosystem to continuously improve, and which gives companies confidence in investing in the Java programming language.

Scala has been a useful vehicle for exploring academically interesting ideas, such as the dot calculus which forms the basis of Scala 3. But in order to enter mainstream, academic experiments must live outside Scala. They must live in new languages, or new dialects of Scala that are well insulated from the commercial edition of Scala.

Scala now needs a firm commitment to maintain backward compatibility indefinitely. We need to be able to run the same Scala 3 programs now that we can run 10 years from now, without having to completely rewrite them (with or without partial automation).

This commitment might mean that new academic explorations might not make it into Scala, but it will also mean Scala's ecosystem (particularly its tooling, but also libraries) will bloom, and companies will have the confidence they need to invest seriously into the language.

Scala also needs a commitment to fixing bugs and improving performance in the core compiler. Scala developers need a compiler that works reliably and performantly, and while that's true in many cases, there is much opportunity for ongoing improvement. 

Finally, Scala needs work on an optimizer that can minimize the cost of abstraction. There is little value in having a language as expressive as Scala if actually taking advantage of its full powers slows applications down. And there is no reason why it should have to, as most forms of abstraction can be eliminated via a sufficiently advanced optimizer.

### World-Class Tooling

Scala desperately needs an IDE that works reliably across complex, real world projects. An IDE is table-stakes for any mainstream programming language, and while IntelliJ IDEA and Scala Metals have both tried to give Scala developers the IDE they need, neither has yet attained that goal, and architectural choices likely prevent these projects from ever attaining this goal.

We need an ultra lean, lightweight architecture for a squeaky clean IDE that can be maintained by a small team with an exceptionally low defect count. Far more important than breadth of features is core stability and developer experience. By leveraging existing open source, by learning from the architectural mistakes of IDEA and Metals, we can engineer a new and extensible solution that delivers the joyful and productive experience that modern developers expect from their IDEs.

Scala also needs a modern build tool, and can chart one of three paths to get there:

1. **Invest in multi-build tool**. Multi-build tools like Pants, Bazel, etc., can be used for building any kind of project. Although support for Scala is weak in such build tools, it could be improved with further investment from the Scala community.
2. **Invest in the most promising SBT alternative**. None of the SBT replacements are currently viable for all use cases being served by SBT. However, if the authors are interested and open to collaboration, it's possible some of these build tools could be improved to handle the full range of use cases handled by SBT, but without SBT's many drawbacks.
3. **Develop a new build tool**. Although this seems like a herculean task, modern Scala OSS includes libraries like ZIO Streams and FS2, which, together with smaller, special-purpose libraries like Coursier, could dramatically reduce development time.

Though we can wait on this one for a while, Scala still needs its own package manager, distinct from Maven, which is built for Scala libraries and applications, which knows about Scala's versioning and cross-platform support, and which provides the joyful experience required by new developers, and the extensible experience required by seasoned professionals.

Scala also could benefit from a first-class REPL experience like Ammonite, polished, extended with full support for Scala 3, and incorporated directly into the official Scala compiler, as a replacement for its existing REPL.

In discussing tooling, I must mention the work that Virtus Lab has done in creating [Scala CLI](https://scala-cli.virtuslab.org/), a simple way to compile, run, test, and package single-module Scala projects that was recently accepted as the new `scala` command.

While not really a build tool right now (for most real world projects), Scala CLI could potentially become a foundation for a build tool, with package management and integrated REPL, as well as IDE support.

Solving the tooling challenges will require investment into promising open source projects, investigating ways to integrate the best parts from multiple OSS projects (for example, Scala CLI, Ammonite, Fury, Ensime), and the creation of new open source projects. Moreover, it will require a commitment to ongoing, broad-based investment from the community at large, as well as partners from academia and industry.

### Ecosystem

The Scala open source ecosystem is strong, but needs to become stronger. Obviously, the respective ecosystems can and should embrace their respective strengths, while addressing their respective weaknesses.

However, I believe Scala can go much further than just ecosystem-siloed improvement.

With a finite resources, we need to make every hour of open source labor count. One of the most rational and effective ways to do this is to reduce duplication across Scala's open source ecosystems.

Does Scala really need 20 JSON libraries, 5 schema libraries, 9 CLI libraries, 4 effect system libraries, and so on? Probably not. We need to consolidate, and rally around strong projects; merge our strengths, cancel our weaknesses.

There's probably a way to do that, in some cases, that makes everyone happy.

Let's say we wanted to consolidate JSON libraries. Each ecosystem having a JSON library could appoint a person to collaborate, who would be an admin in a joint project. The best aspects of each library could be combined into a single library that meets the needs of all users.

In support of this goal, I will personally commit to helping organize a _Scala Open Source Summit_, which encourages all ecosystems to come together to discuss and hack on Scala OSS. This should help foster dialogue between different ecosystems to see if common ground can be found.

If the most brilliant minds in Scala open source work together, then surely, the whole ecosystem can rise to new heights!

### Cross-Platform

The JVM is attempting to innovate, and to its credit, is doing so faster now than at any time in its history. However, the world has moved far past the JVM, with cloud-driven requirements including auto-scaling, fast startup times, and low memory usages, running on heavily virtualized and sandboxed infrastructure, for applications demanding increased efficiency for greater cost savings.

The JVM doesn't have a solution for these problems, and while Oracle can solve this problem, it's not clear they will. There is obvious tension between Oracle's desire to improve the JVM, and Oracle's desire to make money on the JVM, and some of the pain points in the JVM are already being monetized (see _GraalVM Enterprise_). 

If we were to redo the JVM specification from the ground-up, it might look a lot more like the WASM specification. WASM points to a future where a new operating system or perhaps a single application on an existing operating system runs hundreds or thousands of WASM applications in a completely secure, sandboxed fashion, at low cost, with auto-scaling and low memory usage.

In any case, it's clear to me that sticking with the JVM is an increasingly risky strategy. It ties the fate of Scala to the fate of the JVM. But Scala doesn't need the JVM to be a powerful, compelling solution to building modern applications.

Maybe in the early days, Scala needed the JVM. But no longer.

Scala needs to support alternative platforms, and it needs to do so directly, integrated into the compiler. The JVM platform should not be specialized. Rather, each backend should take an intermediate representation (perhaps TASTY, or perhaps something lower-level) and generate platform-specific code, sharing optimization passes where it makes sense to do so.

A huge liability in supporting multiple platforms is the Scala ecosystem's heavy reliance on the JDK. Both Scala.js and Scala Native have to re-implement the JDK, which is a monstrous task to do exhaustively and correctly. Moreover, it's not even really necessary, since an increasing percentage of Scala applications are not written to the JDK, per se, but to Scala's own open source ecosystem.

In order to enable low-cost, bulletproof cross-platform support, Scala *needs* a low-level, cross-platform library that provides the bare-bones essentials for platform support: including threads, files, sockets, processes, and rudimentary time functionality (one should look to various preliminary WASM specs for inspiration).

Scala's vast OSS ecosystem would then adopt this cross-platform library to power their features.

In this fashion, Scala applications depending on Scala's own open source ecosystems would instantly become compatible with all of Scala's supported platforms, to the maximum extent possible. In addition, there would cease being such a thing as "cross-platform libraries" (which are hell to maintain), because all libraries would always be fully cross-platform.

Such a cross-platform library could be built by volunteers, and architected and overseen by leaders from all the major OSS ecosystems in Scala (since ultimately, they would be its direct users, and critical to weaning Scala applications off the JDK).

As CEO of Ziverge and a core contributor to ZIO, I am happy to play a role in developing a low-level cross-platform library, but we need key stakeholders from other ecosystems and companies in the space to rally behind the idea.

### The How

As the Scala programming language and broader ecosystem is not a commercial product, there is no single entity who can fund development and maintenance of critical infrastructure. Although in some cases, I've volunteered my own time and resources, or those of my companies, ultimately, in order to address all of these challenges, we need much broader participation.

Fortunately, there are many organizations who can and do contribute to Scala. The involvement of some of these organizations will be crucial.

#### LAMP

LAMP is the [lab at EPFL](https://www.epfl.ch/labs/lamp/) that is run by Martin Odersky, and which contributes directly to Scala 3 and the broader ecosystem, including open source and tooling. 

LAMP receives its funding from grants (such as the Advanced Grant program from the Swiss National Fund), which are targeted at research and development in specific areas of programming methods.

Due to the way that grants to LAMP work, there is pressure on projects to produce research that is academically novel. Yet, academic novelty is frequently at odds with the needs of industry: commercial developers want languages, tooling, and open source to solve practical problems in straightforward ways, without regard to academic novelty, theory, or value to the research community.

Beyond this basic tension, many projects coming from LAMP are spearheaded by PhD students, who, when they obtain their doctorate, abandon their project and accept a job in industry, creating a steady stream of half-baked abandonware that never realizes its full promise in industry.

Despite these drawbacks, there is no doubt that LAMP has powered tremendous positive change for the Scala programming language, including the Dot calculus which brings a sounder foundation to Scala's advanced type system. The students and assistants working at LAMP are quite brilliant and hard working, but these traits do not solve the tension between academic research and industry needs, nor address the finite tenure of students.

To improve upon its legacy, I believe LAMP could benefit from a new fusion of industry and academia:

1. **Broad industry input into research projects**. There are many possible improvements to the Scala programming language that are both academically interesting, as well as incredibly useful to industry (for example, GC-less allocation and de-allocation through whole program analysis; stack-based storage through local primitive record inlining; lightweight linear typing for leak-proof resources; etc.). If the industry has some input into research projects, it would help ensure they contribute to the commercial success of Scala.
2. **Industry partnership to prevent half-baked abandonware**. Ideally, there would be no LAMP projects targeting the Scala language or ecosystem that do not have an industry partner, who commits to the maintenance of the project after completion. Grant money is non-recurring, and PhD's are one-and-done, so Scala artifacts produced through LAMP need industry partnership to be maintained or polished into commercial grade technology.

By directly involving industry in the development and maintenance of projects, the work that LAMP does will be more relevant than ever, have a much broader impact on commercial software development, and result in long-lasting benefits for the entire Scala community.

#### Scala Center

The Scala Center is a not-for-profit organization backed by EPFL and corporate sponsors, dedicated to open source and education around the Scala programming language. The organization is closely connected to and shares some of the same leadership and team as LAMP.

As with LAMP, the Scala Center has made many positive contributions to the Scala programming language and ecosystem. Originally formed to fill the void in the Scala ecosystem after Lightbend (Typesafe) pivoted away from Scala, the Scala Center has contributed to Scala.js, Scala Metals, Scalafix, Scalafmt, and many other projects.

While numerous smart and talented engineers have worked at Scala Center on important projects, the organization itself has had major challenges, including:

 - **Re-inventing wheels**. Rather than embrace and extend existing open source projects, Scala Center has chosen to build new open source projects (or even new protocols such as BSP!). This can alienate existing OSS contributors (as it did with the author of Ensime), creates more duplicated work, and increases chances of failure due to fewer committed stakeholders.
 - **Abandoning projects**. Scala Center has a long history of creating projects that show potential and then abandoning them.
 - **Breadth over depth**. Scala Center has a demonstrated history of preferring "feature count" to depth and production-worthiness.

While these problems are not particularly hard to solve, they do require effective organization and management, collaboration with industry partners and open source contributors, and a commitment to long-lived and boring (but essential!) open source maintenance.

Instead, at a time when Scala tooling has fallen behind because of Scala 3, the Scala Center chose to spend resources building an [inclusive language guide](https://www.scala-lang.org/blog-detail/2022/04/05/inclusive-language-guide.html). Personally, I don't know any Scala developers who would prefer an inclusive language guide over a working IDE.

Worse still, at a time when the language, tooling, and cross-platform support require more financial investment than ever before, the Scala Center rejected a generous offer of corporate sponsorship from [Ziverge](https://ziverge.com).

As a CEO, I know how to fix these problems, but unfortunately, there is nothing any of us can do because there is no democratic or open mechanism for influencing the organization, management, or priorities of the Scala Center.

In the best case scenario, the Scala Center continues to operate as before, which means the Scala community gets _some_ value from the organization, but cannot rely on the Scala Center to move the needle of adoption or completely solve any of the challenges Scala has.

#### Commercial Organizations

The main commercial organizations involved in contributing to the greater Scala ecosystem include:

 - **JetBrains**. JetBrains is investing quite a lot into Scala via its Scala development team. The problem is that their current approach to re-creating Scala's type system leads to massive waste, and it doesn't work and can't ever work. I would like to see JetBrains invest in the core compiler to make it better suited for use in an IDE (e.g. error recovery, plug-ins to export compile-time information, tooling around TASTY, etc.). I know many Scala developers who would easily pay $500 or more for a Scala IDE that actually worked.
 - **Lightbend**. Lightbend maintains the Scala 2.x compiler, allegedly with revenue from fintech-based support and maintenance contracts. This is important work, as Scala 3 adoption is still low in industry. However, given Lightbend has an uncertain future, and an increasingly tenuous connection to Scala, I believe that the company will stop funding 2.x maintenance. I hope the maintenance contracts will be smoothly transitioned to another organization who can continue to maintain Scala 2.x.
 - **LunaTech**. LunaTech has not been very active in contributing to Scala, but recently the organization lent Chris Kipp to the Scala Center to focus on improved tooling for the Scala ecosystem (donating labor is one way to obtain a seat on the Scala Center advisory board). My hope is that this represents a long-term commitment from LunaTech to improving Scala tooling, and I would love to see them extend this investment.
 - **Scalac**. Scalac is one of the oldest Scala consultancies, and has invested into many open source libraries, particularly around the Akka and ZIO ecosystems. It would be fantastic to see this investment become more consistent, dedicated, and expanded.
 - **SoftwareMill**. SoftwareMill primarily contributes to the Scala open source ecosystem, principally its own projects such as Tapir, sttp, Bootzooka, Magnolia, MacWire, and so forth. These libraries have a positive impact on Scala's adoption in industry. It would be great to see these OSS investments maintained, and perhaps even supplemented with additional investment into tooling or the language itself.
 - **VirtusLab**. Virtus Lab, a consultancy whose primary business line does not involve Scala, generously sponsors some excellent open source work on Scala 3, Metals, and Scala CLI. Given that [Aquiline](https://www.aquiline.com/), a noted private equity firm, acquired a stake in the company and is grooming them for sale to potential buyers, it's unclear if this investment will continue indefinitely, but for now, we can be grateful for the work they are sponsoring.
 - **Xebia Functional**. This is a division of Xebia formed through the acquisition of a company called 47 Degrees. As 47 Degrees, the team once contributed significantly to Scala open source (FreeStyle, Mu, Fetch, etc.). In modern times, the company's focus has shifted to Kotlin and its Arrow OSS ecosystem. Recently the company [controversially](https://contributors.scala-lang.org/t/pre-sip-suspended-functions-and-continuations/5801) proposed to copy Kotlin's pre-Loom hack for the JVM's lack of virtual threads [into Scala](https://github.com/47deg/TBD). Given Xebia has no vested interest in Scala, and given its resources have shifted to Kotlin, it seems unlikely the company will contribute positively to Scala.
 - **Ziverge**. My own company Ziverge contributes mostly to the open source ZIO ecosystem, but also does a large amount of Scala evangelism through annual conferences and hackathons, and weekly educational episodes on YouTube. Ziverge is exclusively focused on Scala, and has a very clear alignment with the commercial success of Scala. I would like to see Ziverge contribute to improved tooling and cross-platform support.

Given the limited resources of LAMP and the Scala Center, some of these organizations (in addition to many individual contributors) must be a part of the solution to challenges facing the Scala programming language. 

## Summary

As a Scala enthusiast and investor, I am keen on increasing Scala's adoption in industry. While Scala has powerful features, it also has some challenges, which prevent Scala from reaching its commercial potential.

I believe that it is time for all of us Scala enthusiasts to take action to improve the following areas:

 - **Industry Language**. Scala needs to become an industry-focused language whose innovation is powered by academia, but tightly and exclusively focused on industry. As part of this transformation, Scala 3 needs to commit to backward compatibility, like Java, Go, and all other commercial programming languages. There can never again be a "Scala 3"-style boil-the-ocean, greenfield language rewrite. It's time for the language to stabilize, and for the focus to shift to commercial interests, including bug fixes, improved performance, enhanced tool support, and optimizing.
 - **World-Class Tooling**. Scala needs significant investment in tooling, most notably, a commercial-grade IDE with a sustainable architecture, and a modern build tool. Beyond the IDE and build tool issues, Maven package management is antiquated and poorly suited for Scala, and Scala's built-in REPL pales in comparison to third-party REPL support.
 - **Ecosystem**. Scala's open source ecosystem is good, but small, and given limited open source resources, we need to find ways to work together. Getting talented folks across ecosystems in the same room is a good first start.
 - **Cross-Platform**. Scala owes a debt of gratitude to the JVM for breathing life into this functional programming language. But ultimately, the JVM is Oracle's baby, and tying the fate of Scala to the fate of the JVM is risky...riskier now in an age that demands applications that are lightweight, with low-memory usage, and fast startup times. Scala must go beyond the JVM in a first-class way.

Given that Scala is not a commercial project, addressing these deficiencies will require coordinated effort across individual open source contributors, LAMP, Scala Center, and organizations like Ziverge. It may require organizational and operational improvements, and will most certainly require new levels of collaboration, as a diverse set of commercial and academic users come together for a common purpose.

Through discipline, hard work, and creative thinking, I believe we can create a new era in Scala's commercial adoption&mdash;a true _Scala resurrection_.

Stay tuned for more!
