---
layout:       post
title:        "Testing Incrementally with ZIO Environment"
description:  "Environmental effects let you easily, quickly, and incrementally build tesable services."
category:     articles
tags:         [fp, functional programming, type classes, scala, monads, lenses, effects, reactive, scalaz, cats, mtl, monad transformers, zio, reader, environmental effects]
---

In my [last post](/articles/zio-environment), I introduced _ZIO Environment_, which is a new feature in ZIO that bakes in a high-performance, type-safe, and fully-inferred reader effect into the ZIO data type.

This capability leads to a way of describing and testing effects that I call _environmental effects_. Unlike tagless-final, which is difficult to teach, difficult to abstract over, and does not infer, environmental effects are simple, abstract well, and infer completely.

Moreover, while proponents of tagless-final argue that tagless-final _parametrically constrains_ effects, my [last post](/articles/zio-environment) demonstrated this is not quite correct: not only can you embed raw effects _anywhere_ in Scala, but even without leaving _Scalazzi_ (the purely functional subset of Scala), you can lift arbitrary effects into any `Applicative` functor.

The inability of tagless-final to constrain effects is more than just theoretical:

 - New Scala functional programmers use effect type classes like `Sync` everywhere (which are themselves lawless and serve only to embed effects), and they embed effects using lazy methods, like `defer` or `point`.
 - Even some experienced Scala functional programmers embed effects in pure methods (for example, exceptions in the functions they pass to `map`, `flatMap`), and some effect types encourage this behavior.

At the end of the day, and after professionally and pedagogically wrestling with these issues for several years now, I've come to the conclusion there are only two highly compelling reasons to use tagless-final:

1. Avoiding commitment to a specific effect type, which can be useful to library authors, but which is less useful for application developers, and often detrimental to them;
2. Writing testable functional code, which is fairly straightforward with tagless-final because you can just create test instances for different effect type classes.

While testability is a compelling reason to use tagless-final, it's not necessarily a compelling reason to choose tagless-final over another approach.

In this post, I'm going to show you how to use environmental effects to achieve testability. I hope to demonstrate that environmental effects provide easier, more incremental, and more modular testability&mdash;all without sacrificing teachability, abstraction, or type inference.

## A Web App

Let's say we are building a web application with ZIO. Suppose the application was originally written with `Future` or perhaps some version of `IO` or `Task`.

Later, the application was ported to ZIO's `Task[A]`, which is a type alias for `ZIO[Any, Throwable, A]`&mdash;indicating an effect that requires no specific environment and which may fail with a `Throwable`.

Now a typical function in our application might look something like this one:

{% highlight scala %}
def inviteFriends(userID: UserID): Task[List[Send]] =
  for {
    user    <- DB.lookupUser(userID)
    friends <- Social.getFriends(user.facebookID)
    resp    <- ZIO.foreach(friends) { friend =>
                 Email.invite(user, friend.email)
               }
  } yield resp
{% endhighlight %}

where portions of the `DB` and `Social` objects are shown below:

{% highlight scala %}
object Social {
  ...
  def getFriends(fid: FacebookID): Task[List[FacebookProfile]] = ???
  ...
}
object DB {
  ...
  def lookupUser(uid: UserID): Task[UserProfile] = ???
  ...
}
object Email {
  ...
  def invite(user: UserProfile, friend: FacebookProfile): Task[Send] = ???
  ...
}
{% endhighlight %}

As currently written, our web application is not very testable. The function `inviteFriends` makes direct calls to database functions, Facebook API functions, and email service functions.

While we may have automated tests for our web service, because our application interacts directly with the real world, the tests are actually _system_ tests, _not_ unit tests. Such tests are very difficult to write, run slowly, prone to random failures, and they test much more than our application logic.

We do not have time to rewrite our application, and we cannot make it testable all at once. Instead, let's try to remove dependency on the database for the `inviteFriends` function.

If we succeed in doing this, we will make our test code a _little better_, and after we ship the new code, we can incrementally use the same technique to make the function _fully_ testable&mdash;fast, deterministic, and without external dependencies.

### Steps Toward Testability

To incrementally refactor `inviteFriends` to be more testable, we're going to perform the following series of refactorings:

1. Introduce a type alias.
2. Introduce a module for the database.
3. Implement a production database module.
4. Integrate the production module.
5. Implement a test database module.
6. Test the `inviteFriends` function.

Each of these steps will be covered in the sections that follow.

### Introduce A Type Alias

Mainly to simplify the process of refactoring our application, especially if we aren't using an IDE, we're going to first introduce a simple type alias that we can use in the definition of `inviteFriends` and the functions that call it:

{% highlight scala %}
type Webapp[A] = Task[A]
{% endhighlight %}

Now with a straightforward change, we will update the `lookupUser` function and any functions that call it to use the type alias:

