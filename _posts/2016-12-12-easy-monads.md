---
layout:       post
title:        "A Beginner-Friendly Tour through Functional Programming in Scala"
description:  "An educational experiment that strips away jargon and develops some functional machinery from first-principles"
category:     articles
tags:         [fp, functional programming, free, mtl, scala]
---
The essential core of functional programming is quite simple: build programs
by composing functions.

In this context, "function" does not refer to a "computer science" function,
that is, a hunk of machine code, but rather, a *mathematical function*:

 1. **Totality**. A function must yield a value for every possible input.
 2. **Determinism**. A function must yield the *same* value for the *same* input.
 3. **Purity**. A function's only effect must be the computation of its return value.

Together, these properties give you an unprecedented ability to reason about
your code: call a function with any input, and you'll always get a valid value,
it will be the same value every time you call the function with that input, and
the function won't try to do anything else funky like launch a nuclear missile.

Such a tiny idea has a profoundly simplifying effect on large-scale
software engineering, because it means there's less your brain has to keep track
of in order to understand the behavior of your program. In fact, you can
understand the behavior of the program by understanding the behavior of its
individual parts — without having to keep it all in your head at one time!

Now, function programming does not always have a reputation for *being* simple,
but I think that's because of several factors:

 1. **Unfamiliarity**. Functional programming is quite different than the type of
 programming that most professionals are used to. Because it's unfamiliar, it
 can seem difficult. It's best to compare functional programming with the
 experience of learning to program *for the first time* (rather than the
 experience of, say, learning a new programming language that's modestly
 different than one you already know).
 2. **Jargon**. There's a lot of jargon in functional programming, such as
 "immutability", "recursion", "induction", "hylomorphism", "transducer",
 "functor", and the list goes on. These concepts are not necessarily hard, but
 they have scary-sounding names and having lots of experience programming the
 imperative way doesn't help you understand any of the new terminology.
 3. **Motivation**. From my experience, people learn things more easily when
 they can clearly see *how* they can be useful to accomplishing the goals these
 people have. Yet, although the situation is improving as more and more people
 learn functional programming (and share their knowledge with others), there's
 not much motivation of the core concepts in functional programming.

Some might add that "functional programming" is *sometimes* associated with
*advanced statically-typed functional programming*. In these cases, people
learning "functional programming" are really learning two things at once:
functional programming, and an advanced type system (most imperative programming
languages have relatively simple type systems).

However, one can program functionally with untyped programming languages, or
program imperatively with more advanced type systems. So the two are not
necessarily linked.

Many of us functional programmers would like to show people why we're so excited
about functional programming. Yet, most of the content that I write is not
geared at the functionally-curious, or even beginner functional programmers.
Rather, I write about what interests me, which tends to be more intermediate to
advanced functional programming, intended to be useful to people who have been
programming functionally for years.

So I'm trying an experiment: I'm going to demonstrate a few ideas in functional
programming that help us build real-world programs. But I'm going to do it in a
way that is hopefully somewhat aware of the above factors.

It's a functional programming blog post for people who aren't functional
programmers. Or maybe for people who know a *little* but wish they understood *more*.

Hopefully you'll find it useful &mdash; but more than useful, I hope you find it
*inspiring*. Inspiring enough to invest the effort necessary to push forward
your knowledge of functional programming, *despite* of how difficult it might
seem.

Or who knows, maybe after this post, it'll all seem easy!

## Impractical FP

A canonical example of the power of functional programming is the following
generic sort function:

{% highlight scala %}
def sort[A: Ordering](as: List[A]): List[A] = as match {
  case Nil => Nil     
  case a :: as =>        
    val (before, after) = as partition (_ < a)

    sort(before) ++ (a :: sort(after))
  }
}
{% endhighlight %}

It's a beautiful example because it demonstrates how functional programming can
have a simplifying effect on how we reason about software.

After looking at the code, you can probably convince yourself of the following:

1. A sorted empty list is an empty list.
2. A sorted non-empty list with a first element `a` and subsequent elements `as`
is the concatenation of the sort of all those elements less than `a`, followed
by the concatenation of `a`, followed by the concatenation of the sort of all
those elements *not* less than `a`.

