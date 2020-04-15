---
layout:       post
title:        "Functional Programming Isn't the Answer"
description:  "Functional programming is a means to an end, not an end in itself."
category:     articles
tags:         [functional programming, CCC, 3 C's]
---

Functional programming has enjoyed a surge in recent years. You can see in the proliferation of books and conferences, in the rapid growth of languages like Scala and Clojure, and in the very public conversions (John Carmack, Bob Martin, etc).

These days, no one would dare announce a new programming language without supporting "functional programming." Heck, even conservative, mild-mannered, Enterprise-certified Java is getting [lambdas](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) and [even](https://download.java.net/jdk8/docs/api/java/util/stream/Stream.html) [monads](https://download.java.net/jdk8/docs/api/java/util/Optional.html)!

Yep, it's a whole new world.

## Why FP, Why Now?

I've been a *functionally-inclined* software engineer for more than four years now. I love FP and learn more every day.

Recently, I accepted a short-term contract for working on an existing Java application. While working on the application (which I would describe as more or less idiomatic, "Enterprise Java"), I had the chance to revisit some of the fundamental reasons why I like functional programming (after a while, you sort of take them for granted).

Among them:

 * **Higher-order functions** (which let you pass functions to functions, or return functions from functions) let you factor out a lot of duplication in the small. I refactored the existing Java app to use higher order functions, and found and fixed several bugs along the way (all related to bugs by copy & paste).
 * **Immutable data structures**, which are often used in FP, free you from having to constantly worry about what code will do to the data that you pass it. In the Java app, I found a lot of "defensive copying", which could simply be deleted as I changed many core data structures from mutable to immutable.
 * **Strong types**, which appear in many functional programming languages (but not all), tell us more about statically-proven properties of the code. In the Java app, I changed a lot of code from using `null` to using a generic optional data structure, which more clearly communicates where values may be absent. This let me delete a lot of defensive `null`-checking, as well as fix a few NPEs in uncommon code paths.
 * **Pure functions**, which are functions that don't have side effects (i.e. their output is a deterministic function of their input), are much easier to understand and test, because you don't have to wonder if the function's behavior will change based on hidden state. In the Java application, I was able to convert a lot of stateful functions to stateless functions, which made the code much simpler and fixed a few bugs.
 
There are other benefits, as well (and of course a few drawbacks), but as one data point, consider the fact that in this Java application, I was able to fix bugs and implement a large amount of new functionality with fewer total lines of code.

That's pretty common in my experience.

These benefits have not escaped anyone's notice. In contrast to even 5 years ago, most programmers today have heard of functional programming, many use some techniques from FP (at least higher-order functions), and a growing number have hopped on the bandwagon and become FP evangelists. 

## The Religion of FP

On the spectrum of functional programming (FP), people fall between one of two extremes. At one extreme, FP is a way to enrich imperative programming (e.g. pass a lightweight callback to a function or a block to a loop). At the other extreme, FP is a way to write so-called *pure* code &mdash; code without side-effects, expressed as pure, referentially transparent functions.

Some people have fallen so much in love with FP (it's hard not to!) that their views are almost religious. Hence, what I refer to as *the religion of FP*.

The problem with any religion is that dogmatism can be blinding. 

In this case, I believe it's blinded many to something that should be obvious:

> Functional programming (pure or otherwise) is not the goal of software engineering. Rather, it's a means to an end, like every other tool in the bag of a software engineer.

Heresy, I know, so let me clarify.

## The Goal of Software Engineering

As a software engineer, my job is usually to produce working, comprehensible, maintainable software.

Most everyone who pays me for my services wants some combination of the following:

 1. Code that works reliably, even in parts of the application that aren't used very often.
 2. Code that can be easily understood by other people (I won't be around forever).
 3. Code that is factored in such a way as to minimize the cost of changing it (because if one thing is certain, it's that requirements never stop changing).

(They also usually want it fast and cheap, too, but that's a topic for another post.)

Maybe surprisingly, the above is what *I* want, too.

I like code that doesn't have bugs (it gives me a sense of pride in my work, and I hate debugging). I want all the code I write to be easily understood, because I'll probably need to come back to the code a few months or years down the road (plus it helps reduce bugs). And I absolutely adore code that's so well-factored, I can easily and safely change it to accommodate new requirements.

So if the goal of software engineering is working, comprehensible, maintainable software, the logical question is, "Does functional programming help us achieve it?"

My answer is: *not necessarily.*

## Rogue FP

To illustrate my point, I decided to implement Quick Sort in the functional programming language *Haskell*.

As per the description on its [home page](https://www.haskell.org), Haskell is an advanced, purely-functional programming language (and presently one of my favorite programming languages).

You almost can't get any more "FP" than Haskell. All programs written in Haskell are purely functional (although there are ways to cheat, we can ignore them here).

With that said, brace yourself, and take a look at my implementation of Quick Sort:

{% highlight haskell %}
module Main (main) where

import Control.Applicative
import Data.Array.MArray
import Data.Array.IO
import Data.IORef

type Array a = IOArray Int a

whileM :: IO Bool -> IO () -> IO ()
whileM pred effect = do
  rez <- pred
  if rez
  then do
    effect
    whileM pred effect
  else return ()
    
quick_sort :: Ord a => Array a -> IO (Array a)
quick_sort a = do
  (m, n) <- getBounds a
  let loop' = loop
  loop' a m (n + 1)
  where 
    loop ary m n = if (n < 2) then return ary else do
      let readVal idx = readArray ary (idx + m)
          
      let writeVal idx = writeArray ary (idx + m)
      
      let readValRef ref = readIORef ref >>= readVal

      let writeValRef ref v = readIORef ref >>= writeVal <*> pure v
  
      pivotVal <- readVal $ n `div` 2
      
      leftIdxRef  <- newIORef 0
      rightIdxRef <- newIORef $ n - 1
      
      let incLeft  = modifyIORef leftIdxRef (+1)
      let decRight = modifyIORef rightIdxRef (subtract 1)
      
      let readLeftIdx = readIORef leftIdxRef
      let readRightIdx = readIORef rightIdxRef
              
      whileM ((<=) <$> readLeftIdx <*> readRightIdx) $ do
        leftVal  <- readValRef leftIdxRef
        rightVal <- readValRef rightIdxRef
        
        if (leftVal < pivotVal) then incLeft
        else if (rightVal > pivotVal) then decRight
        else do 
          writeValRef leftIdxRef rightVal
          writeValRef rightIdxRef leftVal
          incLeft
          decRight
      
      leftIdx  <- readLeftIdx
      rightIdx <- readRightIdx
      
      loop a m       (rightIdx + 1)
      loop a leftIdx (n - leftIdx)

main = newListArray (0, 7) [9, 2, 3, 45, 2, 9, 2, 1] >>= quick_sort >>= getElems >>= putStrLn.show
{% endhighlight %}

Despite the fact that this program is "purely functional", the code is complete and utter *garbage*:

 * It had several bugs when I first wrote it, and it took me a lot of time to track them down.
 * It's hard to understand. In fact, a C implementation would probably be easier to understand.
 * For such a small function, it would be awfully hard to maintain. Making changes safely would require a large amount of mental effort and testing, and you might not be able to reuse much code.

(Note, I said garbage, but I know full well that sometimes you have to take a hit to write systems-level, performance critical code; such is not the case, however, for the majority of modern software engineering.)

There you have it, folks: a purely functional program which is utterly irrelevant to the goal of software engineering (this is a not-so-subtle demonstration, but there are many more real-world demonstrations that are more subtle, which functional programmers would appreciate).

It's FP gone rogue, and proof that just because something is "purely functional", doesn't mean it's worth a damn.

## Lovable FP

Now I'd like to show you the better known example of Quick Sort in Haskell. It's not exactly classic Quick Sort, because it doesn't sort in-place, but it's close enough:

{% highlight haskell %}
quicksort :: Ord a => [a] -> [a]
quicksort []     = []
quicksort (p:xs) = (quicksort lesser) ++ [p] ++ (quicksort greater)
    where
        lesser  = filter (< p) xs
        greater = filter (>= p) xs
{% endhighlight %}

Now *that's* a thing of beauty! And why people love FP.

This code is practically correct by definition. It's trivial to understand (if you know Haskell's syntax), and there's not a lot of sorting code that's easier to maintain (well, `filter` should really be replaced by `partition`, because `filter` destroys information; use of `filter` requires manual negation of the boolean predicate `< p`, which represents duplicated information content).

We now have a *night and day* difference between two *purely functional* programs, both written in the same language.

What gives?

## FP Isn't the Goal

My view is that although FP makes it easier to write good code, just because something's functional, or even "purely functional", doesn't necessarily mean it's any good.

Stated differently, as software engineers trying to improve our craft, we shouldn't latch onto or defend something just because it's "functional" or "purely functional". Although it's possible to write good code using techniques from functional programming, it's also possible to write bad code. 

FP doesn't protect us. We need a different measure for "good code" than just "functional".

I'd suggest the answer has a lot to do with *composability*, *comprehensibility*, and *correctness*.

## good\_code = c^3

Fundamentally, I believe all good code has the following properties:

 1. You can comprehend how it works well-enough to be reasonably confident that it's correct (and be right about that confidence most of the time!).
 2. You can take two pieces of comprehensible, correct code, and easily compose them to create another piece of code that's both comprehensible and correct.

This is a very human, very *cognitive-centric* definition of good software.

After all, we are glorified, talking apes. That bag of fat filling our skulls was never designed to write software.

By recognizing this fact, good code tries to expand our (very limited) capacity for writing and maintaining working software.

## FP Isn't the Answer

In defining good code, I didn't say anything about functional programming, static types, or lots of other things, because those are "just" a means to an end.

Sometimes (but not always, as I've shown in this post!), these means help us create, comprehend, and compose correct code.

But by themselves, they are not the goal of our work.

> Stated differently, whether or not something's "bad" is independent of whether or not it's "purely functional". "Purely functional" is neither a necessary nor sufficient condition for good code.

We can't stop at functional. We can't pat ourselves on the back just because we wrote "purely functional" code. We can't ignore "non-functional" programming techniques, including whole other paradigms like logic programming and reactive programming.

Every technique has to be measured on its own, independent of whether or not it's "functional".

In future posts, I hope to talk more about *comprehensibility*, dig into some meaty examples, and rant against monads (which I believe are often an *anti-pattern* in functional programming).