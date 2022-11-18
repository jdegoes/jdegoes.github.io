---
layout:       post
title:        "Native Config Lands in ZIO 2.0 "
description:  "With logging and metrics already baked in, configuration was inevitable"
category:     articles
tags:         [type classes, haskell, purescript, scala, zio, config, configuration, cloud-native, functional programming, fp, effects, monads, io]
---

In a first for functional effect systems, [ZIO 2.0](https://degoes.net/articles/zio-2.0) brought integrated metrics and logging front-ends directly into the core, which enables application developers to easily track and log what's going on inside their applications for enhanced observability.

Now, some months downstream of this seminal release, we've witnessed rapid adoption of these features by ZIO ecosystem projects, as well as by end-users.

Two important factors are driving this adoption:

1. **Simple, Value-Added Front-ends**. Integrated logging and metrics have very simple, lightweight interfaces that impose zero barrier to adoption, and they bring value-added features powered directly by ZIO's runtime system. For example, logging is enriched by the tracking of full failures (including concurrent failures and finalizer failures), as well as fiber identity and fork location&mdash;rich data that is freely accessible to the ZIO runtime system, and useful for monitoring operations and diagnosing production issues.
2. **Highly Customizable Back-ends**. With the help of separate ecosystem projects [ZIO Logging](http://github.com/zio/zio-logging) and [ZIO Metrics Connectors](http://github.com/zio/zio-metrics-connectors), developers can easily and rapidly wire-up ZIO's logging and metrics front-ends to major backends, such as Logback for logging, or Prometheus for metrics monitoring. This rich integration lets you adopt ZIO's front-end without needing to change any of the services you already use.

With integrated logging and metrics, as well as overhauled dependency injection with automatic wire-up and simplified layers, ZIO 2.0 took a major leap from being a _Better Future_ library to being a _powerful framework for modern app development_, powered by Scala's unique fusion of object-oriented and statically-typed functional programming.

Frameworks like [Spring](https://spring.io), however, have long gone beyond just _logging_, _metrics_, and _dependency injection_: they also provide support for _configuration management_, which has increasingly become a pain point in the ZIO ecosystem.

## The Pain of Config

In the ZIO 2.0 service pattern, we architect our application in _layers_, each composed of _services_. Services are defined with _traits_, and are implemented with _classes_.

A service implementation expresses its dependency on other services via its constructor, which is lifted into a _layer_, which is capable of describing initialization, finalization, creation of state, and so forth, and which benefits from ZIO's compile-time auto-wiring of the application dependency graph.

In ZIO 2.0, all layers are defined along the following lines:

{% highlight scala %}
val myLayer: ZLayer[PaymentRepo, Nothing, MyService] = 
  ZLayer.scoped {
    for {
      repo <- ZIO.service[PaymentRepo]
      ref  <- Ref.make(MyState.Initial)
      impl <- ZIO.succeed(new MyServiceImpl(ref, repo))
      _    <- impl.initialize
      _    <- ZIO.addFinalizer(impl.destroy)
    } yield impl
  }
{% endhighlight %}

This example layer allocates resources, and registers a finalizer to dispose of them properly when the layer is being shutdown at the end of its life. For the simpler case when a service implementation does not use any resources, the `ZLayer.apply` constructor can be used.

It turns out that many, _many_ service implementations require configuration; a requirement that I will hereafter refer to as _the configuration problem_.

For example, a web service implementation may need a token to call an API; a server may need a port number and certificate details; a user repo may need database and table names; and so forth.

This is so pervasive a pattern, that the ZIO ecosystem has developed _two_ separate ways of handling it.

### Solution #1 

In this solution, we solve the configuration problem by turning our layer from an ordinary value into a function, which accepts configuration, and returns the layer (a _layer factory_ of sorts):

{% highlight scala %}
def makeMyLayer(config: MyServiceImpl.Config) = 
  ZLayer.scoped {
    for {
      repo <- ZIO.service[PaymentRepo]
      ref  <- Ref.make(MyState.Initial)
      impl <- ZIO.succeed(new MyServiceImpl(config, ref, repo))
      _    <- impl.initialize
      _    <- ZIO.addFinalizer(impl.destroy)
    } yield impl
  }
{% endhighlight %}

This solution is used probbaly 70% of the time (in my estimation), because it's very straightforward.

Unfortunately, it leads to some amount of awkwardness when wiring up application dependencies, because configuration needs to be loaded _before_ constructing layers:

{% highlight scala %}
def myApp = ...

def loadConfigs: Task[(MyConfig, MyOtherConfig)] = 
  for {
    myConfig      <- loadMyConfig(args)
    myOtherConfig <- loadOtherConfig(args)
  } yield (myConfig, myOtherConfig)

def wireUp = 
  for {
    configs <- loadConfigs
    myLayer <- makeMyLayer(configs._1)
    ...
  } yield myApp.provide(myLayer, ...)
{% endhighlight %}

This two-stage application wireup process has a highly irregular structure that is a pain to teach, and a pain to maintain, and raises the question: why is this so difficult?

### Solution #2

In this solution, we solve the configuration problem by treating the configuration data itself as _another_ service: a dependency of our service implementation.

Our layer definition becomes something like the following:

{% highlight scala %}
val myLayer: ZLayer[PaymentRepo with MyServiceImpl.Config, Nothing, MyService] = 
  ZLayer.scoped {
    for {
      repo   <- ZIO.service[PaymentRepo]
      config <- ZIO.service[MyServiceImpl.Config]
      ref    <- Ref.make(MyState.Initial)
      impl   <- ZIO.succeed(new MyServiceImpl(config, ref, repo))
      _      <- impl.initialize
      _      <- ZIO.addFinalizer(impl.destroy)
    } yield impl
  }
{% endhighlight %}

This approach is more uniform (with a one-stage wire-up), but has a few drawbacks, ranging from being harder to use for simpler cases (like for tests, where we don't need to load configuration data at all), to demanding that we have only one configuration value for each type of configuration.

The latter point is more subtle. Let's say we have 10 services that each require `HostAndPort` as inputs: then when we wire up our application, they will all get the same `HostAndPort` value, rather than each getting separate `HostAndPort` configurations. 

All these problems have workarounds, but it has felt for a long while that we can do better.

In ZIO 2.0.4, we are finally introducing a native configuration front-end!

## Native Config in ZIO 2.0

Just like integrated logging and metrics, ZIO 2.0 is introducing only a configuration _front-end_. This means that for most applications, you will need to choose a configuration _backend_, which will load configuration data and perform validation.

A common choice in the Scala ecosystem has been Typesafe Config, with its support for HOCON, but there are many other choices, including TOML, Yaml, JSON, XML, and of course configuration servers.

ZIO Config has support for some of these, but has not yet been updated to support the new configuration front-end available in ZIO 2.0.4.

The configuration front-end is designed to be lightweight and user-friendly, perfectly integrated with other ZIO features, and extensible via pluggable backends coming soon to _ZIO Config_.

## First Steps

To begin using ZIO 2.0 configuration, you define a `Config[A]` for some data type `A`, which holds your configuration data. A value of type `Config[A]` represents a "recipe" for decoding configuration data into a value of type `A`.

For example, let's say we have the following data type:

{% highlight scala %}
final case class HostPort(host: String, port: Int)
{% endhighlight %}

In order to use this data type for configuration with ZIO 2.0, we need to create a `Config[HostPort]`. We will recommend that users store such recipes _in the companion object of the data type_, because it's an obvious place to look for such data.

To create configs, we can use the constructors in the companion object of `Config`, together with the operators in the `Config` data type itself.

Here's one way to create a `Config[HostPort]`:

{% highlight scala %}
final case class HostPort(host: String, port: Int)
object HostPort {
  val config: Config[HostPort] = 
    (Config.string("host") ++ Config.int("port")).map { 
      case (host, port) => HostPort(host, port)
    }
}
{% endhighlight %}

Once we have defined a `Config[A]` for some data type `A`, we can load this configuration using `ZIO.config(...)`, which will either successfully load the data from the configuration backend, or it will produce a detailed error about what is missing or malformed.

Now layer construction will end up looking something like the following:

{% highlight scala %}
val myLayer: ZLayer[PaymentRepo, Nothing, MyService] = 
  ZLayer.scoped {
    for {
      repo   <- ZIO.service[PaymentRepo]
      config <- ZIO.config(MyServiceImpl.config)
      ref    <- Ref.make(MyState.Initial)
      impl   <- ZIO.succeed(new MyServiceImpl(config, ref, repo))
      _      <- impl.initialize
      _      <- ZIO.addFinalizer(impl.destroy)
    } yield impl
  }
{% endhighlight %}

Notice how this layer is just an _ordinary value_, not a method, and that configuration does not appear _anywhere_ inside the dependencies of the service (the input to the layer).

How is this configuration data loaded?

Through a new service in ZIO 2.0 called `ConfigProvider`. A `ConfigProvider` is capable of loading configuration data from any `Config`. Configuration providers can be composed using fallback semantics, as well as transformed in various ways.

By default, ZIO 2.0 ships with a `ConfigProvider` that looks _first_ at environment variables, and _second_ at system properties. This `ConfigProvider` can be used to bootstrap other, more sophisticated configuration providers, which may need to read files mounted on networked drives or inside an image, or possibly connect to configuration servers.

`ConfigProvider` is stored in a fiber ref, so it can be updated locally in parts of your application, and that's exactly how an upcoming version of _ZIO Config_ will tie different backends into the front-end.

## Disambiguation

In order to disambiguate different instances of the same configuration data, `Config` provides a method called `Config#nested`, which allows you to provide a namespace for configuration data. 

In the preceding example, we might actually load the configuration using `ZIO.config(HostPort.config.nested("MyService"))`. This would allow us to configure `MyService` separately from other parts of the application that also require `HostPort`:

<pre>
[MyService]
host = localhost
port = 8080
</pre>

The `nested` method is currently the recommended way of achieving disambiguation, although future versions of ZIO may be able to take advantage of the application dependency graph.

## Summary

As you can see, the new configuration front-end in ZIO 2.0.4 is simple and cleanly integrated into ZIO 2.0, with powerful flexibility that enables us to reuse our existing application configuration.

Over the course of the next few months, core ZIO libraries like ZIO HTTP, ZIO JDBC, ZIO Redis, and others, will adopt ZIO 2 configuration so that you have a standardized, uniform way of configuring different components across the entire ZIO ecosystem. Meanwhile, _ZIO Config_ will be updated to provide seamless support for both traditional configuration backends (files and the like), as well as more modern configuration backends, such as Docker, Kubernetes, and cloud configuration services.

With this step of integrated configuration, ZIO 2 continues down the path of solving all the common painpoints of cloud application development, with the principled, type-safe, and user-friendly approach that ZIO has pioneered in the space of object-oriented, pure functional programming.

Please give the new feature a whirl, and let us know what you think!