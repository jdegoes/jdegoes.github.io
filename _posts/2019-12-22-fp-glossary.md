---
layout:       post
title:        "A Glossary of Functional Programming"
description:  "Functional programming has a bit of jargon, but that doesn't have to stop you from understanding core concepts"
category:     articles
tags:         [functional programming, fp, scala, haskell, purescript]
---

I've taught functional programming for years now, each time experimenting with different ways of teaching core concepts. Over time, I've collected and converged on simple (but reasonably precise) pedagogical definitions for a range of functional concepts.

In this post, I'll share those definitions with you, in my first ever, *Glossary of Functional Programming*. Enjoy!

## Glossary

**Abstraction**. An _abstraction_ is a precise description of the ways in which different data types share common structure. Abstraction allows different data types to participate in _generic programming_. In functional programming, abstractions are defined by _algebraic structure_, and are usually encoded in a programming language using _type classes_. Common examples of abstractions include monoids, functors, and monads.

**Ad Hoc Polymorphism**. _Ad hoc polymorphism_ is any language feature that allows overloading functions and operators (providing multiple implementations for the same symbol) in such a fashion that the compiler or runtime can automatically select which implementation to call based on the types involved in the function application. Examples of ad hoc polymorphism include method overloading and _type classes_.

**Algebra**. An _algebra_ is a set of _elements_ together with a set of _operations_ on the elements. The operations of an algebra are defined by _algebraic laws_, which give the operations meaning by relating them to each other. For example, addition on integers forms a very simple algebra, where the elements are integers, and the sole operation is addition, which satisfies assocative and commutative laws.

**Algebraic Data Type (ADT)**. An _algebraic data type_ is any data type composed from _product types_ (records) and _sum types_ (enumerations). In functional programming, algebraic data types are used for _data modeling_.

**Algebraic Law**. An _algebraic law_ is a _universally quantified_ statement that asserts a relationship between the _operations_ of an _algebra_. For example, an algebraic law for addition could assert that `(a + b) + c` is equal to `a + (b + c)` (_associativity_). In mainstream programming languages, algebraic laws are usually tested using _property-based testing_.

**Algebraic Structure**. _Algebraic structure_ is any _structure_ that is defined using an _algebra_.

**Applicative**. An `Applicative` Functor is an `Apply` Functor that allows taking a value and converting it into a _functional effect_ that succeeds with the specified value. Under the view of functors as DSLs, `Applicative` adds a "pure return" statement to the DSL.

**Apply**. An `Apply` Functor is a Functor that allows zipping two _functional effects_ together, to get a tuple of their successes. Under the view of functors as DSLs, `Apply` adds a way to combine two programs together into one program, and preserve both successes.

**Associativity**. An _associative_ operator is a binary operator that, when applied to three values, always results in the same value, regardless of the order in which pairwise values are combined. For example, addition on numbers is associative, because for all numbers `a`, `b`, and `c`, `(a + b) + c` is equal to `a + (b + c)`.

**Category Theory**. _Category theory_ is a branch of mathematics that defines mathematical structure using directed labeled graphs (_categories_). An alternative to algebraically-defined structure, category theory is a powerful and precise pattern language for mathematics, computer science, and functional programming. Knowledge of category theory is helpful but not necessary for mastery of applied functional programming.

**Commutativity**. A _commutative_ operator is a binary operator that produces the same result no matter the order of the operands. For example, addition is commutative because `a + b` is equal to `b + a`, for all numbers `a` and `b`.

**Composable**. An operator is _composable_ if it can be applied to its own return value. For example, the addition operator is composable with respect to integers, because it returns another integer, which can then be added to other integers. Similarly, the boolean negation operator is composable, because it is possible to negate a boolean value that has itself been negated. Composability allows a small number of operators (which can be reasoned about simply) to have immense expressive power.

**Contravariant Functor**. A `Contravariant` Functor is a type class that allows transforming the input type to some _functional effect_ using a function. Contravariant functors are often formed by polymorphic types that accept input (such as functions, or stages in a pipeline). 

