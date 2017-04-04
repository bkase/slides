<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Beyond Type Safety in Swift 

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

!!!

## Let's say we want to design an API

!!!

## Uncontroversial Goals

1. Simple
2. Expressive

!!!

## Simple

TODO: Lookup the slide by john-de-goes and cite it

_More simple_ as I have _fewer and fewer primitives_

Note: Thing means a parameters to a function, a new type in the library. A very simple library has one type and one method with one parameter for example. Yes this can actually be useful, we'll see later.

!!!

## Expressive

_More expressive_ as I can _solve more problems_

Note: But it's hard to do more things with fewer things...

!!!

### Simple vs Expressive

(image)

Note: Tradeoff, we want to impact simplicity very trivially and extremely raise expressivity

!!!

### Compositional APIs

```swift
struct Future<V> {
  /*...*/
  func flatMap<U>(f: (V) -> Future<U>) -> Future<U>
}
```

Note: Futures simplify callbacks, I don't have to convince this audience

!!!

### Compositional APIs 2

```swift
??
```

Note: See my composable caching talk for more details??

!!!

Consider doing a series of examples here?

!!!

### Compositional APIs (more)

The local maxima for power you can squeeze out of one type

!!!

### This talk: All about one instance of this composition

```
struct Foo {
  func op(other: Foo) -> Foo
}
// image or diagran of this?
```

Note: Take two of something and combine it to make one thing

!!!

## Closed binary operations

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
protocol Something {
  func op(other: Self) -> Self
}
```

Note: We can talk about all instances of these operations by talking at the level of protocols. I'll make sure to stick lots of examples in throughout the talk as well. But keep in mind, these work for all sorts of types!

!!!

## Closed binary operations

```swift
// assume we don't crash
protocol Magma {
  func op(other: Self) -> Self
}
```

!!!

## Simple Example (definition)

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

## Simple Example (usage)

```swift
let s = Sum(value: 1).op(Sum(value: 1))
print(s.value) // 2
```

Note: This example shouldn't be used in real code though, right? This is because we decreased simplicity (added a type) at no benefit to expressivity(we already have a way to sum integers). But when could it be useful?...

!!!

## What are some magmas?

* summing integers
* subtracting floating point numbers
* dividing doubles
...
* cache composition
* animation composition
* cancel-token composition?

Note: What's not a magma! We haven't constrained ourselves sufficiently -- this is why it's not very useful.

!!!

## Closed binary operations

```swift
// TODO: Does swift have infixl?
infixl operator <>: MultiplicationPrecedence
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

Note: Naming is hard. We're talking about something so general that we have to reach for scary sounding names. But mathemticians deal with these general things so they have names for them. Instead of making up a name, let's use the ones the mathematicians gave us. I'll speak more to that later.

!!!

## Magma

```
∀ a, b ∈ M: a.op(b) ∈ M
```

Note: That's it

!!!

### Not expressive enough

(image)

Note: How can we increase expressive power without needing to introduce more types?

!!!

## Laws: The Journey

(image)

Note: Laws! We're going to enrich our magma with some laws

!!!

## Laws?

A law is an _equivalence_ between two programs that should _always be true_

!!!

## Law 1: Associativity

(image)

!!!

## Review from high-school: Associativity

```
(x + y) + z = x + (y + z)
// ex: (1 + 2) + 3 = 1 + (2 + 3)
```

Note: These are equivalent for any x,y,z for integer addition

!!!

## Associativity

```
(x <> y) <> z = x <> (y <> z)
```

Note: On our closed binary operation, we can reason about associativity with this: any x,y,z under some type with the operation `<>`)

!!!

## Associativity

```
/// Law: (x <> y) <> z = x <> (y <> z)
protocol Semigroup: Magma {}
```


!!!

## Aside: Equatable: example of ascribing laws to protocols

(Screenshot from docs of equatable)

Note: In Swift, we can ascribe laws to protocols. See the equatable

!!!

## Associativity

```swift
extension Sum: Semigroup {}
// We get the op definition from magma
```

Note: Sure but why do I care?

!!!

## Associativity implies parentheses don't matter

```swift
let foo = x <> y <> z <> w
```

Note: No need for parentheses. My code is cleaner!

!!!

## Associativity means I can chunk up code into tmp variables

```swift
let xy = x <> y
let all = xy <> z
// or
let all = x <> y <> z
```

Note: It's up to the consumers of the API. They get more expressive power!

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

## ParReduce

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

## Why don't we just make everything associative!

!!!

## Counterexample: Magma but not semigroup

```
// subtraction over integers
(x - y) - z = x - (y - z)
// TODO: animation this part
// ex: (1 - 2) - 3 = 1 - (2 - 3)
//      -1 - 3     = -1 + 3
//      4          = 2  (counterexample)
```

Note: Subtraction over integers is a binary operation that's closed, but not a semigroup because operatoin is not associative.

!!!

## Law 2: Left-right identity

(image)

TODO: Name of "left right identity"?

!!!

## Identity

```
empty <> x = x
x <> empty = x
// ex: 0 + 1 = 1
// ex: 1 + 0 = 1
```

Note: Some element that when combined on the left or right is the same

!!!

## Expressive power: Inline conditionals

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

## Monoid

```swift
/// Law: empty <> x = x <> empty = x
/// and Semigroup laws
protocol Monoid: Semigroup {
  static var empty: Self { get }
}
```

Note: Just extending semigroup with the identity constant

!!!

## Monoid history?

TODO

!!!

## Exmaple: Sum

```swift
struct Sum{ let value: Int }
extension Sum: Monoid {
  static var empty: Sum { return Sum(value: 0) }
  /* sum is a semigroup, so we already have op defined */
}
```

Note: We already saw that though, let's see something a bit more interesting

!!!

