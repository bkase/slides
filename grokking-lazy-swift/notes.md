generic version is:

Start with the photo problem

You have a lot of photos
and computing the photo is expensive (reading the metadata)
so you only want to compute the photos you'll use

You have a lot of X things
and computing X is expensive
so you only want to compute the Xs you'll use

So we need to hold the things, and to delay the computation of the work within it.

Let's talk about how we have lots of things in swift

Arrays? But that's not abstract enough.

In Swift we can use Sequences and Collections.

Go into sequences

Then go into collections

And then we go back to the photos example
How do we delay the computation of the expensive things?

That's called Laziness

LazySequence, LazyCollection
Burritos...

How do we transform photo metadata into photos

This is how transform lazy collection/sequence (this is what we do to the photos)

(show the code for LazyMapSequence)

How does map exist on sequences and collections and compile?

We solved transformation by wrapping things in a struct, but now that we used tortillas OUR

New Problem: Function signatures become unmaintainable
Solution: Use type erasure

Let's take every other photo






* ORDER of slides
* put simpler iterator example first
* go more into the burrito
* remove the burrito slides
* go through the code line by line-ish
* use the UIView thing the whole time
* pictures
* problem: extension function doesn't work, shorts photos.   solution: use the machinery for lazy stuff
* make the benefits CLEAR
* heirarchy of headers more (clear asides)
* kill the indexes slide
* kill unary (even though I love it)
* add the other method to A
* more the difference between collection and sequence
* add complicated thing to the end of the slide (anticipate things)
* The REAL reason: you want to use map and filter and don't want to load the data that backs your views (since the metadata access is expensive)









. * Why can't I just override the subscript operator
. * Give full example of access

. ```swift
let u: Unary</*...*/>
u[""] // 1st element
u["x"] // 2st element
u["xx"] // 3st element
u["xxx"] // 4st element
```

. * Move the aside to where it makes sense
. * Evens is sort-of inherently lazy, so show a linked list
. * Any sequence that is merely an iterator should be lazy
. * Figure out the burritos
* Go into erasure more
* Why would you want to use the more specific erases
     Maybe take it out
. * Runtime of the indexing is the real different
. * LazySequence vs Sequence is about the operators being lazy or not


## Lazy Transforms
Brandon Kase

!!!


Plan:

1. To grok means you really need to understand
2. To understand how lazy sequences and collections, you need to understand a bunch of other stuff
3. I'm going to assume you are comfortable with protocols, and assume you are NOT comfortable with the internals of Swift's std library
4. When I say Swift's std library, I mean the latest Swift 3 beta's std library because things have changed
5. First of all, what does it mean to be lazy (vs eager)
6. Let's talk about eager things first
7. IteratorProtocol
* just a `mutating next()`
* one-pass only (how can you go backwards?)
* example?
8. Sequence
* no contract of one-pass or multipass
* must be able to produce an iterator (could be self)
* example? (same as iterator one and a stable one)
9. Collection
* has addressable positions (index) -- address doesn't have to be an int
* must support multipass
* must have a finite range of indices (has to have an endIndex and a count) corresponding exactly to each element -- no infinite structures
* expect O(1) subscript access
* the index type determines expected runtime for count (RandomAccessIndex has O(1), ForwardDirection or Bidirection has O(N)) -- but we're not going to go more into that (talk to me after if you're interested)
Notably this is a large change between Swift 2 and 3 here
10. Unary collection (wrapping)
11. This wrapping intuition will help you understand how lazy transformations work
12.
LazySequenceProtocol inherits from Sequence
LazyCollectionProtocol inherits from Collection
but specializes strict operations like `map` and `filter` to be lazy
How does it do this?
(1..<4).lazy.map{ x in x + 1 }
`LazyMapRandomAccessCollection<CountableRange<Int>, Int>(_base: CountableRange(1..<4), _transform: (Function))`
What is a LazyMapRandomAccessCollection?

It looks a lot like Unary
13. 
When you're using this stuff, you need the type inference to manage it because of all the associated types with the protocols
but sometimes you do need to have the type annotation (like on a func return value)
You can use type-erasure to help deal with this
Any*
Swift stdlibrary includes type-erased collections and sequences for Forward, Bidirectional, and Random acess variants

14.
Couldn't find documentation on protocol method precedence, but through experimentation, I've determined that protocol inheritance sets method precendence. A subprotocol? can "override" an extension method in the parent protocol EVEN changing it's return type (see appendix for proof)

This is how map returns a `LazyMapSequence` for sequences and a `LazyMapRandomAccessCollection` on random access collections 