**Data Modeling**. _Data modeling_ is the act of constructing a data model of domain objects (such as people, products, schedules, etc.). A goal of data modeling in statically-typed functional programming is to construct a data model so precise, it is impossible to construct bad data, a process sometimes called, _making illegal states unrepresentable_.

**Declarative Programming**. _Declarative programming_ is a style of programming in which solutions are constructed by specifying goals, rather than specifying the sequential steps necessary to achieve these goals (_what_ instead of _how_). Declarative programming is the opposite of imperative programming. Some languages, such as SQL and Datalog, are inherently declarative; but declarative programming can be practiced in any programming language, typically by defining declarative DSLs atop imperative implementations.

**Dependency Injection**. _Dependency injection_ refers to a framework, library, language feature, or architectural pattern that facilitates threading dependencies throughout an application, and wiring up those dependencies for different scenarios. Dependency injection is one of the primary architectural needs of large-scale, complex applications.

**Dependent Typing**. _Dependent typing_ is a method of defining types that permits types to depend on values. For example, in dependently typed programming languages, one can define the type of vectors whose length is equal to some value. Examples of dependently typed programming languages include Idris, Coq, and Agda. Dependently typed programming languages allow proving more properties about programs at compile-time, albeit at increased development cost.

**Deterministic**. A _deterministic_ procedure returns the same output for the same input.

**Domain-Specific Language (DSL)**. A _domain-specific language_ (DSL) is a "mini-language" that is designed to solve a certain subproblem in an application. In functional programming, DSLs are generally _embedded_, which means users of these DSL use data types and operators, defined in the host language, to construct programs in the DSL.

**Effect Rotation**. _Effect rotation_ refers to the construction of highly polymorphic data types (typically _indexed monads_) that simultaneously support multiple _functional effects_. Each effect supported by the data type is represented with a different type parameter, and each effect may be introduced or eliminated using certain operations. Effect rotation provides much of the benefit of monad transformers, but with substantially improved performance, and in some languages like Scala, significantly better type inference.

**Existential Quantification**. _Existential quantification_ asserts that a statement holds for some choice of a given variable. For example, the value `list: List[_]` is existentially quantified over the `List` type parameter, asserting that there exists some (unknown) type that describes the type of elements in the list, without specifying what this type is.

**Existential Type**. An _existential type_ is a specific, definite type, which is unknown to code interacting with the type. In programming languages, existential types are used to "forget" information that need not be carried around in type parameters. In Scala, abstract type members are always existential types.

**Expression**. An _expression_ is a value constructed from the application of functions or operators to other values. The parameters to the functions or operators are called the _terms_ of the expression. For example, `a + b` is an expression, which computes the result of adding the term `a` to the term `b`.

**F-Algebra**. An F-algebra is a generalization of algebraic structure that comes from category theory, which makes it possible to represent algebraic laws without universal quantification, using only morphisms. In functional programming, F-algebras appear more directly as interpreters for both tagless-final and free monadic programs (*natural transformations*), and as algebras and coalgebras in recursion schemes.

**First-Class**. _First-class_ constructs are those which can be created, manipulated, and abstracted over using the most powerful machinery a language exposes for these tasks. For most programming language, _first-class_ means the construct is a _value_. First-class constructs give programmers much more power than second-class constructs. Some constructs which are not first-class can nonetheless be _refied_ to obtain some of the benefits of being first-class.

**Flow analysis**. Type-directed _flow analysis_ is a process whereby the flow of information in the declaration of a data type or a function is analyzed using type information alone. Performing flow analysis can yield useful information about the implementation of polymorphic declarations. For example, for a function `def foo[A](a: A): A` (`foo :: a -> a`), one can tell using flow analysis that the output of the function must have come from its single input parameter, because there is no other way for the function to return a value of the polymorphic type.

