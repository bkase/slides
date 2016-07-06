<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Grokking Lazy Sequences and Collections

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

!!!

# Photos

(screenshot)

!!!

# Problems

* There are a lot of photos
* Loading a photo's data is expensive

!!!

# Problems

* There are a lot of things
* Loading a thing is expensive

Note: More generally...

!!!

## Holding Things

!!!

## Holding Things

* Arrays?

!!!

## Holding Things

```swift
let a = Array(PHAsset.fetchAllAssets/*...*/())
// slowwwwww
```

Note: Not abstract enough

!!!

## Holding Things

We need a `Collection`

!!!

### Groups of Things

* `Collection`s are `Indexable` `Sequence`s <!-- .element: class="fragment" data-fragment-index="1" -->
* `Sequences` are `Iterator` makers <!-- .element: class="fragment" data-fragment-index="2" -->
* `Iterator`s can produce `Element`s <!-- .element: class="fragment" data-fragment-index="3" -->

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

```swift
struct Evens: IteratorProtocol {
  var state: Int = 0

  mutating func next() -> Int? {
    let curr = state
    state += 2
    return .some(curr)
  }
}
```

!!!

## IteratorProtocol

```swift
let s = Evens()
for _ in 1...5 {
  print(s.next()!) // 0, 2, 4, 6, 8
} 
```

Note: This kinda sucks having to call next()

!!!

## Sequence

(image here)

Recall: `Sequences` are `Iterator` makers

!!!

## IteratorProtocol

```swift
struct Evens: IteratorProtocol, Sequence
```

Note: Solution: we make it a Sequence

!!!

## Sequence

```swift
for x in Evens().prefix(5) {
  print(x) // 0, 2, 4, 6, 8
}
```

!!!

## Sequence

```swift
extension Sequence where 
    Iterator == Self,
    Self: IteratorProtocol {
  func makeIterator() -> Iterator {
    return self
  }
}
```

Note: makeIterator() is inferred

!!!

## Sequence

So what else can we do with a sequence?

!!!

## Sequence

```swift
for x in someSequence {
  print(x)
}
```

!!!

## Sequence

* `map`, `filter`, `reduce`
* `prefix`, `dropFirst`
* much, much more

!!!

## Sequence

* no contract of one-pass or multipass (no indexing!)
* no contract on finite-ness
* must be able to produce an iterator (could be self)

!!!

## Collection

(image here)

Recall: `Collection`s are `Indexable` `Sequence`s

!!!

## Collection

```swift
protocol Collection: Sequence, Indexable
```

!!!

## Collection

(images of photos in a collection)

Note: iOS SDK has an API method for getting photo metadata of many photos

!!!

## Collection

```swift
struct PhotosMetadata: Collection {
  /*...*/
  var startIndex: Int { /*...*/ }
  var endIndex: Int { /*...*/ }
  func index(after i: Int) -> Int { /*...*/ }
  subscript(i: Int) -> PHAsset { /*...*/ }
}
```

!!!

## Collection

```swift
  private let base: PHFetchResult

  init(_ base: PHFetchResult) {
    self.base = base
  }
```

!!!

## Collection

```swift
  var startIndex: Int {
    return 0
  }
```

!!!

## Collection

```swift
  var endIndex: Int {
    return base.count
  }
```

!!!

## Collection

```swift
  func index(after i: Int) -> Int {
    return i + 1
  }
```

!!!

## Collection

```swift
  subscript(i: Int) -> PHAsset {
    return base[i] as! PHAsset
  }
```

!!!

## Collection

* Why did we have to implement all that?

!!!

## Collection

* Alternate index types allow for LinkedLists
* Alternate index types let you index with Unary numbers
* ... see appendix

!!!

## Photos

```swift
let a = PhotosMetadata(PHAsset.fetchAllAssets/*...*/())
// not slow!
```

!!!

## Photos

Now we want to turn our `PHAsset`s into `Photo`s

!!!

## Photos

