
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


Law is an equivalence between two programs that should always be true



Beyond Type Safety in Swift -- The Virtues of following the Law

Despite Swift's expressive type system there is only so much we can ask our type checker to verify. Laws take us one step further.

Laws are simple mathmatical equations that you promise to uphold when implementing a protocol. For example, when implementing `Equatable` you promise that your implementation of `==` is symmetric: `a == b` implies `b == a` for any `a` and `b`.

This talk will explore the life of a simple `(T, T) -> T` binary operation in the context of several different protocols: Each protocol only differs in it's laws over this operation and through the definition of one or more constants of type `T`.

Each new law that this binary operation demands gives power to our protocol consumers. Associativity, for example, gives us parallelization!

We will discover the meaning behind scary names given by mathmaticians to describe collections of laws over our simple operation.

Theorems on these scary sounding algebraic structure become extension methods you get for free in your protocol!

By understanding and embracing these simple mathmatical laws, not only can we enrich and simplify our APIs in an objective (mathmatical) way -- but also, we, as an iOS development community, can talk to each other at a higher level of abstraction, and therefore find clean solutions to more problems together faster.

