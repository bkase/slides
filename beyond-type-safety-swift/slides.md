<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Beyond Type Safety in Swift 

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

!!!

## Let's say we want to design an API

!!!

### Async Work Cancellation

(image)

!!!

### Approach 1: Cancel is a method on our work

```swift
extension Work {
  func cancel(q: DispatchQueue) -> Void { /*...*/ }
}
```

Note: Go through this process... to make it better, what is better?

!!!

### Can't express easily

* Cancelling something twice should be okay
* Invoke cancel on many pieces
* Automatically cancel work on deinit

Note: We could go through and ... Let's discover this functionality

!!!

### Uncontroversial Goals

1. Minimize Complexity (Simple)
2. Maximize Expressivity

!!!

### Complexity: How to measure

Complexity increases as public interface _primitive count_ increases

Note: Primitive means a parameters to a function, a new type in the library. A very simple library has one type and one method with one parameter for example. Yes this can actually be useful, we'll see later.

!!!

### Expressivity: How to measure

Expressivity increases as _solvable problems_ increase

Note: Empower to write clean code in applications. But it's hard to do more things with fewer things...

!!!

### Tradeoff: Simple vs Expressive

(image)

Note: Tradeoff, we want to impact simplicity very trivially and extremely raise expressivity.

!!!

### Approach 1: Cancel is a method on our work

```swift
extension Work {
  func cancel(q: DispatchQueue) -> Void { /*...*/ }
}
```

Note: Simple but not expressive

!!!

## Emergent expressivity without complexity

!!!

### Approach 2: Composable Cancellation

```swift
struct Canceller {
  let ctx: ExecutionContext
  let cancel: () -> Void
}
```

Note: Still pretty simple, not expressive yet

!!!

### Composable APIs: Expressive and Simple

The local maxima for expressiveness you can squeeze out of one type

Note: In other words, a way to increase expressiveness without harming simplicity as much

!!!

## This talk: All about one flavor of composable method

!!!

### Closed Binary Operation

```swift
infix operator <>: MultiplicationPrecedence {
  associativity: left
}
func <>(lhs: A, rhs: A) -> A {
  // implement this for some A
}

// Usage: `x <> y <> z`
```

Note: This is so general, We could call it a frog, or a car...

!!!

### Magma

```swift
protocol Magma {
  func op(other: Self) -> Self
}
```

```swift
func <><M: Magma>(lhs: M, rhs: M) -> M {
  return lhs.op(rhs);
}
```
<!-- .element: class="fragment" data-fragment-index="1" -->


Note: We can define an operator to make our lives better

!!!

### Magma: The name

![Brandt](img/brandt.jpg)

> Heinrich Brandt (1926) and Hausmann & Ore (1937)

Note: Naming is hard. We're talking about something so general that we have to reach for scary sounding names. But mathemticians deal with these general things so they have names for them. Instead of making up a name, let's use the ones the mathematicians gave us. I'll speak more to that later.

!!!

### Magma: Simple Example (definition)

```swift
struct Sum{ let value: Int }
extension Sum: Magma {
  func op(other: Sum) -> Sum {
    return Sum(value: self.value + other.value)
  }
}
```

Note: We wrap the int in a struct so we can implement multiple different instances

!!!

### Magma: Simple Example (usage)

```swift
let s = Sum(value: 1) <> Sum(value: 1)
print(s.value) // 2
```

Note: This example shouldn't be used in real code though, right? This is because we decreased simplicity (added a type) at no benefit to expressivity(we already have a way to sum integers). But when could doing this be useful?...

!!!

### Canceller combining forms a magma

```swift
extension Canceller: Magma {
  func op(other: Canceller) -> Canceller {
    return Canceller(
      cancel: {
        self.cancel()
        other.cancel()
      })
      // mem cycles here so make sure you weak properly in production
  }
}
```

Note: Don't stop here!

!!!

## Laws: The Journey

Note: Laws! We're going to enrich our magma with some laws

!!!

### Laws?

A law is an _equivalence_ between two programs that should _always be true_

!!!

## Law 1: Associativity

(image)

!!!

### Review from high-school: Associativity

