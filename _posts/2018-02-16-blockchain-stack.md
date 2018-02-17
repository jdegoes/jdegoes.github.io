---
layout:       post
title:        "Will The Real Blockchain Stack Please Stand Up?"
description:  "Successful blockchain applications won't be built on top of blockchain platforms."
category:     articles
tags:         [blockchain, bitcoin, ethereum, eth, btc, fp, oss, floss]
---

In my recent post on [blockchain myths](/articles/blockchain-myths), I argued that successful blockchain applications will *not* be built on blockchain platforms (Myth 4).

In this followup post, I want to talk about why I believe this is _necessarily_ the case, and what it means for the future of blockchain development stacks.

## Levels of Blockchain

Historically, tech has divided software into horizontal layers, stratified according to their distance from end-users:

1. **Infrastructure**. The infrastructure layer sits at the base of all other software, providing low-level functionality, including storage, compute, and network.
2. **Platform**. The platform layer sits on top of infrastructure, providing middleware functionality in the form of frameworks or services.
3. **Application**. The application layer sits on top of platform, providing tailored *applications* that end-users use to solve specific problems.

As cloud computing emerged to become the dominant model for software development and deployment, cloud analogues of these strata have materialized&mdash;IaaS, PaaS, and SaaS. Most SaaS applications are built on top of PaaS, which are in turn built on top of IaaS, replicating the familiar stack from before the days of cloud computing.

The dominant vision for blockchain applications pictures a similar stack emerging over time:

1. **Infrastructure**. This layer would include the low-level software building blocks on top of which platforms are developed.
2. **Platform**. This layer would include Ethereum, Filecoin, TiesDB, and other similar platforms, which are blockchain-based technologies that offer middleware functionality useful to building applications.
3. **Application**. This layer would include end-user applications like CryptoKitties, Ethercraft, Akasha, and so forth.

In the future, many would have you believe, successful blockchain applications&mdash;be they social media networks, contact relationship management applications, or ride-sharing applications&mdash;will be built on blockchain platforms.

This is a very natural, intuitive vision of the future. Unfortunately, it's also completely wrong.

## The Profile of Success

As of 2009, SalesForce hosted its CRM software on 1,000 nodes, while Blizzard utilized 20,000 nodes to provide hosted game services.

In general, successful SaaS software is hosted on anywhere from hundreds to thousands (and in many cases, tens of thousands!) of nodes.

That's a crap load of computing power.

The crypto space requires dramatically more compute power, because of the overhead of untrusted decentralization. Ethereum runs on 25,000 nodes, and Bitcoin runs on 7,000 nodes (many heavily specialized, with massive raw processing power).

Despite this incredible compute power, all nodes in these blockchains are running more or less the same computations, which is an extraordinarily high cost to pay for untrusted decentralization&mdash;and one that has real implications on the network size of successful blockchain applications.

## Success on the Blockchain

If we take a modern-day SaaS application that requires hundreds of nodes, and port it to blockchain technology, it's clear the number of nodes it requires will increase tremendously. If we naively assumed that one non-blockchain node can be emulated by 7,000 blockchain nodes (Bitcoin), then we're looking at 7 million nodes to power a decentralized CRM application!

Now clearly, innovations like sharding, proof-of-stake, variable trust topologies, and so forth, can improve efficiency by orders of magnitude. But even in the most optimistic scenarios, we are still looking at tens of thousands of nodes required to run a decentralized CRM application.

In other words, a successful blockchain application, using next-generation blockchain technology that hasn't even been invented yet, will require as many (or more) nodes as all of the massive Ethereum or Bitcoin networks!

If successful blockchain applications will be as large (or potentially much larger) than current day networks, then they will necessarily possess the capability to run their *own* blockchains.

If successful blockchain applications have the capability to run their own blockchains, then any decentralized service they require&mdash;currently provided by platforms like Ethereum, Filecoin, or TiesDB&mdash;can be layered into those blockchains. In fact, the code for all these platforms is open source, so they could just fork and use the same code!

Manifestly, the value of a blockchain is not the *software* that runs the protocol, but the *size of the network* that participates in the protocol. Successful blockchain applications will already possess *massive networks* due to their raw requirements for compute and storage.