{% highlight scala %}
def inviteFriends(userID: UserID): Webapp[List[Send]] =
  for {
    user    <- DB.lookupUser(userID)
    friends <- Social.getFriends(user.facebookID)
    resp    <- ZIO.foreach(friends) { friend =>
                 Email.invite(user, friend.email)
               }
  } yield resp
{% endhighlight %}

As an alternative to this technique, we could simply delete the return types entirely, but it's considered a good practice to place return types on top-level function signatures, so developers without IDEs can easily determine the return type of functions.

After this step, we are ready to introduce a service for the database.

### Introduce a Database Module

The database module will provide access to a database service.

As discussed in my post on [ZIO Environment](/articles/zio-environment), the database module is an ordinary interface with a single field, which contains the database service; and the database service is just an ordinary interface.

We can define both the module and the service very simply:

{% highlight scala %}
// The database module
trait Database {
  val database: Database.Service
}
object Database {
  // The database service
  trait Service {
    def lookupUser(uid: UserID): Task[UserProfile]
  }
}
{% endhighlight %}

Notice how we have decided to place only a single method inside the database service: the `lookupUser` method. Although there may be many database methods, we don't have time to make all of them testable, so we will focus on the one required by the `inviteFriends` method.

We are now ready to implement a production version of the service.

### Implement Production Module

We will call the production database module `DatabaseLive`. To implement the module, we need only copy and paste the implementation of `Database.lookupUser` into our implementation of the service interface:

{% highlight scala %}
trait DatabaseLive {
  val database: Database.Service = 
    new Database.Service {
      // Implementation copy/pasted from
      // DB.lookupUser:
      def lookupUser(userID: UserID) = 
        ...
    }
}
object DatabaseLive extends DatabaseLive
{% endhighlight %}

For maximum flexibility and convenience, we have both a _trait_ that implements the database module, which can be mixed into other traits, and we have an _object_ that extends the trait, which can be used standalone.

### Integrate Production Module

We now have all the pieces we need to replace the original `DB.lookupUser` method, whose actual implementation now resides inside our `DatabaseLive` module.

The new version of the `DB` object looks like this:

{% highlight scala %}
object DB {
  ...
  def lookupUser(uid: UserID): ZIO[Database, Throwable, UserProfile] = 
    ZIO.accessM(_.database lookupUser uid)
  ...
}
{% endhighlight %}

The `lookupUser` method merely delegates to the database module, by accessing the model through ZIO Environment (`ZIO.accessM`).

Here we don't use the `Webapp` type alias, because the functions in `DB` will not necessarily have the same dependencies as our web application.

However, after enough refactoring, we might introduce a new type alias in the `DB` object: `type DB[A] = ZIO[Database, Throwable, A]`. Eventually, all methods in `DB` would return effects of this type.

At this point, our refactoring is nearly complete. But we have to take care of one last detail: we have to provide our database module to our production application.

There are two main ways to provide the database module to our application. If it is inconvenient to propagate the `Webapp` type signature to the top of our application, we can always supply the production module somewhere inside our application.

In the worst case, if we are pressed for time and need to ship the code today, maybe we choose to provide the production database wherever we call `inviteFriends`.

{% highlight scala %}
inviteFriends(userId).provide(DatabaseLive)
{% endhighlight %}

If we have a bit more time, we can push the `Webapp` type synonym to the entry point of our purely functional application, which might be the main function, or it might be where our web framework calls into our code.

In this case, instead of using the `DefaultRuntime` that ships with ZIO, we can define our own `Runtime`, which provides the production database module):

{% highlight scala %}
val myRuntime = 
  Runtime(DatabaseLive, PlatformLive)
{% endhighlight %}

The custom runtime can be used to run many different effects that all require the same environment, so we don't have to call `provide` on all of them before we run them.

Once we have this custom runtime, we can run our top-level effect, which will supply its required environment:

{% highlight scala %}
myRuntime.unsafeRun(effect)
{% endhighlight %}

At this point, we have not changed the behavior of our application at all&mdash;it will work exactly as it did before. We've just moved the code around a bit, so we can access a tiny effect through ZIO environment.

Now it's time to build a database module specifically for testing.

### Implement Test Module

We could implement the test database module using a mocking framework. However, to avoid all magic and use of reflection, in this post, we will build one from scratch.

For maximum flexibility, our test database module will track all calls to `lookupUser`, and supply responses using a `Map`, which can be dynamically changed by the test suite.

To support this stateful behavior, we will need a `Ref`, which is a concurrent-safe ZIO data structure that models mutable references. We will also need a simple (immutable) data structure to hold the state of the test database module.

We define the following test data structure, which is capable of tracking a list of `UserID` values, and holding data that maps from `UserID` to `UserProfile`.

{% highlight scala %}
final case class TestDatabaseState(
  lookups : List[UserID],
  data    : Map[UserID, UserProfile]
) {
  def addLookup(uid: UserID): TestDatabaseState = copy(lookups = uid :: lookups)
}
{% endhighlight %}

