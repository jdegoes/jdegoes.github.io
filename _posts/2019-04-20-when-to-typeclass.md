---
layout:       post
title:        "Type Classes: When To Use Them, When To Avoid Them"
description:  "Type classes offer powerful capabilities, but knowing when to use them and when to avoid them is not simple."
category:     articles
tags:         [type classes, haskell, purescript, scala, cats, scalaz, mtl, tagless-final, functional programming, fp]
---

Type classes are a powerful tool for abstraction in functional programming. But they are poorly understood, and often abused.

In this post, I want to explain what a type class is&mdash;in a way that will be _quite_ foreign to most readers&mdash;and offer my own personal recommendations on when you should use type classes to solve a problem, and when you should avoid them.

_Note: This post assumes you already know how to create and use type classes._

## Type Class 101

In the simplest possible case, a _type class_ is a function from a type to a set of lawful operations on values of that type.

Let's take this definition apart, one piece at a time:

1. _A type class is a function ..._. This is not a function in the _programming language_ sense of the word, but rather, a function in the _mathematical_ sense of the word. We give a type class some type, like `Int`, and we get back something (just _one_ something). Moreover, when we give the type class the _same_ type, we get back the _same_ thing. How this function is encoded varies depending on the capabilities of the host language. For example, Scala and Haskell have quite different mechanisms for representing type classes.
2. _...from a type..._. Technically, type classes can have multiple parameters, not just one; and technically, they may not be _types_, but rather, _type constructors_. But in the common case, type classes are functions that take one type. Some languages, like Java or Kotlin, don't have higher-kinded type parameters.
3. _...to a set..._. Like _function_ above, this is not a _programming language_ set, but rather, a _mathematical_ set. Each operation in the set is unique (there are no duplicates). Like with "function" above, how the set is encoded varies depending on the programming language.
4. _...of lawful operations..._.  The operations provided by the type class are governed by laws. Laws are rather like unit tests, except they apply in _all_ cases, not just _some_ cases. A few programming languages (like Idris) can express laws in a way the compiler will enforce, but most programming languages rely on manual use of property-checking to verify laws are satisfied as part of a test suite.
5. _...on values of that type_. The operations provided by the type class are all operations on _values_ that have the type described in (2).

A type class _instance_, on the other hand, is a _piecewise definition_ of the type class function for a given type. The type class function is not defined for _all_ types, but only for _some_ types, and those definitions are described by type class _instances_. A type class is not defined for a given type when there does not exist an instance of the type class for that type. On the other hand, if a type class is defined for some type, then the instance provides the definition of the operations for that type.

Type classes compliment _parametric polymorphism_. Parametric polymorphism allows us to create data structures and functions that are _polymorphic_ in a type _parameter_. 

Parametric polymorphism serves two useful functions:

1. It allows us to create reusable data structures and functions, which can be used across not just a single (monomorphic) type, but across lots of types;
2. It allows us to throw away information that is not relevant to a problem, constraining possible solutions, and improving guarantees of correctness.

In the following code, we write _parametrically polymorphic_ code that accesses the 2nd element of a tuple:

{% highlight scala %}
def snd[A, B](tuple: (A, B)): B = tuple._2
{% endhighlight %}
{% highlight haskell %}
snd :: (a, b) -> b
snd (_, b) = b
{% endhighlight %}

This code can work for all types `A` and `B`, and can therefore work for all tuples of arity 2. It is very resuable, and doesn't know more about the types than it needs to.

While parametric polymorphism allows us to _throw away_ structure, to make code more reusable and constraint implementations, sometimes it throws away _too much_ structure to solve a given problem. In the above definition of `snd`, the function does not know _anything_ about the types `A` and `B`, which is fine for this problem, but which is not fine for most problems.

Most polymorphic functions will require _some_ structure from a type: they will need the ability to do _something_ with values of that type. For example, they might need to combine them, compare them, iterate over them, or construct them.

To add structure to a type, to give us some _operations_, we use type classes. Type classes, being functions from types to a set of lawful operations on values of those types, give us the ability to operate with values of some unknown type. Although we may not know what the type is, thanks to type classes, we have enough structure to generically solve our problem.

The final piece of the puzzle is called a _type class constraint_. Type class constraints are the mechanism by which a language allows us to _statically require_ that some polymorphic type have an _instance_ for a given type class.

