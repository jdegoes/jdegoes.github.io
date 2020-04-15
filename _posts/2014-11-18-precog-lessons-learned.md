---
layout:       post
title:        "10 Lessons I Learned from Doing My First Real Startup"
description:  "Personal and business lessons learned from my experiences funding, growing, and selling Precog."
category:     articles
tags:         [startups, precog, entrepreneurs, entrepreneurship, techstars]
---

It's been more than a year since Precog was [acquired by RichRelevance](https://techcrunch.com/2013/08/14/richrelevance-acquires-precog-to-add-large-scale-analytics-engine-to-e-commerce-personalization-platform/), and I've had some time to reflect on what I learned from my experiences there.

## Born to Build

My Mom used to say that I was practically *born* an entrepreneur, albeit not in the Valley sense of the word.

I taught myself BASIC when I was 8, and in my early teens, I started selling my first shareware product (Home Movies for Windows, a video cataloging system).

Before I graduated high school, I had developed and sold no less than three software programs, as well as written a best-selling book on 3D game programming.

During college, I co-founded GameInstitute (as well as a competitor!), and wrote curriculum and taught classes in everything from artificial intelligence to game mathematics. I went on to found N-Brain, where I funded and helped build a 400K LOC collaborative IDE for software developers.

Between and sometimes during these adventures, I managed to find time to consult for dozens of companies (mostly contract software development) and work for a few large ones.

I got sucked into the Valley in 2009, when I went to work as VP of Engineering for SocialMedia.com during their reboot due to the collapse of independent Facebook ad networks. 

At SocialMedia.com, I was instrumental in helping turn around their technology, rebuild their engineering team, and finally selling the company to LivingSocial (though we had a number of suitors at the time).

Precog, however, was my first experience founding and building a "real" startup.

## Getting Serious

What's a *real* startup? In my mind, it's one where you put everything on the line. 

You quit your day job. You put money into the company. You're all in, and your only safety net is the cold hard concrete which will smash you into a bloody pulp if you step too far to the right or left.

That's exactly what I did at Precog.

## Precog in 10 Seconds

In 2011, I applied to [TechStars Boulder](https://www.techstars.com/program/locations/boulder/), and managed to get in despite having a terrible demo, a practically worthless product hacked together in a few weekends, and a strongly  engineering-biased background.

I invested my own money in the company, often took no salary or minimum wage, and worked like a dog.

Days turned to weeks, weeks to months, and months to years.

By the end of the two year saga, I had raised almost $3M in capital, built what was arguably the best Scala engineering team on the planet, and designed a solution that, to this day, still has ardent fans using the technology.

We were acquired by RichRelevance a couple months after we came out of beta. In those few months before acquisition, our revenue went from $0 to nearly $200k ARR, thanks to the efforts of my colleague Jeff Carr.

Yet, we weren't looking to be acquired. Our technology had finally matured to the point where we could charge more than $1k / month per customer. We were just beginning to figure out how to position and sell the technology.

It felt like, after years in beta, we were *just* getting started.

We sold Precog not because we *wanted* to, but because we *had* to. We were in that tricky spot between seed and Series A that some have called the *Series A Crunch*. 

We hadn't made enough progress to justify a Series A, and we'd already taken almost $3M in seed funding. I tried desperately to raise a bridge round, but that didn't work out, and it was literally *sell* or *shutdown*.

The last-minute acquisition by RichRelevance was not the outcome I preached to investors or employees.

We weren't a billion dollar company redefining data analytics for the 21st century, or even hundred million dollar company. We were a broke little startup shit out of options.

## Mistakes Were Made

Precog was my first real startup. My first experience as CEO. My first experience fund-raising. My first experience making difficult decisions that would impact other people's lives.

In retrospect, I had a *lot* to learn.

*Mistakes were made*, as they say, and I've had a lot of time to think about what they were. 

In this post, I've tried to distill ten of these learnings. 

No single mistake was fatal. Indeed, the history of every successful startup is littered with countless mis-steps and blunders. But I do believe that, if I knew *then* what I know *now*, the outcome would have been very different.

### 1. Go Deep Not Broad

Probably the single most important lesson I learned from Precog is that you need to apply *super-human focus* to solving an extremely *well-defined pain* held by an extremely *well-defined market*.

Make your solution so *impossibly good* that no one who has the pain would even *consider* using anything else. If you don't have the resources to do that, it means you need more focus.

At Precog, we were building a very broad platform to solve a huge range of use cases. The reasons why this strategy doesn't work so well are pretty obvious in retrospect:

1. **Broad solutions cannot easily compete against deep ones.** If you choose to go broad, you will be competing with numerous point solutions across different domains and markets, all of which were cheaper to build and evolve much faster than your solution. Users are more likely to choose the deep solution because it's easier to find, easier to understand, easier to use, solves their exact problem in an obvious way, and delivers value sooner... *even if in the long-run, they might have preferred your solution.*
2. **You can monetize earlier and at a higher price point with a deep solution.** This reduces the need to raise money, extends your runway, and accelerates the growth of your business. It also has profound ramifications if you run into any trouble: a broad solution is like a massive cruise ship that requires huge resources to propel forward or change direction, while a point solution is more like a yacht. 

Starting deep doesn't mean you can't end up broad. Starting deep will give you the traction you need to have the luxury of considering whether or not broad makes sense for your solution.

There are clearly places where broad technology is needed. Most solutions like this are open source (e.g. Hadoop or httpd), and I personally wouldn't recommend companies attempt to build proprietary broad solutions unless they can raise whatever they like (I personally can't command a $20M raise on a PowerPoint deck, but some clearly can).

