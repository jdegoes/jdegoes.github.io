---
layout:       post
title:        "The Rise (and Fall?) of NoSQL"
description:  "NoSQL is exploding in popularity, with serious ramifications for the entire database industry."
category:     articles
tags:         [nosql, mongodb, oracle, json]
---

If you haven't heard of the meteoric rise of NoSQL, you've been living in a hole. Likely at the bottom of the ocean. In the Mariana trench. Buried under a heap of rubble.

NoSQL adoption is exploding, and not just in the startup scene. Even big companies like Apple and Comcast have skin in the game, with large NoSQL deployments that probably dwarf those at your company.

MongoDB, the most widely-adopted NoSQL database, [recently raised](https://www.crunchbase.com/funding-round/5576d45dd1a3b6f606ba5c478660a4e3) $150 million dollars on a **$1,200,000,000** dollar valuation. 

Yes, that's more than a billion dollars for a "boring" database company built around pure open source software!

What you probably *haven't* heard, what's been *lost* in all the hype, is that NoSQL's unprecedented growth has very little to do with "big data" or "SQL"!

Sure, many NoSQL systems are "big data", but the vast majority of actual deployments are "small data". In fact, these days, plenty of [SQL databases](https://www.splicemachine.com) are "big data", too.

And sure, many NoSQL systems don't use SQL as their primary interface. After all, this is where the term *NoSQL* originally came from (since then it's been retconned to mean "Not Only SQL"). [But some](https://www.couchbase.com/communities/n1ql) NoSQL databases do use SQL as a primary query interface, and not all relational data tools use SQL as their primary interface (remember Excel, the #1 BI tool in the world?).

To *really* understand the reason for NoSQL's popularity, you have to look a lot further than just *big data* and *not SQL*.

## The Scoop on NoSQL's Hotness

It's well-known that the time required to develop a new application has been steadily falling. All thanks to new technologies, the cloud, and the growth of open source.

The proverbial developer in a garage can now crank out a fully featured app in a weekend &mdash; an app that used to take *months* for a whole team of engineers. That's astounding when you think about it.

NoSQL databases *accelerate* this trend even further. With NoSQL:

* Developers can stuff any kind of data into their database, not just flat, uniform, tabular data. When building apps, most developers actually use *objects*, which have nesting and allow non-uniform structure, and which can be stored natively in NoSQL databases. *NoSQL databases fit the data model that developers already use to build applications.*
* Developers don't have to spend months building a rigid data model that has to be carefully thought through, revised at massive cost, and deployed and maintained by a separate database team within ops.

The pervasive theme here is that NoSQL *empowers* developers to rapidly create new applications or change existing applications &mdash; an order of magnitude faster than they could do with legacy technologies.

Indeed, I'd argue that the macrosopic trend behind *many* of today's buzzwords (cloud, APIs, devops, PaaS, etc) is *developer empowerment* (AKA "The Rise of the Maker").

As [Google Trends](https://www.google.com/trends/explore#q=mongodb%2C%20oracle%20db&cmpt=q) illustrates (albeit unscientifically!), more and more developers are turning to these databases to build new applications. MongoDB alone has been downloaded more than 7 million times (with 2 million of those downloads occurring in the last 6 months)!

The large database vendors either haven't tried their hand at NoSQL, or haven't seen adoption like MongoDB. 

It's true that some vendors are *genuinely* trying to innovate. But others are so busy selling licenses, products, and solutions based on 40 year old legacy technology that the *biggest disruption* in databases since the invention of the RDBMS is happening right under their noses!

## The Achilles' Heel of NoSQL

While adoption is staggering, not everything is rainbows and sunshine in the land of NoSQL. NoSQL databases have a *critical vulnerability* that could ultimately curb or even reverse their exponential growth.

It's quite simple: analytics tooling for NoSQL databases is almost non-existent. Apps stuff a lot of data into these databases, but legacy analytics tooling based on relational technology can't make any sense of it (because it's not uniform, tabular data).

So what usually happens is that companies extract, transform, normalize, and flatten their NoSQL data into an RDBMS, where they can slice and dice data and build reports.

The cost and pain of this process, together with the fact that NoSQL databases aren't fully self-contained (using them requires using their "competition"" for analytics!) is the biggest threat to the possible dominance of NoSQL databases.

If RDBMS vendors get their act together and move closer to NoSQL databases, while still preserving their compatibility with legacy analytic toolchains, they may be able to slow or even reverse the tide of adoption.

PostgreSQL is leading the way here, even though it has a *long* way to go. An open source RDBMS developed by and for developers, recent additions make  PostgreSQL behave more like NoSQL databases and less like traditional RDBMS technology (albeit at the cost of breaking compatibility with existing analytic toolchains, which sort of levels the playing field).

## What's Next?

NoSQL adoption is currently driving the creation of billion dollar database companies, the likes of which have not been seen in *decades*. The money being poured into this sector will spawn other billion dollar NoSQL industries, in much the same way that RDBMS spawned BI, ETL, data warehousing, and many others.

It's a *massive* landgrab right now, as every NoSQL vendor tries to carve out the largest set of use cases that their technology can handle.

In the next few years, expect major database vendors to snap up the players who have seen substantial traction, but who haven't made it big enough to go public (or in some cases, even those who *do* go public!).

Earlier this year, [IBM acquired Cloudant](https://www-03.ibm.com/press/us/en/pressrelease/43342.wss) for a tidy sum. That's not the last acquisition you'll hear about, but the first of many.

The lack of analytic tooling that natively supports NoSQL databases is a serious issue to continued adoption. Either this will create a market for a new wave of analytic companies (much like the major BI and analytic database vendors were born in the wake of the original RDBMS solutions), or it will curb the adoption and force RDBMS to become more like NoSQL.

In either case, the database world is never going to be the same again. 

Strap yourselves in for one heck of a ride.