```swift
struct Photo {
  let url: NSURL
  /*...*/
}
```

!!!

## Photos

```swift
let metadata: PhotosMetadata
let photos = metadata.map{ Photo(url: $0.url) }
// slowwww
```

!!!

## Photos

* Delay metadata load

!!!

## Photos

* Delay operations on collections or sequences

!!!

## Lazy Groups of Things

Computed when the information is _forced_ out

!!!

## LazySequence

`protocol LazySequenceProtocol: Sequence`

!!!

## LazyCollection

`protocol LazyCollectionProtocol: LazySequence, Collection`

!!!

### Normal Map

```swift
// eager sequence/collection
[1,2,3].map{ $0 + 1 } // [2, 3, 4]
```

!!!

### LazySequence Map

```swift
Evens().lazy.map{ $0 + 1 }
    // LazyMapSequence<Int, Int>
// nothing is computed yet!
```

!!!

### LazyCollection Map

```swift
[1,2,3].lazy.map{ $0 + 1 } 
    // LazyMapRandomAccessCollection<Int, Int>
// nothing is computed yet!
```

!!!

#### How can an overriden method return different types?

!!!

### Protocol default implementations

Note: I said I would assume you knew this, but; this is nuanced and cool

!!!

### Default Implementations

```swift
protocol A { }
extension A {
  func woo() -> Self { return self }
  func hi() -> String {
    return "A"
  }
}
```

!!!

### Default Implementations

```swift
struct ConcA: A {}
print(ConcA().woo().hi()) // "A"
```

!!!

### Default Implementations

```swift
protocol B { }
extension B {
  func hi() -> Int {
    return 0
  }
}
```

!!!

### Default Implementations

```swift
struct Mystery: A, B { }
print(Mystery().woo().hi()) // compile error!
```

!!!

### Default Implementations

```swift
// now B subsumes A
protocol B: A { }
extension B {
  func hi() -> Int {
    return 0
  }
}
```

!!!

### Default Implementations

```swift
struct Mystery: B { }
print(Mystery().woo().hi()) // 0
// NOT a compile error!
```

!!!

// insert lazy map deep-dive
// insert examine types here, burritos

!!!

## Photos

```swift
let metadata: PhotosMetadata
let photos = metadata.map{ Photo(url: $0.url) }
// not slow!
```

!!!

## Photos

```swift
func prepareData(d: PhotosMetadata) -> ? {
  return d
    .filter{ $0.isPhoto() } // skip videos
    .map{ Photo(url: $0.url) } // make photos
    .map{ PhotoView(photo: $0) } // make view
}
```
 
Note: What is the return type?

!!!

## Photos

```swift
LazyFilterBidirectionalCollection<
  LazyMapRandomAccessCollection<
    LazyMapRandomAccessCollection<
      PHAsset, Int
    >, Int
  >
>
```

Note: Ahhhhhh

!!!

## Tortilla Inference

(picture)

!!!

## Type Inference

```swift
let m = d.map/*...*/
return m.first
```

Note: Can we avoid the need to ever write the type

!!!

## Tortilla Erasure

(picture)

!!!

## Type Erasure

```swift
let m = AnyCollection(
  d.filter{}.map{}/*...*/
)
// m: AnyCollection<PHAsset>
```

Note: Trade type information for maintainable code

!!!

## Type Erasure

* AnySequence
* AnyCollection
* ... see appendix for more

!!!

## Photos

Product wants `everyOther` photo

(picture)

!!!

## New Tortillas

```swift
let a = [1,2,3,4]
let b = a.lazy.everyOther()
for x in b {
  print(x) // 2,4
}
```

!!!

## New Tortillas

```swift
// the tortilla sequence
struct LazyEveryOtherSequence
    <S: Sequence>: LazySequenceProtocol {
  var base: S
  func makeIterator() -> LazyEveryOtherIterator<S.Iterator> {
    return LazyEveryOtherIterator(base: base.makeIterator())
  }
}
```

!!!

## New Tortillas

