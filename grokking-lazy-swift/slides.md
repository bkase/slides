<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Grokking Lazy Sequences and Collections

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

Note: Hi I'm brandon kase. Shorts .. We've encountered a bunch of challenges around this..

!!!

# Photos

<img src="./img/photo-stack.jpg" height=500 />

> http://friendlycomputersspokane.com/wp-content/uploads/2014/06/photo-stack.jpg

!!!

# Photos

<img src="./img/cards1.jpg" height=500 />

!!!

# Problems

* There are a lot of photos
* Loading a photo's data is expensive

Note: More generally...

!!!

# Problems

* There are a lot of _things_
* Loading a _thing_ is expensive

!!!

## Holding Things

<img src="./img/holding-things.jpg" height=500 />

> http://odditymall.com/includes/content/wall-mounted-hands-for-holding-stuff-0.jpg

!!!

## Holding Things

* Arrays?

!!!

## Holding Things

```swift
let fetchresult = PHAsset.fetchAllAssets/*...*/()
```

```swift
var arr = [PHAsset]()
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
fetchresult.enumerateAssetsUsingBlock { obj, /*...*/
  arr.append(obj as! PHAsset)
}
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
// slowwwwww
```
<!-- .element: class="fragment" data-fragment-index="3" -->

Note: Go over each element, one at a time and insert the elements;; Solve the slowness...

!!!

## Holding Things

We need a `Collection`

!!!

### Groups of Things

* A `Collection` is an `Indexable` `Sequence`
* A `Sequence` is an `Iterator` maker
* An `Iterator` can produce one or more `Element`s

Note: Slowly!

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

Let's say we want to iterate over the even numbers

<img src="./img/2468.jpg" height=400 />

> https://i.ytimg.com/vi/A9NfHIIwNN4/maxresdefault.jpg

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

Note: This kinda sucks having to call next() with !

!!!

## Sequence

<img src="./img/recall.jpg" height=400 />

> http://ichef.bbci.co.uk/news/660/media/images/80295000/jpg/_80295405_thinking522738989.jpg

Recall: A `Sequence` is an `Iterator` maker

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

<img src="./img/questions.jpg" height=300 />

> http://customer-blog-images.s3.amazonaws.com/questions.jpg

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

<img src="./img/what-else.jpg" height=500 />

> http://www.thedirtondusters.com/wp-content/uploads/2012/08/WHAT-ELSE-NO-TEXT.jpg

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

* must be able to produce an _iterator_
* could be _one-pass_ or _multipass_
* could be _finite_ or _infinite_

!!!

## Collection

<img src="./img/recall.jpg" height=500 />

> http://ichef.bbci.co.uk/news/660/media/images/80295000/jpg/_80295405_thinking522738989.jpg

Recall: A `Collection` is an `Indexable` `Sequence`

!!!

## Collection

```swift
protocol Collection: Sequence, Indexable
```

!!!

## Collection

<img src="./img/photo-stack.jpg" height=500 />

> http://friendlycomputersspokane.com/wp-content/uploads/2014/06/photo-stack.jpg

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

`Why do we need to deal with the index crap?` <!-- .element: class="fragment" data-fragment-index="1" -->

Note: Point!

!!!

## Collection Rules

* has addressable positions (index)
* must support multipass iteration
* exists a one-to-one mapping of indices to elements
  * No infinite structures!
* expect O(1) subscript access

!!!

## Collection

* Non-integer indices are useful:

<img src="./img/linked-list.png" height=300 />

> https://upload.wikimedia.org/wikipedia/commons/thumb/d/d4/CPT-LinkedLists-deletingnode.svg/380px-CPT-LinkedLists-deletingnode.svg.png

!!!

## Collection

```swift
struct PhotosMetadata: Collection {
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

## Photos

```swift
let a = PhotosMetadata(PHAsset.fetchAllAssets/*...*/())
// not slow!
```

Note: Since it's a collection, we can transform everything

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

Note: In other words...

!!!

## Photos

* Delay operations on collections or sequences

!!!

## Lazy Groups of Things

<img src="./img/lazy.gif" height=500 />

> https://media.giphy.com/media/AyXMnDH4nA7jW/giphy.gif

Transformations computed when the information is _forced_ out

!!!

## LazySequence

`protocol LazySequenceProtocol: Sequence`

!!!

## LazyCollection

`protocol LazyCollectionProtocol:
        LazySequenceProtocol, Collection`

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

### Wait a second

`protocol LazyCollectionProtocol:
    LazySequenceProtocol, Collection`

!!!

#### How can an overriden method return different types?

!!!

### Protocol default implementations

<img src="./img/gears.png" height=500 />

> https://pptcrafter.files.wordpress.com/2013/01/gears-0.png

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

## Maps

```swift
Evens().lazy.map{ $0 + 1 }
    // LazyMapSequence<Int, Int>