**Free Structure**. A _free structure_ is a data structure that _reifies_ operations of some algebra into a tree-like data structure. Later, the data structure can be traversed, and the operations applied to concrete values&mdash;a process called _interpretation_ of the free structure. An example free structure is `List` (`[]`), which is a free monoid for any type `A`; that is, for any type `A`, `List[A]` (`[a]`) provides both a neutral value (the empty list) and an append operation (list concatenation). Another example is `Free`, which provides a free monad for any type constructor `F[_]` (`f : * -> *`).

**Function**. A mathematical _function_ `f : A => B` (`f :: a -> b`) associates every value in a set `A` (called the _domain_) with a single value in a set `B` (called the _codomain_). For a given function, this mapping is static (it does not change). In programming languages, the domains and codomains of value-level functions are types, and all such functions are _total_ (they map every input to some output), _deterministic_ (they map the same input to the same output), and _pure_ (they only map inputs to outputs).

**Functional Effect**. A _functional effect_ is an immutable data type that describes (or _models_) the computation of one or more values, where the computation may require an additional feature like optionality, logging, access to context (like configuration), errors, state, or input/output. Using effect-specific operations, functional effects can be transformed and composed to model solutions to complex problems out of solutions to smaller problems. The ways of constructing a functional effect, together with the operations on effects of that type, form a DSL, whose capabilities are dictated by the constructors and operations. Frequently, functional effects are _run_ or _interpreted_ into plain values or into other functional effects. Common functional effects include `Option` (`Maybe`), which models missing values; `Either`, which models failure; and `ZIO` (`IO`), which models side-effects, including asynchronous side-effects.

**Functional Programming**. _Functional programming_ is a style of programming in which solutions are constructed by defining and applying (mathematical) _functions_. Many programs are not "purely" functional, but contain parts that are written in a functional style, as well as parts that are written in a procedural style. Functional programming can be practiced in statically-typed as well as untyped programming languages.

**Functional-Imperative**. _Functional-imperative_ is a hybrid style of programming that uses higher-order functions to embed an imperative model of computation inside functional code. Functional-imperative style relies on _monads_ or an equally powerful abstraction to represent the stateful sequentiality of imperative computations.

**Functor**. A `Functor` is a type class that allows _mapping_ the success value of some _functional effect_ using a function. Every functor can be regarded as a type of DSL, where the terms in the functor sum type correspond to different instructions in the DSL. Under this view of functors as DSLs, `Functor` provides a way to change the success value of a DSL "program" into some other success value, by applying a specified function.

**Generalized Algebraic Data Type (GADTs)**. _Generalized algebraic data types_ are polymorphic ADTs whose component types may introduce existential types or impose type equality constraints on specific type parameters. GADTs enable easier modeling and use of type-safe DSLs.

**Generic Programming**. _Generic programming_ is a style of programming that uses _polymorphism_ to "forget" irrelevant details about data types, which allows code to be used across many different data types that share common structure. Generic programming can be thought of as defining a "template" that is then specialized to concrete types.

**Global Reasoning**. _Global reasoning_ is a property of some code wherein the correctness of the code cannot be inferred without considering prior application state or all possible inputs.

**Higher-Kinded Types**. _Higher-kinded types_, more precisely called _higher-kinded generics_, is a language feature in which type parameters may have a kind higher than `*`. The two most mainstream languages with higher-kinded types are _Scala_ and _Haskell_.

**Higher-Order**. _Higher-order_ refers to higher-order functions, which are functions that accept functions as input, or return functions as output. For example, `map` on a list is a higher-order function, because it accepts a function as one of its parameters; similarly, the `Monad` type class is a higher-order type class, because the kind of its type parameter is `(* => *) => *`, which is a higher-order type-level function.

**Homomorphism**. _Homomorphisms_ are a special type of function `f : A => B` that preserve a given _algebraic structure_. For example, the absolute value function (which returns the absolute value of some number) is a semigroup homomorphism, because it preserves additive and multiplicative semigroups.

