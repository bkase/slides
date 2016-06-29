## Grokking Lazy Sequences and Collections
Brandon Kase

!!!

Warning: I find this stuff _super_ fascinating, but probably most will find it boring.

I've found that putting in a billion slides, and going super fast through them makes things more interesting.

!!!

Shout-out to the IBM Swift sandbox

Seriously. Really awesome.

Super easy to experiment with Swift3isms

!!!

# Grok

!!!

## Assumptions

1. Comfortable with protocols
2. Uncomfortable with Swift3 standard library details

!!!

Prepare yourself

for information <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

## Lazy

What does it mean to be lazy?

!!!

## Lazy

Computed at the last possible moment, when the information is _forced_ out

!!!

## Eager

Eager (or strict) is the opposite of lazy

Computed _always_

!!!

## Lazy

Why would you _want_ lazy?

!!!

## Lazy

1. A bunch of UIViews
2. Tapping on one of many views does a lot of computation <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

## Eager

```swift
class A {
  var eagerField: Int = 1 + 1
  /* ... */
}
```

!!!

## Lazy

```swift
class A {
  lazy var lazyField: Int = 1 + 1
  /* ... */
}
```

!!!

# Eager Things

!!!

# Sequence

!!!

## Sequence

Something you can iterate over

!!!

## Sequence

Specifically, you need a `makeIterator() -> Iterator`

!!!

## Sequence

The simplest sequence is it's own iterator

!!!

## IteratorProtocol

What's an iterator?

!!!


## IteratorProtocol

```swift
protocol IteratorProtocol {
  associatedtype Element
  mutating func next() -> Element?
}
```

Note: move forward by calling next() until nil

!!!

## IteratorProtocol

* just a `mutating next()`
* one-pass only (how can you go backwards?)

!!!

## Self Iterating Sequence

```swift
struct EagerEvens: Sequence, IteratorProtocol {
  var state: Int = 0

  mutating func next() -> Int? {
    let curr = state
    state += 2
    return .some(curr)
  }
}
```

!!!

## Self Iteratoring Sequence

```swift
// now we can use it
let zeroThroughEight = EagerEvens().prefix(5)
print(zeroThroughEight)
// [0, 2, 4, 6, 8]
```

!!!

## Self Iterating Sequence

* The associated type (Element) was inferred
* Where is the `makeIterator()`?

Take a look again <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

## Self Iterating Sequence

```swift
struct EagerEvens: Sequence, IteratorProtocol {
  var state: Int = 0

  mutating func next() -> Int? {
    let curr = state
    state += 2
    return .some(curr)
  }
}
```

!!!

## Self Iterating Sequence

```swift
extension Sequence where 
    Iterator == Self,
    Self: IteratorProtocol {
  func makeIterator() -> Iterator {
    return self
  }
}
```

!!!

### Aside: Protocol default implementations

!!!

## Default Implementations

```swift
protocol A { }
extension A {
  func hi() -> String {
    return "A"
  }
}
```

!!!

## Default Implementations

```swift
struct ConcA: A {}
print(ConcA().hi()) // "A"
```

!!!

## Default Implementations

```swift
protocol B { }
extension B {
  func hi() -> Int {
    return 0
  }
}
```

!!!

## Default Implementations

```swift
struct Mystery: A, B { }
print(Mystery().hi()) // 0
// NOT a compile error!
```

!!!

## Default Implementations

You can override not just the behavior of extension methods

But also the return type!!!

!!!

WHAT

so cool

!!!

## Sequence

The sequence/collection protocol infrastructure goes crazy with this stuff.

!!!

## Sequence

So what can we do with a sequence?

!!!

## Sequence

```swift
for x in someSequence {
  print(x)
}
```

!!!

## Sequence

`map`, `filter`, `reduce`
`prefix`, `dropFirst`
much, much more

!!!

## Sequence

* no contract of one-pass or multipass
* no contract on finite-ness
* must be able to produce an iterator (could be self)

!!!

## Sequence

Thus, you cannot index into a sequence!
And you need to be careful about consumption of elements.

!!!

## Sequence

```swift
// for struct=EagerEvens
let evens = EagerEvens()
for e in evens.prefix(5) {
  print(even) // 0, 2, 4, 6, 8
}
for e in evens.prefix(5) {
  print(even) // 0, 2, 4, 6, 8
}
```

!!!

## Sequence

```swift
// if class EagerEvens
let evens = EagerEvens()
for e in evens.prefix(5) {
  print(even) // 0, 2, 4, 6, 8
}
for e in evens.prefix(5) {
  print(even) // 10, 12, 14, 16, 18
}
```

!!!

## Collection

`protocol Collection: Sequence, Indexable { /*...*/ }`

!!!

## Collection

It's a sequence, but

We have addressable elements.
We can index!

!!!

## Collection Rules

* has addressable positions (index)
* must support multipass iteration
* must have a finite range of indices corresponding exactly to each element -- no infinite structures
* expect O(1) subscript access