```swift
struct LazyEveryOtherIterator
    <I: IteratorProtocol>: IteratorProtocol {
  var base: I
  mutating func next() -> I.Element? {
    return base.next()?.next()
  }
}
```

!!!

## New Tortillas

```swift
extension LazySequenceProtocol {
  func everyOther() -> LazyEveryOtherSequence<Self> {
    return LazyEveryOtherSequence(base: self)
  }
}
```

!!!

## Photos

```swift
let skipped = photos.everyOther()
```

!!!

## Photos

Product wants every 4th photo

!!!

## Photos

```swift
let skipped = photos.everyOther().everyOther()
```

Note: Precisely why you WANT to use the Swift machinery when you can

!!!

## Photos

* Collections hold our photo metadata and photos

!!!

## Photos

* LazyCollection's map transforms our data

!!!

## Photos

* AnyCollection helps us maintains our type signatures

!!!

## Photos

* We can create new operators that compose

!!!

## Photos

Our app works!

(picture)

!!!

<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Grok it?

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

P.S. IBM Swift Sandbox is legit






!!!

# Grok

!!!

## Assumptions

1. Comfortable with protocols
2. Uncomfortable with Swift3 standard library details

!!!

## Ready?

Prepare yourself <!-- .element: class="fragment" data-fragment-index="1" -->

for information <!-- .element: class="fragment" data-fragment-index="2" -->

!!!

## Lazy

What does it mean to be lazy?

!!!

## Lazy

Computed at the last possible moment, when the information is _forced_ out

!!!

## Eager

Eager?

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

## Lazy

> "infinite" cards

(insert shorts screenshot here)

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

# Eager Groups of Things

!!!

# Eager Groups of Things

Eager means _operations_ on the groups of things are eager

The groups of things themselves can be strict or lazy

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

* Just a `mutating next()`
* One-pass only

Note: How can you go back?

!!!

## Self Iterating Sequence

```swift
struct Evens: Sequence, IteratorProtocol {
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
let zeroThroughEight = Evens().prefix(5)
print(zeroThroughEight)
// [0, 2, 4, 6, 8]
```

!!!

## Self Iterating Sequence

* The associated type (`Element`) was inferred
* Where is the `makeIterator()`?

Take a look again <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

## Self Iterating Sequence

```swift
struct Evens: Sequence, IteratorProtocol {
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

Note: Here is is

!!!

## Sequence

The sequence/collection infrastructure goes crazy with this stuff.

Read through the Swift source if you get a chance, it's cool

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

* `map`, `filter`, `reduce`
* `prefix`, `dropFirst`
* much, much more

!!!

## Sequence

* no contract of one-pass or multipass (no indexing!)
* no contract on finite-ness
* must be able to produce an iterator (could be self)

!!!

## Collection

`protocol Collection: Sequence, Indexable`

!!!

## Collection

It's a sequence, but

We have addressable elements.
We can index!

!!!

## Collection Rules

* has addressable positions (index)
* must support multipass iteration
* exists a one-to-one mapping of elements to indices
  * No infinite structures!
* expect O(1) subscript access

!!!

## Simple Collection

Let's make a unary collection

!!!

## Unary

```swift
let u: Unary</*...*/>
u[""] // 1st element
u["x"] // 2st element
u["xx"] // 3st element
u["xxx"] // 4st element
```

!!!

## Unary

Given any input collection addressable via Ints, 

Create a collection addressable via unary numbers!

!!!

## Unary

More powerful than overriding the subscript operator!

!!!

## Unary

```swift
print(u.lazy.map{ x in x + 1 }["xxx"])
```

!!!

## Unary

Let's build it

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

!!!

## Burritos

Look at `Unary<Array<Int>>`

Let's call the `Unary` part the tortilla

Wrapping the collection burrito

!!!

## Collection Variants

How can you move to the next index?

* Collection (ForwardDirection)
* BidirectionalCollection
* RandomAccessCollection

Note: Have to mention it, but let's not spend too much time on it. Collection is just increment in O(1); Bidirection is go forward or backward in O(1); RandomAcessCollection can get the index difference in O(1)

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

Operations on lazy collections/sequences have different return types

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
Evens().lazy.map{ $0 + 1 } // LazyMapSequence<Int, Int>
// nothing is computed yet!

// lazy collection
[1,2,3].lazy.map{ $0 + 1 } // LazyMapRandomAccessCollection<Int, Int>
// nothing is computed yet!
```