Now we can define the service of our test database module. The service will require a `Ref[TestDatabaseState]`, so it can not only use test data, but update the test state:

{% highlight scala %}
final case class TestDatabaseService(ref: Ref[TestDatabaseState]) extends Database.Service {
  def lookupUser(uid: UserID): Task[UserProfile] = 
    for {
      _       <- ref.update(_.addLookup uid)
      data    <- ref.get.map(_.data)
      profile <- Task.fromEither(data.get(uid)
                   .fold(Left(new DBErr))(
                     Right(_)))
    } yield profile
}
{% endhighlight %}

Notice how the `lookupUser` function stores the `UserID` of every call in the `lookups` field of the `TestDatabaseState`. In addition, the function retrieves test responses from the map. If there is no response in the map, the function fails, presumably in the same way the production database would fail.

The test service must be placed in a module. In general, we should wait to create the module until the test suite, because then we will know the full set of dependencies for each test.

However, at this stage, the database service is the only dependency in our application, so we can make a helper function to create the test database module:

{% highlight scala %}
object TestDatabase {
  def apply(ref: Ref[TestDatabaseState]): Database = 
    new Database {
      val database: Database.Service = 
        new TestDatabaseService(ref)
    }
}
{% endhighlight %}

We now have all the pieces necessary to write a test of the `inviteFriends` function, which will use our test database module.

### Write the Test

To more easily test the `lookupFriends` function, we will define a helper function. Given test data and input to the function, the helper will return the final test state and the output of the `lookupFriends` function:

{% highlight scala %}
def runLookupFriends(data: Map[UserID, UserProfile], uid: UserID): Task[(TestDatabaseState, List[Send])] = 
  for {
    ref   <- Ref.make(TestDatabaseState(Nil, data))
    resp  <- lookupFriends(uid)
               .provide(TestDatabase(ref))
    state <- ref.get 
  } yield (state, resp)
{% endhighlight %}

The helper function creates a `Ref` with the initial test data, uses the `Ref` to create the `TestDatabase` module, and then supplies the database module to the effect returned by `lookupFriends`.

With this helper function, writing a test becomes quite simple:

{% highlight scala %}
class TestSuite extends DefaultRuntime {
  def testLookupFriends = {
    val (state, resp) = 
      unsafeRun {
        runLookupFriends(
          Map(TestUserID -> TestUserProfile),
          TestUserID
        )
      }
    
    (state.lookups must_=== List(TestUserID)) and
      (resp must_=== TestResponse)
}
{% endhighlight %}

This test for `inviteFriends` is not perfect. It still interacts with a real Facebook API and a real email service. But compared to whatever tests already exist, at least this test does not interact with a real database.

Moreover, we were able to make this change in a _minimally disruptive_ manner.

### A Glimpse Beyond

After a little more refactoring, of course, we would succeed in making `inviteFriends` fully testable. Even after the full refactoring, the code for `lookupFriends` would not change.

Instead, our type alias for `Webapp` would expand to include new environmental effects:

{% highlight scala %}
type WebappFX = 
  Database with Social with Email

type Webapp[A] = ZIO[WebappFX, Throwable, A]
{% endhighlight %}

Now all the methods in the `DB`, `Social`, and `Email` objects would simply delegate to their respective modules using `ZIO.accessM`.

Running our application would now look a little different:

{% highlight scala %}
val myRuntime = 
  Runtime(
    new  DatabaseLive 
    with SocialLive 
    with EmailLive, PlatformLive)
...
myRuntime.unsafeRun(effect)
{% endhighlight %}

Finally, testing `lookupFriends` would be entirely fast, deterministic, and type-safe, without any dependencies on external systems, or use of any reflection.

## Summary

Environmental effects make it easy to test purely functional applications&mdash;significantly easier and with less ceremony than tagless-final, with full type inference, and without false promises.

Morever, with environmental effects, we can make even just a _single function_ testable, by pushing that function into an environmental effect. This requires just a few small changes that can be done incrementally without major disruption to the application.

This ability lets us make incremental progress towards better application architecture. We don't have to solve all the problems in our code base at once. We can focus on making our application a little better each day.

While this post focused on ZIO Environment, if you're using `Future`, Monix, or Cats IO, you can still use this approach with a `ReaderT` monad transformer. With a monad transformer, you will lose type inference and some performance, but you will gain the other benefits in a more familiar package.

If you'd like to give ZIO Environment a try, hop over to the [Github project page](https://github.com/scalaz/scalaz-zio), and be sure to stop by the [Gitter channel](https://gitter.im/scalaz/scalaz-zio) and say hello.

In future posts, I will cover how to provide partial dependencies, how to model services that require other services (the _graph problem_), how to hide implementation details, and how this approach differs from the classic cake pattern. Stay tuned for more!