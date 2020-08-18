---
layout:       post
title:        "An Introduction to Functional Design"
description:  "Functional design is a framework for using functional programming to solve every day problems, without jargon or fancy techniques"
category:     articles
tags:         [functional design, domain-driven design, functional programming, functional domains, algebra]
---

Functional programming doesn't need to be complex, confusing, or theoretical. 

Instead, functional programming can be simple, natural, and practical, helping you solve problems 
with more power and more joy than ever before.

A key to unlocking this potential is understanding _functional design_, a framework for applying 
functional programming to real world problems that I have been teaching at 
[Ziverge](https://ziverge.com) as well as on my [Patreon mentorship program](https://patreon.com).

Functional design helps you put the "practical" in functional programming, distilling the essence of 
functional programming into a toolkit that you can use anywhere, any time, in any programming 
language, and with either greenfield or legacy code bases.

Although developing skills in functional design requires work, time, and practice, the concepts are 
few and simple&mdash;perfect content for a blog post, in fact. 

So in this post, I'm going to introduce you to functional design!

## Elements of Functional Design

In _functional design_, we create _immutable data types_ that _model_ solutions to problems in some 
_domain of interest_, and equip these data types with _constructors_, which let us build _simple 
solutions_, and _composable operators_, which let us transform and combine solutions in a 
predictable way, so that we can use a small set of primitive operators and constructors to solve all 
possible problems in the domain of interest.

That's quite a mouthful! So let's break it down, one bite at a time:

 - _Functional design_. Functional design is an alternative to object-oriented design. Both try to 
 solve business problems in ways that reduce defects and lower the cost of maintenance.
 - _Immutable data types_. Functional programming uses immutable data types, which are a better fit 
 for concurrent applications and which invert control (the functions that you call cannot modify the 
 data that you pass them), making code maintenance easier and less risky.
 - _Model_. Most functional code doesn't solve problems _directly_. Instead, it creates 
 _models_ of solutions to problems, which are later executed (or _interpreted_). For example, a 
 route in a REST web server models handling a URL path, which is later executed on each request. 
 Models are always represented using immutable data types.
 - _Solutions_. At every level of an application, there are problems that require solutions. Data 
 needs to be validated, requests need responses, business logic needs to trigger emails, numbers 
 need to be aggregated, and so forth.
 - _Domain of interest_. In most applications, there are dozens to hundreds of different domains of 
 interest, each of which contains related entities and similar problems that we would like to solve 
 in a way that is easy to test and maintain.
 - _Constructors_. Constructors let us build values using our _immutable data types_ that model 
 solutions to _simple_ problems.
 - _Composable operators_. Operators let us transform and combine solutions to subproblems so we can 
 make values that model solutions to larger, much more complex problems.

Functional design works great with legacy code and greenfield code, although you may use slightly 
different techniques in each of these cases.

A _functional domain_ is a shorthand way to refer to some particular domain of interest, together 
with its immutable data type that models solutions to problems, and the model's constructors and 
operators.

In the next section, we'll take a look at a few example functional domains.

## Example Domains

Functional design is at the heart of all great functional programming libraries. 

Below are a few of the examples you may have already seen:

* **Parser Combinators**. Parser combinator libraries are designed for the domain of parsing text 
formats into data structures. A parser itself is a model that describes how to parse some piece of 
data from a character input sequence. Parser constructors let you build simple parsers, while 
operators let you transform and combine parsers so you can parse more complicated formats. You can 
execute parsers, which involves feeding them input to produce either an error or a value that 
captures the structure in the parsed data.
* **Functional Effects**. Functional effect libraries like [ZIO](https://zio.dev) are designed for
the domain of concurrent programming, with safe error management and resource handling. An effect is 
a model that describes how to execute a series of sequential and concurrent operations. Effect 
constructors let you build simple (non-concurrent) operations, while operators let you transform and 
combine effects so you can model much more sophisticated concurent tasks. You can execute effects, 
which performs all the operations they describe.
* **Optics**. Optics libraries like [Monocle](https://www.optics.dev/Monocle/) are designed for the 
domain of accessing and transforming immutable data structures. An optic is a model that describes 
how to zoom into a piece of data, and retrieve or modify it in some fashion. Optic constructors let 
you build simple accessors and modifiers, while operators let you transform and combine optics so 
you can model access and modification on gigantic, highlly nested data structures. You can 
"execute" optics, which involves feeding them data and transformations to get transformed data.
* **Streams**. Stream libraries like [ZIO Stream](https://zio.dev) are designed for the domain of 
concurrent streaming data. A stream is a model of a concurrently generated sequence of values. 
Stream constructors let you build simple streams, while operators let you transform and combine 
streams so you can model sophisticated data pipelines. You can execute streams, which involves 
reading elements from their source locations, and concurrently transforming and combining them as 
per the model.

I will take one example functional domain in ZIO and drill down to the level of actual code, to make 
everything as concrete as possible.

### In Depth: ZIO Schedule

_ZIO Schedule_ is an immutable data type that models a solution to the problem of specifying a 
finite or infinite recurring schedule.

As with every functional domain, there is a model, constructors, and operators.

The model is the data type `zio.Schedule[Env, In, Out]`, which describes a schedule that runs in 
some environment `Env`, requires inputs of type `In`, and produces outputs of type `Out`.

When schedules are used to retry failed effects, the input type to the schedule is the error type of 
the effect. When schedules are used to repeat successful effects, the input type to the schedule is 
the success type of the effect.

Like all models, a `Schedule` doesn't actually do anything except describe a recurring schedule
(models _describe_, they don't _do_). To execute a schedule model, you can call the `retry` or 
`repeat` methods on any ZIO effect, and feed them the schedule.

The constructors give us schedules that model solutions to very simple problems. For example, if we 
wish to describe a recurrence on Thursday, we can use the `Schedule.dayOfWeek` constructor:

{% highlight scala %}
val onThursday = Schedule.dayOfWeek(DayOfWeek.THURSDAY)
{% endhighlight %}

Similarly, if we wish to describe a recurrence at 6:00 AM and 12:00 PM, then we can use the 
`Schedule.hourOfDay` constructor:

{% highlight scala %}
val sixAndTwelve = Schedule.hourOfDay(6, 12)
{% endhighlight %}

Like all models in functional design, the schedule model has operators that let us transform and 
combine schedules, so we can solve bigger problems from solutions to much simpler subproblems.

For example, if we want a schedule that recurs on Thursdays at 6:00 AM and 12:00 PM, then we can 
use the intersection operator, which produces the intersection of two schedules:

{% highlight scala %}
val thursdaysSixAndTwelve = 
  onThursday && sixAndTwelve
{% endhighlight %}

Similarly, if we want to modify this schedule, so that it produces a constant output (say, the unit
value `()`), we can use the `map` method, which allows us to produce a new schedule with a 
transformed output:

{% highlight scala %}
val finalSchedule = thursdaysSixAndTwelve.map(_ => ())
{% endhighlight %}

Because *ZIO Schedule* is a functional domain, it has the ability to solve a huge number of 
scheduling problems using a small set of constructors and operators. Thanks to following the 
principles of functional design, schedules are fully testable, easy to understand, and easy to 
modify in the face of changing business requirements.

Now that you've seen an example of functional design up close, we're going to build our own simple
functional domain and explore different choices for encoding the model.

## Email Filtering

Let's say we're building an email web application. Users of the application need the ability to 
construct email filters, so they can trigger actions like forwarding emails, moving emails to 
certain folders, and deleting emails.

There are a huge number of possible filters that users might want to construct. Moreover, as 
developers of the application, we want the ability to support these numerous use cases in a way that 
is highly testable and has a low cost of maintenance.

To achieve these benefits, we will use functional design techniques.

### The Model

The first thing we need in this new functional domain is a *model*. This model will be an immutable
data type (like _all_ models), and will describe a particular filter that the user has constructed
and wishes to apply to incoming emails.

One of the ways we want to *execute* this model is to apply it to an email and see if the filter 
matches the email. This consideration can influence the design of our model.

In functional design, there are two ways to encode any model:

 * **Executable Encoding**. In this encoding, we express every constructor and operator for our 
 model in terms of its execution.
 * **Declarative Encoding**. In this encoding, we express every constructor and operator for our 
 model as pure data in a recursive tree structure.

The first encoding is sometimes called _final_ (because the constructors and operators are expressed
in terms of its final executed form, which is predetermined in advance), while the latter is 
sometimes called _initial_ (because given this initial form, the model can later be executed into 
any form, even those not predetermined in advance).

Let's explore both encodings of an email filter.

### Executable Encoding

Executable encodings are represented using `case class` types or open `trait` types that store
a bunch of functions, each of which executes the model in some predetermined way.

In our case, we want to execute an email filter by applying it to an email, and seeing if the filter 
matches the email. So to create an executable encoding of an `EmailFilter`, we will define a 
`case class`, and store a function that takes an email and returns a boolean value:

{% highlight scala %}
final case class EmailFilter(matches: Email => Boolean)
{% endhighlight %}

To execute this model, we need only call the `matches` function that we have stored in the model,
and provide it an email, which will then give us a boolean value that tells us whether or not the 
email filter matches the specified email.

We may now proceed to implement constructors, which build solutions to simple problems, such as 
matching emails that have some subject:

{% highlight scala %}
def subjectContains(phrase: String): EmailFilter = 
  EmailFilter(_.subject.contains(phrase))
{% endhighlight %}

In a realistic example, other constructors would be needed, which would allow filtering on bodies, 
or filtering on the recipient list, or filtering on the send date (for example).

Constructors only solve simple problems. To solve more complex problems, we need operators that 
allow transforming and composing models. 

We can add these methods directly inside the `EmailFilter` class:

{% highlight scala %}
final case class EmailFilter(matches: Email => Boolean) { self =>
  def &&(that: EmailFilter): EmailFilter = 
    EmailFilter(email => self.matches(email) && that.matches(email))

  def ||(that: EmailFilter): EmailFilter = 
    EmailFilter(email => self.matches(email) || that.matches(email))

  def unary_! : EmailFilter = 
    EmailFilter(email => !self.matches(email))
}
{% endhighlight %}

These two binary operators and one unary operator are enough to solve quite a few interesting 
problems in email filtering. For example, the following filter accepts emails whose subject contains 
the words "discount" or "clearance", and which do not contain the word "liquidation":

{% highlight scala %}
val filter = 
  (subjectContains("discount") || subjectContains("clearance")) &&
  !subjectContains("liquidation")
{% endhighlight %}

The executable encoding is nice and straightforward: it's obvious what an email filter is, even to 
someone who is not very familiar with functional design, because every email filter is expressed
directly in terms of its execution.

### Declarative Encoding

Declarative encodings are represented using `sealed trait` types or `enum` types (Scala 3), which 
have as many subtypes (or _cases_) as the model has primitive constructors and operators. The 
purpose of these subtypes is to capture the arguments of every constructor and operator, and store 
them as pure data in a recursive data type.

In our case, if we want to replicate the functionality of the previous executable encoding, then we 
need four subtypes of a `sealed trait`, one for the subject constructor, one for the unary operator,
and two more for the binary operators:

{% highlight scala %}
sealed trait EmailFilter { self =>
  def &&(that: EmailFilter): EmailFilter = And(self, that)

  def ||(that: EmailFilter): EmailFilter = Or(self, that)

  def unary_! : EmailFilter = Not(self)
}
final case class SubjectContains(phrase: String) extends EmailFilter
final case class And(left: EmailFilter, right: EmailFilter) extends EmailFilter 
final case class Or(left: EmailFilter, right: EmailFilter) extends EmailFilter 
final case class Not(value: EmailFilter) extends EmailFilter

def subjectContains(phrase: String): EmailFilter = 
  SubjectContains(phrase)
{% endhighlight %}

As with all declarative encodings, the subtypes just store the arguments to the constructors and the 
operators&mdash;they don't actually do anything except "record" the way email filters are 
constructed, transformed, and composed.

If we take the previous example `filter`, then we can see it ends up building a tree:

{% highlight scala %}
And(
  Or(SubjectContains("discount"), SubjectContains("clearance")),
  Not(SubjectContains("liquidation"))
)
{% endhighlight %}

This tree can be inspected and transformed, like any data tree.

Now because a model using the declarative encoding is pure data, there is no built-in way to execute 
the model: we have no way to test an email to see if it matches a filter. However, it's easy to 
write a standalone function that can execute the model (called an *interpreter* or *executor* of the 
model).

The following executor takes a model, and tests an email to see if it matches the filter:

{% highlight scala %}
def matches(filter: EmailFilter. email: Email): Boolean = 
  filter match {
    case And(l, r) => matches(l, email) && matches(r, email)
    case Or(l, r) => matches(l, email) || matches(r, email)
    case Not(v) => !matches(v, email)
    case SubjectContains(phrase) => email.subject.contains(phrase)
  }
{% endhighlight %}

The declarative encoding has a layer of indirection, which adds additional ceremony. However, it is 
a nice way to think about models in general, because it is obvious they describe solutions to 
problems&mdash;the execution of the model requires a separate traversal of the data structure.

## New Interpreters

In the previous examples, we executed the `EmailFilter` model by passing an email, and we got back 
a boolean value indicating whether or not the filter matched the email.

However, that's just one of many ways we might want to execute a model. For example, we might also
want to execute the model into a string, which tells us how the email filter was constructed, 
transformed, and composed (for debug purposes or rendering to users).

We can do this for both the executable and the declarative encodings.

For the executable encoding, we just add another function of type `() => String` to the 
`case class`. This second executor creates a string representation of the email filter:

{% highlight scala %}
final case class EmailFilter(matches: Email => Boolean, describe: () => String) { self =>
  def &&(that: EmailFilter): EmailFilter = 
    EmailFilter(
      email => self.matches(email) && that.matches(email),
      () => s"(${self.describe()} && ${that.describe})")

  def ||(that: EmailFilter): EmailFilter = 
    EmailFilter(
      email => self.matches(email) || that.matches(email),
      () => s"(${self.describe()} || ${that.describe})"))

  def unary_! : EmailFilter = 
    EmailFilter(email => !self.matches(email),
    () => s"!${self.describe()}"))
}
def subjectContains(phrase: String): EmailFilter = 
  EmailFilter(_.subject.contains(phrase), () => s"(subject contains ${phrase})")
{% endhighlight %}

In general, if we want `n` distinct ways to execute a model, then our executable encoding will have 
`n` functions all stored inside the `case class` (or `n` methods inside an open `trait`). 

Every time we add a new interpreter to a model expressed with the executable encoding, we have to 
update all the code that uses the constructor for the model, which could potentially be a lot of 
code. However, the maintenance burden can be minimized by creating _derived_ operators and 
constructors, which are expressed in terms of other constructors and operators (and not the primary
constructor for our `case class`).

For the declarative encoding, because the model is just data, if we want another interpreter, we can 
just add another standalone function:

{% highlight scala %}
def describe(filter: EmailFilter): String = 
  filter match {
    case And(l, r) => s"(${describe(l)} && ${describe(r)})"
    case Or(l, r) => s"(${describe(l)} || ${describe(r)})"
    case Not(v) => s"!${describe(v)}"
    case SubjectContains(phrase) => s"(subject contains ${phrase})"
  }
{% endhighlight %}

The ease with which one can add new ways to execute a model is indeed a strength of the 
declarative encoding.

### Encoding Tradeoffs

Executable and declarative encodings have *opposite* strengths and weaknesses.

In the executable encoding, new constructors and operators can be added freely, without updating
any existing code. However, new interpreters cannot be added without updating all existing 
constructors and operators.

In the declarative encoding, new interpreters can be added freely, without updating any existing
code. However, new (primitive) operators and constructors cannot be added without updating all 
existing interpreters.

In functional programming, the executable and declarative encoding are considered "duals": they are 
mirror images of each other, sitting at opposite ends of the same spectrum.

In both executable and declarative encodings, you can add new operators and constructors; and in 
both executable and declarative encodings, you can add new interpreters, which execute the model 
in some way (for example, testing to see if an email filter matches an email).

Now, as pure data, the declarative encoding is amenable to optimizations that are not possible with 
the executable encoding (at least, not without essentially reinventing data). So for cases where 
performance is critical, the declarative encoding can sometimes have an edge.

In addition, the declarative encoding is ideal when persistence of the model is required: as pure 
data, it is straightforward to define a way to save and restore a model value, using a relational 
database or other persistence scheme.

Meanwhile, the executable encoding tends to be a better fit for legacy code, because it is very 
lean and can seamlessly reuse existing (impure) interfaces and classes. For example, if we have a 
lot of code using an `InputStream`, we can write our own simple functional stream like so:

{% highlight scala %}
final case class Stream(create: () => InputStream)
{% endhighlight %}

Such a data type can have many constructors and operators that let us program in a more declarative 
fashion (without worrying about resource handling or error propagation for intermediate input 
streams), while still maintaining easy interop with existing code bases.

## Where From Here

Functional design is an immensely powerful tool for making functional programming practical. It lets
you solve real world problems in a way that makes your code easy to test, cheap to maintain, and fun
to work on.

Although this post introduces the basics, there's much more beyond what I've talked about here, 
including how to make functional domains more type safe (using generics, phantom types, and type-
level programming); how to design good constructors and operators using the principles of 
orthogonality, expressivity, and composability; common patterns of transformation and 
composition; and identifying functional domains inside business applications.

If you've enjoyed this post and would like to learn more, check out my 
[exercises on Github](https://github.com/jdegoes/functional-design), sign up for my next workshop 
at [Ziverge](https://ziverge.com), or join my Spartan tier on [Patreon](https://patreon.com/jdegoes).

Those for whom this post was an easy read are encouraged to explore 
[the expression problem](https://en.wikipedia.org/wiki/Expression_problem), 
[object algebras](https://www.cs.utexas.edu/~wcook/Drafts/2012/ecoop2012.pdf),
and [tagless-final style](http://okmij.org/ftp/tagless-final/index.html), all of which provide more 
background on the tradeoffs of different encodings.

That's all for now!