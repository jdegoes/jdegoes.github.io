---
layout:       post
title:        "Modern Functional Programming: Part 2"
description:  "The onion architecture may be the future of large-scale FP"
category:     articles
tags:         [fp, functional programming, haskell, purescript, free, mtl]
---

Late last year, I wrote my thoughts on what the [architecture of modern
functional programs should look like](/articles/modern-fp).

The post generated vigorous discussion in the community, perhaps because I
railed against the `IO` monad and advocated for `Free` monads, which are now
used pervasively in [Quasar Analytics Engine](https://github.com/quasar-analytics/quasar/),
one of the open source projects that [my company](https://slamdata.com) develops.

Since then, I've had a chance to read responses, look at [equivalent
architectures](https://gist.github.com/ocharles/6b1b9440b3513a5e225e) built upon
Monad Transformers Library (MTL), and even [talk about](https://www.slideshare.net/jdegoes/mtl-versus-free) my recent experiments
at [LambdaConf 2016](https://lambdaconf.us).

The result is a sequel to my original post, which consists of a newly-minted,
tricked-out recommendation for architecting modern functional programs, along
with new ways of thinking about the structure of this architecture.

# Onion Architecture

The modern architecture for functional programming, which I will henceforth
call the *onion architecture* (because of its similarity to [a pattern of the
same name](https://jeffreypalermo.com/blog/the-onion-architecture-part-1/)),
involves structuring the application as a series of layers:

1. At the center of the application, semantics are encoded using the language
of the domain model.
2. Beginning at the center, each layer is translated into one or more
languages with lower-level semantics.
3. At the outermost layer of the application, the final language is that of the
application's *environment* — for example, the programming language's standard
library or foreign function interface, or possibly even machine instructions.

At the top-level of the program, the final language of the application is
trivially executed by mapping it onto the environment, which of course involves
running all the effects the application requires to perform useful work.

## The Free Edge

The onion architecture can be implemented in object-oriented programming or in
functional programming.

However, the limitations of type systems in most object-oriented programming
languages generally imply that implementations are *final*. Pragmatically
speaking, what this means is that object-oriented programs written using the
onion architecture cannot benefit from any sort of runtime introspection and
transformation.

Within functional programming, the choices are Monad Transformers Library
(MTL), or something equivalent to it (the *final* approach, though note that the
type classes from MTL can be subverted to build Free structures); or Free
monads, or something equivalent to them (the *initial* approach).

It's even possible to mix and match MTL and Free within the same program, which
comes with the mixed tradeoffs you'd expect.

As shown by [Oliver Charles](https://gist.github.com/ocharles/6b1b9440b3513a5e225e),
MTL can indeed quite simply implement the onion architecture, without any help
from Free. However, the following caveats apply:

1. **Tangled Concerns**. MTL implies a linear stack of monads. Since interpreter
logic goes into type class instances, this involves tangling concerns. For
example, a logging interpreter must also delegate to some other interpreter to
expose the semantics of the logging class.
2. **No Introspection**. MTL does not allow introspection of the structure of the
program for the purpose of applying a dynamic transformation. For one, this means
program fragments cannot be optimized. Other implications of this limitation are
being explored as new ways are discovered to use free monads.
3. **Over-specified Operations**. Monad classes in MTL must be over-specified
for performance reasons. If one operation is semantically a composite of others,
the type class must still express both so that instances can provide high-
performance implementations. Technically, this is an implication of (2), but
it's important enough to call out separately.

Free monads have none of these drawbacks:

1. Free monads permit more decoupling between interpreters, because one
interpreter does not have to produce the result of the operation being
interpreted (or any result at all, in fact).
2. Free monads permit unlimited introspection and transformation of the structure
of your program (*EDIT*: up to the information-theoretic limit; see [my talk on
Free applicatives](https://www.youtube.com/watch?v=H28QqxO7Ihc), which support
sequential code just like free monads but allow unbounded peek-ahead).
3. Free monads allow minimal specification of each semantic layer, since
performance can be optimized via analysis and transformation.

On the second benefit, I have previously discussed optimization of programs via
[free applicatives](https://www.youtube.com/watch?v=H28QqxO7Ihc), and I also
recently demonstrated a [mocking library](https://github.com/slamdata/purescript-mockfree)
that exposes composable, type-safe combinators for building expectations &mdash;
something not demonstrated before and apparently impossible using monad
transformers.

For all these reasons, I endorse free monads as the *direction* of the future.
However, most functional programming languages have derived approaches
that reduce or eliminate some of the boilerplate inherent in the original
approach (see [FreeK](https://github.com/ProjectSeptemberInc/freek) and
[Eff-Cats](https://github.com/atnos-org/eff-cats) in Scala, for example).

In my opinion, the wonderful polymorphism of monad type classes in MTL is the
best thing about MTL (rather than the transformers themselves), and clearly
superior to how early Free programs were built.

Nonetheless, Free has an equivalent mechanism, which I'm dubbing *Free
Transformers* (FT), which goes head-to-head with MTL and even allows
developers to target both MTL and Free, for portions of their code.

## Free Transformers

Old-fashioned Free code is monomorphic in the functor type. With functor
injection, this becomes more polymorphic, but functor injection is just a
special case of polymorphic functors whose capabilities are described by type
classes.

Fortunately, we can replicate the success of type classes in MTL in
straightforward fashion.

Let's say we're creating a semantic layer to describe console input. The first
step is to define a type class which describes the operations of our algebra:

{% highlight haskell %}
class Console f where
  readLine :: f String

  writeLine :: String -> f Unit
{% endhighlight %}
{% highlight scala %}
trait Console[F[_]] {
  def readLine: F[String]

  def writeLine(line: String): F[Unit]
}
{% endhighlight %}

Notice that unlike in MTL, `Console` is *not* necessarily a monad. This
weakening allows us to create instances for data types that capture the
structure of these operations but do not provide a context for composing them.
This allows code that is polymorphic in the type of data structure used to
represent the operations.

Of course, monadic instances may be defined for Free, and any code that
requires monadic or applicative composition can use additional constraints:

{% highlight haskell %}
myProgram :: forall f. (Console f, Monad f) => f Unit
{% endhighlight %}
{% highlight scala %}
def myProgram[F[_]: Console: Monad]: F[Unit]
{% endhighlight %}

Laws for type classes like this can be specified by embedding the functor into
a suitable computational context such as `Free`.

The name "transformers" comes from the fact that functors compose when nested.
The outer functor "transforms" the inner functor to yield a new composite
functor, and Free programs are usually built from compositional functors.

The Free Transformers approach allows maximum polymorphism. In fact, there's
enough polymorphism that a lot of your code doesn't need to *care* whether you
implement with Free or monad transformers!

## A Worked Example

Let's work a simple example in the onion architecture using free monads and the
Free Transformers approach to abstracting over functor operations.

Suppose we're building a banking service with the following requirements:

1. The accounts may be listed.
2. The balance in an account may be shown.
3. Cash may be withdrawn from an account in multiples of $20.
4. Cash may be transferred from one account to another.

Our first step is to create a type class to represent the operations available
in our domain model:

{% highlight haskell %}
data From a = From a
data To a = To a

type TransferResult = Either Error (Tuple (From Amount) (To Amount))

class Banking f where
  accounts :: f (NonEmptyList Account)
  balance  :: Account -> f Amount
  transfer :: Amount -> From Account -> To Account -> f TransferResult
  withdraw :: Amount -> f Amount
{% endhighlight %}
{% highlight scala %}
case class From[A](value: A)
case class To[A](value: A)

type TransferResult = Either[Error, (From[Amount], To[Amount])]

trait Banking[F[_]] {
  def accounts: F[NonEmptyList[Account]]
  def balance(account: Account): F[Amount]
  def transfer(amount: Amount, from: From[Account], to: To[Account]): F[TransferResult]
  def withdraw(amount: Amount): F[Amount]
}
{% endhighlight %}

Our next step is to create a data structure for representing these operations
independent of any computational context:

{% highlight haskell %}
data BankingF a
  = Accounts (NonEmptyList Account -> a)
  | Balance Account (Amount -> a)
  | Transfer Amount (From Account) (To Account) (TransferResult -> a)
  | Withdraw Amount (Amount -> a)

instance bankingBankingF :: Banking BankingF where
  accounts = Accounts id
  balance a = Balance a id
  transfer a f t = Transfer a f t id
  withdraw a = Withdraw a id
{% endhighlight %}
{% highlight scala %}
sealed trait BankingF[A]
case class Accounts[A](next: NonEmptyList[Account] => A) extends BankingF[A]
case class Balance[A](next: Amount => A) extends BankingF[A]
case class Transfer[A](
  amount: Amount, from: From[Account], to: To[Account],
  next: TransferResult => A) extends BankingF[A]
case class Withdraw[A](amount: Amount, next: Amount => A) extends BankingF[A]
{% endhighlight %}

Now we can create an instance for Free that can be automatically derived from
any suitable functor:

{% highlight haskell %}
instance bankingFree :: (Banking f) => Banking (Free f) where
  accounts = liftF accounts
  balance a = liftF (balance a)
  transfer a f t = liftF (transfer a f t)
  withdraw a = liftF (withdraw a)
{% endhighlight %}
{% highlight scala %}
implicit def BankingFree[F[_]](implicit F: Banking[F]): Banking[Free[F, ?]] =
  new Banking[Free[F, ?]] {
    def accounts: Free[F, NonEmptyList[Account]] = Free.liftF(F.accounts)
    def balance(account: Account): Free[F, Amount] = Free.liftF(F.balance(account))
    def transfer(amount: Amount, from: From[Account], to: From[Account]):
      Free[F, TransferResult] = Free.liftF(F.transfer(amount, from, to))
    def withdraw(amount: Amount): Free[F, Amount] = Free.liftF(F.withdraw(amount))
  }
{% endhighlight %}

At this point, we can define high-level programs that operate in our business
domain, without tangling other concerns such as banking protocols, socket
communication, and logging:

{% highlight haskell %}
example :: forall f. (Inject BankingF f) => Free f Amount
example = do
  as <- accounts
  b  <- balance (head as)
  return b
{% endhighlight %}
{% highlight haskell %}
def example[F[_]: Inject[BankingF, ?]]: Free[F, Amount] =
  for {
    as <- F.accounts
    b  <- F.balance(as.head)
  } yield b
{% endhighlight %}

After we've defined our high-level program, we can formally express the meaning
of this program in terms of the next layer in the onion.

But before we do that, let's first introduce a notion of `Interpreter` that can
give one layer semantics by defining it in terms of another:

{% highlight haskell %}
type Interpreter f g = forall a. f a -> Free g a

infixr 4 type Interpreter as ~<
{% endhighlight %}
{% highlight scala %}
type Interpreter[F[_], G[_]] = F ~> Free[G, ?]

type ~<[F[_], G[_]] = Interpreter[F, G]
{% endhighlight %}

An interpreter `f ~< g` (`F ~< G`) provides a meaning for each term in `F` by
attaching a sequential program in `G`. In other words, interpreters define the
meaning of one layer of the onion in terms of another.

(Other definitions of interpreters are possible and useful, but this suffices
for my example.)

These interpreters can be composed horizontally, by feeding the output of one
interpreter into a second interpreter, or vertically, by feeding values to
two interpreters and then appending or choosing one of the outputs.

When using this notion of sequential interpretation, it's helpful to be able to
define an interpreter that doesn't produce a value, which can be done using the
`Const`-like construct shown below:

{% highlight haskell %}
type Halt f a = f Unit
{% endhighlight %}
{% highlight scala %}
type Halt[F[_], A] = F[Unit]
{% endhighlight %}

Then an interpreter from `f` to `g` which produces no value is simply
`f ~< Halt g` (`F ~< Halt[G, ?]`). These interpreters are used purely for their
effects, and arise frequently when weaving aspects into Free programs.

Now, let's say that we create the following onion:

1. Banking is defined in terms of its protocol, which we want to log.
2. The protocol is defined in terms of socket communication.
3. Logging is defined in terms of file IO.

Finally, at the top level, we define both file IO and socket communication
in terms of some purely effectful and semantic-less `IO`-like monad.

Rather than take the time to define all these interpreters in a realistic
fashion, I'll just provide the type signatures:

{% highlight haskell %}
bankingLogging :: BankingF ~< Halt LoggingF

bankingProtocol :: BankingF ~< ProtocolF

protocolSocket :: ProtocolF ~< SocketF

loggingFile :: LoggingF ~< FileF

execFile :: FileF ~> IO

execSocket :: SocketF ~> IO
{% endhighlight %}
{% highlight scala %}
val bankingLogging : BankingF ~< Halt LoggingF

val bankingProtocol : BankingF ~< ProtocolF

val protocolSocket : ProtocolF ~< SocketF

val loggingFile : LoggingF ~< FileF

val execFile : FileF ~> IO

val execSocket : SocketF ~> IO
{% endhighlight %}

After implementing these interpreters, you can wire them together by using a
bunch of seemingly unfamiliar utility functions that ship with Free
implementations (more on this later).

The final composed program achieves complete separation of concerns and domains,
achieving a clarity, modularity, and semantic rigor that's seldom seen in the
world of software development.

In my opinion, this clarity, modularity, and semantic rigor &mdash; rather than
the specific reliance on a Free monad &mdash; is the future of functional
programming.

Now let's take a peek into the structure of this approach and see if we can
gain some additional insight into why it's so powerful.

# Higher-Order Category Theory

If you spend any time writing programs using Free, you'll become quite good
at composing interpreters to build other interpreters.

Before long, you will identify similarities between various utility functions
(such as free's `liftF`, which lifts an `f a` into a `Free f a` /
`F[A] => Free[F, A]`) and functions from the standard functor hierarchy (`point`).

In fact, these similarities are *not* coincidental. Due to language limitations,
the notion of "functor" that we have baked into our functional programming
libraries are quite specialized and limited.

Beyond this world lies another one, far more powerful, but too abstract for us
to even express properly in the programming languages of today.

In this world, Free forms a higher-order monad, capable of mapping, applying,
and joining ordinary (lower-order) functors!

If we want to express this notion in current programming languages, we have to
introduce an entirely new set of abstractions &mdash; a mirror functor
hierarchy, if you will.

These abstractions would look something like this:

{% highlight haskell %}
class Functor1 t where
  map1 :: forall f g. (Functor f, Functor g) => f ~> g -> t f ~> t g

class (Functor1 t) <= Apply1 t where
  zip1 :: forall f g. (Functor f, Functor g) => Product (t f) (t g) ~> t (Product f g)

class (Apply1 t) <= Applicative1 t where
  point1 :: forall f. (Functor f) => f ~> t f

class (Functor1 t) <= Bind1 t where
  join1 :: forall f. (Functor f) => t (t f) ~> t f

class (Bind1 t, Applicative1 t) <= Monad1 t
{% endhighlight %}
{% highlight scala %}
trait Functor1[T[_[_], _]] {
  def map1[F[_]: Functor, G[_]: Functor](fg: F ~> G): T[F, ?] ~> T[G, ?]
}

trait Apply1[T[_[_], _]] extends Functor1[T] {
  def zip1[F[_]: Functor, G[_]: Functor]:
    Product[T[F, ?], T[G, ?], ?] ~> T[Product[F, G, ?], ?]
}

trait Applicative1[T[_[_], _]] extends Apply1[T] {
  def point1[F[_]: Functor]: F ~> T[F, ?]
}

trait Bind1[T[_[_], _]] extends Functor1[T] {
  def join1[F[_]: Functor]: T[T[F, ?], ?] ~> T[F, ?]
}

trait Monad1[T[_[_], _]] extends Bind1[T] with Applicative1[T]
{% endhighlight %}

Composing interpreters together becomes a matter of using ordinary functor
machinery, albeit lifted to work on a higher-order!

These higher-order abstractions don't stop at the functor hierarchy. They're
everywhere, over every bit of machinery that has a category-theoretic basis.

For example, we can define higher-order categories:

{% highlight haskell %}
class Category1 arr1 where
  id1 :: forall f. (Functor f) => arr1 f f

  compose1 :: forall f g h. (Functor f, Functor g, Functor h) =>
    arr1 g h -> arr1 f g -> arr1 f h
{% endhighlight %}
{% highlight scala %}
trait Category1[Arr1[_[_], _[_]]] {
  def id1[F[_]: Functor]: Arr1[F, F]

  def compose1[F[_]: Functor, G[_]: Functor, H[_]: Functor](
    gh: Arr1[G, H], fg: Arr1[F, G]): Arr1[F, H]
}
{% endhighlight %}

The advantage of recognizing these abstractions and pulling them out into type
classes is that we can make our code *way* more generic.

For example, we can write code to compose and manipulate interpreters that
doesn't depend on Free, but can operate on other suitable computational
contexts (such as `FreeAp`).

The discovery of whole new ways of building programs may depend on our ability
to see past the crippled notions of category theory baked into our libraries.
Notions ultimately rooted in the limitations of our programming languages.

# Denotational Semantics

Denotational semantics is a mathematical and compositional way of giving meaning
to programs. The meaning of the program as a whole is defined by the meaning of
the terms comprising the program.

Denotational semantics provide an unprecedented ability to reason about programs
in a composable and modular fashion.

The *onion architecture* provides a way of specifying *whole programs* using
denotational semantics, where the meaning of one domain is precisely and
compositionally defined in terms of another domain.

# Recursion Schemes

Recursion schemes are generic ways of traversing and transforming data structures
that are defined using fixed-point types (which are capable of "factoring out"
the recursion from data structures).

Recursion schemes are useful in lots of places, but where they really shine is
complex analysis and transformation of recursive data types. They are used in
compilers for translating between higher-level layers (such as a program's AST)
to lower-level layers (such as an intermediate language or assembly language),
and for performing various analyses and optimizations.

If you know about recursion schemes and start using free monads, eventually you
discover that `Free` is a fixed-point type for describing value-producing
programs whose unfolding structure depends on runtime values.

What this means is that you can leverage suitably-generic recursion schemes to
analyze and transform Free programs!

Any program written using the onion architecture can be viewed as a compiler.
The source language is incrementally and progressively translated to the target
language (through one or more composable intermediate languages).

With the onion architecture, all programs are just compilers!

Recall the rise and fall of domain specific languages (DSLs), which held enormous
promise, but were too costly to implement and maintain. `Free`, combined with a
suitably powerful type system, provides a way to create type-safe domain-specific
languages, and give them precise semantics, without any of the usual overhead.

# Conclusion

The onion architecture has proven an enormously useful tool for structuring
large-scale functional programs in a composable and modular way. This
architecture isolates separate concerns and separate domains, and allows a
rigorous treatment of program semantics.

While the onion architecture can be implemented in many ways, I prefer
implementations using `Free`-like structures, because of the better separation
of concerns and potential for program introspection and transformation. When
combined with the Free analogue of MTL's type classes, the approach becomes
easier to use and much more polymorphic.

This architecture's connection to denotational semantics, the surprising
emergence of higher-order abstractions from category theory that arise from
composing interpreters, and the natural applicability of recursion schemes, are
all promising glimpses at the future of functional programming.

To be clear, I don't think `Free` is the future of functional programming. The
`Free` structure itself is insufficiently rich, a mere special case of something
far more general. But it's enough to point the way, both to a "post-Free" world,
and to a distinctly *algebraic* future for programming.

To get *all* the way there with high performance and zero boilerplate, we're
going to need not just new *libraries*, but most likely, whole new *programming
languages*.

But there's still a lot of development possible in our current programming
languages, and lots of people are working in the space.

Stay tuned for more, and please share your own thoughts below.

**EDIT**: Please see [this repository](https://github.com/notxcain/onion-architecure)
for an example implementation in Scala/Cats constructed by [Denis Mikhaylov](https://twitter.com/notxcain).