```
(x + y) + z = x + (y + z)
// ex: (1 + 2) + 3 = 1 + (2 + 3)
```

Note: These are equivalent for any x,y,z for integer addition

!!!

### Associativity

```
(x <> y) <> z = x <> (y <> z)
```

Note: On our closed binary operation, we can reason about associativity with this: any x,y,z under some type with the operation `<>`)

!!!

### Associativity

```
/// Law: (x <> y) <> z = x <> (y <> z)
protocol Semigroup: Magma {}
```

!!!

### Aside: Equatable: example of ascribing laws to protocols

![equatable definition](img/equatable.png)

Note: In Swift, we can ascribe laws to protocols. See the equatable

!!!

### Associativity

```swift
struct Sum: Semigroup {}
// We get the op definition from magma
```

Note: Sure but why do I care?

!!!

### Associativity power: Freedom to chunk work

```swift
let xy = x <> y
let all = xy <> z
// or
let all = x <> y <> z
```

Note: It's up to the consumers of the API. They get more expressive power!

!!!

### Associativity power: Paralellization is safe

```swift
// work I need to do: x <> y <> z <> w
let a = x <> y // on thread1
let b = z <> w // on thread2
return a <> b
```

!!!

### FoldPar definition

(image of execution trace tree)

Note: split into pieces and use the empty case for off-by-one

!!!

### FoldPar code

```swift
extension Semigroup {
  func foldPar(empty: Self, work: [Self], ctx: ExecutionContext) {
    // split into pairs, use empty if odd number
    // dispatchAsync all the work
    // recurse
  }
}
```

Note: You could probably do this more generically with sequences

!!!

### Why don't we just make everything semigroups!

!!!

### Counterexample: Magma but not semigroup

```swift
// subtraction over integers
(x - y) - z = x - (y - z)
```

```swift
// ex: (1 - 2) - 3 = 1 - (2 - 3)
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
//      -1 - 3     = -1 + 3
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
//      4          = 2  (contradiction)
```
<!-- .element: class="fragment" data-fragment-index="3" -->

Note: Subtraction over integers is a binary operation that's closed, but not a semigroup because operatoin is not associative.

!!!

### Is Canceller a Semigroup?

```swift
(c1 <> c2) <> c3 = c1 <> (c2 <> c3)
```

Note: As long as your cancel functions are well-behaved (expand on this)

!!!

### Counterexample

```swift
var doCancel2 = true
let cancel1 = { doCancel2 = false }
let cancel2 = { if (doCancel2) { cancelWork() } }
```

Note: For example, setting a global variable that would cause a cancel to misbehave. Globals are bad. Why would that break associativity? We could set a global if one operation runs first and then check the global in the next operation (which would be unse if ...)

!!!

### Custom Law: Cancels must not be coupled

```swift
/// Cancelling something can't have side-effects that affect other cancellations in any way
/// Law: c1.cancel() /* doesn't affect */ c2.cancel()
```

!!!

### Canceller is a semigroup now!

```swift
extension Canceller: Semigroup {}
```

Note: Parens don't matter! Moving on to the next law...

!!!

## Law 2: Left-right identity

(image)

TODO: Name of "left right identity"?

!!!

### Identity

```
empty <> x = x
x <> empty = x
// ex: 0 + 1 = 1
// ex: 1 + 0 = 1
```

Note: Some element that when combined on the left or right is the same

!!!

### Identity power: Inline conditionals

```swift
func annoyinglyNeedOptional() -> Foo? {
  return something ? x <> y : nil
}
// to
func expressiveNoNeedOptional() -> Foo {
  return something ? x <> y : empty
}
// or: optFoo ?? empty
```

Note: Again this is up to the client to decide if the inlining will make the code cleaner at the client's callsite. More expressive power!

!!!

## Idenity + Semigroup = Monoid

```swift
/// Law: empty <> x = x <> empty = x
/// and Semigroup laws
protocol Monoid: Semigroup {
  static var empty: Self { get }
}
```

Note: Just extending semigroup with the identity constant

!!!

### Monoid Example: Sum

```swift
struct Sum{ let value: Int }
extension Sum: Monoid {
  static var empty: Sum { return Sum(value: 0) }
  /* sum is a semigroup, so we already have op defined */
}
```

