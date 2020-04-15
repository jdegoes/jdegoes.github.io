---
layout:       post
title:        "Haskell's Type Classes: We Can Do Better"
description:  "Ideas on how to fix some of what's wrong with type classes."
category:     articles
tags:         [functional programming, typeclasses, ml, modules, haskell, scala, java, idris, purescript]
---
Type classes, object-oriented interfaces, and ML-style modules were all invented to solve the same core problem: how do we abstract over a set of functions and types that might have multiple concrete implementations?

For example, a *monoid* is an algebraic structure with an associative binary operation and an identity element (natural numbers form a monoid under the addition operator, with `0` as the identity element of the monoid).

There are lots of possible monoids, and we'd like to be able to write generic code which can work with *any* monoid.

In Haskell, we do that with type classes:

{% highlight haskell %}
class Monoid a where
  empty :: a
  append :: a -> a -> a
{% endhighlight %}

In SML, we do it with modules:

{% highlight SML %}
signature MONOID = 
 sig
   type t
   val append: t*t -> t
   val empty: t
 end;
{% endhighlight %}

And in Java, we do it with interfaces:

{% highlight java %}
interface Monoid<A> {
  A empty();
  A append(A v1, A v2);
}
{% endhighlight %}

Of these solutions, Haskell type classes are by far the sexiest because of compiler magic that makes using them so effortless. 

Let's take a closer look at that magic before I dive into some of my gripes against even Haskell's type classes.

## The Magic of Type Classes

In Haskell, you provide a concrete implementation for a type class by defining an *instance* of that type class for some type.

For example, we can create the additive monoid for natural numbers as follows:

{% highlight haskell %}
instance Monoid Integer where
  empty = 0 :: Integer
  append a b = a + b
{% endhighlight %}

Ordinarily, we would define this instance in either the module that `Monoid` is defined, or the module that `Integer` is defined (if we were defining it at all, and we wouldn't for reasons I'll explain later).

If we play by these rules, then any module that imports both `Monoid` and `Integer` will be able to use this `Monoid` instance trivially and automatically:

{% highlight haskell %}
Prelude> append 1 2
3
{% endhighlight %}

When I called the `append` function, I did not specify which type class instance the compiler should use. The compiler infers the type of the parameters to the `append` function, and finds a suitable instance (in this case, the instance we just defined).

This magic is one of the things that makes Haskell code so clean. Unlike in Java or SML, where we would have to pass around their versions of 'type classes' manually, Haskell figures that out for us!

Scala has poor-man's type classes. Or rather, you can simulate type classes (and a great many other things) with *implicits*, which are function parameters that Scala will pass around for you.

The above example might look like this in Scala:

{% highlight scala %}
trait Monoid[A] {
  def empty: A
  def append(v1: A, v2: A): A
}

def empty[A: Monoid] = implicitly[Monoid[A]].empty
def append[A: Monoid](v1: A, v2: A) = implicitly[Monoid[A]].append(v1, v2)

implicit val MonoidInt = new Monoid[Int] {
  def empty = 0
  def append(v1: Int, v2: Int) = v1 + v2
}

scala> append(1, append(2, empty[Int]))
res2: Int = 3
{% endhighlight %}

Beautiful, right? Well, yes &mdash; type classes are indeed a wonderful creation responsible for much clean, generic, highly-maintainable, *beautiful* code.

But type classes, as implemented in both Haskell, Scala, and most other languages, have a darker side, too.

## Unprincipled Type Classes

Fundamentally, I want to argue that type classes in most all languages are *ad hoc* rather than principled, relying on personal discipline and conventions instead of the compiler to define and use correctly.

My criticisms aren't mean to argue for *abolishing* type classes, but rather, for making them *better* &mdash; something I believe is surely possible, and which I provide some ideas for at the end of the post.

### Laws

Type classes can't stand alone. To be useful writing generic code, they need to ship with *laws* that describe how they behave.