!!!

## Simple Collection

Given any input collection addressable via Ints, 
Create a collection addressable via unary numbers!

!!!

## Simple Collection

We didn't say our addressable positions had to be addressed by an Int!

!!!

## Unary

```swift
struct Unary<C: Collection>: Collection 
    where C.Index == Int, 
          C.IndexDistance == Int {
```

!!!

## Unary

```swift
{
  var startIndex: String { /*...*/ }
  var endIndex: String { /*...*/ }
  func index(after i: String) -> String { /*...*/ }
  subscript(i: String) -> C.Iterator.Element { /*...*/ }
}
```

!!!

## Unary

```swift
    private let xs: C
    
    init(_ xs: C) {
        self.xs = xs
    }
```

!!!

## Unary

```swift
    var startIndex: String {
        return ""
    }
```

!!!

## Unary

```swift
    /// O(n) endIndex
    var endIndex: String {
        return String((0...xs.count).map{ _ in "x"})
    }
```

!!!

## Unary

```swift
func index(after i: String) -> String {
  return i + "x"
}
```

!!!

## Unary

```swift
subscript(i: String) -> C.Iterator.Element {
  return xs[i.characters.count]
}
```

!!!

## Unary

```swift
let u: Unary<Array<Int>> = Unary([1, 2, 3, 4, 5, 6])
print(u.lazy.map{ x in x + 1 }["xxx"])
```

!!!

## Burritos

The unary tortilla wraps our collection into a burrito

!!!

## Collection Variants

The index type determines expected runtime for count (RandomAccessIndex has O(1), ForwardDirection or Bidirection has O(N))

If you implement the RandomAccessCollection protocol, for example you need to provide more methods (and more implied runtimes)

!!!

#### What can we do to collections

Everything we can do with sequences

!!!

#### What can we do to collections

And index
And stuff that depends on finiteness

!!!

## Lazy stuff

!!!

## LazySequence

`protocol LazySequenceProtocol: Sequence`

!!!

## LazyCollection

`protocol LazyCollectionProtocol: LazySequence, Collection`

!!!

## Lazy

Recall: When you inherit from protocols you can override return types of methods

!!!

## Map

```swift
// eager sequence/collection
[1,2,3].map{ $0 + 1 } // [2, 3, 4]
```

!!!

## Map

```swift
// lazy sequence
EagerEvens().lazy.map{ $0 + 1 } // LazyMapSequence<Int, Int>
// nothing is computed yet!
```

!!!

## Map

```swift
// lazy collection
[1,2,3].lazy.map{ $0 + 1 } // LazyMapRandomAccessCollection<Int, Int>
// nothing is computed yet!
```

!!!

## LazyMapSequence?

A sequence wrapped sequence, just like Unary

!!!

## LazyMapSequence

```swift
struct LazyMapSequence<S: Sequence>: LazySequenceProtocol {
  func makeIterator() -> LazyMapIterator</*...*/> { /*...*/ }
}
```

!!!

## LazyMapSequence

```swift
struct LazyMapIterator<Base: Iterator, Element>: IteratorProtocol, Sequence {
  var _base: Base
  let _transform: (Base.Element) -> Element

  mutating func next() -> Element? {
    return _base.next().map(_transform)
  }
}
```

!!!

## Burritos

Each operator applied on the lazy sequences and collections wrap the sequence

!!!

## Burritos

```swift
let a = [1,2,3].lazy.map{ $0 + 1 }.filter{ $0 != 3 }
// a: LazyFilterBidirectionalCollection<LazyMapRandomAccessCollection<Array<Int>, Int>>
```

!!!

## Burritos

When the burritos have too many tortillas, types become a nightmare

!!!

## Tortilla Inference

If you can get by doing _all_ the transformations locally, the types will just be inferred.

!!!

## Tortilla Inference

```swift
func foo() -> Int {
  let a = [1,2,3].lazy.map{ $0 + 1 }.filter{ $0 != 3 }
  return a[2]
}
```

!!!

## Tortilla Erasure

Type erasure in Swift help you deal with the nightmare if you need to return it.

!!!

## Tortilla Erasure

```swift
// a: LazyFilterBidirectionalCollection<LazyMapRandomAccessCollection<Array<Int>, Int>>
let b = AnyCollection(a)
// b: AnyCollection<Int>
```

!!!

## Tortilla Erasure

Type erasure works by moving the generic craziness to the constructor. Keep in mind you and your compiler _lose_ information about your type.

!!!

## Tortilla Erasure

```swift
// one of the inits for AnySequence
init<S : Sequence where S.Iterator.Element == Element, S.SubSequence : Sequence, S.SubSequence.Iterator.Element == Element, S.SubSequence.SubSequence == S.SubSequence>(_ base: S)
```

!!!

## Tortilla Erasure

Swift provides `AnySequence` and one `AnyCollection` variant per Index variant.

`AnyBidirectionalCollection`, `AnyRandomAccessCollection`, etc.

!!!

## New tortillas