Note: We already saw that though, let's see something a bit more interesting

!!!

### Monoid Example: TwoSums

```swift
struct TwoSums{
  let a: Sum
  let b: Sum
}
extension TwoSums: Monoid {
  static var empty: TwoSums = { return TwoSums(a: Sum.empty, b: Sum.empty) }
  func op(other: TwoSums) -> TwoSums {
    return TwoSums(
      a: self.a <> other.a
      b: self.b <> other.b
    )
  }
}
```

Note: In fact, any two monoids tupled form a monoid by applying the operations to the two pieces. Unfortunately this is hard to express in Swift automatically for any tuple of two monoids, see appendix for a hack to do it, but there are tradeoffs (show the struct thing in the appendix)

!!!

### Monoid: FoldPar better

```swift
extension Monoid {
  func foldPar(rest: [Self], ctx: ExecutionContext) {
    /* … */
  }
}
```

Note: Now what was the _empty_ parameter of the `foldPar` function is implicit. Simplifies our API.

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

### Does Canceller have an identity?

(image)

!!!

### Canceller is a Monoid

```swift
extension Canceller: Monoid {
  static var empty = Canceller(
    ctx: .immediate,
    _cancel: { } // no-op
  )
  // already a semigroup
}
```

!!!

### Identity well-behaved proof

(diagram)

!!!

### Monoids are powerful

(image)

Note: Once you have a monoid, I've found that you have a SICK API. If you think about a solution to your problem and it's monoid. Be happy. Your API is clean. But why stop there...

!!!

## Law 3: Commutativity

!!!

### Commutativity

```swift
x <> y = y <> x
```

!!!

### Comutativity Power: Operations can be moved around

TODO

!!!

### Commutative Monoid

```swift
/// Law: x <> y = y <> x
/// and Monoid laws
protocol CommutativeMonoid: Monoid {}
```

!!!

### Example: Average

```swift
// a = totalSum
// b = totalCount
struct TwoSums{ a, b }
extension TwoSums: CommutativeMonoid {
  func avg() -> Double {
    return a * 1.0 / b
  }
}
// already a monoid, so we don't have to do anything else
```

Note: A bit cooler

!!!

### DistFold with current progress!

```swift
extension CommutativeMonoid {
  func distFold(rest: [Self], ips: [IpAddress]) {
    // distribute work across all ips
    // every device does some work
    // put it back together
    // *** SHOW RESULTS INCREMTNALLY WHEN ANY DEVICE FINISHES ***
    /* … */
  }
}
```

Note: Imagine doing averages across many machines. Super computer vs smartphone

!!!

### Canceller combining is commutative!

(diagram)

Note: So canceller is a CommutativeMonoid

!!!

### Canceller cancels are threadsafe without extra locking!

```swift
// random work spawning in a threadpool
func onSpawn(c: Canceller) {
  combined = c <> combined
}
```

Note: Order that this is executed doesn't matter!

!!!

## Law 4: Idempotence

!!!

### Idempotence

```swift
x <> x <> y = x <> y
```

Note: Doing something twice is the same as once

!!!

### Bounded Semilattice

```swift
/// Law: x <> x <> y = x <> y
/// and CommutativeMonoid laws
protocol BoundedSemilattice: CommutativeMonoid {}
```

Note: Really good for distributed systems

!!!

### Bounded Semilattice Example: Max

```swift
struct Max{ let v: Int }
extension Max: BoundedSemilattice {
  static var empty = Int.min
  func op(other: Max) -> Max {
    return max(self.v, other.v)
  }
}
```

!!!

### DistFold over UDP!

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

### Canceller: Is it idempotent?

```swift
c1 <> c1 <> c2 = c1 <> c2
```

Note: No! We can make it idempotent though

!!!

### Canceller where cancel only runs once

```swift
struct Canceller {
  let ctx: ExecutionContext
  private let _cancel: () -> Void
  func cancel() -> Void {
    dispatch_once {
      self.ctx.dispatch { self._cancel() }
    }
  }
  // mem cycle here
}
```

Note: Now it's idempotent! Because of the dispatch_once, we can always just apply the same cancellation again!

