---
layout:       post
title:        "Everything You Know About the Blockchain Is Wrong"
description:  "Prepare to unlearn everything you have learned to discover what blockchain is really about."
category:     articles
tags:         [blockchain, bitcoin, ethereum, eth, btc, fp, functional programming, oss, floss]
---

You don't understand blockchain.

Well, _maybe_ you do. But if you don't see what all the fuss is about, wonder why anyone uses blockchain technology instead of Postgres, or think Tor figured out decentralization long before Bitcoin, then I have some news for you:

*Every damn thing you know about the blockchain is wrong.*

In this post, I going to talk about what I see as the top 8 myths of blockchain. Be prepared for bold claims and new ways of looking at the space.

## Myth 1: Blockchain Is Digital Currencies

Many early applications of blockchain technology have been directed at the creation of digital assets that can be used as currencies (more precisely: de-centralized, consensus-driven, append-only ledgers).

Blockchain technology itself, however, is neither about nor restricted to the creation of digital currencies.

In the most general possible sense, _blockchain technology_ refers to _a mathematical innovation that allows us to incentivize independent parties in an untrusted, purely consensual network to provide well-defined, agreed upon services_.

> "Blockchain technology refers to a mathematical innovation that allows us to incentivize independent parties in an untrusted, purely consensual network to provide well-defined, agreed upon services."

It is hard to overstate just how important this innovation is. One of the oldest problems in the history of civilization is figuring out how to get different parties to work together.

Previously, legal markets have heavily relied on _physical force_ to incentivize independent parties to provide (contractually) agreed upon services. Indeed, this is one of the important functions of governments.

Force works reasonably well for trustworthy parties in the same jurisdiction, but is costly and slow. Force doesn't work in cases where the parties are untrusted, in cases where the parties span different jurisdictions, or in cases where speed or low costs are critical.

Blockchain technology provides a shocking answer to one of civilization's oldest questions. Without force&mdash;indeed, with just math&mdash;we can engineer cooperation between different parties in a well-defined way.

Classifying the set of viable blockchain solutions is not trivial, but it's clear based on this definition that any problem uniquely solved by blockchain technology will involve some combination of untrusted parties, multiple jurisdictions, high speed, and low costs.

It should also be clear that some solutions currently marketed as blockchain solutions are not actually blockchain solutions, _per se_, even though they share some of the underlying math (zero-knowledge proofs, homomorphic computing, lattice cryptography, etc). All blockchain tech involves cryptographic tech, but not all cryptographic tech involves blockchain tech.

## Myth 2: Tokens Are Currencies

Tokens, such as Bitcoins, can be _used_ as currencies, but fundamentally, tokens are _not_ currencies, but _capabilities_.

Possession of tokens gives you the _capability_ to avail yourself of the well-defined, agreed upon services provided by a blockchain technology. _Incidentally_, people may be willing to give you some form of currency in order to acquire a token, if they want those services, or if they are speculating on the value of the tokens.

For _pure cryptocurrencies_ such as Bitcoin, the services revolve around providing a global, distributed ledger _for the tokens_, which blurs the line between capability and currency. But the world doesn't need more than a few pure cryptocurrencies, so the majority of successful blockchain applications will _not_ be pure cryptocurrencies.

A better analogy for tokens is _corporate stock_: just like stock gives you capabilities to avail yourself of the services provided by the corporation to stockholders (such as the right to vote, right to dividends, etc.), tokens let you avail yourself of the services provided by the blockchain platform.

Stock usually has real value, because you can often exchange it for money, but it's less a currency than it is a set of rights to do something.

## Myth 3: Blockchain Isn't Scalable

Today's blockchains are generally _not_ scalable. Bitcoin can handle a few transactions per second. Ethereum can handle about five times that amount. To give you a sense for how terrible this performance is, Visa handles 65,000 transactions per _second_!

Now, since _tokens are not currency_, blockchain platforms don't _necessarily_ need to scale to the level of payment systems. That said, it's clear numerous applications require performance orders of magnitude greater than currently offered, as well as far cheaper storage and compute than currently possible on any existing blockchain.

Nonetheless, there are no theoretical reasons why even public blockchain technology cannot scale. Ongoing work in proof-of-stake consensus algorithms, sharding, efficient routing, reputation networks and heterogeneous (specialized) networks, all provide a clear (if difficult) path toward building highly scalable blockchain platforms.

Blockchain platforms can be thought of as highly-constrained, specialized databases. First generation systems are architecturally similar to flat file managers (how data was stored prior to databases). Later systems will be more architecturally similar to highly-scalable databases like Google Spanner.

## Myth 4: Blockchain Platforms Will Replicate Tech Stacks