// nothing is computed yet!
```

!!!

## LazyMapSequence?

```swift
struct LazyMapSequence<S: Sequence>: LazySequenceProtocol
```

Note: Sequence wrapped sequence

!!!

## LazyMapSequence

```swift
struct LazyMapSequence<S: Sequence>: LazySequenceProtocol {
  func makeIterator() -> LazyMapIterator</*...*/> { /*...*/ }
}
```

!!!

## LazyMapIterator

```swift
struct LazyMapIterator<Base: Iterator, Element>:
      IteratorProtocol, Sequence {
```

```swift
  var _base: Base
  let _transform: (Base.Element) -> Element
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
  mutating func next() -> Element? {
    return _base.next().map(_transform)
  }
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
}
```

!!!

## LazyMapSequence

```swift
struct LazyMapSequence<S: Sequence>: LazySequenceProtocol
```

<img src="./img/burrito.png" height=300 />

> http://www.tacobueno.com/media/1381/beefbob.png?quality=65

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

<img src="./img/flour-tortilla.jpg" height=500 />

> http://images.wisegeek.com/flour-tortilla.jpg

!!!

## Photos

```swift
let metadata: PhotosMetadata
let photos = metadata.lazy.map{ Photo(url: $0.url) }
// not slow!
```

!!!

## Photos

```swift
func prepareData(d: PhotosMetadata) -> ? {
  return d.lazy
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
      PHAsset, Photo
    >, PhotoView
  >
>
```

Note: Ahhhhhh

!!!

## Tortilla Inference

<img src="./img/magic-tortilla.jpg" height=500 />

> http://m.jumpstart.com/JumpstartNew/uploadedFiles/sne/small-screenshots/the-magic-tortilla.jpg

!!!

## Type Inference

```swift
let m = d.lazy.map/*...*/
return m.first
```

!!!

## Tortilla Erasure

<img src="./img/tortilla-erasure.jpg" height=500 />

> https://2e0a24317f4a9294563f-26c3b154822345d9dde0204930c49e9c.ssl.cf1.rackcdn.com/9218426_the-tortilla-pencil-case-deliciously-wraps_11d463da_m.jpg?bg=DFC1B7

!!!

## Type Erasure

```swift
let m = AnyCollection(
  d.lazy.filter{}.map{}/*...*/
)
// m: AnyCollection<PhotoView>
```

Note: Trade type information for maintainable code; see appendix for more info

!!!

## Type Erasure

* AnySequence
* AnyCollection
* ... see appendix for more

!!!

## Photos

```swift
func prepareData(d: PhotosMetadata) -> AnyCollection<PhotoView>
```

!!!

## Photos

Product wants `everyOther` photo

<img src="./img/every-other.jpg" height=500 />

> http://www.giftofcuriosity.com/wp-content/uploads/2016/04/Letter-hopscotch-4.jpg

!!!

## New Tortillas

```swift
let a = [1,2,3,4]
for x in a.lazy.everyOther() {
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
    if let _ = base.next() {
      return base.next()
    } else {
      return nil
    }
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

Product wants every _4th_ photo

<img src="./img/every-other.jpg" height=500 />

> http://www.giftofcuriosity.com/wp-content/uploads/2016/04/Letter-hopscotch-4.jpg

!!!

## Photos

```swift
let skipped = photos.everyOther().everyOther()
```

Note: Precisely why you WANT to use the Swift machinery when you can

!!!

## Recap

* <!-- .element: class="fragment" data-fragment-index="1" --> Collections hold our photo metadata and photos <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> LazyCollection's map transforms our data <!-- .element: class="fragment" data-fragment-index="2" -->
* <!-- .element: class="fragment" data-fragment-index="3" --> AnyCollection gives maintainable type signatures <!-- .element: class="fragment" data-fragment-index="3" -->
* <!-- .element: class="fragment" data-fragment-index="4" --> We can create new operators that compose <!-- .element: class="fragment" data-fragment-index="4" --> 

!!!

## Finally

Our app works!

<img src="./img/cards2.jpg" height=500 />

!!!

<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Grok it?

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

P.S. IBM Swift Sandbox is legit

P.P.S Check out Unary indexed collections in appendix

!!!

## Appendix

!!!

## Eager

Eager (or strict) is the opposite of lazy

Computed _always_

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

## Unary

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

## Collection-Index variants

How can you move to the next index?

* Collection (ForwardDirection)
  * Go forward in O(1)
* BidirectionalCollection
  * Go forward or backward in O(1)
* RandomAccessCollection
  * Get the difference between two indices in O(1)

!!!

## Type Erasure

!!!

## Type Erasure

Type erasure works by moving the type info to the constructor

!!!

## Type Erasure

```swift
// one of the inits for AnySequence
init<S : Sequence where S.Iterator.Element == Element,
     S.SubSequence : Sequence,
     S.SubSequence.Iterator.Element == Element,
     S.SubSequence.SubSequence == S.SubSequence>(_ base: S)
```

