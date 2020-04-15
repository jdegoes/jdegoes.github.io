---
layout:       post
title:        "Dear Engineer: So, You Wanna Become an Entrepreneur?"
description:  "Lots of engineers have gone from coding to business-building, but it's not an easy transition."
category:     articles
tags:         [startups, engineering, entrepreneurs, business]
---

Dear Engineer:

I'm so excited that you're interested in starting your own company! The world needs more entrepreneurial engineers, who blend a vision for a better future with the technical chops to get there.

You'll be in good company. Bill Gates, Mark Zuckerberg, and many others transitioned from coding to building their own companies.

It *isn't* easy. If not properly directed, many of the skills that make you an *amazing* engineer can actually work *against you* as you try to build a business. 

But it *is* possible. To help you get there, I want to share some of the lessons I've learned over the years.

These aren't *generic* lessons for starting your own company. You can find those [elsewhere](https://degoes.net/articles/precog-lessons-learned/). These are lessons that *engineers* in particular need to learn.

### Make sure you have something to lose.

The biggest divide you have to cross is the employer / employee divide.

When you're an employee, it's easy to complain and find faults in everything. You can complain about low pay, lack of benefits, technical debt, crappy technologies, pivots, troublesome customers, and so much more. 

If *you* were in charge, you'd fix all those problems, because that's what engineers do: fix problems. *Right?*

It's all a matter of perspective. While there are plenty of crappy employers, there are lots of *good* ones who struggle to do the best they can with the meager resources they have.

To truly cross the divide, you need *your own skin* in the game. Invest your own money, go without a salary until the business achieves break-even, or live hand-to-mouth until you get the company off the ground.

*Anyone* can complain about their employer. *Anyone* can spend someone else's money like crazy. *Anyone* can succeed in a world of infinite time and money. But very few can make the *difficult trade-offs* that come from choosing between a large number of suboptimal solutions to an inherently over-constrained problem.

### Be wary of generalizing the pain.

As an engineer, one of your most powerful skills is that of *abstraction*.

You see patterns everywhere, and this ability helps you generalize from specific examples to highly abstract code that can easily and robustly solve a wide range of problems.

Unfortunately, this ability to see patterns can prove *catastrophic* as you attempt to build your own company. The more you generalize the *solution* to a particular pain, the further removed it becomes from that *specific* pain. While it might end up being able to solve a *lot* of pains, it won't be very good at solving any *particular* one.

An example will make this clearer. Let's say you believe in the following pain:

 > People are fed up with the high price of hotels.

From this pain, you use your engineer's mind to come up with a solution: A lot of people have spare rooms they aren't using, so why not build a solution that lets them rent out those rooms to people looking for cheaper options?

You don't stop there, however. You see this pain is just a specific example of a more general pattern: the pattern of connecting price-sensitive buyers with sellers having remnant inventory.

You then build a fancy marketplace solution that does everything from Uber to Airbnb, and while it might be possible to buy or sell *anything* on your platform, it's not very good at any one thing in particular, and you get lost in an infinite sea of noise.

More importantly, you fail to build *Airbnb*.

Or maybe the user's pain is that they want to know which online channels are driving the most lifetime revenue, so they can shift marketing dollars to them. But you end up generalizing the solution to a monster analytics platform that can do *anything at all*, but isn't particularly good at doing the one thing the user wanted.

It's quite natural for *maturing companies* to generalize their solutions to siphon dollars from adjacent markets, but *premature generalization* can kill your company faster than most other techniques combined.

So fight your instinct to generalize, and instead, *specialize*. 

In geek-speak, you don't want a transducer. You don't even want a generic sorting algorithm. You're looking for an merge sort specialized to 32-bit unsigned integers that runs only on Xeon E3 1240v2 processors.

### Don't run from messy pains.

As as engineer, you probably have a bias toward elegant solutions. You like order and have a strong aversion to chaos. Systems that run according to well-defined, deterministic laws are easier to understand and easier to predict.

In business, this tendency toward elegance, order, and simplicity can work against you.