Many people who are sold on the promise of blockchain technology imagine that the distributed applications of tomorrow will be built on blockchain platforms that replicate the form and function of today's tech stacks.

Just like today's applications are built on file systems, databases, message queues, compute nodes, and so forth, many imagine that distributed applications will be built on combinations of services like Filecoin (for storage), IPDB (for database), Ethereum (for logic), and so forth.

Billions have been invested in this vision of the future, which is so utterly and catastrophically wrong, entire blockchain platforms will collapse in a heap of rubble. Or at least be reduced to ghosts of their former selves.

The reason is simple: any distributed application that has significant market value can easily attract enough participation to provide all of the services it needs for itself, on its own blockchain, without having to pass along numerous third-party costs to end-users.

The stack for new distributed applications is _not_ going to be a bunch of blockchain platforms, but rather _open source software_. Developers will assemble their applications from open source components that provide different distributed services, such as queuing and storage.

The only distributed applications that will be built in Frankenstein fashion are those that cannot easily attract enough outside compute and storage&mdash;i.e. the set of unsuccessful distributed applications.

## Myth 5: Blockchains Are Anti-Government

Lots of early vocal blockchain proponents were outspoken libertarians, or even anarchists, which led many to believe that blockchains are inherently anti-government.

Nothing could be further from the truth.

As tokens are not currency, but assets, proceeds from the sale of such assets can be taxed in the same manner as other assets. In addition, if blockchain technology is eventually regulated differently, the capability to provide verifiable audits on blockchain transactions could ensure _perfect_ compliance with regulation, which is a standard no other industry can match.

Blockchain also provides the technology necessary to power many functions of government in a way that can be audited, and yet preserves a tunable degree of confidentiality in transactions. For example, a government powered by blockchain technology could issue currency and tax participants in an extraordinarily efficient, software-defined way, which leaves no room for tax evasion, has none of the overhead of the existing system, and gives citizens trust through tunable transparency.

Blockchains are not anti-government, they are just a new technological tool, one that actually has the potential to radically _improve_ the efficiency and transparency of government, and usher in an era of software-defined (and therefore, software-enforced) regulation.

## Myth 6: Blockchain Is An Append-Only Chain of Blocks

Technically speaking, the word _blockchain_ stems from Bitcoin's append-only chain of blocks, a data structure produced as part of the "mining" process that confirms transactions.

These days, blockchain technology has evolved past linear append-only chains. Sharded systems utilize Directed Acyclic Graphs (DAGs) of blocks, not linear chains, and there is no specific requirement for ever-growing, append-only chains (we'll see other types of chains in the future that preserve the ability to audit but discard some historical information).

While some developers may not like the imprecision or terminology drift, _blockchain_ now refers generically to a _space_ of solutions, not any specific _implementation techniques_ or _data structures_.

## Myth 7: Blockchains Should Be Implemented in Go or C/C++

![](https://i.imgur.com/6VzdZiS.gif)

Given the high stakes involved, blockchain platforms should be written in languages that permit as much static verification as possible, and which allow straightforward and easily verifiable translation of math and formal protocol semantics into executable code.

Languages with strong, static type systems, which possess semantics amenable to formal verification, and which support functional programming (a mathematical style of developing software), are an excellent fit for implementation of blockchain technology.

Haskell in particular is proving to be a [robust choice](https://www.cardanohub.org/en/home/) for implementing new blockchain technology, and other functional programming languages show promise as well (OCaml, Scala).

(Sorry, I couldn't resist!)

## Myth 8: Blockchain Is a Bubble

No one can perfectly predict the future of Bitcoin, Ethereum, and other major players in the blockchain space&mdash;precisely because the success of those platforms depends on people. Developers have to write code, pull requests must be accepted or rejected, and miners must adopt (or not adopt) upgrades.

While it's impossible to perfectly predict what will become of the first-movers in the blockchain space, I feel confident in saying that the market cap of all blockchain technologies today is completely insignificant compared to what it will ultimately become.

In other words, while individual cryptocurrencies may (or may not) be in a bubble, the overall blockchain market is in its early days. All the real growth lies in the future, not the past.

Blockchain is a technology that will change the world forever.

## The Future

In this post, I've outlined what I see as the major myths around blockchain technology that impede communication and funnel investment toward dead-end solutions.

What I _haven't_ done is talk about what blockchain technology is good for.

What are the killer applications for this type of technology, how will they emerge, and how will they be structured? What are the innovations and players we should be paying attention to, and the ones that are safe to ignore? How can blockchain technology provide a radically new way of monetizing some types of open source projects?

All these are excellent topics...for different posts. Let me know what you're interested in hearing about in the comments below!