Appendix. Indices
* Collection Indexable (forward index aka singly-linked-list)
* BidrectionalCollection contains s.t. BidrectionalIndexable (forward or back aka doubly-linked-list
  has efficient last/reverse()
* RandomAccessCollection contrains s.t. RandomAccessIndexable (random access aka array)

Appendix. Protocol method overrides
```swift
//  Write some awesome Swift code, or import libraries like "Foundation",
//  "Dispatch", or "Glibc"

protocol A {}
extension A {
    func hi() -> String {
        return "A"
    }
}
protocol B: A {}
extension B {
    func hi() -> Int {
        return 5
    }
}

struct X: A {}
struct Y: B {}

print(X().hi())
print(Y().hi())
```



# Overview

example for each

IteratorProtocol
* just a `mutating next()`
* one-pass only (how can you go backwards?)

Sequence
* no contract of one-pass or multipass
* must be able to produce an iterator (could be self)

Collection
* has addressable positions (index) -- address doesn't have to be an int
* must support multipass
* must have a finite range of indices (has to have an endIndex and a count) corresponding exactly to each element
* expect O(1) subscript access
* the index type determines expected runtime for count (RandomAccessIndex has O(1), ForwardDirection or Bidirection has O(N))

In Swift 3 index advancement is part of the collection

Indices
* Collection Indexable (forward index aka singly-linked-list)
* BidrectionalCollection contains s.t. BidrectionalIndexable (forward or back aka doubly-linked-list
  has efficient last/reverse()
* RandomAccessCollection contrains s.t. RandomAccessIndexable (random access aka array)

https://github.com/apple/swift/blob/master/docs/SequencesAndCollections.rst


LazySequenceProtocol inherits from Sequence

but specializes strict operations like `map` and `filter` to be lazy

How does it do this?

(1..<4).lazy.map{ x in x + 1 }

`LazyMapRandomAccessCollection<CountableRange<Int>, Int>(_base: CountableRange(1..<4), _transform: (Function))`

What is a LazyMapRandomAccessCollection?


These types get crazy fast. How to deal with them across functions? Type erasure!
Swift stdlibrary includes type-erased collections and sequences for Forward, Bidirectional, and Random acess variants


http://swiftdoc.org/v3.0/type/LazyMapRandomAccessCollection/



While a sequence may be consumed as it is traversed, a collection is guaranteed to be multi-pass: Any element may be repeatedly accessed by saving its index. 

Collections don't have to be indexed by integers!
Here's a lazy random access collection indexed by a unary number denoted by repeated `"x"`s

```swift
struct SampleCollection: LazyCollectionProtocol, RandomAccessCollection {
  // you need this so Swift knows you want random and not bidirectional access
  typealias Indices = DefaultRandomAccessIndices<SampleCollection>

  let xs = ["a","b","c"]

  var startIndex: String {
    return ""
  }

  var endIndex: String {
    return "xxx"
  }

  func index(after i: String) -> String {
    return i + "x"
  }

  func index(before i: String) -> String {
    return String((0..<(i.characters.count-1)).map{ _ in "x" })
  }

  subscript(i: String) -> String {
    return xs[i.characters.count]
  }
}
```

You can even transform it and still index by the unary number!

```swift
print(SampleCollection().map{ x in x + "!" }["xx"])
```

Let's make a collection that wraps collections

```swift
struct Unary<C: Collection>: Collection 
    where C.Index == Int, 
          C.IndexDistance == Int {

    private let xs: C
    
    init(_ xs: C) {
        self.xs = xs
    }
    
    var startIndex: String {
        return ""
    }
    
    /// O(n) endIndex
    var endIndex: String {
        return String((0...xs.count).map{ _ in "x"})
    }
    
    func index(after i: String) -> String {
        return i + "x"
    }

    subscript(i: String) -> C.Iterator.Element {
        return xs[i.characters.count]
    }
}

let u: Unary<Array<Int>> = Unary([1, 2, 3, 4, 5, 6])
print(u.lazy.map{ x in x + 1 }["xxx"])
```

Randomize fast and randomize slow


Understanding Lazy Sequences and Collections

When we code, we love to abstract over same-shaped things. In Swift, we do so with sequences and collections, and transform these structures using operators like map, reduce, and filter. Sometimes we want to delay execution of our element transformations until we actually look at those elements -- if such transformations are in some way computationally expensive for example. In this talk, you will learn what makes lazy sequences and collections tick, how to define new operators for them, and when you should consider using lazy sequences and collections in your code.

Lazy sequences and collections are protocols that delay transformations until access. 




I found this library while I was preparing for this talk https://github.com/oisdk/SwiftSequence