### Example

In the following example, we define a type class called `Ord`, which provides a single lawful operation for some type `A`.

{% highlight scala %}
sealed trait Ordering
object Ordering {
  object LT extends Ordering
  object EQ extends Ordering
  object GT extends Ordering
}
trait Ord[A] {
  def compare(l: A, r: A): Ordering
}
object Ord[A] {
  def apply[A](implicit A: Ord[A]): Ord[A] = A
}
{% endhighlight %}
{% highlight haskell %}
data Ordering = LT | EQ | GT
class Ord a where 
  compare :: a -> a -> Ordering
{% endhighlight %}

In Scala, if we wish to obtain the operation for some type `A`, let's say `Int`, we utilize the expression `Ord[Int]`, which gives us the _instance_ for the type `Int`. If such an instance has not been defined, then we will get a compile-time error. 

Similarly, in Scala, if we wish to add a _constraint_ on some parametric parameter, we can use context bounds, which are how constraints are encoded in Scala:

{% highlight scala %}
def sort[A: Ord](list: List[A]): List[A])
{% endhighlight %}

In Haskell, if we wish to obtain the operation for some type `A`, let's say `String`, we cannot call the type class function directly, because instances are not first-class values in Haskell. However, we can use the type class operation, and Haskell will automatically "call" the type class function to retrieve the instance for that type. If such an instance has not been defined, we will get a compile-time error.

In Haskell, if we wish to add a constraint, we use a type class constraint:

{% highlight scala %}
sort :: Ord a => [a] -> [a] -> [a]
{% endhighlight %}

In both of these examples, the use of parametric polymorphism allows us to throw away irrelevant details, making our sort function more generic, and constraining possible implementations, leading to stronger guarantees on correctness.

With this background out of the way, let's talk about when to use type classes, and more importantly, when to _not_ use them!

## Type Class Best Practices

The purpose of type classes is to add _structure_ to polymorphic types&mdash;just enough structure to allow us to solve our problem, but not more structure than necessary, so we can benefit from maximum code reuse, and maximally constrain our implementation.

The word _structure_ is critically important. By _structure_, I mean _algebraic structure_: the operations that a type class provides us with satisfy some properties (the _laws_) across _all types_ for which there is an instance.

Without a notion of _structure_, we cannot know the meaning of generic code. In the previous example of `sort`, our polymorphic function requires the ability too totally order the elements of the list it is sorting. Without a notion of _total order_, embedded into the laws of the `Ord` type class, we cannot meaningfully define what it is to "sort" elements.

Structure allows _abstraction_: we can write code that is generic across a range of different types _precisely_ because we can _define_ the ways in which those types are similar. In the case of sorting, we can abstract over the element type, because we can define what it means to compare elements of a given type. Although `Int` and `String` are quite different, they both posess enough structure to provide a total ordering.

Abstraction is _not_ the same as _indirection_. _Indirection_ provides a layer of insulation between our code and concrete data types, and is useful for testability and modularity.

Indirection is not necessarily governed by laws (algebraic or otherwise). In object-oriented programming languages, interfaces are often used to provide _indirection_, where it is sometimes mistakenly called _abstraction_. In functional programming languages, records of functions or modules can be used to provide indirection.

_The structure that type classes provide is the foundation for writing well-defined generic code; without structure, there is no abstraction&mdash;only indirection._

This leads to my rule of when to use type classes:

> **Consider using type classes whenever you can identify enough common structure across a range of data types that is by itself wholly sufficient for writing well-defined generic code across these data types**

Conversely, if you cannot identify common structure (through a set of well-defined algebraic laws), then you cannot write well-defined generic code across different data types. This is a strong sign that what you really want is _indirection_, not abstraction.

This leads to my rule of when **not** to use type classes:

> **Consider using indirection whenever you cannot identify enough common structure across a range of data types that is wholly sufficient for writing well-defined generic code**

Now sometimes we can define structure across a range of data types, but that structure is not wholly sufficient for writing generic code. 

An example is the algebraic structure called _magma_. A _magma_ provides an composition operation with _closure_ (that i8s, if you compose two values of the same type, you get back another value of the same type). This is less structure than a semigroup, because semigroups also provide a guarantee of _associativity_.