### 2. Be Non-Disruptively Disruptive

Many startups are fond of saying how "disruptive" their technology or solution is. 

What I now know is that there is a *good* way to be disruptive, and a *bad* way:

 * **Good Disruption**. Your solution solves some aspect of a problem *so astoundingly well*, that a segment of users would tolerate many other deficiencies in order to benefit from your solution. An example is Google Docs: it gets collaboration so phenomenally right, a segment of users put up with a laundry list of deficiencies about as big as the feature list in Microsoft Word.
 * **Bad Disruption**. To adopt your solution, users must be completely re-educated, or they must completely rip out and replace parts of their existing solution, or your solution doesn't easily fit with all the other pieces of the bigger picture. Note: it doesn't matter how technologically or architecturally or philosophically superior your solution is &mdash; all these forms of disruption are bad and will impair your business (albeit, not necessarily kill it).

At Precog, you had to learn a new query language called *Quirrel*, which was a brilliant piece of work and highly acclaimed... by those who took the time to learn it, anyway.

Beyond that, you had to copy all your data to our cloud analytics platform, and learn our tools and APIs.

That's an example of both good and bad disruption. Strive to be the former, *not* the latter.

Now, if you really *have* to be disruptive along both dimensions (and you probably *don't*!), then you need to combine either high usability or deep pain (ideally both) with the open source model. That combination has proven a fairly reliable way to introduce "disruptively disruptive" solutions to the market, but it slows the business down by *years* (see Hadoop, MongoDB, Neo4j, etc.).

### 3. Keep your Game Plan Unfundable

Many investors say they'll only invest in "billion dollar ideas". The trouble with billion dollar ideas is that, if you try to execute on them directly, you'll likely run your company into the ground.

Billion dollar tech companies have massive sales, marketing, support, and engineering teams, and extremely complex products with a bewildering array of confusing features, because that's what was necessary to keep an existing customer or land a new whale.

The billion dollar idea behind Precog was to become the data analytics infrastructure powering all business applications and serving up all manner of insights to the users of those applications across all domains.

That's a big, exciting, bold vision. In the end, it proved too difficult to execute on.

I now believe that your *game plan* should be *unfundable*. If you tell an investor what *you're actually doing*, and they tell you, "that's not a standalone business," then there's a good chance you're on the right track.

It seems pretty obvious, in retrospect, as there are plenty of examples: if Facebook had tried to be the social network for the world, they'd be a footnote in history. Instead, they executed on an unfundable game plan (social network for a college), and are now doing nearly $10B in annual revenue.

Yes, have a big vision that gets investors excited, and makes you want to get up in the morning. But keep your game plan *unfundable*: small and immediate, focused not on the billion dollar idea, but on producing something that someone somewhere gives a shit about.

Succeeding *there* will give you the traction necessary to go after that billion dollar idea.

### 4. Choose a Right-Sized Problem

Success in early fund-raising is heavily dependent on your pedigree, education, connections, and past successes. 

If you were engineer number #2 at Facebook, have a Stanford degree, know the right people, and have prior successful exits, you can raise a *lot* of money on a PowerPoint.

Otherwise, you're probably not going to be so lucky.

If you don't have the luxury of raising whatever you want, then *it's a mistake to assume you can work on any problem.* At Precog, we bit off a problem too hard for our capital constraints, and we tried to make the difference up with incredibly smart engineers and an insane work ethic. It wasn't enough, and I've learned my lesson.

Choose a *right-sized problem* for your capital constraints. Don't try to build the next space shuttle if you only have capital to build the next bicycle.

### 5. Deep Domain Knowledge is Your Most Powerful Asset

When I started Precog, I already had chops in big data analytics. I was an early adopter of Hadoop and other relevant technologies, and as a consultant, I had architected, implemented, or worked on many large-scale distributed data processing systems (EDA rule checker, gravity wave simulator, 3D density reconstructor, etc).

So I thought of myself as a "big data analytics expert", but I found out I was totally wrong.

Understanding the technology is *one* thing. Understanding the business is *completely* different.

To understand the *business*, you need to have a firm grasp on at least the following:

 * **The pains.** Yes, I mean *all* of the pains in the space you are in. Multiple pains lead to multiple solutions, which lead to integration of solutions, and you have to understand not just the pain *you* solve, but how you fit (or do not fit) in to all the other related solutions the customer has.
 * **The solutions.** You have to understand how customers solve their existing pains. Which departments solve which pains? What do those solutions look like? How do they integrate with each other?
 * **The competition.** If I rattle off the name of a competitor, you should be able to tell me without thinking what they do, how it differs from your solution (and why that difference will drive sales), what their price points are, what their typical customer looks like, how much they've raised and roughly what their revenue is like, and so forth. Too many CEOs hide in the sand and actively avoid looking at the competition (for reasons that befuddle me).
 * **The sales cycle.** Who are your champions? Who are your saboteurs? Who discovers your solution and why? Who makes the purchasing decision? What ROI metrics will be used to evaluate your solution?

It took me hundreds of calls, several proof-of-concepts, and dozens of in-person meetings to finally understand the *business* of analytics, especially in the Enterprise, which is a completely different animal than the typical tech company.

Now I almost feel obligated to stick in the same space, if only to leverage years of building domain knowledge.

### 6. Be Responsive, but Stay Focused

Towards the end of Precog, we had thousands of users, and we had several calls with prospects every single day.

Many times, a prospect's use case would be *dangerous close enough* to our technology. We'd juggle around some priorities, shift the product roadmap, devote some engineers to adding a bell or whistle. 

I've seen firsthand how destructive that pattern is to becoming *phenomenally* good at *something* to *someone* (which, as covered above, is a key ingredient of success).

In the quest for product / market fit, one of your most powerful weapons is the word *no*. As in, *No*, we're not going to add that feature. *No*, your use case isn't a fit for our product. *No, no, no!*

To become great, truly great, at something, you have to accept being terrible at a lot of other stuff (at least initially).

That doesn't mean you have a license to ignore prospects or customers. On the contrary, you have to pay extreme attention to feedback, who it comes from, how big the pain is, and how deep the pockets are. But you have to pick and choose your battles, and saying no *a lot* will give you the ability to say *yes* to the one or two things that can move the needle.

### 7. Choose Investors Carefully

At Precog, the reason we weren't able to raise a bridge round was because of negative signaling around one investor's refusal to participate (and of course they had no obligation to participate).

This same investor also inadvertently harmed our fund-raising efforts by giving erroneous information to third-party investors, thus painting our company in a negative light.

With a bridge round, I can't guarantee the outcome would have been any different, but I do feel we had an excellent shot at hitting at least $1M in ARR on a relatively conservative bridge.

The lesson I've learned is that it only takes one bad egg. 

In an ideal world, everyone would invest solely based on the merits of the company. In the real world, signaling dominates. However, I think you can protect yourself with a few steps:

1. **Insist on experience.** Don't accept money from a green investor unless the investor is unknown and the amount is small. If a name-brand investor or (worse) a venture fund gets cold feet at any bump in the road, the negative signaling can doom the entire enterprise.
2. **Syndicate the round.** While you do want someone who is highly committed to your success, I think you should aim to raise from several "equally prominent" investors. In fact, recently I've heard this same advice from VCs: "Don't raise a seed from just us, because if we don't participate in the next round (and we often don't!), it will kill your company."
3. **Talk to references.** Don't talk to the ones experiencing meteoric growth. Talk to the ones that were shut down or are struggling or which took a while to get off the ground.
4. **Find out their bridging policy.** You may expect a meteoric rise to super-stardom, but what if you hit a few snags? You want to make sure the majority of your investors have a *demonstrated track record* of bridging companies that fall between rounds.