In a market in which there exists two blockchain applications, and one of them pays transaction fees to numerous third-party blockchain platforms (each of which must pay for compute and storage and also make miners a profit), and one of them runs its own blockchain without any dependencies, the latter one will always win, because it will be cheaper, with fewer points of failure, lower latency, and full feature parity (since it could even use the same code as the blockchain platforms).

The implications of this line of reasoning are dramatic. It means successful blockchain applications will *not* be built on blockchain platforms&mdash;although they could very well be built on the *source code* powering blockchain platforms.

## Death of Platforms?

Just because successful blockchain applications will run their own stack of distributed services doesn't mean blockchain platforms are going away.

In fact, *unsuccessful* blockchain applications&mdash;which might better be termed *niche* blockchain applications&mdash;will need *somewhere* to run, and blockchain platforms can pool together the needs of these niche applications to generate networks of significant size and value.

In addition, Ethereum, Filecoin and some other platforms are more than *just* platforms&mdash;they are *applications* in and of themselves. Filecoin can be used to store your photos or movies. Ethereum can be used to run smart contracts between untrusted parties.

Blockchain platforms like Ethereum and Filecoin can be very large and offer significant value. Most blockchain platforms, however, will be dwarfed by the most successful blockchain applications.

## The Real Stack

In my opinion, the real stack for successful blockchain applications *isn't* blockchain platforms, but rather, ordinary *FLOSS* (free/libre open source software).

I predict the rise of very modular frameworks (similar to [Scorex](https://github.com/scorexfoundation/scorex)), which allow trivial composition of the different services required by blockchain applications (file storage, messaging, identity management, smart contracts, etc). These frameworks will allow developers to quickly create purpose-built decentralized applications that run on their own blockchain.

This means, among other things, a proliferation of digital currencies. Every successful blockchain application will have its own token type. This means exchanges become extraordinarily important, because there will be hundreds of thousands of different token types!

Modular frameworks will probably accelerate standardization, which will have many benefits. Dedicated miners (AKA *cloud infrastructure providers 2.0*) will eventually use blockchain container technology to allow them to dynamically shift mining power to the most profitable decentralized applications.

## Funding the Stack

If modular, open source frameworks are the future of successful blockchain applications, a natural question is *who's going to pay for them*.

It's possible the answer to this question is *no one*, like most open source software is developed today. But I actually think blockchains open up remarkable new pathways for funding open source software.

In the context of blockchain frameworks, I'll mention two possibilities that might be viable:

1. **Insurance**. If you build your application on a framework that is insecure, you risk complete destruction of your market. Developers of a framework might offer insurance against this possibility, which helps fund development of the framework, similar to how some open source software is monetized with commercial SLAs.
2. **Taxes**. A framework can embed a small tax into every transaction, payable to the developers of the framework, in the "currency" of the application. Framework developers could offer dedicated support only to applications that do not disable taxation. In addition, a framework could provide a common set of cross-application services (such as seeding, name resolution, or authentication) in exchange for the tax.

Of these two options, I think taxation may be more likely, because it aligns incentives and doesn't require upfront investment for application developers.

If developers of a framework tax applications too heavily, the taxes will be disabled or the project forked by other developers with lower taxation.

It's like government taxation, only exit has no cost.

## Summary

In this post, I've tried to argue that the software and cloud models do not apply to the blockchain space, at least not in the way many think.

Successful blockchain applications will require networks so large, they can provide all their own services on their own blockchain. This doesn't imply blockchain platforms can't be large and provide significant value, but it does have significant implications for the development of blockchain stacks.

In particular, blockchain stacks are likely to emerge in the FLOSS world, and allow developers to modularly snap together the services required to implement a decentralized application running on its own blockchain. This will lead to a proliferation of token types and make exchanges even more crucial.

Modular blockchain stacks may well emerge organically, possibly as forks of blockchain platforms, or they may be engineered and funded with novel blockchain-centric methods, like insurance or taxation.

If this thesis is correct, it has implications for near-term funding and development of blockchain technology&mdash;some of which should be obvious, and others of which are more subtle. I hope to elaborate in future posts.

If you have your own thoughts on what blockchain stacks of the future will look like, please share them in the comments section below!
