---
layout:       post
title:        "Destroy All Ifs — A Perspective from Functional Programming"
description:  "Replace conditionals with lambdas for simpler code that's more powerful"
category:     articles
tags:         [fp, functional programming, conditionals, if, anti-if, booleans]
---

The [Anti-IF Campaign](https://antiifcampaign.com) currently stands at 4,136
signatures, and there's a reason: conditional statements are frequently a
troublesome source of bugs and brittle logic, and they make reasoning about
code difficult because they multiply the code paths.

The problem is not necessarily with conditionals: it's with the boolean values
that are required to use conditionals. A boolean value reduces a huge amount of
information to a single bit (`0` or `1`). Then on the basis of that bit,
the program makes a decision to take one path or a totally different one.

What could possibly go wrong — Right?

# The Root Problem

I've long argued that, despite all its flaws, one of the great things about
functional programming is its intrinsic *inversion of control*.

In an impure imperative language, if you call some function
`void doX(State *state)`, the function can do anything it wants. It can modify
state (both passed in and global), it can delete files, it can read from the
network, and it can even launch nukes!

In a pure functional language, however, if you call some function
`doX :: State -> IO ()`, then at most it's going to return a value. It can't
modify what you pass it, and if you like, you can ignore the value returned by
the function, in which case calling the function has no effect (aside from
sucking up a little CPU and RAM).

Now granted, `IO` in Haskell is **not** a strong example of an abstraction that
has superior reasoning properties, but the fundamental principle is sound: in
pure functional programming languages, the control is *inverted* to the caller.

Thus, as you're looking at the code, you have a clearer sense of what the
functions you are calling can do, without having to unearth their foundations.

I think this generalizes to the following principle:

> Code that inverts control to the caller is generally easier to understand.

(I actually think this is a specialization of an even *deeper* principle — that
code which destroys information is generally harder to reason about than code
which does not.)

Viewed from this angle, you can see that if we embed conditionals deep into
functions, and we call those functions, we have lost a certain amount of
control: we feed in inputs, but they are combined in arbitrary and unknown
ways to arrive at a decision (which code path to take) that has humongous
ramifications on how the program behaves.

It's no wonder that conditionals (and with them, booleans) are so widely despised!

# A Closer Look

In an object-oriented programming language, it's generally considered a good
practice to [replace conditionals with polymorphism](https://refactoring.com/catalog/replaceConditionalWithPolymorphism.html).

In a similar fashion, in functional programming, it's often considered good
practice to replace boolean values with algebraic data types.

For example, let's say we have the following function that checks to see if
some target string matches a pattern:

```haskell
match :: String -> Boolean -> Boolean -> String -> Bool
match pattern ignoreCase globalMatch target = ...
```

(Let's ignore the boolean return value and focus on the two boolean parameters.)

Looking at the `match` function, you probably see how it's very signature is
going to cause lots of bugs. Developers (including the one who wrote the function!)
will confuse the order of parameters and forget the meaning.

One way to make the interface harder to misuse is to replace the boolean
parameters with custom algebraic data types:

```
data Case = CaseInsensitive | CaseSensitive

data Predicate = Contains | Equals

match :: String -> Case -> Predicate -> String -> Bool
match pattern cse pred target = ...
```

By giving each parameter a type, and by using names, we force the developers
who call the function to deal with the type and semantic differences between
the second and third parameters.

This is a big improvement, to be sure, but let's not kid ourselves: both `Case`
and `Predicate` are still "boolean" values that have 1 bit of information each,
and inside of `match`, *big decisions* will be made based on those bits!

We've cut out *some* potential for error, but we haven't gone as far as our
object-oriented kin, because our code still contains conditionals.

Fortunately, there's a simple technique you can use to purge almost all
conditionals from your code base. I call it, *replace conditional with lambda*.

# Replace Conditional with Lambda

When you're tempted write a conditional based on boolean values that are passed
into your function, I encourage you to rip out those values, and replace them
with a lambda that performs the *effect* of the conditional.

For example, in our preceding `match` function, somewhere inside there's
probably a conditional that looks like this:

```haskell
let (pattern', target') = case cse of
  CaseInsensitive -> (toUpperCase pattern, toUpperCase target)
  CaseSensitive   -> (pattern, target)
```

which normalizes the case of the strings based on the sensitivity flag.

Instead of making a decision based on a bit, we can pull out the logic into
a lambda that's passed into the function:

```haskell
type Case = String -> String

data Predicate = Contains | Equals

match :: String -> Case -> Predicate -> String -> Bool
match pattern cse pred target = ...
  let (pattern', target') = (cse pattern, cse target)
```

In this case, because we are accepting a user-defined lambda which we are then
applying to user-defined functions, we can actually perform a further
refactoring to create a match combinator:

```haskell
type Matcher = String -> Predicate -> String -> Bool

caseInsensitive :: Matcher -> Matcher

match :: Matcher
```

Now a user can choose between a case sensitive match with the following code:

```haskell
match a b c                 -- case sensitive
caseInsensitive match a b c -- case insensitive
```

Of course, we still have another bit and another conditional: the predicate flag.
Somewhere inside the `match` function, there's a conditional that looks at `Predicate`:

```haskell
case pred of
  Contains -> contains pattern' target'
  Equals   -> eq pattern' target'
```

This can be extracted into another lambda:

```haskell
match :: String -> (String -> Bool) -> String -> Bool
```

Now you can test strings like so:

```
caseInsensitive match "foo" eq       "foobar" -- false
caseInsensitive match "fOO" contains "foobar" -- true
```

Of course, now the function `match` has been simplified so much, it no longer
needs to exist, but in general that won't happen during your refactoring.

Note that as we performed this refactoring, the function became more general-
purpose. I consider that a side-benefit of the technique, which replaces highly-
specialized code with more generic code.

Now let's go through a real world case instead of a made-up toy example.

# psc-publish

The PureScript compiler has a publish tool, with a [function defined approximately as follows](https://github.com/purescript/purescript/blob/9f292cf64e23723f0851f59b2672a3c1e2c546ed/psc-publish/Main.hs#L42-L50):

```haskell
publish :: Bool -> IO ()
publish isDryRun =
  if isDryRun
    then do
      _ <- unsafePreparePackage dryRunOptions
      putStrLn "Dry run completed, no errors."
    else do
      pkg <- unsafePreparePackage defaultPublishOptions
      putStrLn (A.encode pkg)
```

The purpose of `publish` is either to do a dry-run publish of a package (so the
user can make sure it works), or to publish the package for real.

Currently, the function accepts a boolean parameter, which indicates whether or
not it's a dry-run. The function branches off this bit to decide the behavior of
the program.

Let's unify the two code branches by extracting out the options, and introducing
a lambda to handle the different messages printed after package preparation:

```haskell
type Announcer = String -> IO String

dryRun :: Announcer
dryRun = const (putStrLn "Dry run completed, no errors.")

forReal :: Announcer
forReal = putStrLn <<(A.encode pkg)

publish :: PublishOptions -> Announcer -> IO ()
publish options announce = unsafePreparePackage options >>= announce
```

This is better, but it's still not great because we have to feed two parameters
into the function `publish`, and we've introduced new options that don't make
sense (publish with dry run options, but announce with `forReal`).

To solve this, we'll extend `PublishOptions` with an `announcer` field, which
lets us collapse the code to the following:

```haskell
forRealOptions :: PublishOptions
forRealOptions = ...

dryRunOptions :: PublishOptions
dryRunOptions = ...

publish :: PublishOptions -> IO ()
publish options = unsafePreparePackage options >>= announcer options
```

Now a user can call the function like so:

```haskell
publish forRealOptions
publish dryRunOptions
```

There are no conditionals and no booleans, just functions. You never need to
decide what bits mean, and you can't get them wrong.

Now that the control has been inverted, the *caller* of the `publish` function
has the ultimate power, and can decide what happens deep in the program merely
by passing the right functions in.

# Summary

If you don't have any trouble reasoning about code that overflows with booleans
and conditionals, then more power to you!

For the rest of us, however, conditionals complicate reasoning. Fortunately,
just like OOP has a technique for getting rid of conditionals, we functional
programmers have an alternate technique that's just as powerful.

By replacing conditionals with lambdas, we can invert control and make our code
both easier to reason about and more generic.

So what are you waiting for?

Go sign the [Anti-IF Campaign](https://antiifcampaign.com) today. :)

*P.S.* I'm just joking about the Anti-IF campaign (but it's kinda funny).

# Addendum

After writing this post, it has occurred to me the problem is larger than just
boolean values.

The problem is fundamentally a *protocol* problem: booleans (and other types) are
often used to encode program semantics.

Therefore, in a sense, a boolean is a serialization protocol for communicating
intention from the caller site to the callee site.

At the caller site, we serialize our intention into a bit (or other data value),
pass it to the function, and then the function deserializes the value into an
intention (a chunk of code).

Boolean blindness errors are really just *protocol errors*. Someone screwed up
on the serialization or deserialization side of things; or maybe just caller
or callee misunderstood the protocol and thought different bits had different
meanings.

In functional programming, the use of lambdas allows us to propagate not merely
a *serialized* version of our intentions, but our *actual* intentions! In other
words, lambdas let us pull the intentions out of the callee site, and into the
caller site, since we can propagate them directly without serialization.

In older programming languages, this could not be done: there was no way to ship
semantics (a hunk of code) from one part of the program to another. Today's
programming languages support this style of programming (to varying degrees),
and functional programming languages encourage it.

Inversion of control, in this limited sense, is about removing the possibility
of a protocol error by giving the caller direct control over semantic knobs
exposed by the callee.

Kinda cool, eh?
