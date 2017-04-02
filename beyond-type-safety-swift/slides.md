<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Beyond Type Safety in Swift 

## The Virtues of following the Law

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

!!!

Some intro

!!!

## In Common

All are _closed binary operations_

!!!

## What

(image)

!!!

## Closed

(image)

Note: The thing we're doing yields us the same type

!!!

## Binary

(image)

Note: Takes two inputs

!!!

## Operation

(image)

Note: a function

!!!

## Closed binary operation

```swift
func op(a: T, b: T) -> T // for some T
```

Note: Function signature looks something like this

!!!

## Closed binary operations

(image)

Note: We want to talk about types that satisfy these constraints

!!!

## Closed binary operations

```swift
protocol Magma {
  func op(other: Self) -> Self
}
```

Note: In Swift, we talk about groups of things with protocols

!!!

## Closed binary operations

```swift
infix operator <>: MultiplicationPrecedence
func <><M: Magma>(lhs: M, rhs: M) -> M {
  return lhs.op(rhs);
}

// Usage: `x <> y`
```

Note: We can define an operator to make our lives better

!!!

## Magma?

(image)

> Heinrich Brandt (1926) and Hausmann & Ore (1937)

!!!

## Magma

```
∀ a, b ∈ M: a.op(b) ∈ M
```

!!!

## Closed binary operations

```swift
// assuming we don't crash
protocol Magma {
  func op(other: Self) -> Self
}
```

Note: Be well-behaved

!!!

## Magma

There are so many magmas, but saying something is a magma doesn't get you much

!!!

## The Journey

(image)

Note: We're going to enrich our magma with some laws

!!!

## Laws?

A law is an _equivalence_ between two programs that should _always be true_

!!!

## Associativity

```
(x <> y) <> z = x <> (y <> z)
```

Note: 

!!!

## Associativity

```
/// Law: (x <> y) <> z = x <> (y <> z)
protocol Semigroup: Magma {}
```

!!!

## Associativity

(image)

Note: Sure but why do I care?

!!!

## Associativity implies parentheses don't matter

```swift
let foo = x <> y <> z <> w
```

!!!

## Associativity implies paralellization

```swift
// work I need to do: x <> y <> z <> w
let a = x <> y // on thread1
let b = z <> w // on thread2
return a <> b
```

!!!

## ParReduce

(image of execution trace tree)

Note: split into pieces and use the empty case for off-by-one

!!!

```swift
extension Semigroup {
  func foldPar(empty: Self, work: [Self], q: DispatchQueue) {
    // split into pairs, use empty if odd number
    // dispatchAsync all the work
    // recurse
  }
}
```

Note: You could probably do this more generically with sequences

!!!

## Identity

```
0 <> x = x
x <> 0 = x
```

Note: Some element that when combined on the left or right is the same; we implicitly required this for our empty case for foldPar

!!!

Show benefit of identity in expressions

!!!

## Monoid

```swift
protocol Monoid: Semigroup {
  var empty: Self { get }
}
```

Note: Just extending with a constant

!!!

## ParFold better

```swift
extension Monoid {
  func foldPar(rest: [Self], q: DispatchQueue) {
    /* … */
  }
}
```

!!!

## Monoid lets us lift that empty out

Now what was the _empty_ parameter of the `foldPar` function is 


!!!

## Monoids are powerful

!!!

## Move over ParFold -- DistFold

```swift
extension Monoid {
  func distFold(rest: [Self], ips: [IpAddress]) {
    // distribute work across all ips
    // every device does some work
    // put it back together
    /* … */
  }
}
```

!!!

## Commutativity

```swift
x + y = y + x
```

!!!

## Commutativity

```swift
// Non-deterministic concurrency okay
// races don't matter
```

!!!

## Commutative Monoid

```swift
protocol CommutativeMonoid: Monoid {}
```

!!!

## DistFold with current progress!

```swift
extension Monoid {
  func distFold(rest: [Self], ips: [IpAddress]) {
    // distribute work across all ips
    // every device does some work
    // put it back together
    // *** SHOW RESULTS INCREMTNALLY WHEN ANY DEVICE FINISHES ***
    /* … */
  }
}
```

!!!

## Idempotence

```swift
x <> x <> y = x <> y
```

Note: Doing something twice is the same as once

!!!

## Bounded Semilattice

```swift
extension BoundedSemilattice: CommutativeMonoid {}
```

Note: Really good for distributed systems

!!!

## DistFold over UDP!

```swift
extension Monoid {
  func distFold(rest: [Self], ips: [IpAddress]) {
    // distribute work across all ips
    // *** USE CUSTOM TRANSPORT PROTOCOL THAT DOESNT CARE ABOUT DUPLICATES ***
    // every device does some work
    // put it back together
    // *** SHOW RESULTS INCREMTNALLY WHEN ANY DEVICE FINISHES ***
    /* … */
  }
}
```

!!!

## Bounded Semilattice

```swift

```