For example, an order type class (`Ord` in Haskell) can provide laws saying the ordering is *transitive* (that is, if `a <= b` and `b <= c`, then `a <= c`), that it's *anti-symmetric* (if `a <= b` and `b <= a`, then `a = b`), and that it's *total* (either `a <= b` or `b <= a`). Armed with these laws, you can then write a generic sorting algorithm which requires only that it's elements have an order type instance defined.

If you can't define laws for a type class, then it's not useful as an abstraction, and you should not try to define a type class. Instead, just write lawless functions and pass them around (or think harder about those laws!).

Unfortunately, while laws are sometimes documented in source code comments, they are not enforced by the compiler, and it's very common for developers to unintentionally (or even intentionally!) create instances that break those laws in ways that can cause nasty bugs.

In the ordering example, consider an instance for integers which says `1 <= 2` and `2 <= 1`, but denies that `1 = 2`. Then think about what happens when your generic sorting routine gets ahold of this instance!

Type class laws are the essence of abstraction: thanks to laws, they enable you to generically reason about how functions work independent of the particular types they operate on. Yet, laws are given no special treatment by most languages and compilers! 

### Orphans

For a given type class and type, there is often *more* than one way to write an instance for the type that satisfies its laws.

For integers, we can define multiple instances of the `Monoid` type class which satisfy all the laws. The additive monoid defines `empty` to be `0`, and `append` to be integer addition (`+`). The multiplicative monoid defines `empty` to be `1` and `append` to be integer multiplication (`*`).

Alarms should be going off in your head right now. If you can define multiple instances for a given type, which have completely different behavior (but nonetheless satisfy the type class laws), which one will the compiler choose?

You might write code that expects the additive monoid, but Haskell chooses the multiplicative monoid. Oh no!

For Haskell, at least, it turns out that if you define your instances in either the module in which the type is defined, or the module in which the type class is defined, then you're safe, because the compiler *won't let you* define multiple instances. Any code that imports both the type and the type class will automatically see the right (and only!) type class instance.

If for a given type class and type, instances are defined elsewhere (not in the modules that declare the type or the type class), the instances are referred to as *orphan instances*. 

For orphan instances, which type class instance you get depends on what you import!

We can't just ignore this issue or the reason why it exists. For a given type, there may be *many* (perhaps even infinitely many!) ways to implement an instance that satisfies the laws of the type class. 

In Haskell, the community's preferred solution is to create wrapper types (`newtype`) and define instances on the wrappers instead. You can then "force" the compiler to choose the instance you want by using the right wrapper type.

This practice, which I lovingly call *newtype abuse*, works well enough, but it's totally ad hoc and unprincipled. While you can force the compiler to select a desired instance by wrapping values in the right `newtype`, the instance satisfies *no more laws* than another `newtype` wrapper. 

Stated more forcefully, when we abuse newtypes, we are encoding functional requirements into the *name* of a `newtype` wrapper, instead of making the compiler verify those requirements!

If the *correctness* of your monoidal code depends on more properties being satisfied by an instance than just the `Monoid` laws (e.g. if you require the multiplicative monoid for integers), then the compiler should be able to verify those properties. 

The alternative is ascribing arbitrary and unenforceable significance to the names of data constructors.

#### Brief Digression on Newtypes

Newtypes aren't always bad, but they're often an example of "programming by name". For example, if you define `newtype Email = Email String`, you haven't restricted the set of valid values for `Email` &mdash; every valid string is also a valid email, and visa versa. On the other hand, if we use smart constructors for an `Email` type, then we can restrict the set of `Email` to be those strings which are actually valid emails (or better yet, use a more precise representation which corresponds 1-to-1 to an email!). 

Both of these approaches make the code easier to read, but only one of them actually leverages the compiler to enforce properties of correctness.

### Abstraction

In Haskell, you cannot abstract over type classes in the same way you can abstract over functions and other values. Why? Because type classes are magical &mdash; they're neither values nor types, so you can't rely on the ordinary machinery of abstraction.