## Example: TwoSums

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

## Show foldmap; Foldmap is mapreduce?

TODO

!!!

## ParFold better

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

### But not every semigroup is a monoid

(image)

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

## Monoids are powerful

(image)

Note: Once you have a monoid, I've found that you have a SICK API. If you think about a solution to your problem and it's monoid. Be happy. Your API is clean. But why stop there...

!!!



## Law 3: Commutativity

!!!

## Commutativity

```swift
x <> y = y <> x
```

!!!

## Expressive Power: Operations can be moved around

TODO

!!!

## Commutative Monoid

```swift
/// Law: x <> y = y <> x
/// and Monoid laws
protocol CommutativeMonoid: Monoid {}
```

!!!

## Example: Average

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

Note: Imagine doing averages across many machines. Super computer vs smartphone

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

## Law 4: Idempotence

```swift
x <> x <> y = x <> y
```

Note: Doing something twice is the same as once

!!!

## Bounded Semilattice

```swift
/// Law: x <> x <> y = x <> y
/// and CommutativeMonoid laws
protocol BoundedSemilattice: CommutativeMonoid {}
```

Note: Really good for distributed systems

!!!

## Example: Max

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

## Case Study: Async Work Cancellation

(image)

Note: Let's say we want to make our "future" or something cancelable

!!!

## Approach 1: Cancel is a method on futures

```swift
extension Future {
  func cancel(ctx: ExecutionContext) -> Void { /*...*/ }
}
```

Note: Simple but not expressive

!!!

### Approach 2: A Canceller is a value with a cancel method

```swift
struct Canceller {
  let ctx: ExecutionContext
  let cancel: () -> Void
}
```

Note: Still pretty simple, not expressive yet

!!!

### Canceller what does it mean to combine them?

```swift
// for a,b,c: Canceller
let c = a <> b
```

Note: How about the new cancel cancels both the inner ones

!!!

### Canceller cancels both

(diagram of cancelling both)

!!!

### Canceller cancels both

```swift
extension Canceller: Magma {
  func op(other: Magma) {
    return Canceller(
      cancel: {
        self.cancel()
        other.cancel()
      })
      // mem cycles here so make sure you weak properly in production
  }
}
```

Note: okay cool that works

!!!

### But don't stop here!

!!!

### Canceller associative?

```
(c1 <> c2) <> c3 = c1 <> (c2 <> c3)
```

Note: Not necessarilly, but we really want this, can we change our op to support this?

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

Note: This also made our API simpler! We don't have to remember if we've already cancelled something

!!!

### Canceller is associative now?

```swift
(c1 <> c2) <> c3 = c1 <> (c2 <> c3)
```

Note: Kind of, as long as your cancel functions are well-behaved (expand on this)

!!!

### Custom Law: Cancels must not be coupled

```swift
/// Cancelling something can't have side-effects that affect other cancellations in any way
/// c1.cancel() /* doesn't affect */ c2.cancel()
```

Note: For example, setting a global variable that would cause a cancel to misbehave. Globals are bad. Why would that break associativity? We could set a global if one operation runs first and then check the global in the next operation (which would be unse if ...)

!!!

### Example of breaking the no coupling law

```swift
var doCancel2 = true
let cancel1 = { doCancel2 = false }
let cancel2 = { if (doCancel2) { cancelWork() } }
```

!!!

### Can we think of an identity?

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

### Does it commute?

```swift
c1 <> c2 = c2 <> c1
```

Note: Yes! Commutative Monoid

!!!

### Is it idempotent?

```swift
c1 <> c1 <> c2 = c1 <> c2
```

Note: Yes! Because of the dispatch_once, we can always just apply the same cancellation again!

!!!

### Canceller is a BoundedSemilattice

```
extension Canceller: BoundedSemilattice
```

!!!

### Why do we care about this?

Note: Was that a waste of time? No!

!!!

### Formal semantics of Canceller

* Our docs our extremely precise
* We can use _property based testing_ tools like quickcheck for correctness tests

!!!

### That's not all

(image)

Note: There is real value. We discovered a clean API just by thinking about composition and laws. What can we do with this API?

!!!

### Scenario 1: (Future.cancel) I want to cancel all async work for a thing that does a lot of stuff

```swift
class LotsOfAsyncWork {
  // manually remove futures here that are finished or I cancelled already
  var manyFutures: [Future] = []

  func cleanup() {
    manyFutures.forEach{ $0.cancel() }
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
struct AutoCanceller {
  let canceller: Canceller
  // TODO: Look this up and make sure it works
  deinit {
    canceller.cancel()
  }
}
extension AutoCanceller: BoundedSemilattice
```

Note: I'm not even going to attempt to write the messy code that would be required with the first implementation. Plus this is coupled to this view controller. Look how easy this is!! We just wrap the canceller with auto-canceller and we're done.

!!!

### Even more useful real example

TODO: Link to talk here...

Caches are monoids

!!!

### Animations!

I'm working on treating animations in this manner. Please ask me about it and we can discuss. Not ready to share this just yet.

TODO: Read pan and fran

!!!

### Aside: Names

Names matter. Many people may dismiss this stuff because they are afraid to learn the names. Don't be one of those people. <-- less agressive

It's important that we have consistent names for these abstractions across programming languages and disciplines (mathmatics). Then we can all be on the same playing field. The names already exist -- mathamticians have been using them for a hundred years. Don't be so arrogant that you can make up a different name. If we all learn the same names for things we can solve more problems together across disciplines.

TODO: Make that less ranty

!!!

### Take-away

* Optimal API design is about maximizing simplicity and expressivity
* Compositional APIs maximize these properties
* Don't stop at composition. More laws = More expressivity
* This stuff is actually useful in real life!
* Learn the names! And teach your coworkers and friends!

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

