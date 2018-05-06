
(actually make an animation of something flying across)

!!!

### Uncontroversial Goals

1. Minimize Complexity (Simple)
2. Maximize Expressivity

!!!

### Complexity: How to measure

Complexity increases as public interface primitive count increases

Note: Primitive means a parameters to a function, a new type in the library. A very simple library has one type and one method with one parameter for example. Yes this can actually be useful, we'll see later.

!!!

### Expressivity: How to measure

Expressivity increases as problem solutions increase

Note: Empower to write clean code in applications. But it's hard to do more things with fewer things...

!!!

### Tradeoff

<img alt="expressivity vs complexity" src="img/graph.png" width="80%" height="80%"/>

> created with https://jakevdp.github.io/blog/2012/10/07/xkcd-style-plots-in-matplotlib/

Note: Tradeoff, we want to impact simplicity very trivially and extremely raise expressivity.

!!!

### Composable APIs: Expressive and Simple

The local maxima for expressiveness squeezed from one type

Note: In other words, a way to increase expressiveness without harming simplicity as much

!!!

### Composable Animations

(image or something)

Note: Composable animations let us describe primitives once and then re-use them like lego bricks to make complex reusable layer two primitives.....

!!!

### Okay we want composition... how can we find it in animations

(thinking face)

!!!

### One facet of composition: Sequencing animations

!!!

### Another facet of composition: Playing two animations simultaneously

!!!

### Another facet of composition: Changing time

!!!

### You could stop here, and build out a library, if you use purity, you'll already create something better than 99% of the design space

!!!

### But there's a secret, we can do more thinking up-front; look to mathematics

!!!

### Category Theory is the obvious place, but I think category theory is hard. There is something more primitive, easier to understand, and possible to instantiate in "every-day" modern languages (like swift) not just Haskell/Scala

!!!

<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Abstract-algebra-driven Design

## Animation in Swift as (almost)-Semirings

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

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

Note: Explain closed! This is so general...

!!!

### Magma

```swift
protocol Magma {
  static func <>(lhs: Self, rhs: Self) -> Self
}
```

!!!

### Magma: Diagram view

![magma diagram](img/magma-diagram.png)

!!!

### Magma: The name

![Brandt](img/brandt.jpg)

> Heinrich Brandt (1926) and Hausmann & Ore (1937)

Note: Naming is hard. We're talking about something so general that we have to reach for scary sounding names. But mathemticians deal with these general things so they have names for them. Instead of making up a name, let's use the ones the mathematicians gave us. I'll speak more to that later.

!!!

### Magma (example)

```swift
// sequencing two animations
translate <> fadeOut
```

!!!

### So far, we haven't recovered any powers other than a name; We can think about how our composition behaves

!!!

### Using Laws!

!!!

### Laws?

A law is an _equivalence_ between two programs<br> that should _always be true_

!!!

## Law 1: Associativity

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
/// Law: (x <> y) <> z = x <> (y <> z) (associativity)
protocol Semigroup: Magma {}
```

!!!

### Semigroup: Diagram view

![semigroup diagram](img/semigroup-diagram.png)

!!!

### Sequencing Animations

(associative)

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

## Law 2: Left-right identity

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

### Identity power: Drop the optional

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
/// Law: empty <> x = x <> empty = x (identity)
/// and is a Semigroup
/// (x <> y) <> z = x <> (y <> z) (associativity)
protocol Monoid: Semigroup {
  static var empty: Self { get }
  // func op(other: Self) -> Self
  // (from Magma)
}
```

Note: Just extending semigroup with the identity constant

!!!

### Monoid: Diagram view

![monoid diagram](img/monoid-diagram.png)

!!!

### Sequence Animations Monoid

(image)

Note: This is a little tougher, but if we think really carefully, we can make sure to include some notion of duration in our animation primitive

!!!

## Law 3: Commutativity

!!!

### Commutativity

```swift
x <> y = y <> x
```

!!!

### Comutativity Power: Operations can be reordered

```swift
var combined = Sum.empty
/* ... */
func onNewInt(v: Int) {
  combined = Sum(value: v) <> combined
}
```

!!!

### Commutative Monoid

```swift
/// Law: x <> y = y <> x (commutativity)
/// and the Monoid laws:
/// empty <> x = x <> empty = x (identity)
/// (x <> y) <> z = x <> (y <> z) (asociativity)
protocol CommutativeMonoid: Monoid {}
```

!!!

### CommutativeMonoid: Diagram view

![commutativemonoid diagram](img/commutativemonoid-diagram.png)

!!!

### Sequencing animations NOT a commutative monoid

!!!

### Parallel composition?

Note: This IS commutative, but is it also a monoid?

!!!

### Parallel composition -- magma

!!!

### Parallel composition -- semigroup

!!!

### Parallel composiiton -- monoid?

??? let's just skip this for now

!!!

### There are more laws for a single operation but let's skip that for now

Note: such as idempotence

!!!

### Consider how sequence interacts with parallel

!!!

### Law 4: Distributivity

!!!

TODO

!!!

### Let's take addition to be parallel composition and multiplication to sequence

!!!

### Semirings! Animations are so close to semirings, but don't have a simple additive identity

!!!

### Finally, we can think about implementing

Now we have our abstract model in our head. We know how we want it to behave. Now we can start implementing it.

!!!

### Creativity

Note: I sat down with very smart people and we played around with different representations for a bit until we found one that works

!!!

### What is an animation

A value changing over time

!!!

### Remember, we need to keep the duration to get all our laws to work

Animaation = Value changing over time AND a duration

!!!

### We use the algebra of datatypes and parametricity

```swift
public struct Animation<A> {
  let duration: CFAbsoluteTime
  let _value: (CFAbsoluteTime) -> A
}
```

!!!

### Let's start with the sequence identity

```swift
  /// A multiplicative identity
  public static var one: Animation {
    return .init(duration: 0) { _ in fatalError() }
  }
```

!!!

### Sequence composition (mulitiplication)

```swift
  public static func * (lhs: Animation, rhs: Animation) -> Animation {
    let sum = lhs.duration + rhs.duration
    let ratio = lhs.duration / sum

    return Animation(duration: sum) { t in
      (t <= ratio && lhs.duration != 0)
        ? lhs.value(t / ratio)
        : rhs.value((t - ratio) / (1 - ratio))
    }
  }
```


!!!

### It's more powerful if we define the parallel composition only when A is a semigroup

```swift
extension Animation where A: Semigroup {
  /// Runs two animations in paralllel and combines the results. If one is longer than the other, the shorter one will stop at it's
  /// last value until the longer one finishes.
  public static func + (lhs: Animation, rhs: Animation) -> Animation {
    let newDuration = max(lhs.duration, rhs.duration)
  
    return .init(duration: newDuration) { t in
      let a1 = lhs.value(min(1, t * newDuration / lhs.duration))
      let a2 = rhs.value(min(1, t * newDuration / rhs.duration))
      return a1 <> a2
    }
  }
}
```

!!!

### (It turns out we do some category-theoretic composition as well, so we can animate As and Bs)

It's an applicative functor

!!!

Maybe a few more animations???

!!!

# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

Slide Deck: [https://is.gd/Qv7rC6](https://is.gd/Qv7rC6)

!!!

## Appendix

!!!