**Imperative Programming**. _Imperative programming_ is a style of programming in which solutions are constructed from step-by-step instructions, where later instructions can depend on the result of previous instructions. In procedural programming, imperative programming is typically done with side-effecting statements, whereas in functional programming, imperative programming is typically done with _monads_.

**Indexed Monad**. An _indexed monad_ is a generalization of `Monad` for highly polymorphic data types having multiple type parameters, which form ordinary `Monad` instances for any possible choice of the extra type parameters. Indexed monads allow increased type precision, and allow operations that cannot be expressed with non-indexed monads. Examples of indexed monads included indexed state and `RIO` (in Haskell), and `ZIO` (in Scala).

**Indirection**. _Indirection_ is any programming language feature (interfaces, type classes, records of functions, module definitions) that allows multiple implementations of some required functionality to be defined and provided to code that requires these capabilities. Indirection facilitates testability, code reuse, and modularity, both in functional programming, and in other paradigms.

**Isomorphism**. An _isomorphism_ is an equivalence between two data types that have the same information. An isomorphism is defined by two functions: one to translate from the first data type to the second; and one to translate from the second data type to the first. These functions must satisfy "roundtrip" laws in order for them to form an isomorphism, which demonstrate the two representations contain the same information. In functional programming, many things are considered equal _up to isomorphism_. For example, `(A, Unit)` and `A` are _equal up to isomorphism_, meaning these two different types have the same information.

**Kind**. A _kind_ is a description of the structure of a type or type constructor. Simple types like `Boolean` have kind `*`, which can be regarded as the set of all types. Type constructors like `List` (`[]`) and `Option` (`Maybe`) have kind `* => *`, which indicates they are type-level functions that, if given a type, can return a type. For example, if you give the `List` type constructor the type `Boolean`, then you get back the type representing lists of boolean values.

**Lambda**. A _lambda_ is an anonymous function, which is a function value that has no name. Because lambdas are values, lambdas may be passed to functions, and returned from functions. All languages that support functional programming provide a way to encode lambdas.

**Lambda Calculus**. The _lambda calculus_ is a way of expressing arbitrary computation using only _lambdas_. The lambda calculus is an alternative to Turing machines and other calculi of computation, and gives meaning to the term _functional programming_. There are many different formulations of the lambda calculus, not just one.

**Lenses**. _Lenses_ are _optics_ that allow getting and setting terms in _product types_. For example, a lens could allow getting and setting the _name_ field inside objects of type `Person`. In the functional context, _setting_ a term in a product means creating a new modified product, given the new value for the term, and the old product value.

**Liskov Substitution Principle**. The _Liskov substitution principle_ says that if a function accepts an input of type `A`, then it should be valid to pass the function any subtype of `A`, without altering the correctness of the function. Similarly, for a function returning an output of type `B`, it should be valid for the function to return a subtype of `B`. The Liskov substitution principle enables principled reasoning about subtyping.

**Local Reasoning**. _Local reasoning_ is a property of some code wherein the correctness of the code can be inferred locally under specified assumptions, without considering prior application state or all possible inputs.

**Map**. To _map_ over a polymorphic data type is to change one type parameter into another type, by specifying a way to transform values of that type. For example, a list of integers can be mapped into a list of strings, by specifying a way to transform an integer into a string. To be well-defined, and to constitute a valid `Functor`, mapping over any polymorphic data type with an identity function must yield something that is equal to the original value. 

**Minimal**. A set of operators is _minimal_ if there exist no smaller set of orthogonal operators that has the same expressive power.

**Modular**. A factoring of a solution to a problem is _modular_ when it has been divided into independent concerns called _modules_, such that each module knows no more than necessary about other modules. Modularity helps tame the complexity of large-scale software development.

**Monad**. A `Monad` is an `Applicative` Functor that allows combining a first _functional effect_, together with a function that takes the success value of the first effect and returns a second effect (_continuation_). Monads represent the essence of imperative programming: do a first thing, and then do this second thing, which depends on the value computed by the first thing. Under the view of functors as DSLs, `Monad` adds sequential "statements", where subsequent statements depend on previous ones.