!!!

### Protocol default implementations

Note: I said I would assume you knew this, but; this is nuanced and cool

!!!

### Default Implementations

```swift
protocol A { }
extension A {
  func hi() -> String {
    return "A"
  }
}
```

!!!

### Default Implementations

```swift
struct ConcA: A {}
print(ConcA().hi()) // "A"
```

!!!

### Default Implementations

```swift
protocol B { }
extension B {
  func hi() -> Int {
    return 0
  }
}
```

!!!

### Default Implementations

```swift
struct Mystery: A, B { }
print(Mystery().hi()) // compile error!
```

!!!

### Default Implementations

```swift
// now B subsumes A
protocol B: A { }
extension B {
  func hi() -> Int {
    return 0
  }
}
```

!!!

### Default Implementations

```swift
struct Mystery: B { }
print(Mystery().hi()) // 0
// NOT a compile error!
```

!!!

### Default Implementations

You can override not just the behavior of extension methods

But also the return type!!

!!!

## WHAT

so cool <!-- .element: class="fragment" data-fragment-index="1" -->

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
struct LazyMapIterator<Base: Iterator, Element>:
      IteratorProtocol, Sequence {
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
// a: LazyFilterBidirectionalCollection<
//        LazyMapRandomAccessCollection<
//          Array<Int>, Int
//        >
//    >
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
// a: LazyFilterBidirectionalCollection<
//      LazyMapRandomAccessCollection<
//        Array<Int>, Int>>
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
init<S : Sequence where S.Iterator.Element == Element,
     S.SubSequence : Sequence,
     S.SubSequence.Iterator.Element == Element,
     S.SubSequence.SubSequence == S.SubSequence>(_ base: S)
```

!!!

## Tortilla Erasure

Swift provides 

* `AnySequence`
* `AnyCollection`
* `AnyBidirectionalCollection`, `AnyRandomAccessCollection`

!!!

## New Tortillas

!!!

## New Tortillas

Let's make an `everyOther` transformation

!!!

## New Tortillas

```swift
// Given a sequence, return a sequence
// that only iterates over every other element
let a = [1,2,3,4]
let b = a.lazy.everyOther()
for x in b {
  print(x) // 2,4
}
```

!!!

## New Tortillas

```swift
// the tortilla sequence
struct LazyEveryOtherSequence
    <S: Sequence>: LazySequenceProtocol {
  var base: S
  func makeIterator() -> LazyEveryOtherIterator<S.Iterator> {
    return LazyEveryOtherIterator(base: base.makeIterator())
  }
}
```

!!!

## New Tortillas

```swift
// the tortilla iterator
struct LazyEveryOtherIterator
    <I: IteratorProtocol>: IteratorProtocol {
  var base: I
  mutating func next() -> I.Element? {
    return base.next()?.next()
  }
}
```

!!!

## New Tortillas

```swift
extension LazySequenceProtocol {
  func everyOther() -> LazyEveryOtherSequence<Self> {
    return LazyEveryOtherSequence(base: self)
  }
}
```

!!!

## New Tortillas

```swift
// Given a sequence, return a sequence that 
// only iterates over every 4th element
let a = (0...10)
let b = a.lazy.everyOther().everyOther()
for x in b {
  print(x) // 4,8
}
```

!!!

### Back to reality

!!!

### Back to reality

> "infinite" cards

(insert shorts screenshot here)

!!!

### Back to reality

This is useful. Try it!

!!!

<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Grok it?

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

P.S. IBM Swift Sandbox is legit