Let's say I want to take two type classes, and programmatically combine them (and possibly one or more values) to create another type class. (TODO: Example.)

How would you express this in Haskell? You can't, because type classes are magical!

Ironically, in weaker languages that don't support type classes, abstraction is usually possible, because the type classes are emulated using first-class language features, such as dictionaries, interfaces, traits, or modules.

## Principled Type Classes?

I don't know what the *best* solution to these issues is. But I do have some ideas about what *I* want from type classes:

1. **Haskell-style**. A baked-in notion of type classes in the overall style of Haskell, Purescript, Idris, etc.
2. **Lawful**. First-class laws for type classes, which are enforced by the compiler.
3. **Hierarchical**. A compiler-verified requirement that a subclass of a type class must have at least one more law than that type class.
4. **Globally Unambiguous**. Type class resolution that produces an error if there exists more than one instances which satisfies the constraints at the point where the compiler must choose an instance.
5. **Abstractable**. The ability to abstract over type classes themselves.

I'll explore what I mean by each of these in the sections that follow.

### Haskell-Style

Some have made [compelling arguments](https://www.haskellforall.com/2012/05/scrap-your-type-classes.html) for abolishing type classes in Haskell, at least as baked-in language constructs. 

I'm obviously sympathetic to this point of view, but I just view the problems with built-in type classes as a reason to *make them better* rather than to *abolish them*.

The advantage for built-in type classes is not just concise code: if the compiler knows about type classes, it can make sure they're used in sensible ways that lead to the construction of correct, comprehensible software.

If you emulate type classes using ordinary values, then while you gain a whole host of benefits, you also lose some. I don't personally think the tradeoff is worthwhile &mdash; at least for "improved type classes" of the type I'm advocating.

### Lawful

By lawful, I want the ability to state the laws for a type class when defining that type class. In fact, I want the compiler to *force* me to do so!

Dependently-typed languages can do this easily in many cases. Idris, for example, introduces a `VerifiedMonoid` class which cannot be implemented for a type without proving its laws are satisfied for that type (unless you cheat!):

{% highlight haskell %}
class (VerifiedSemigroup a, Monoid a) => VerifiedMonoid a where
  total monoidNeutralIsNeutralL : (l : a) -> l <+> neutral = l
  total monoidNeutralIsNeutralR : (r : a) -> neutral <+> r = r
{% endhighlight %}

Dependent-typing, however, is not a panacea. You can't use it to prove laws about effects, like proving the monad laws for Haskell's `IO`. I'm not sure whether that's a good thing or a bad thing.

But I do think that dependent-typing isn't necessary to have lawful type classes. For non-dependently-typed languages, QuickCheck-style property verification is probably sufficient (`doctest` is already used for this purpose in the Haskell community).

One can imagine defining laws like this (warning: bullshit syntax):

{% highlight haskell %}
class Semigroup a => Monoid a where
  empty :: a
  
  law emptyIsIdentity a = ((a `append` empty) .==. a) .&&. (empty `append` a) .==. a
{% endhighlight %}

where the compiler would infer constraints for law parameters (for example, an `Arb` instance for `a`), and require that instances pass the laws of both `Monoid`, as well as its superclass `Semigroup`.

Interestingly, this approach might be possible in present-day Scala, although it would require macros.

### Hierarchical

Two type classes with the same laws have no useful distinction (different *names* are not a meaningful distinction!). Hence, the compiler should not allow them to exist. 

Every type class should have at least one more law than its super class. The (implicit) super class of all base type classes has no laws, thus forcing every base type class to have at least one law.

### Globally Unambiguous

At the point of instance selection &mdash; that is, when the compiler chooses an instance based on inferred types &mdash; there should be no ambiguity. If the compiler detects more than one instance satisfying the constraints, then the compiler should reject the program with an appropriate error message.

The compiler should not just consider instances defined in imports, but should *globally* consider *all instances* defined in the *entire* project and *all* its dependencies.

Notice I'm focused on *instance selection* and not *instance definition*. I'm actually in favor of allowing multiple instances that satisfy the same laws, as long as these instances are for different (if related) type classes.

For example, I'm OK with defining instances for `Integer` of two distinct subclasses of `Monoid`.

Why? Because fundamentally, *laws are all that matter*. Subclasses have even more laws than superclasses, and satisfying these laws might require a more specific kind of implementation, which satisfies superclass laws in a different (if still valid) way.

So even though I couldn't create two `Monoid` instances for `Integer`, one for addition and one for multiplication, I *would* be allowed to create two classes that derive from `Monoid` and define instances of each class for `Integer`. 

See the distinction?

Let's say I want to define both additive and multiplicative monoids for integers. My first step would be to create type classes which have *even more laws* that further restrict the implementation to have the properties I want.

For example:

{% highlight haskell %}
class Integral a where
  toInteger   :: a -> Integer  
  fromInteger :: Integer -> a
  
  law isoInteger a = (toInteger . fromInteger) a .==. a
  law isoA a = (fromInteger . toInteger) a .==. a

class (Monoid a, Integral a) => IntMultMonoid a where
  law appendIsIntMult a b = (fromInteger $ (toInteger a) * (toInteger b)) .==. a `append` b
  
class (Monoid a, Integral a) => IntAddMonoid a where
  law appendIsIntAdd a b = (fromInteger $ (toInteger a) + (toInteger b)) .==. a `append` b
{% endhighlight %}

(These are over-constrained &mdash; many uses could get by with weaker laws &mdash; but they're enough to demonstrate my point!)

Now assume we've defined instances of both of these type classes for `Integer`. How do I use them?

If I tried the naive approach, I'd run into problems:

{% highlight haskell %}
1 `append` 2
{% endhighlight %}

This would generate a compiler error, because there exist two instances which satisfy the laws for `Semigroup` and `Integer`.

However, I can further constrain my code as follows:

{% highlight haskell %}
addOneTwo :: (IntAddMonoid a) => a -> a -> a
addOneTwo a b = a `append` b

addOneTwo 1 2
{% endhighlight %}

My function `addOneTwo` has stronger constraints than just `Monoid`. So when I call that function with `Integer`, the compiler is able to select the unique instance which satisfies my stronger constraints.

Note that in this code, I'm explicitly capturing the fact that I *care* about more than just the `Monoid` laws when the compiler chooses the instance for `Integer`. I'm capturing that fact in a way that is *rigorous* and *compiler-verified*.

There would be several positive side effects of this approach:

1. **No Newtype Abuse**. There's no need for the ad hoc practice of abusing `newtype` to force Haskell to select the "right" type class instance. (In fact, I'd like the compiler to forbid this abuse of `newtype` for instance selection.)
2. **More Generic Code**. For example, the above function `addOneTwo` works on any type that's provably isomorphic to `Integer` (and some use cases wouldn't even require laws that strong).
3. **Lawful Thinking**. The compiler forces you to think about which additional laws you need to choose the instance you want, and to state those laws in compiler-verifiable properties.

### Abstractable

I think that making type classes abstractable and fully composable requires two things:

1. First-class modules, which can be passed around and manipulated like values. 
2. Probably named instances (which, for example, Purescript and Idris already support!). Named instances aren't necessary for instance selection, but they would be helpful as a concise way of referring to a specific "module".

Obviously, in languages that don't have type classes, the emulated type classes are ordinary values and can be abstracted and composed like other values.

First-class modules also seem to provide a clean answer to the problem of "type class coherency", but that's a topic for another post.

## Summary

Type classes have a *long ways* to go before they become *principled*.

Their current problems range from lawlessness to global ambiguity to complete failure to abstract.

My ideal solution would look a lot like Haskell's type classes, but add compiler-enforced, hierarchical laws, make type class selection (but not definition!) globally unambiguous, and allow abstraction and composition of type classes in all the same ways we can compose and abstract over values.

Where do you think type classes should go? Share your thoughts in the comments below.