The guarantee of closure is so weak, that we cannot write much (if any) generic code that only requires this guarantee. In fact, closure is so weak, no laws are necessary to express: a parametrically polymorphic type system provides us this guarantee "for free".

This is why it is so critically important that the structure provided by the type class be sufficiently strong that useful generic code is possible.

If you are looking at some structure that is sufficiently rare so as to permit very few useful instances for a given type, then there's a good chance that a type class will allow you to write generically useful code. Conversely, if you are looking at some structure that permits infinitely many equally useful instances, then there's a good chance having a type class will not allow you to write generically useful code.

I consider these best practices for when to use and when not to use type classes. However, in the next section, I want to provide some practical reasons for following this advice, as well as show you what to do when type classes aren't a good fit.

## The Power of Values

Type classes provide a means of accessing structure as a function of type. 

As I have defined them, anyway, a type class has a _single_ instance for any type. If for any reason a type class had _multiple_ instances for a given type, then this would imply that type classes are _not_ functions&mdash;at least, not _functions_ in the mathematical sense of the word, because they would map the same type to _different_ sets of operations.

Such non-determinism would mean that any given type would no longer map to a unique set of operations. It would further imply that we could not know, just looking at a concrete type, what set of operations a type class would give us access to. 

Indeed, one of the chief problems of introducing non-determinism into my definition of a type class is that simple refactorings, like moving a function from one file to another, can change the runtime behavior of your program&mdash;a fact that Scala developers know all too well, thanks to their exposure to _implicit abuse_.

Ensuring that type classes are _globally coherent_ gives us the ability to clearly and easily understand how our program will behave, and to preserve this behavior in the presence of refactoring, without having to be cautious.

Yet, while global coherence (_determinism_ of the type class function) is important for reasoning and refactoring, there is something distinctly anti-modular about type classes. Type classes are, after all, _global_, _piecewise_ defined functions from types to sets of operations.

_Global_ definitions are opposed to _local_ definitions, and _local, compositional definitions_ are the foundation of _modularity_.

Herein lies the fundamental tradeoff of type classes: type classes provide global (anti-modular), type-directed reasoning that stems from the very algebraic structure of the types in question; while modules and their weaker variants (records of functions), provide local (modular), value-directed reasoning that stems from the makeup of the values themselves.

Although the battle between type classes and modules goes back for decades, there is no real competition, because while type classes are a natural fit for algebraic structure, modules (or records of functions) are a natural fit for indirection.

Indirection gives us loose coupling between pieces of the whole, allowing us to, for example, connect to different types of databases at runtime, accessing them through a uniform interface; allowing us to support local logging, but also remote logging; allowing us to have a local file system, and a remote one; and allowing us to have test implementations, as well as production implementations.

Type classes give us algebraic structure in the small, and indirection gives us modularity in the large. They do not compete with each other, but rather, compliment each other, and the weaknesses of the one are the strengths of the other.

Some beautiful properties of modules (and records of functions) include the following:

- **Superior Composition**. We can take values, and compose them with other values, to yield values. While type class composition is possible, it looks and feels awkward because it has to be encoded in a different language than the primary, expression-oriented language that we write most of our code in.
- **No Dependence on Type**. While type classes are functions of types to values, modules are just ordinary values, so for a given type (or set of types), we can have as many different modules as we like. For example, we can have a different database module for each underlying database type (MySQL, PostgreSQL, SQL Server, etc.), not connected to the type of effect we are using or any other type.
- **Superior Customization**. We can easily and programmatically mix and match pieces of different modules, for example, drilling into a module and adding instrumentation to a single operation. Optics make these types of customizations trivial, but optics work on data, and type classes are not data.

The benefits of working with ordinary values are tremendous and should not be overlooked. Now it's also true that type classes have their own benefits. The lure of these benefits is sometimes so strong, it leads to _type class abuse_.

## Type Class Abuse

Type class abuse is what happens when we use a type class to represent something that _should_ be a module (or record of functions). 

Typical symptoms of type class abuse include the following:

 - The type class has _no laws_ that permit well-defined generic use of the type class
 - There are _many_ possible useful instances of the type class for a given type
 - The type class exists primarily to avoid passing or composing values