**Monad Transformer**. A _monad transformer_ is a data type that adds one functional effect atop any other functional effect. For example, the `OptionT` (`MaybeT`) monad transformer adds the effect of optionality to any other functional effect. A monad transformer, when "stacked" atop a monad, itself forms a monad, which allows building up large stacks of monad transformers, each layer adding some desired functional effect. Monad transformers have significant performance overhead in languages like Scala. An alternative to monad transformers is _effect rotation_.

**Monoid**. A monoid is a semigroup equipped with a _neutral element_ (often called _zero_), such that appending the neutral element to any other element (on either side) returns that same element unchanged. 

**Monomorphic**. _Monomorphic_ is the opposite of _polymorphic_, and refers to a function that takes no type parameters (instead, all of its inputs are concrete types), or a data type that takes no type parameters (rather, it defined entirely with concrete types).

**Natural Transformation**. A _natural transformation_ is any polymorphic function between _Functors_.

**Nominal Typing**. _Nominal typing_ is a method of defining types solely based on their names. Two nominal types are equal if and only if they have the same fully-qualified name. Nominal typing stands in contrast to _structural typing_, in which two types that have different names but the same structure are regarded as the same.

**Onion Architecture**. _Onion architecture_ is a method of architecting programs that involves gradually translating business logic into lower-layer levels, until at the edges of the application, logic is translated into system calls. Applications written in the onion architecture bear similarities with multi-staged compilers, where every layer of the onion contains DSLs that are "compiled" into other DSLs.

**Optics**. _Optics_ are _values_ that allow getting and setting smaller parts of a larger data structure, in a purely functional way. In the functional context, _setting_ means to produce a new copy from an old copy, with some modification applied. Optics allow zooming into small parts of very large structures, and making pinpoint modifications, without having to deal with all the boilerplate involved in deconstructing and reconstructing every level of the data structure.

**Orthogonal**. Roughly speaking, a set of operators is _orthogonal_ when no operator performs the function of any other operator. This is the functional programming analogue of _Single Responsibility Principle_ (SRP). More precisely, a set of operators `S` is _orthogonal_ if there exists no other factoring of the operators `S1` such that any operator in `S` can be expressed as a composition of two or more operators in `S1`.

**Parametric Polymorphism**. Sometimes called _generics_, _parametric polymorphism_ is a feature of some programming languages that allows _universally quantifying_ a function or a data type over one or more type parameters. Such _polymorphic functions_ and _polymorphic data types_ are said to be _parameterized_ by their type parameters. Parametric polymorphism allows the creation of generic code, which works across many different data types, and generic data types, like collections. Parametrically polymorphic code must behave uniformly across all choice of type parameters, which allows a powerful way of reasoning about such code called _parametric reasoning_.

**Parametric Reasoning**. _Parametric reasoning_ is a type of reasoning that enables one to state facts about an implementation of a polymorphic function or polymorphic data type based only on type signatures. Parametric reasoning is typically accomplished using _flow analysis_ on polymorphic declarations.

**Partial Function**. A _partial function_ is a function that is only defined for a subset of its domain. Partial functions can be modeled as total functions by expanding the codomain to include at least one new value, which indicates the function does not handle some input.

**Polymorphism**. _Polymorphism_ is a feature of programming languages that allows a variable or function to take on many different forms. Common types of polymorphism include _parametric polymorphism_, _subtype polymorphism_, and _ad hoc polymorphism_.

**Prisms**. _Prisms_ are _optics_ that allow getting and setting terms in _sum types_. For example, a prism could allow getting and setting the `Left` term inside objects of type `Either[A, B]` (`Either a b`). In the functional context, _setting_ a term in a sum means creating a new sum, given the new value for the term.

**Procedures**. In programming languages, _procedures_ are named entities that contain ordered sets of instructions for a machine to perform. Usually called _functions_, _procedures_, or _subroutines_ in common parlance, procedures are not necessarily _functions_ in the mathematical sense of the word, although all mathematical functions on values constitute valid procedures.