Every part of this function can be reasoned about independently. If we believe
the pieces are correct, we can trust the correctness of the whole.

Moreover, this `sort` function, because it's a function *in the mathematical
sense*, is easy to test and reuse. We can send any list to the function and
expect it will return the sort of the list. Further, we know that the function
will always return the same output for the same input.

So our tests can be stated quite simply:

{% highlight scala %}
assert(sort[Int](Nil) == Nil)
assert(sort[Int](3 :: 2 :: 1 :: Nil) == 1 :: 2 :: 3 :: Nil)
{% endhighlight %}

(In fact, they can be stated much more powerfully in functional programming, but
that's a topic for another post.)

While explaining the benefits of functional programming in this one case seem
simple enough, it's a big stretch to imagine if or how these benefits might
scale as our example moves beyond a toy example.

In fact, the number one objection to functional programming is that it only
works for toy examples, but fails utterly in "real-world" programming.

Let's take the simplest real-world problem I can think of: a function that
doesn't return a value because it doesn't succeed.

## Totality

I've defined a function as *total* and *deterministic*. What happens if we drop
the totality requirement?

Well, the function need not return anything. Practically speaking, this means
the function either runs forever, or "escapes" the need to return a value by
using some other mechanism supported by the host language &mdash; typically
by "throwing" an exception.

Functions that never return because they run forever are clearly broken (runaway
loop due to an edge condition, or similar), but what about exceptions?

In the days before exceptions, programmers just used return values to indicate
the success or failure of a function. In C code, for example, we'd write long
sequences of code that looked like this:

{% highlight c %}
int parse_config(config *cfg) {
  FileHandle handle;
  char *bytes;

  handle = new_handle();

  handle = file_open("config.cfg");

  if (handle == NULL) return -1;

  char *bytes = file_read(handle);

  if (bytes == NULL) return -2;

  ...

  return 0;
}
{% endhighlight %}

In this world, the introduction of exception handling seemed almost magical.
Programmers could avoid tangling error-handling concerns with application logic,
and benefit from the short-circuiting behavior of exceptions.

The main problem with exceptions is that, in a language that supports them, you
have no guarantee where they will occur. Which means they can occur *everywhere*.
Which means that, over a sufficiently long period of time, they *do* occur
virtually everywhere (even in Java, unchecked exceptions can be thrown anywhere
and often contain "checked" exceptions inside them!).

As with nulls, a proliferation of exceptions leads to defensive and aggressive
exception handling, even in cases where it doesn't make sense, which leads to
more bugs and bizarre edge cases.

Fortunately, we can easily achieve both totality *and* clean code, but it
requires more machinery than older programming languages supported.

To develop this idea, let's take a function which returns the first element of a
list:

{% highlight scala %}
def head[A](as: List[A]): A = as.head
{% endhighlight %}

This function is not total. Depending on which list you pass the function, it
may or may not return the first element of the list. If you pass it an empty
list, the function will never return, but instead throw an exception.

To make this function a total function, we need only introduce a data structure
that can model the concept of *optionality* &mdash; something that *might* or
*might not* be there.

Let's call this `Maybe`:

{% highlight scala %}
sealed trait Maybe[A]
final case class There[A](value: A) extends Maybe[A]
final case class NotThere[A]() extends Maybe[A]
{% endhighlight %}

With this data structure, we can turn the `head` pseudo-function into a real
function:

{% highlight scala %}
def head[A](as: List[A]): Maybe[A] = as match {
  case Nil => NotThere[A]()
  case a :: _ => There[A](a)
}
{% endhighlight %}

Now when we think about code that uses the function, we don't have to consider
the possibility the function won't return. Because the function will *always*
return. Since we don't have to consider this possibility, our reasoning about
code that uses the `head` function is simpler, containing fewer cases to analyze.

The `Maybe` data structure doesn't provide the same power as exceptions,
however. For one, it doesn't have any information on the error, which is perhaps
OK in the case of the `head` function (because we know what the error is), but
isn't going to work well for most other functions.

To solve this problem, we can introduce a new data structure, called `Result`,
which models the concept of *exceptionality*:

{% highlight scala %}
sealed trait Result[E, A]
final case class Error[E, A](error: E) extends Result[E, A]
final case class Success[E, A](value: A) extends Result[E, A]
{% endhighlight %}

This type lets us make functions like `file_open` total functions:

{% highlight scala %}
def file_open(path: String): Result[FileError, FileHandle] = ???
{% endhighlight %}

Now we have all the same information that exceptions provide us with. However,
if we have lots of different operations to perform, and each of them returns a
`Result`, then it looks like we would need a lot of boilerplate, reminiscent of
the days before exceptions:

{% highlight scala %}
def parse_config: Result[FileError, Config] = {
  file_open("config.cfg") match {
    case Error(e) => Error(e)
    case Success(handle) =>
      file_read(handle) match {
        case Error(e) => Error(e)
        case Success(bytes) =>
          ???
      }
  }
}
{% endhighlight %}

We've made the functions `file_open`, `file_read`, and so on *total*, which
simplifies our reasoning about them, but at the same time, we have introduced
additional boilerplate, which makes the code harder to read.

To recapture the detangling and short-circuiting benefits of exceptions, we
need to recognize the pattern in the above code. If you study the code for a
few minutes, you should be able to see the following pattern:

{% highlight scala %}
doX match {
  case Error(e) => Error(e)
  case Value(x) => doY(x) match {
    case Error(e) => Error(e)
    case Success(y) => doZ(y) match {
      case Error(e) => Error(e)
      case Success(z) => doW(w) match {
        ...
      }
    }
  }
}
{% endhighlight %}

If you examine what the types of `doY`, `doZ`, and `doW` must be, you discover
they accept the `A` of the `Result[E, A]` of the previous operation, and using
this `A`, they produce a new `Result[E, B]`.

This suggests we can factor out the duplication by introducing a `chain` method:

{% highlight scala %}
def chain[E, A, B](result: Result[E, A])(f: A => Result[E, B]): Result[E, B] =
  result match {
    case Error(e) => Error(e)
    case Success(a) => f(a)
  }
{% endhighlight %}

Indeed, we can use such a `chain` method in our original `parse_config` method:

{% highlight scala %}
def parse_config: Result[FileError, Config] = {
  chain(file_open("config.cfg")) { handle =>
    chain(file_read(handle)) { bytes =>
      ???
    }
  }
}
{% endhighlight %}

This reduces the boilerplate to calling `chain` on every `Result[E, A]`. In
doing this, we achieve the short-circuiting benefits of exceptions, and the
error-handling logic is factored out into the `chain` method, so the application
logic doesn't need to concern itself with it.

If you use a structure like `Result` to model exceptional cases, eventually
you'll find yourself in need of a utility function that looks like this:

{% highlight scala %}
// Change the `A` in a `Result[E, A]` into a `B` by using the provided
// function `f`:
def change[E, A, B](result: Result[E, A])(f: A => B): Result[E, B] = result match {
  case Error(e) => Error(e)
  case Success(a) => Success(f(a))
}
{% endhighlight %}

This is a map-like function (similar to mapping the elements of an array into a
different type). If you like programming in an object-oriented style, you can make the functions `chain` and `change` methods on the `Result` class:

{% highlight scala %}
sealed trait Result[E, A] {
  def change[B](f: A => B): Result[E, B] = this match {
    case Error(e) => Error(e)
    case Success(a) => Success(f(a))
  }

  def chain[B](f: A => Result[E, B]): Result[E, B] = this match {
    case Error(e) => Error(e)
    case Success(a) => f(a)
  }
}
final case class Error[E, A](error: E) extends Result[E, A]
final case class Success[E, A](value: A) extends Result[E, A]
{% endhighlight %}

This makes the code read a little better:

{% highlight scala %}
def parse_config: Result[FileError, Config] = {
  file_open("config.cfg").chain { handle =>
    file_read(handle).chain { bytes =>
      ???
    }
  }
}
{% endhighlight %}

Further, if you call the `change` and `chain` methods `map` and `flatMap`,
respectively, then Scala provides some handy syntax that can further simplify
the way the code looks:

{% highlight scala %}
def parse_config: Result[FileError, Config] =
  for {
    handle <- file_open("config.cfg")
    bytes  <- file_read(handle)
  } yield ???
{% endhighlight %}

At this point, we have regained totality of our functions, but with the full
benefits of exceptions (short-circuiting and untangled concerns).

All we needed was the `chain` function (AKA `flatMap`) and the `Result` data
structure. The rest falls into place naturally.

Note that the `chain` function accepts a function as a parameter. This parameter
is supplied as an anonymous function (which captures arbitrary references in its
context), which should help demonstrate why this technique never really emerged
in C code. If C had first-class functions and garbage collection or
reference counting, it's quite probable this pattern would have emerged on its
own, without any input from the functional programming community.

The remaining requirements for functions are determinism and purity. Let's take
a look at these requirements in the next section.

## Determinism & Purity

Functions that aren't deterministic or which do more than just compute a return
value can be notoriously difficult to reason about.

For example, I have used libraries (and probably created some!), which perform
IO inside a constructor:

{% highlight java %}
class Logging {
  private OutputStream ostream;

  public Logging(File file) {
    ostream = new FileOutputStream(file);
  }
}
{% endhighlight %}

This constructor may throw exceptions, which is very unexpected!

As another example, the `URL` class in Java may perform a network connection if
you place an instance of this class into a data structure (whoa!). This happens
because the equals & hash code methods may trigger resolution of the address.

In addition to the unexpected nature of impure functions (who knows what they
will do and when they will do it!), non-determinism makes testing and reasoning
about the functions particularly difficult. You may be forced to mock code or
reason about the external state that can affect the behavior of such a function.

Pure, deterministic functions don't do anything dirty behind the scenes, and
you can count on them to always return the same output for the same input, which
means the code is easier to test and easier to understand.

But if you try to make all your functions total, deterministic, and pure, you'll
quickly hit into a wall when you try to perform any input/output or other effects
&mdash;a wall that has convinced too many that functional programming is
impractical for real-world software.

In fact, the wall has long since been shattered, but rather than go down those
paths, let's look at a simple example of IO and see if we can bootstrap a solution.

### Console IO

Let's say we are developing a console program, which needs to read input from
the console, and write output to the console. Many useful problems can be
solved with console programs, and if we can figure out how to make a console
program deterministic and pure, we can generalize to other types of programs.

If you think about the problem for a while, you may come up with the following
idea: instead of calling functions that aren't deterministic or pure, we can
build up a data structure that describes the console effects that comprise
our program.

Suppose that `ConsoleIO` is a description of our program. To start with, we need
a way to describe the effect of writing output to the console.

One approach might look like this:

{% highlight scala %}
sealed trait ConsoleIO
final case class WriteLine(line: String) extends ConsoleIO
{% endhighlight %}

This approach is a bit too simple, because we can only build a program that
writes a single line of text to the console:

{% highlight scala %}
def helloWorld: ConsoleIO = WriteLine("Hello World!")
{% endhighlight %}

This is a total, deterministic, and pure function&mdash;however, it's not that
useful. At most it can describe a program that writes a single line of text to
the console. The text can change, of course, even be fed in as a parameter to
the function, but ultimately, the program will have just a single effect on the
console.

Fortunately, it's pretty easy to extend the structure to support *multiple*
effects in sequence. Here's one way:

{% highlight scala %}
sealed trait ConsoleIO
final case class WriteLine(line: String, then: ConsoleIO) extends ConsoleIO
{% endhighlight %}

Using this approach, we can construct more complex descriptions of effectful
programs, such as this example:

{% highlight scala %}
def infiniteHelloWorld: ConsoleIO = WriteLine("Hello World", infiniteHelloWorld)
{% endhighlight %}

If we introduced a "thunk" to avoid blowing the stack during construction, this
structure would describe a program that writes "Hello World" an infinite number
of times to the console. Infinite programs like this aren't that useful, but
we can fix this problem by adding another term to `ConsoleIO`, one which allows
program termination:

{% highlight scala %}
sealed trait ConsoleIO
final case class WriteLine(line: String, then: ConsoleIO) extends ConsoleIO
final case object End extends ConsoleIO
{% endhighlight %}

Now we can construct, for example, a description of a program that repeats
the printing of a certain line of text a certain number of times:

{% highlight scala %}
def printTextNTimes(text: String, n: Int): ConsoleIO =
  if (n <= 0) End
  else WriteLine(text, printTextNTimes(text, n - 1))
{% endhighlight %}

This function is total, pure, and deterministic. Of course, it doesn't actually
print anything out to the console, but we'll get to that soon enough.

Currently, we can only describe programs that write text to the console. It's
easy enough to extend this approach so we can read text from the console:

{% highlight scala %}
sealed trait ConsoleIO
final case class WriteLine(line: String, then: ConsoleIO) extends ConsoleIO
final case class ReadLine(process: String => ConsoleIO) extends ConsoleIO
final case object End extends ConsoleIO
{% endhighlight %}

Notice how constructing a `ReadLine` value requires we supply a function that,
given the line of input read from the console, returns another `ConsoleIO`,
representing the rest of the effectful program.

We pass our function to `ReadLine`, and it's a promise that at some point in the
future, someone will give us a `String` (the line of input from the console)
that we can use to return the "rest" of the program.

This structure is sufficiently powerful for us to describe interactive programs.
For example, the following program asks you for your name, then prints out "Hello"
followed by your name:

{% highlight scala %}
def socialProgram: ConsoleIO = WriteLine(
  "Hello, what is your name?",
  ReadLine(name =>
    WriteLine("Hello, " + name + "!", End)
  )
)
{% endhighlight %}

Remember this function is total, deterministic, and pure. One can imagine
building very complex descriptions of effectul programs using this structure.
In fact, *any program* that requires just console IO effects could be described
with this structure!

Notice that any program described by `ConsoleIO` simply terminates at some point.
These programs cannot return a value.

If we want "composable programs" that is, programs which produce things that
other programs can depend on, then we'll need to generalize the `End` term to
accept some value of type `A`, which will necessitate we add a new type
parameter `A` to `ConsoleIO` and thread it through the other terms.

The end result looks only slightly more complex:

{% highlight scala %}
sealed trait ConsoleIO[A]
final case class WriteLine[A](line: String, then: ConsoleIO[A]) extends ConsoleIO[A]
final case class ReadLine[A](process: String => ConsoleIO[A]) extends ConsoleIO[A]
final case class EndWith[A](value: A) extends ConsoleIO[A]
{% endhighlight %}

Now, `ConsoleIO[A]` is a description of a program that can read and write from
the console, and terminate with a value of type `A`. This lets us build programs
that produce values, which can be consumed by other programs.

Using this structure, we can build the "Hello, <name>!" program of before, but
this time, we can return the name of the user from the program:

{% highlight scala %}
val userNameProgram: ConsoleIO[String] = WriteLine(
  "Hello, what is your name?",
  ReadLine(name =>
    WriteLine("Hello, " + name + "!", EndWith(name))
  )
)
{% endhighlight %}

The only capability we are missing from `ConsoleIO` is the ability to change
the return value into some other type. For example, what if we want to build
another console program which does not return the name of the user, but returns
how  many characters are in the name, by reusing `userNameProgram`?

Currently, this is not possible. We need something more powerful to capture
this notion. If we had a list, we would be able to "map" over its elements to
change the result type. This is precisely the capability we need for `ConsoleIO`.

We can add such a map function directly to `ConsoleIO`:

{% highlight scala %}
sealed trait ConsoleIO[A] {
  def map[B](f: A => B): ConsoleIO[B] = Map(this, f)
}
final case class WriteLine[A](line: String, then: ConsoleIO[A]) extends ConsoleIO[A]
final case class ReadLine[A](process: String => ConsoleIO[A]) extends ConsoleIO[A]
final case class EndWith[A](value: A) extends ConsoleIO[A]
final case class Map[A0, A](v: ConsoleIO[A0], f: A0 => A) extends ConsoleIO[A]
{% endhighlight %}

Now given any `ConsoleIO[A]`, we can map it into a `ConsoleIO[B]`, given a function
`A => B`. So we can write a new program, `userNameLenProgram`, which computes the
number of characters in the user's name, by writing the following snippet:

{% highlight scala %}
def userNameLenProgram: ConsoleIO[Int] = userNameProgram.map(_.length)
{% endhighlight %}

With the addition of `map`, the utility of `EndWith` has changed: we no longer
need it to return a value from the program, because we can map whatever value
we have to the return value. For example, if we have `ConsoleIO[String]`, we can
map that to `ConsoleIO[Int]` by specifying a function `String => Int`.

However, `EndWith` could still be used to construct a "pure" program that performs
no effects (which can be composed with other programs). So it's still useful,
although not for its original purpose. Therefore, we can rename it to `Pure` as
follows:

{% highlight scala %}
sealed trait ConsoleIO[A] {
  def map[B](f: A => B): ConsoleIO[B] = Map(this, f)
}
final case class WriteLine[A](line: String, then: ConsoleIO[A]) extends ConsoleIO[A]
final case class ReadLine[A](process: String => ConsoleIO[A]) extends ConsoleIO[A]
final case class Pure[A](value: A) extends ConsoleIO[A]
final case class Map[A0, A](v: ConsoleIO[A0], f: A0 => A) extends ConsoleIO[A]
{% endhighlight %}

With these capabilities, we can build and reuse descriptions of console IO
programs without any limitation or restrictions. All such descriptions are
total, deterministic, and pure, and therefore benefit from the enhanced
abilities of functional code: easier to understand, easier to test, easier to
change safely, and easier to compose.

Ultimately, however, we have to *convert* a description of a program into
an actual program&mdash;something that can actually perform effects (which means indeterminism and impurity). Typically, this process is called *interpretation*.

One can write a straightforward, if indeterministic and impure, interpreter for
`ConsoleIO[A]` using code similar to the following:

{% highlight scala %}
def interpret[A](program: ConsoleIO[A]): A = program match {
  case WriteLine(line, next) => println(line); interpret(next)
  case ReadLine(process) => interpret(process(readLine()))
  case Pure(value) => value
  case Map(v, f) => f(interpret(v))
}
{% endhighlight %}

There are better ways, but we shall not cover them there. For now, I think it's
interesting enough that we have both been able to describe effectful programs,
which satisfy the requirements of totality, determinism, and purity, and which
can be easily converted into real-world programs that perform effects.

Note that the conversion could (and should!) happen at the end of the program.
In Haskell, it happens outside the main function, in Haskell's runtime. However,
for other languages that don't have the same level of FP machinery baked in,
you can always effectfully interpret your programs at the entry point of your
program.

What this does is ensure you have maximum totality, determinism, and pureness
throughout the vast majority of your program. At the edge, you have a nasty
little layer that may be hard to reason about, but which faithfully translates
the description of your program into executable effects on the substrate.

### Extensibility

This approach works well in a world where the set of effects is fixed. In our
cases so far, it has been console IO — reading and writing lines to and from a
console. However, in real world programs, the set of effects under consideration
is far more complex and we benefit from the ability to compose programs with
different effects into a union of all their effects.

The first step is to recognize additional structure. In essence, we can break down
our description of a console program into parts that produce values (`WriteLine`)
or expect values (`ReadLine`). The rest of the program consists of pure
boilerplate: either changing one program into another by mapping over the value
it returns (`map`), or by chaining two programs together in such a fashion that
the second depends on the value output by the first (`chain` / `flatMap`).

If this doesn't make any sense, study the following types for a while:

{% highlight scala %}
sealed trait ConsoleIO[A] {
  def map[B](f: A => B): ConsoleIO[B] = Map(this, f)
  def flatMap[B](f: A => ConsoleIO[B]): ConsoleIO[B] = Chain(this, f)
}
final case class WriteLine(line: String) extends ConsoleIO[Unit]
final case class ReadLine() extends ConsoleIO[String]
final case class Pure[A](value: A) extends ConsoleIO[A]
final case class Chain[A0, A](v: ConsoleIO[A0], f: A0 => ConsoleIO[A]) extends ConsoleIO[A]
final case class Map[A0, A](v: ConsoleIO[A0], f: A0 => A) extends ConsoleIO[A]
{% endhighlight %}

This is just a different, admittedly more confusing way to represent the same
thing. Using this new structure, our interactive program becomes a little more
complex to express:

{% highlight scala %}
def userNameProgram: ConsoleIO[String] =
  Chain[Unit, String](
    WriteLine("What is your name?"),
    _ => Chain[String, String](
      ReadLine(),
      name => Chain[Unit, String](
        WriteLine("Hello, " + name + "!"),
        _ => Pure(name))))
{% endhighlight %}

In this model, a `ConsoleIO` has the sort of generic-looking `Chain`, `Map`, and
`Pure` terms, which don't really talk about the effects of our console program,
and then it has two additional terms, which describe those effects: the `ReadLine`
and `WriteLine` effect, which have been simplified under this model.

An interpreter for this model is only a bit more involved:

{% highlight scala %}
def interpret[A](program: ConsoleIO[A]): A = program match {
  case WriteLine(line) => println(line); ();
  case ReadLine() => readLine()
  case Pure(value) => value
  case Map(v, f) => f(interpret(v))
  case Chain(v, f) => interpret(f(interpret(v)))
}
{% endhighlight %}

This formulation may not seem all that different, but the key insight is that
there are only 2 terms that describe effects. The rest is pure machinery. This
machinery can be abstracted into *another* class and reused for *all possible
effect types*. For example:

{% highlight scala %}
sealed trait Sequential[F[_], A] {
  def map[B](f: A => B): Sequential[F, B] = Map[F, A, B](this, f)

  def flatMap[B](f: A => Sequential[F, B]): Sequential[F, B] = Chain[F, A, B](this, f)
}
final case class Effect[F[_], A](fa: F[A]) extends Sequential[F, A]
final case class Pure[F[_], A](value: A) extends Sequential[F, A]
final case class Chain[F[_], A0, A](v: Sequential[F, A0], f: A0 => Sequential[F, A]) extends Sequential[F, A]
final case class Map[F[_], A0, A](v: Sequential[F, A0], f: A0 => A) extends Sequential[F, A]

sealed trait ConsoleF[A]
final case class WriteLine(line: String) extends ConsoleF[Unit]
final case class ReadLine() extends ConsoleF[String]

type ConsoleIO[A] = Sequential[ConsoleF, A]
{% endhighlight %}

The `Sequential` class doesn't directly reference `ConsoleIO`, so it can be
reused for may different effect types.

This allows us to cleanly separate effects from computational machinery. The new
version of the hello world program is below:

{% highlight scala %}
def userNameProgram: ConsoleIO[String] =
  Chain[ConsoleF, Unit, String](
    Effect[ConsoleF, Unit](WriteLine("What is your name?")),
    _ => Chain[ConsoleF, String, String](
      Effect[ConsoleF, String](ReadLine()),
      name => Chain[ConsoleF, Unit, String](
        Effect[ConsoleF, Unit](WriteLine("Hello, " + name + "!")),
        _ => Pure(name))))
{% endhighlight %}

If we take advantage of Scala's `for` notation, and add a single implicit class
to make it easier to wrap the effects inside the `Effect` constructor, we end
up with the following simplification:

{% highlight scala %}
def userNameProgram: ConsoleIO[String] =
    for {
      _    <- WriteLine("What is your name?").effect
      name <- ReadLine().effect
      _    <- WriteLine("Hello, " + name + "!").effect
    } yield name
{% endhighlight %}

This approach can be simplified even further — for example, adding helper
functions for all the terms (such as `writeLine`, `readLine`, and so forth).
Such helpers make the program even simpler:

{% highlight scala %}
def userNameProgram: ConsoleIO[String] =
  for {
    _    <- writeLine("What is your name?")
    name <- readLine()
    _    <- writeLine("Hello, " + name + "!")
  } yield name
{% endhighlight %}

There's no major difference between this and an imperative program. The
structure is roughly the same, and the syntax differs only a little. In our
case, however, we've built a *description* of a program, which is total,
deterministic, and pure, but which itself has no effects on the outside world
(aside from utilizing memory and CPU to compute the structure).

Moreover, with this clean separation, the actual set of effects can be extended,
too, which means we can take two programs with different effects, and combine
them together into a program which has the effects of both.

Do to this, all you need is some sort of "EitherOr" term that can say it's one
effect (console IO) or another (file IO):

{% highlight scala %}
sealed trait EitherOr[F[_], G[_], A]
final case class IsLeft[F[_], G[_], A](left: F[A]) extends EitherOr[F, G, A]
final case class IsRight[F[_], G[_], A](right: G[A]) extends EitherOr[F, G, A]

type CompositeProgram[A] = Sequential[EitherOr[ConsoleIO, FileIO, ?], A]
{% endhighlight %}

Interpreters can be written on this structure quite simply, and can reuse other
interpreters for the individual effect types (console versus file).

While we've developed it from first principles and without any jargon, this
abstraction that you see is actually the legendary "Free monad", which strikes
terror into unsuspecting Scala developers everywhere!

That wasn't so bad, was it?

Let's take a look at a few perks of this abstraction before we wrap up the tour.

## Perks of Pure

A common reaction to this sort of encoding of effects is, "What's the point?"

There are a few really practical benefits that come out of this style of describing
effectful programs:

1. The types of your program describe exactly what it's capable of doing.
Reasoning about the code becomes tremendously simpler because there are no
"hunks of machine code" embedded everywhere that can literally do anything.
Instead, you precisely describe exactly what effects your program requires in a
declarative fashion (and in more advanced versions, you describe how effects at
one layer of abstraction translate to effects at a lower level of abstraction).
2. You can separate the description of a program from what it actually does. For
example, a web program can build HTML, but that description can be interpreted to
HTML on a DOM node, HTML on a Canvas node, or PDF on the server.
3. You can mock out dependencies. If your external dependencies (such as file
systems, network systems, APIs, etc.) are all described by data structures, then
you can easily traverse such data structures to make sure that when you feed them
the right values, they return the right responses. Unlike mocking libraries, this
approach is totally type-safe and doesn't require any runtime support.
4. You can inspect the program as it's being run. For example, you can add logging
lines to print out what each instruction is about to do, and what it succeeded in
doing. These log lines are quite detailed and can replace the need for manual logging.
Moreover, since they can be "weaved in" during the act of composing interpreters,
logging can truly have no overhead when it is disabled. No boilerplate and no
overhead — sounds good to me!
5. Beyond just logging, you can add whole aspects to your program at runtime.
For example, adding authentication, authorization, and auditing into a program
by composing together interpreters (this is how my company produces a secure
version of the [Quasar Analytics](https://github.com/quasar-analytics/quasar))
project without tampering with the code base. It's like aspect-oriented
programming, but type safe and much more flexible!

In addition to all these benefits, you get the very tangible benefits of being
able to reason about total, pure, deterministic code, even in the presence of
effects (at least, for the majority of your code). The benefits of that cascade
to new development, maintaining existing code, adding tests, fixing bugs, adding
team members, and so much more.

# Summary

I hope at least some of what we've covered in this post made sense. More than
that, I hope you have a glimpse for how we can write full programs using total,
deterministic, pure functions — and how that such an approach leads to other
benefits, some of which we are just starting to discover.

If nothing else, this is a call to learn more functional programming! To persevere
in spite of the strange names, incomplete documentation, and confusing types.

There is power in functional programming. *Tremendous* power. In my experience,
those who take the time to learn functional programming end up becoming very
passionate about it and never return to the old ways. Functional programming
gives developers superpowers&mdash;the ability to write software in simpler ways
that result in cleaner code, lower cost of maintenance, lower technical debt, easier
composition, easier reasoning, and much more.

If you're intrigued, I urge you to stick with it, and if you get stuck, please
know that myself and numerous other members of the functional programming
community have your back. We'll help you get to the next level, value by value,
type by type, lambda by lambda.

See you on the other side, partner!