You might learn that hospitals are looking for better reporting on patient medical records. But the deeper you dig into the problem, the messier it seems:

 * Medical records aren't stored in one format, they're stored in literally thousands of totally incompatible formats.
 * Medical record systems are fragmented, incompatible, and hospitals are sometimes using versions of software that pre-date the Internet.
 * Hospitals don't standardize on terminology; so what one hospital calls *gender* another hospital calls *sex*.
 * Access and storage of medical data is regulated by HIPAA and other laws, as well as hospital-specific policies.

Your first instinct to uncovering this mess is probably repulsion (as it should be!). But there are two quite different ways you can respond:

1. **Avoid the mess.** Solve the problem with a generic, self-service solution that requires customers themselves deal with the messy bits.
2. **Embrace the mess.** Solve the problem with a full-service solution that relies on a crap-load of integration middleware and maybe even services.

Most first-time *engineer-entrepreneurs* will choose the first option. Instead of solving the customer's pain, they'll build a tool, or a "platform", that lets customers *solve their own pains*.

Unfortunately, this approach usually ends in disaster. The messy parts of any problem are the ones that cause the *most* pain, and the ones that drive differentiation, high business value, and rapid growth.

So instead, I urge you to *run* to messy pains, and use that geeky brain of yours to find out how to solve them in a scalable way that delights the customer.

### Remember that technology is irrelevant.

As engineers, we love technology. Great programming languages dazzle us, and powerful libraries fill us with awe and wonder.

But one of the most important lessons you can learn is that *the customer doesn't give a shit about technology*.

A spaghetti-code PHP 3.0 application that uses text files for its database may be an engineer's worst nightmare.  But customers care only about the surface area they interact with: elegant architecture, beautiful abstractions, brilliant programming languages, and world-class libraries don't mean squat.

*You and I know* that all these things have an effect on long-term maintainability, ease of adding new members to the team, testability of the code, defect rates, engineer satisfaction, and lots more. And I don't want to discount any of these measures, just to emphasize the following: your business will succeed or fail *not based on the technology*, but based on how well it solves customer pain (which is *almost* completely independent of the technology!).

If anything, you are likely to become *too distracted* by technology, so much so you may forget the customer in your quest to build something really cool.

If you can't check that quest for cool tech at the door, *don't start your own business*.

### Embrace inferior but successful crap.

In many tech businesses, your success is going to depend on how well you integrate with stuff the customer is *already using*.

This holds true even when *the customer is using utter crap*. 

As engineers, we gravitate toward well-engineered systems, and we have a tendency to *strongly* prefer supporting and integrating with these systems.

Unfortunately, the best-engineered systems do not often win: rather, what wins are the systems that are "good enough" to get the job done (but just passably), and which have the lowest barrier to entry. This means there's a lot of successful crap out there, which is poorly engineered, buggy, and far from best-in-class.

Look for what your customers are already using, whether that's SOAP, Internet Explorer 6, or whatever. Then build your product to integrate well with these technologies, in a seamless way.

Deal with your repulsion to crap with hot showers, *not* by avoiding it in your business.

### Don't confuse yourself with the customer.

One of the worst mistakes you can make as an engineer is building something that you would use. Although there are exceptions, by and large, if you build something that *you* would use, you'll be building for a *market of one*. 

You are *not* your customer. You probably *won't* be interested in using whatever you build, *even if you are targeting a market of developers*.

Your product decisions need to be driven by the needs of your target market, who are going to have a profile that you gradually discover as you find product-market fit.

### Finally, don't get cocky.

We are makers. Creators of beautiful and intricately-engineered systems that power the modern world.

By all means, be proud of your maker heritage. But remember, just because you *can* build anything, doesn't mean you *will* build something that people *actually want*. 

Your capacity to build is something you should be thankful for every day. But it's not enough. 

More than knowing *how* to build, you're going to have to figure out *what* to build that can drive revenue. That, in the end, is going to be the most difficult problem your engineer's mind will ever have to solve.

Good luck.