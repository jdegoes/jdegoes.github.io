---
layout:       post
title:        "New Scala Build Tool"
description:  "Scala needs a new build tool to replace the aging SBT, but that new build tool needs to learn from mistakes of the past, rather than repeat them"
category:     articles
tags:         [scala, scala3, tooling, ides, functional programming, build tools, sbt]
---

I previously [made a case](https://degoes.net/articles/scala-resurrection) that Scala needs coordinated and radical changes in order to remain relevant to industry. 

On the tooling front, the two areas where Scala needs the most help are IDEs and build tools.

In this post, I'd like to outline what I believe are the three separate paths to a best-in-class build tool, and share my thoughts on each strategy.

## Path 1: Ditch & Run

The first path to having a good build tool for Scala is, ironically, to ditch all of our Scala build tools! Rather than using a Scala build tool, we embrace and extend an existing multi-language build tool, such as Gradle, Pants, or Bazel.

In order for this strategy to be successful, three ingredients are necessary:

1. We collectively decide upon a multi-language build tool, and we migrate our projects to it.
2. We identify weak areas in the built tool's support for Scala, and we contribute to smooth them over, with particular attention to new developer onboarding experience (required for the top of the funnel) and complex build requirements (required for the bottom of the funnel, where the very few large commercially relevant projects exist).
3. We engage in evangelism and education to teach Scala developers how to use this multi-language build tool, deprecating Scala build tools.

This strategy was successfully adopted by Kotlin, which largely piggybacks on Gradle (most Kotlin projects, including Kotlin itself, are built using Gradle).

Personally, I do not think Bazel is appropriate, because of the extreme difficulty making Bazel accessible to top-of-the-funnel. The other two obvious choices are Gradle and Pants.

Pants V2 is written partially in Rust and is quite fast. Gradle is written in Groovy and Java, and inherits slow JVM startup time and large memory usage, though attempts to workaround the former with build daemons (like SBT and other Scala build tools).

The support for Scala in these build tools is immature (Gradle is [here](https://github.com/gradle/gradle/tree/master/subprojects/scala/src/main/java/org/gradle/api) and Pants is [here](https://github.com/pantsbuild/pants/tree/main/src/python/pants/backend/scala)), but the code base is relatively small and manageable, and should be straightforward to bring them up to speed with Scala build tools.

The main advantage of embracing an existing multi-language build tool is that the steps involved in building a software application are largely common across different languages. So rather than re-invent what all other build tools invent, we can just leverage existing machinery, and put our limited resources into polishing and extending the Scala plug-ins in these build tools.

Essentially, it's like getting a huge workforce for free to build and maintain every part of the build tool except the Scala-specific parts, which includes not only the core build machinery (watching files, downloading dependencies, caching artifacts, etc.) but also a massive ecosystem of plug-ins that add features like publishing, packaging, deployment, and so forth.

The main disadvantages of adopting an existing multi-language build tool are as follows:

1. Multi-language build tools don't know anything about Scala, so they can't tailor their user-interfaces and base feature sets specifically to Scala. Obviously, however, the language-specific plug-ins to these build tools can be built for Scala developers, so this limitation only affects the base feature set of the build tools.
2. Multi-language build tools are not written in Scala. To contribute to Gradle, you should know Java and Groovy. To contribute to Pants, you should know Python and maybe Rust. There is a much smaller pool of Scala developers who are able and willing to contribute in these languages.

It's worth pointing out that many large projects (including, for example, Twitter's own code base) have long ago ditched Scala build tools, because their needs (especially for performance) grew more sophisticated than what our own build tools offer.

## Path 2: Double-Down

The second path to having a good build tool for Scala is to double-down on existing build tools, and try to address their weaknesses. 

The dominant Scala build tool is SBT, which has [serious drawbacks](https://degoes.net/articles/scala-resurrection). So realistically, we are talking about doubling down on SBT.

The main problem with this path is that fixing everything that's wrong with SBT would completely break backward compatibility. Which means, in essence, it would be a Python 2/3 or Scala 2/3 catastrophe, where users would have to develop new build files or try to "port" their build files to what is essentially a new build tool with a new ecosystem.

If you are going to break backward compatibility, and force a painful and time-consuming transition, then effectively, this path is the same as building a new build tool.

## Path 3: New Scala Build Tool

The final path to having a good build tool for Scala is to _create_ a new Scala build tool, either from scratch or by forking an existing Scala build tool without the constraint of backward compatibility.

I think it quite likely any new Scala build tool will end in ruin. There are dozens of dead Scala build tools, very few of which ever managed to gain any traction.

There are a few reasons for this inglorious end for new Scala build tools:

1. **Hobbies**. Scala build tools are written by hobbyists with an itch to scratch, not by commercial developers who are being paid by their employers to solve real problems. Hobbyists frequently abandon their work, and just as frequently only solve the "interesting" part of a problem, not the "boring" part. For build tools, however, solving the boring part is extremely important to adoption.
2. **Two-Sided Market**. There is no successful build tool that is successful by itself: they all have large ecosystems that add incredibly useful features to the build, package, and deployment process. For example, SBT has a large network of plug-ins that solve common problems. Without such an ecosystem, a build tool has trouble attracting users. Yet, without users, developers aren't motivated to build out an ecosystem. It's a chicken-egg problem.
3. **Inexperience**. Most developers have no experience in building successful products, and every open source project (including tools) is a product (or a mini-startup, if you will, unfunded and broke!). Building successful products requires knowledge and discipline: knowledge of what to build (features) and how to build it (usability), how to talk about it in a way that drives growth (marketing), what features to focus on at what stage (product triage), and so much more.

Without taking an opinion on whether or not investing in a new Scala build tool is a great idea, let me outline my thoughts on what such a build tool should focus on to succeed. Rather than focus on obvious features, I'll cover some of the less obvious but crucial features.

In no particular order:

### Build-as-Data
 
A build file should be pure data, not code, a mistake that SBT and other build tools have replicated. The idea of expressing a build in a general-purpose programming language is attractive to generalists, but is a massive drawback to tooling, which wants to be able to read and write build files (this is especially important for IDEs).

In addition, build tools powered by programming languages trade off usability for expressive power that is utilized by almost no one, and which can be obtained in other ways (plug-ins). A user creating a new build file does not want to start from an empty Scala file, having no idea what to type or why. They want a few obvious and build parameters they can set. They want a Scala build-specific "language" that's so highly constrained, it's obvious what to write. 

It's important that the data be in a standardized format, which can be written and read by tooling. Moreover, unlike Bazel, I would say it's important the build file format be high-level, and suitable for humans to create and maintain. Finally, it's important the format be textual, so that developers can work on independent changes to the build file, and merge them in using standard Git-based workflows and tooling. 

A proprietary binary format for the build file (or even a textual format that doesn't merge well using version control) would be a catastrophic mistake, even if advanced front-end tooling existed to create and maintain such build data.

### All-in on Scala

Let's face it: many programmers like to abstract. They like to build platforms, not products. They like the general, not the specific. This is doubly-true for developers who are attracted to Scala, because one of the main reasons to use Scala is its immense power of abstraction.

Making a Scala build tool that is so general-purpose it can build any project in any language is a gigantic mistake. Because there are already best-in-class multi-language build tools, which are effectively impossible to compete with on hobbyist resources.

It's even a mistake to focus the build tool on "generic Scala", without deeply integrating into the existing quirks and features of the Scala languages and compilers. SBT, despite being a Scala build tool, knows almost nothing about Scala, forcing developers to hard-code aspects of the Scala compiler and language into their build files, repeated endlessly across all projects.

A Scala build tool should have first-class support for the following Scala-specific features:

 - **Platform**. Scala already supports three platforms, including JVM, JS, and native (via LLVM). In the future, Scala may support more platforms. A Scala build tool should allow developers to specify which platforms a project supports, including compiling overlapping but slightly different code bases for each platform. The build tool should also know that when targeting the JVM, you can choose to target one or more versions of the JDK, and that some versions of the JDK might require new or different source files (for example, supporting Loom in JDK 19 or later).
 - **Versions**. Scala exists in multiple flavors, which are incompatible with each other, and all important to support: 2.12, 2.13, and the radically different 3.x line, with potentially more versions down the road. A Scala build tool should allow developers to specify which of these versions they support, including compiling overlapping but slightly different code bases for each platform (for example, one file may compile under Scala 2.12, but another one might be required for both Scala 2.13 and Scala 3).
 - **Warnings**. A Scala build tool should know about and standardize warnings across different versions of Scala. Warnings are important to configure for all major projects, but some Scala versions only support some warnings, and right now developers have to configure them as error-prone strings without any guidance from the build tool.
 - **Experimental Features**. A Scala build tool should know about and standardize experimental features, which can be turned on and off in different versions of Scala.

### First-Class JVM

The JVM is still the only commercial-grade backend platform for Scala. As a result, any new Scala build tool must have strong support for the JVM.

With SBT, a common experience is exhausting all heap or metaspace while compiling. Diagnosing these problems takes expertise and knowledge of the JVM. Fixing them requires arcane and mostly undocumented knowledge, passed from developer-to-developer by copy/paste and lore.

A Scala build tool should, for itself, as well as the JVM target, have explicit knowledge and support for common JVM parameters, including heap size, metaspace size, stack size, and so forth. It should be a simple per-project setting to choose how much space to allocate to compiling or testing a project.

### Batteries Included

A new Scala build tool must resist the temptation to push all functionality into ecosystem projects, which may or may not exist. Rather, a new Scala build tool must directly tackle the two-sided market problem by baking in the features common across all Scala builds: features that are so wholly indispensible, if your build tool doesn't have them, it won't get adoption.

These features include:

1. Starting a REPL for a given project (and platform/version)
2. Testing a project, with integrations for common Scala test frameworks
3. Packaging a project, including generating Scaladoc 
4. Publishing a project to a repository
5. Running main functions
6. Producing containers

Testing, packing, and publishing should be CI-friendly, with the realization that the CI tool (e.g. Github Workflows, Jenkins, etc.) will provide authentication details for wholly automated workflows.

These features, some of which are trivial, while two are more involved (publishing packages and building containers) are a compelling feature set that is sufficient to drive adoption, assuming the build tool is in reasonable shape otherwise.

### Performance

A new Scala build tool should be committed to performance. Especially as performance is the main reason companies abandon Scala build tools. 

A fast build is a delight, and if Scala has its own build tool, it should focus on delighting developers.

### Plug-ins

A new Scala build tool should support plug-ins. Developers universally push back against plug-ins, because designing a plug-in architecture is complex, and maintaining it is expensive. 

However, there is no possible way SBT could have become the dominant build tool in Scala without plug-ins. Plug-ins allow ordinary developers to reuse the build and deployment features common across large chunks of the ecosystem, thus helping them rapidly assemble and tweak their builds to their specific needs, no matter how complex.

It might be possible to support plug-ins as ordinary command-line processes, which are fed details from the build process. However, care should be taken with this approach, because if a Scala developer builds a JVM-based plug-in, then startup times could be prohibitive (there's nothing to say, however, that plug-ins should be developed in Scala, even though it will be the first choice for many).

Any new Scala build tool that rejects plug-ins on grounds they are "too hard" will fail with the largest and most commercially relevant projects (even though it could obtain some traction for toy / hobbyist projects, which have unsophisticated needs met easily without plug-ins).

### Bonus Points

A new Scala build tool should, ideally, be geared at *eventually* solving two other concerns:

1. **Package Management**. Publishing of packages on Scala-specific repository. For too long, Maven has been abused for publishing Scala packages. But with Scala going cross-platform, and having completely different compatibility considerations, a Scala-specific packaging solution could be a far better fit. In addition, Maven is old-school and it's difficult to host or find a free host to publish new libraries, and this discourages new Scala developers who are also new to the JVM.
2. **Language-Server Protocol**. A build tool already has to interface with the Scala compiler, and has intimate knowledge of the build settings. An LSP server could be directly integrated into a build tool, improving performance, simplifying architecture, reducing points of failure, reducing code size, and making maintenance easier.

These concerns don't have to be tackled right away, but they represent the sort of bold new thinking that's required to effectively utilize limited resources to produce a best-in-class experience for Scala developers.

## Summary

In this blog post, I discussed the three potential paths to creating a best-in-class build tool for Scala: ditching Scala build tools in favor of multi-language build tools, doubling down on existing build tools such as SBT, or creating a new Scala build tool from scratch.

Of these paths, I believe that only the first and third are viable. Fixing what's wrong with SBT would require changes of such magnitude, it's effectively a new build tool, anyway.

If we end up creating a new Scala build tool (from an existing or new code base), I believe that it must focus on build-as-data, providing first-class support for Scala-specific features, embracing the JVM, offering batteries-included functionality, prioritizing performance, supporting plug-ins, and eventually solving packaging and language-server protocol concerns.

In the [ZIO ecosystem](https://github.com/zio/), we are hungry for a new build tool, and at [Ziverge](https://ziverge.com), we have clients who are frustrated with existing Scala build solutions.

Even though I have no intention to directly solve Scala's build tool problem, I will be keeping an eye on developments in the space, and if I see significant investment into Pants or Gradle, or if I see a new Scala build tool that is getting the right foundation, then I will not hesitate to support these investments, including migrating ZIO ecosystem projects, sponsoring development of features, and publicly endorsing and evangelizing such solutions.

