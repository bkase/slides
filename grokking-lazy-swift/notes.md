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

