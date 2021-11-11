---
layout:       post
title:        "Improvements to the ZIO Organization"
description:  "The ZIO organization needs to use its strong position in the Scala community to bind together all Scala developers for a common good"
category:     articles
tags:         [typelevel, functional programming, scala, zio, community, open source, leadership]
---

In the past few months, the TypeLevel organization has become increasingly hostile toward the ZIO community (whose software directly competes with TypeLevel):

 - Rob Norris [removing](https://github.com/tpolecat/doobie/pull/1587) an existing Quill integration from Doobie because Quill decided to move to the ZIO organization
 - Rob Norris [opting to not help a user](https://github.com/tpolecat/skunk/issues/438) specifically because they were using Skunk with ZIO
 - Ross Baker and Christopher Davenport [rejecting a ZIO integration](https://github.com/http4s/http4s/issues/4718) on grounds that ZIO is "inconsistent with our values"
 - Rob Norris and Michael Pilquist referring to ZIO as `Z**` or `*IO`, with Rob Norris [leaving](https://twitter.com/tpolecat/status/1424368733976416257) the [Scala Discord](https://discord.gg/scala) because ZIO was [mentioned](https://twitter.com/nafg613/status/1424542295244873730)
 - Oscar Boykin [attacking](https://twitter.com/flaviusbraz/status/1456615716619055106) both an Orthodox Jew and a Latino who contribute to Quill and ZIO

As an open source developer myself, I am keenly aware of the fact that OSS is a "nights and weekends" hobby for most of us. We freely volunteer our time, often sacrificing other pursuits in order to contribute to open source and support our end-users.

As a result of this reality, I whole-heartedly support the right of every OSS developer to contribute on their own terms, whatever those terms may be. Non-paying users of free software should _not_ get to dictate these terms.

At the same time, I recognize that the actions of the TypeLevel organization may have undesirable side-effects on the Scala community:

 - They undermine the trust that end-users and companies place in Scala OSS
 - They increase the risk involved in deploying solutions based on Scala OSS
 - They decrease the network value of Scala OSS
 - They make Scala look unprofessional to many developers, especially compared to Java or other ecosystems where major OSS projects would never behave in such a fashion

As a passionate fan of the Scala programming language, a well-known advocate for functional programming, a leader in the ZIO open source ecosystem, and a founder of a [company devoted to Scala OSS](https://ziverge.com), I want Scala to prosper.

I have no ability to influence the TypeLevel organization. However, as [BDFL](https://en.wikipedia.org/wiki/Benevolent_dictator_for_life) of the [ZIO organization](https://github.com/zio), I am in a position where I can learn from these behaviors, and implement changes within the ZIO organization that I believe will benefit the whole Scala community.

Effective immediately, I hereby codify and will support the following:

 - **Pro-Community**. All ZIO projects hosted within the [official ZIO organization](https://github.com/zio) will gladly accept and host integrations for Akka, TypeLevel, and other Scala or JVM ecosystems, without consideration of the relationships, dispositions, or politics between these projects and those ecosystems, and they will provide non-discriminatory support for end-users, regardless of their disposition to or affiliation or association with other Scala community members or ecosystems.
 - **Pro-Professionalism**. Although behavior within ZIO organization projects is already governed by the [Scala Code of Conduct](https://scala-lang.org/conduct/), I want to strengthen this code of conduct by making it clear that ad hominem and career sabotage has no place in the ZIO community. Projects in the ZIO organization exist only to help developers solve problems, regardless of their race, sexual orientation, gender identity, or disposition to or affiliation or association with other Scala community members or ecosystems.

ZIO is already doing a great job at being _pro-community_, as ZIO JSON hosts an integration for the TypeLevel projects _http4s_ and _Refined_, and ZIO Core hosts integrations for _Cats Effect 2_, _Cats Effect 3_, and _Akka Cluster_ (among other integrations too numerous to list). However, I believe that making this behavior official will further increase trust and expand integrations, and also clearly set expectations for new projects coming into the organization.

As for being _pro-professionalism_, within ZIO official spaces (Github, Discord, etc.), I have only ever seen welcoming, inclusive, and non-discrimantory behavior, without ad hominem or career sabotage. But explicitly committing to this high-standard of professionalism can only help to set expectations and provide guidance for leaders as the organization continues to grow.

I encourage other Scala open source organizations to follow ZIO's lead, and adopt these _pro-community_ and _pro-professsionalism_ measures, without imposing any negative repurcussions on Scala open source contributors (including TypeLevel) who choose to take a different path (OSS entitlement is a [real problem](https://medium.com/@fommil/the-open-source-entitlement-complex-bcb718e2326d)).

As [Martin Odersky](https://en.wikipedia.org/wiki/Martin_Odersky) once wrote, [Scala is a big tent](https://www.scala-lang.org/blog/2019/05/02/community.html). It is not necessary that open source contributors have the same views or even like each other. But if we put Scala first, by being pro-community and pro-professionalism, we can find a way to co-exist peacefully inside this big tent, and together, we can, in different ways and with different audiences, show the world that Scala is a force to reckon with.