**Procedural Programming**. _Procedural programming_ is a style of programming in which solutions are constructed from the construction and invocation of procedures. All procedural programming is _imperative_ in nature, although not all imperative programming is _procedural_.

**Product Type**. A _product type_ is the composition of `n` types, referred to as the _terms_ of the product, together into a new type, such that a value of the product type contains a value from each of its terms. Structs, classes, records, and tuples are all examples of product types. A table is also an example of a product type, because each row in a table contains a value from each of its columns.

**Profunctor**. A _profunctor_ is type class defined over type constructors of two type parameters that describes polymorphic data types that form a `Functor` in one type parameter, and a `Contravariant` Functor in the other type parameter. Functions are examples of profunctors, which can be mapped on the output channel, and contramapped on the input channel. Profunctors can be thought of as a generalization of functions, which accept input and produce output, but most profunctors are not functions.

**Projection**. A _projection_ is a "lossy" mapping from one set to another set, such that if applied again to the output of itself, returns an equivalent result. An example is pulling the _age_ field out of a _Person_, in which case repeated application produces the same _age_ field. All extractions of terms from product types can be regarded as projections.

**Property-based Testing**. _Property-based testing_ is a method of testing _algebraic laws_ by pseudo-randomly generating example values, and testing to see if the laws are satisfied. For example, property-testing could generate many integers `a`, `b`, and `c` to check if `(a + b) + c` is equal to `a + (b + c)`. Property-based testing cannot _prove_ laws, but it can disprove them, as well as ensure there are no trivial law violations.

**Pure**. A _pure_ procedure combines inputs into an output, and does not interact with the world outside the function in any way that can be observed. All pure procedures are _deterministic_, but not all _deterministic_ procedures are pure.

**Record**. A _record_ contains `n` fields, each accessed with some _label_, called the _name_ of the field, and each with some associated type. All records are product types, although not all product types are records (for example, terms of tuples are accessed by ordinal values, not names).

**Recursive Data**. _Recursive data_ is any data type that appears as a value somewhere inside the data type. Common examples of recursive data include linked-lists and trees.

**Recursive Functions**. _Recursive functions_ are functions that call themselves. In functional programming, recursive functions allow iteration, like processing elements in a list or accepting socket connections.

**Recursion Schemes**. _Recursions schemes_ refers to any of the many different ways to traverse recursive data types, such as folding (catamorphisms) or unfolding (anamorphisms). Typically _recursion schemes_ refers to the use of fixed-point data types to factor out recursion from data types, and minimize the boilerplate involved in creating different recursion schemes for different recursive data types. However, fundamentally, any recursive data type can be constructed, deconstructed, and transformed in any of the different schemes.

**Referential Transparency**. _Referential transparency_ is a property of functional programs in which any expression may be replaced by the value it computes without changing the behavior of the program. Referential transparency allows refactoring programs without changing their behavior, and is necessary (but not sufficient) for _local reasoning_.

**Reified**. A _reified_ construct is some non-value construct (such as a type, a field, or a package) that has been represented as a value. For example, reified generics store the type of generic type parameters as values at runtime. Similarly, lenses are a type of reified field, which provide a way to treat a field of a record as a first-class value, which can be passed around, stored, and transformed.

**Semigroup**. A _semigroup_ is a type class with a single binary operator, often called _append_ or _combine_, that is _assocative_.

**Side-Effect**. A _side-effect_ occurs when evaluation of an expression does anything more than computing a value. Side-effects are interactions that can be observed from outside a procedure or expression. Common side-effects include database access, file access, network access, system calls, modifying mutable memory, or calling procedures that do any of the above.

**State**. _State_ refers to any data that is used to iteratively process information, either in a loop, or using recursion.

**Structural Typing**. _Structural typing_ is a method of defining types solely based on their structure. Two structural types are equal if and only if they have the same structure. Structural typing stands in contrast to _nominal typing_, in which two types that have the same structure are regarded as different if they have different names.

