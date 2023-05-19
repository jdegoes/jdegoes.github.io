---
layout:       post
title:        "A Splendid Scala Journey"
description:  "My heartfelt thoughts, reflections, and notes on my 15 year Scala journey, and on what's next for me"
category:     articles
tags:         [scala, scala3, tooling, ides, functional programming]
---

I have been a Scala developer for 15 long years, which makes me one of the oldest Scala programmers around.

My journey began when, as VP of Engineering at a San Francisco startup, I needed to choose a programming language for our new ad-tech platform. 

Without hesitation, I chose Scala, because I had a delightful experience dabbling with functional programming in Java, where I had spent the previous 10 years mastering object-oriented programming.

Unfamiliar with Scala, I delved into Odersky's book, devouring it cover-to-cover during my weekly commutes between Denver and San Francisco

The more I learned about Scala&mdash;its lambdas, futures, actors, generics, powerful collection library, and the expression-orientation of the language&mdash;the more enthralled I became. I felt like a kid in a candy store, delighting in each new discovery!

In the 15 years since that fateful decision to build an ad-tech platform in Scala, I have built 3 more companies on Scala, spent more than 20 million on Scala engineers, and built the [large, thriving Scala ZIO ecosystem](https://zio.dev), which has gone on to inspire ports in TypeScript, Kotlin, F#, and other programming languages.

Through countless talks, articles, 1-on-1 mentorship, and educational workshops, I have become a vocal Scala evangelist and champion, tirelessly advocating for Scala within industry. Thanks to my efforts, tens of thousands of developers around the world have learned Scala and leveled-up their skills and knowledge.

I witnessed the rise of Spark and Twitter Scala, the evolution and transformation of Lightbend (formerly Typesafe), and so much more.

Yet, despite these successes and my deep-rooted passion for Scala, I find recent trends drawing me towards new horizons.

## State of Scala

Ever since 2017, I have observed a decline in Scala's adoption within industry&mdash;a trend I [first spoke about in 2018](https://www.youtube.com/watch?v=v8IQ-X2HkGE). I've discussed Scala's adoption challenges more recently in a [post](/articles/scala-resurrection) and [several](https://twitter.com/jdegoes/status/1652940041524486145) [Twitter](https://twitter.com/jdegoes/status/1656566825356754945) [threads](https://twitter.com/jdegoes/status/1658801254196232195).

The transition from Scala 2 to Scala 3 has proven extremely challenging for the industry. Scala 3 is essentially a new language, and upgrading large projects that rely on unmaintained libraries or macros is not a simple task. The majority of commercial users are still on Scala 2 and lack significant incentives to upgrade to Scala 3. Confronted with investing months into upgrades that offer no new business value, many companies are looking to alternative languages that better meet their needs.

Further complicating matters, Scala 3 has had numerous bugs, including compiler crashers, and has seen rapid evolution. It has introduced new features that the industry isn't demanding and still lacks industry-grade tooling. These issues risk [burning out organizations](https://www.reddit.com/r/scala/comments/12zhsoz/is_scala_losing_ground_in_your_organization/), developers, and OSS contributors.

Despite my ongoing efforts to champion and evangelize Scala, watching the decline in adoption year after year has left me with a mix of frustration and sadness. Frustration, because I wanted to address the core challenges hindering its adoption; sadness, because I longed for Scala to become mainstream.

Initially, I tried to influence change by becoming a commercial sponsor of the official Scala Center and joining the advisory board. When that didn't happen, I started speaking candidly about Scala's challenges and sharing my thoughts on how to improve adoption.

Today, I can say without hesitation that I have done literally everything that I know how to do. I have said everything I can possibly say, even at significant personal cost, in order to maximize the chance of Scala's growth.

Letting go of something you deeply love is very difficult, especially if you believe (rightly or wrongly) that you understand the problems at hand and how to address them. The struggle has been genuine and often painful for me.

Yet, acknowledging that I've given my all and there's nothing more to be done has led to a gradual fading of my frustration and disappointment.

Today, I'm neither frustrated nor disappointed. Instead, I feel a growing sense of acceptance and inner peace.

## The Future of Scala

At this point, I do not know if Scala will grow within industry, or if it will become a boutique programming language that a few lucky companies wield for its incredible expressive power and strong type-safety&mdash;paying for it, as they must, with the taxes demanded by all boutique programming languages.

Looking at Clojure, I appreciate that even though Clojure is a boutique programming language, it is still a beast of a programming language, which gives its commercial users a competitive edge, primarily through the senior talent it attracts, who is able to effectively channel its powers.

If the future of Scala is similar to Clojure, then I fully accept it. This future was not my preferred choice for the language, but it's still a win for some savvy companies and talented developers, and that's enough for me.

Moreover, there's another, more positive way to look at Scala 3 adoption. Scala 3 is essentially a new programming language. Viewed in this light, its adoption and tooling are actually quite impressive!

Whatever happens to Scala, I know that it is not going to disappear. There will always be companies using Scala. Whether the market is large or boutique, I accept the fate of Scala, without reservation or complaint.

Que sera, sera.

## My Future with Scala

In light of the shifting market, as well as my long tenure in the Scala community, I am officially stepping down as Scala evangelist and champion. 

I no longer consider myself a dedicated Scala developer, and I will no longer lobby for any Scala improvements, whether at the language or organizational level. I will not involve myself in any discussions regarding the language, standard library, or tooling (though I may promote or sponsor tooling). After 15 long years, it's time for me to make room for the next generation of highly-capable Scala champions.

In response to these changes, I am diversifying my educational content, open source contributions, and mentorship, expanding into other programming languages. I'm especially interested in new languages that, like Scala, empower us with expressive power and support us in writing correct code.

Let me be clear: I still have a deep affection for Scala, both the 2.x and 3.x dialects. The language has been my trusty steed, and together we have fought and won many battles in industry.

However, while the past 15 years of my life have been dedicated almost exclusively to Scala, the next 15 years (should I be so fortunate) will expand beyond Scala and to new technologies.

I am grateful for having the chance to grow as a software engineer using such a powerful, type-safe programming language as Scala. Simultaneously, I am excited about the new journey ahead and the opportunities it presents for personal and professional growth.

Currently, I'm thoroughly enjoying writing some Rust (alongside Scala), and I look forward to teaching a [Rust workshop](https://www.eventbrite.com/e/learn-the-rust-programming-language-by-john-a-de-goes-tickets-630610863067) for anyone interested in learning this powerful, type-safe, and remarkably efficient programming language.

## The Future of ZIO

As I am the founder of the ZIO ecosystem, it's reasonable to ask: _How will this new journey affect the ZIO ecosystem?_

To begin with, it's crucial to recognize that even though I currently serve as the BDFL of the ZIO organization, I am only _one_ contributor among hundreds. The significant contributions made by other skillful and productive developers have long surpassed my own.

In the early days of ZIO, yes, it was primarily _me_, working tirelessly during nights and weekends. Today, however, ZIO comprises hundreds of contributors from dozens of companies, including multiple Scala consultancies. ZIO has grown to be far larger than any one person, myself included. 

These days, I am not even involved in the majority of ZIO projects!

Next, despite my upcoming ventures beyond Scala, I have no plans to stop contributing to the ZIO ecosystem, both directly and through mentorship of new contributors. Currently, I am heavily involved with _ZIO HTTP_, which I believe is a game-changer for ZIO users. I will continue pushing the ZIO ecosystem forward and supporting those who share this goal.

As for my own company, [Ziverge](https://ziverge.com), we continue to offer commercial support contracts for ZIO, as does [Scalac](https://scalac.io). Both companies provide commercial training, feature acceleration, and priority fixes across the whole ZIO ecosystem. With ZIO's monthly downloads doubling to 8 million in the past year, growth remains strong, and this momentum will certainly drive further improvements.

Regardless of whether Scala's adoption surpasses 2017 levels or settles into a small but highly skilled community like Clojure, functional Scala developers will always need a technology stack to rapidly build cloud-native applications. It is my goal for ZIO to remain one of the top choices!

## A Moment of Reflection

Looking back at my 15-year journey with Scala, I recall mostly good times. However, I have also navigated through rough patches, which have provided me ample opportunities for introspection.

I've had [two talks of mine cancelled](https://www.linkedin.com/pulse/dark-side-speaking-tech-conferences-john-de-goes/), I was shadow-banned from ScalaDays, and I endured years of [cyber-stalking, defamation, and harassment](/articles/travis-brown-abuser). Over the years, personal attacks, false accusations, and attempts to harm my professional relationships have become sadly familiar occurrences. Some have refused to provide proper attribution, a basic courtesy in our field.

Sometimes, the Scala community can feel like a magnet for drama, and I acknowledge that I have unintentionally contributed to this atmosphere at times, through either my choice of topics or the language I used.

When we suffer wrongs, we naturally experience hurt and anger, and we yearn to fight back. Despite my best efforts to manage these feelings, I acknowledge that I occasionally struggle with them, and these emotions might bleed into my words.

However, I remain steadfast in my belief in and practice of forgiveness. This is not merely because releasing negative emotions is healthy for us, but also because we ourselves need forgiveness. To be forgiven, we must forgive.

Forgiveness does not imply discarding the boundaries that protect us from further harm. Nor does it require trusting someone who has broken our trust. Forgiveness entails releasing negative emotions, allowing them to flow through us rather than get lodged within us, and ceasing our attempts to retaliate.

To those who have, in my eyes, wronged me&mdash;whether intentionally or not&mdash;I unconditionally forgive you, with no expectations in return. If I have said or done anything to hurt you, even unintentionally, I am open to a personal conversation to own my part.

**To Odersky**: 

I extend my heartfelt gratitude to you for creating such a powerful, type-safe language that has managed to engage me deeply for a span of 15 years.

You paved the way for object-oriented programmers like me to explore the magical new realm of functional programming. This shift has drastically altered my perspective and elevated my skills.

We do not always agree. In fact, our disagreements could probably fill a small book! But, regardless of our differences, I am profoundly thankful that you chose to create and share the Scala programming language with the world. Its influence on other ecosystems ensures Scala's legacy will continue for decades.

Please bear in mind that when developers appear somewhat harsh in their feedback, it's often driven by a deep-seated love for Scala. They might feel frustrated because they can't shape the language as they wish. Every technology has its frustrated lovers, who should resist the temptation to become haters.

I have come to terms with whatever direction Scala takes from here. I understand the benefits of both language experimentation and stabilization. While I might place greater weight on one over the other, I recognize that each holds value for different audiences and for diverse reasons.

I wish you, and the entire team at EPFL, the very best for the future!

**To the Scala Center**:

I have not expressed my gratitude nearly as much as I should have, but I want to acknowledge the immense contributions you make to the Scala community. Without the Scala Center, we would be in a far less favorable position.

Over the past few years, your mission to expand Scala's adoption has been my own personal mission. Although I may have strong opinions (perhaps unreasonably so!) on how to achieve this goal, I fully recognize that it is, ultimately, _your responsibility_ to decide how to advance your mission.

Even though we have had disagreements on focus and organization at times, I wholeheartedly want you to succeed in your mission. With the many talented engineers in your organization, I am confident you possess the necessary engineering prowess to tackle some of Scala's challenges.

I extend my best wishes for your success, and I stand prepared to personally contribute financially to your organization should you so desire.

**To SoftwareMill**:

I want to express my deep appreciation for all that you contribute to the open-source community. Your contributions to the Scala ecosystem have been invaluable, providing enormous value to Scala developers.

Rather than dwelling on recent incidents, I will strive to remember the numerous good times we shared.

My heartfelt congratulations on your merger, and I sincerely hope that both SoftwareMill and VirtusLab achieve great success, not only within the realm of Scala but beyond it as well.

**To Typelevel**:

I want to take this opportunity to appreciate the significant influence you've had on the advancement of pure functional programming in Scala.

Your tireless effort has led to the development of a rich ecosystem of purely functional software, going far beyond Scalaz, and providing substantial value to the industry.

We may always disagree over `F[_]`, type-class-first development, the names of type classes and methods, and occasionally, concurrency, interruption, and back-pressure semantics.

But even if I have spent many words devoted to our differences, ZIO and Typelevel are more similar than different.

We both have seen and believe in the power of pure functional programming to solve real problems. We both believe in the power of programs-as-values.

I miss the times when we would grab beers in some bar in San Francisco and complain about `PartialFunction[Any, Unit]`. What happened to those days?

Well, a lot happened, and if I am being honest with myself, we each contributed to the outcome. For my part, I wish I had argued for improvements in a more skillful way that made everyone feel good about the countless, thankless hours they poured into open source.

We always agreed more than we disagreed, and I know I appreciated so much about the early design of Cats, FS2, and IO, but maybe I should have said this early and often to balance out my technical criticism.

I probably should have dialed down my marketing for ZIO, more diligently pointed out the usual disclaimers with micro-benchmarks, and more frequently complimented you on the innovation occurring in your ecosystem (polling IO? Crazy cool! Smithy4s? Damn!).

I wonder if, through conversations that could only ever happen in person, we had found constructive, and uplifting ways of achieving our community-oriented goals. Could I have addressed your concerns, and could you have addressed and dealt with the negative experiences of my friends in the Scala community?

I don't know if we would have found common ground, but we probably should have tried to have those conversations, face-to-face.

In any case, even though it's beyond my power to make it happen at this point, I think ZIO and Typelevel working together could help Scala, and perhaps this can happen through the new generation of Scala open source contributors, who don't have our baggage, and just want to see functional Scala succeed.

Whether that collaboration involves consolidating libraries that have a similar purpose, or working together on a _Better For_, the functional Scala community is strongest united on the side of *every* functional Scala developer.

Regardless of what happens in the future, I wish all of you the best of luck.

**To the ZIO community**:

I am so happy and proud to be part of the ZIO open source ecosystem.

I love the fact that all of you are constantly challenging existing patterns and practices in functional programming, producing software that is powerful, practical, functional, accessible, and joyful.

I love the culture of the ZIO community. Not only are you talented, but you are open to work with everyone, non-judgmental, positive and upbeat (something I've failed at recently!), and eager to welcome and mentor all.

I've been completely blown away by the powerful, elegant libraries that you have created (ZIO Query, ZIO HTTP, Shardcake, Caliban, ZIO ElasticSearch, etc.), and the many new contributors that you have cultivated around these libraries.

Over the past couple of years, ZIO developers and users have been my primary motivation to invest into Scala. Even as I begin a new journey, expanding my horizons beyond Scala, the ZIO community remains the shining light that will keep me hooked on contributing to Scala open source.

I am eternally grateful to all of you for your kindness and support, for the amazing libraries that you create, and for your feedback using ZIO libraries, which help us make our open source libraries even better.

_"ZIO soon,"_ as they say, to all my friends in the ZIO community! We need to finish that Spring killer, and we're getting awfully close!

## Summary 

As one of the more seasoned Scala developers in the community, my 15-year journey with Scala has been an enriching experience. I've built four companies, employed numerous Scala developers, nurtured a flourishing open source ecosystem, and advocated tirelessly for Scala as a commercially viable functional programming language.

Despite my best efforts, industry adoption of Scala has faced certain challenges. After much deliberation, struggle, and public discourse, I've come to deeply accept this reality. Scala may well follow a path similar to that of Clojure. This wasn't my initial vision for the language, but it's still a positive outcome.

In recognizing this shift, I am officially stepping down from my role as a Scala evangelist and champion, and am warmly passing the baton to the next generation&mdash;those brilliant young minds tirelessly working on Scala's compiler, tooling, and open source ecosystem. This new generation, with its fierce spirit and boundless energy, is surely up for the challenge.

While my love for Scala is unchanged, and I will continue to happily write Scala, my journey will take me to new places, and on new adventures.

The ZIO ecosystem will continue to flourish under the diligent stewardship of countless talented and committed individuals. Commercial users will still have access to support, training, feature acceleration, and priority fixes from two reputable consultancies. I will maintain my involvement by contributing to and guiding core projects.

As I embark on an exciting new journey, I reflect with profound gratitude on the extraordinary saga that has been my very splendid Scala journey.

Consummatum est.