!!!

### Canceller: Idempotence

We don't need to *remember* if we've cancelled something

Note: You may say that this is obvious from the dipatch_onces, was this a waste of time? No!

!!!

### Canceller API discovered

We *discovered* a *simple* and *expressive* API for work cancellation

!!!

### That's not all

* Our docs our extremely precise
* We can use _property based testing_ tools like quickcheck for correctness tests

Note: And that's not all all, here's what we can clean up now...

!!!

### Scenario 1: (Work.cancel) I want to cancel all async work for a thing that does a lot of stuff

```swift
class LotsOfAsyncWork {
  // manually remove futures here that are finished or I cancelled already
  var manyWorks: [Work] = []

  func cleanup() {
    manyWorks.forEach{ $0.cancel() }
  }
}
```

Note: This sucks. I need to care about cancel remembering that I've already called it. Plus the future could be loading something big like a bitmap, and I can leak it easily.

!!!

### Scenario 1: (Canceller) I want to cancel all async work for a thing that does a lot of stuff

```swift
class LotsOfAsyncWork {
  var canceller: Canceller = Canceller.empty
  // add new cancellers by <>ing them
}
```

Note: There is no cleanup function. Just do `self.canceller.cancel()`. Semantically clean. Readable code.

!!!

### Scenario 2: Automatically cancel on view controller deinit

```swift
class AutoCanceller {
  let canceller: Canceller

  deinit {
    canceller.cancel()
  }
}
extension AutoCanceller: BoundedSemilattice
```

Note: Look how easy this is!! We just wrap the canceller with auto-canceller and we're done.

!!!

### Even more useful real example

![Caches are monoids 1]()
![Caches are monoids 2]()

Note: Plus I go over it slowly

!!!

### Animations!

I'm working on a formalization of animations. Please ask me about it and we can discuss. Not ready to share more publicly just yet.

!!!

## Names

!!!

### Names matter

Names matter. Many people may dismiss this stuff because they are afraid to learn the names. Don't be one of those people. <-- less agressive

Important that we have consistent names for these abstractions across programming languages and disciplines (mathmatics). Then we can all be on the same playing field. The names already exist -- mathamticians have been using them for a hundred years. Don't be so arrogant that you can make up a different name. If we all learn the same names for things we can solve more problems together across disciplines.

TODO: Make that less ranty

!!!

### Take-away

* <!-- .element: class="fragment" data-fragment-index="1" --> Optimal API design is about maximizing simplicity and expressivity 
<!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> Compositional APIs maximize these properties
<!-- .element: class="fragment" data-fragment-index="2" -->
* <!-- .element: class="fragment" data-fragment-index="3" --> Don't stop at composition. More laws = More expressivity
<!-- .element: class="fragment" data-fragment-index="3" -->
* <!-- .element: class="fragment" data-fragment-index="4" --> This stuff is actually useful in real life!
<!-- .element: class="fragment" data-fragment-index="4" -->
* <!-- .element: class="fragment" data-fragment-index="5" --> Learn the names! And teach your coworkers and friends!
<!-- .element: class="fragment" data-fragment-index="5" -->

!!!

<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

Slide Deck: [https://is.gd/KzVKWh](https://is.gd/KzVKWh)

!!!

## Appendix

!!!

Put bit defining stuff here.

!!!

### Counterexample: Semigroup but not Monoid

```swift
// linked list with at least one element
indirect enum Nel<T> {
  case one(T)
  case more(T, Nel<T>)
}
// let a = .one('a')
// let abc = .more('a', .more('b', .one('c')))
```

!!!

### Counterexample: Nel concattenation not monoid

```swift
extension Nel: Semigroup {
  func op(other: Self) -> Self {
    // concatenation
  }
  // I have no identity!
}
```

!!!

### Counterexample: Nel concat associative

(diagram)

Note: Non-empty-lists is probably another talk, but hopefully you at least have an inution

!!!

### Counterexample: Monoid but not commutative monoid

```swift
extension Array: Monoid {
  static var empty = []
  func op(other: Self) -> Self {
    return self + other
  }
}
// [1] + [2] != [2] + [1]
```

!!!