### 8. Don't Create a Burn-out Culture

The company values of Precog could best be summarized as *intelligence* and *hard work*. A lot of smart people worked at Precog, and they worked very, very hard. 

I always strove to be the hardest working person in the company. The day after my Mom died, I worked an 18 hour day. Yes, part of that was just me trying to (not) deal with the grief, but I wasn't working alone, either (in fact, for most of the day, I was pair programming with another engineer).

While we got a lot done &mdash; sometimes it astounds me how fast we moved &mdash; the ripples of that culture could be felt months after the acquisition. Most ex-Precogs stopped tweeting, stopped blogging, stopped replying to emails, and stopped contributing to open source projects. They were totally and utterly burnt out.

It took months for some to recover, and it may take years for others. I'm only beginning to recover myself.

While in the thick of it, I came to believe that we were aiming too high for the capital we had raised, but that we could possibly overcome our lack of capital by working harder. It was a race against the clock.

Now I see the toll that wrought on people, and that it didn't really change anything, and I have adopted a different philosophy: if the only way you can succeed is by burning out your employees, then you're working on the wrong problem.

That doesn't mean you shouldn't work hard. That goes without saying. And yeah, sometimes, you may have to put in some time on the nights or weekends. But generally, you should run the company as if you will be running it for 10 or 20 years.

