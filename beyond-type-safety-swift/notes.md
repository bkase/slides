25-35min

1. What in common question?

What do parallel composition of animation choreographs, layering of caches, and task cancellation combination have in common?
(maybe simplify)

They are all examples of closed binary operations

2. Closed
3. Binary
4. Operations
5. Protocol for it
6. Okay but sooo many things conform to this protocol
7. Equatable is this
8. Let's simplify
9. The JOURNEY
10. Associativity -- meaning with simple expression
11. =Semigroup (free stuff)
11. Aside: Scary vocabulary
11. Identity
    Identity + associativity = monoid
12. Idempotence -- at-least-once semantics
    =Semilattice
13. Commutativity -- order doesn't matter; non-deterministic concurrency okay
    =Commutative semilattice
14. Vocabulary is important; talking at a higher level
16. Takeaways: Type safety is important. Laws are the next step. Just as type safety improves correctness so do laws


Consider using "Average" example throughout
make sure to mention that there are many examples that are just Semigroups, just monoid, just commutative monoids etc. Since "Average" is many things.
Average is:
  Magma
  Semigroup
  Monoid
  Cummutative Monoid

Sum is a monoid
Count is a monoid
SumAndCount is a monoid because any two monoids tupled are monoid. Unfortunately this is hard to express in Swift automatically, see appendix for a hack to do it, but there are tradeoffs (show the struct thing in the appendix)
SumAndCount means an Average

* Explain why arrive at least once
* Motivate why obvious cancel implementation can be improved
* Remove specialization ... you will converge on this if you do a good job. But you get there for free by thinking about what things apply to it from the beginning.
* Not talk about execution context
* Types in the _public interface_ of the library
* Omit the ones digit crap in the beginning
* Rephrase introduction to magma, we can call it x y or z, we might as well call it Magma


* Cut magmas a lot to a few slides
* How to get from associativity to a semigroup
* Combine chunk work example with multithreading
* The diamond operation can be really heavy so it's advantageous, to make it easy to picture I'm going to use this example of averaging.

Abstract Motivation for this operator:

Optimal API design.
Composable-first design.
Minimize necessary types?
Once you have the composition. Law it up.
Now you have expressive power.
Examples that are (T,T) -> T
* Caching (see my other talk)
* Cancellation tokens of async work are idempotent monoids
* See a future talk/blog post/library/or paper about Animations as Algebraic Rings
What's a ring?
two monoids stuck together with some more laws about the interaction between them.

Example that isn't this (T,T) -> T operation:
* Futures/Promises better than callbacks for fire-once responses?
  * I think this audience doesn't need convincing of this (and I don't have time)
  * Why are they better?
      Composasable!
      I can take a future and transform it and I get back a future
      Thus I can lift the code out into a temp variable


Do you want to learn more?
I do!
Basic abstract algebra and category theory will help.

For a really low core intro read "How to Bake a Pi"


Beyond Type Safety in Swift -- The Virtues of following the Law

Despite Swift's expressive type system there is only so much we can ask our type checker to verify. Laws take us one step further.

Laws are simple mathmatical equations that you promise to uphold when implementing a protocol. For example, when implementing `Equatable` you promise that your implementation of `==` is symmetric: `a == b` implies `b == a` for any `a` and `b`.

This talk will explore the life of a simple `(T, T) -> T` binary operation in the context of several different protocols: Each protocol only differs in it's laws over this operation and through the definition of one or more constants of type `T`.

Each new law that this binary operation demands gives power to our protocol consumers. Associativity, for example, gives us parallelization!

We will discover the meaning behind scary names given by mathmaticians to describe collections of laws over our simple operation.

Theorems on these scary sounding algebraic structure become extension methods you get for free in your protocol!

By understanding and embracing these simple mathmatical laws, not only can we enrich and simplify our APIs in an objective (mathmatical) way -- but also, we, as an iOS development community, can talk to each other at a higher level of abstraction, and therefore find clean solutions to more problems together faster.