**Structure**. The _structure_ of a data type is the set of statements we know to be true about the data type. Parametrically polymorphic types (as well as `Any` in Scala) have *no* structure, but we can add structure by requiring these types have _instances_ for one or more _type classes_.

**Subtyping**. _Subtyping_ is a method of defining types that permits "subset" relations, where one type is defined to be a subset or superset of another type. For example, all dogs are animals, and a type system with subtyping can recognize the fact that a `Dog` type is a subset of an `Animal` type.

**Sum Type**. A _sum type_ is the composition of `n` types together in an enumeration type, where each case of the enumeration is referred to as a _term_ in the sum type. A value from a sum type contains a value from exactly one of its terms. For example, a value of a `Suite` enumeration is either `Hearts`, `Diamonds`, `Spades`, or `Clubs`. In Scala 2.x, enumerations are emulated with `sealed trait`, where the terms of the sum type are subtypes of the `sealed trait`.

**Tagless-Final**. _Tagless-final_ refers to a way of encoding the interpretation of a DSL using parametric polymorphism. In tagless-final, code written in a DSL is polymorphic in the data type used for interpretation. This polymorphic code is interpreted in different ways by instantiating it to concrete evaluation types. In Scala, _tagless-final_ refers almost exclusively to using _indirection_ to provide ad hoc operations across polymorphic effect types.

**Total**. A _total_ procedure returns an output for every input. Total procedures, unlike partial procedures, always terminate, and they never throw exceptions.

**Traversals**. _Traversals_ are _optics_ that allow getting and setting elements in collection-like data structures.

**Type**. A _type_ is information that exists at compile-time, stored in the computer's memory as the program is compiling. Types are used by the compiler to describe the structure of variables and functions, to ensure programs are well-defined and consistent. Informally, a type can be regarded as a mathematical set of values; the type `Boolean`, for example, contains the values _true_ and _false_. To ascribe a variable a type in a programming language is to assert the value is a member of the set described by the type, and the act of constructing such a value can be regarded as a proof that the assertion is correct.

**Type Class**. A _type class_ is a language-level encoding of an _abstraction_. Type classes can be viewed as functions from some type to an _algebraic structure_ for that type. For example, an `Ord` type class might encode the algebraic structure of total ordering, and might allow us to access ordering operations for types that have this algebraic structure.

**Type Constructors**. A _type constructor_ is a type that is itself parameterized by other types. To construct a type, the type constructor must be _fully applied_ to all of its parameters, which then results in a type. For example, `List` (`[]`) is a type constructor. Given a type, such as `Int` (`Integer`), then "passing" that type to the `List` type constructor then constructs a type: namely, the type of lists with that specified element type.

**Universal Quantification**. _Universal quantification_ asserts that a statement holds for all possible choices of a given variable. All _parametrically polymorphic_ functions and data types universally quantify over their type parameters, asserting the function or data type has an implementation for all possible choices of the type parameters. For example, the function `def empty[A]: List[A]` (`empty :: [a]`) universally quantifies over the type parameter `A`, asserting that for any element type, the function can produce a list of that element type (namely, the empty list).

**Universal Type**. A _universal type_ is any type variable used in _univeral quantification_. In programming languages, universal types are introduced by _parametrically polymorphic_ function and data type declarations.

**Value**. A _value_ is information that exists at runtime, stored in the computer's memory as the program is executing. Values are used to hold and transmit information within a program, and across process boundaries, to the operating system, file system, network, and beyond. In programming languages, values are constructed from literals or from _expressions_.

## FAQ

*Question*: Can you define some other term that I find confusing?

*Answer*: Sure, send it on and I'll add it to the glossary!

*Question*: Wikipedia doesn't agree with your definitions.

*Answer*: Oh no!

*Question*: I don't agree with your definitions.

*Answer*: Ok.

*Question*: Change your definitions to what I want!

*Answer*: Nope. But if you have ideas to improve these definitions, please share your feedback!