As the saying goes, it's a *marathon*, not a *sprint*.

### 9. No One Really Knows Anything

I've pitched to most of the top-tier VC firms, and met some of the most famous personalities in venture capital. I've chatted with and learned from CEOs from all walks of life. I've had mentors and advisors galore, both official and not.

I have stories of warmth and intelligence and stories of stupidity and pomposity.

But if there's one thing I've learned from all this, it's that there's no such thing as an all-knowing oracle. No one *really* knows which paths will lead to success, and which to failure. 

Mentors, even those from the same industry, often *fiercely* disagree with each other. Investors, including venture capitalists, have *no idea* why they do or do not invest. While they sometimes give reasons, those reasons are made up after the fact, which is why they're usually inconsistent with other investments.

Finally, ideas that seem ridiculously stupid (like, "We're gonna create an app that lets you share touched-up photos with your friends!", or "We're gonna help people rent out their spare rooms!") can sometimes pan out in a big way.

There's a lesson in all of this: listen to everyone, and take all criticism to heart, because no point of view is *entirely* wrong. There's a kernel of truth in everything.

But at the end of the day, realize that no one *really* knows anything. There's no wizard behind the curtain.

It's up to *you* to become master of your business, to become that oracle who can help you guide your business to success.

### 10. Don't Let Failure Destroy You

Failure isn't easy to deal with, either socially or professionally.

I've seen firsthand some of the fallout:

* A former mentor studiously avoided me and couldn't even make eye-contact when we ran into each other at tech functions. It was like I had the plague, and it might be contagious, so they had to keep their distance.
* I ran into a Precog investor on the street, and the investor stopped me, and accused me of being a quitter, insisting that if I *really* believed in the company, I would have found a way to make the bridge round happen.
* One investor whom I've never met or even exchanged emails with told a friend of mine, *"I'd never invest in any company that John De Goes was a part of."* Hmmm, OK. 
* Several fellow entrepreneurs whom I knew well from the Precog days couldn't even be bothered to return my emails. I'm not one of the cool kids anymore.

Intellectually, I know that even if you do *a lot* of things right, that doesn't mean you will succeed. There's a large element of chance, and failure is something that *every* entrepreneur may have to contend with.

For all the stories of last-minute miracles, [like the one that saved Evernote](https://techcrunch.com/2013/10/19/phil-libin-evernote/), there are a hundred more startups that *might* have turned out just as well. 

We'll never know, because they never got a last-minute miracle.

Emotionally, however, scraping off that bloody pulp from the concrete and trying to turn it into something resembling a normal human being is a difficult undertaking.

Good thing we entrepreneurs were born to do the impossible, right? Or at least, born crazy enough to *think* so, which is sometimes all that matters.

## Continuing the Journey

While things didn't turn out the way I hoped for Precog, I wouldn't take back the experience for anything. 

Our life experiences mold us into the people we become. I learned so much over such a short span of time, and it's fundamentally changed the way I think about business and look at the world.

I will be taking all of these lessons and many others with me into my next great adventure. And some years down the road, I hope I'm lucky enough to be able to write *another* post like this one, and reflect on all the *new* lessons I've learned.

Until *then*, it's time for me to show that concrete who's boss...