<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Composable Caching in Swift

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

!!!

## Photos

(image)

Note: We want to have an app with photos in it

!!!

## Photos

(image)

Note: Bandwidth, data caps, battery, coverage, etc

!!!

## Caching

(image)

Note: Ad-hoc caching logic makes code messy fast

!!!

## Image Cache

```swift
class ImageCache {
```

```swift
  func get(url: Url) -> Future<JpegImage>
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
  func set(url: Url, image: JpegImage) -> Future<Unit>
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
}
```

!!!

## Image Cache (get)

```swift
func get(url: Url) -> Future<JpegImage> {
  return checkRam(url).orElse{
    checkDisk(url)
  }.orElse{
    hitNetwork(url)
  }
}
```

!!!

## Image Cache (set)

```swift
func set(url: Url, image: JpegImage) -> Future<Unit> {
  return Futures.join(
    setRam(url, image),
    setDisk(url, image)
  )
}
```

Note: Okay so this sort of makes sense

!!!

## Image Cache

This isn't what your real code will look like though...

!!!

## Production Image Caches

We need to know the progress of things as they download

(image)

!!!

## Production Image Caches

We need to reuse inflight requests, instead of loading some asset twice

(image -- like avatars in a comment thread)

!!!

## Production ~~Image~~ Video Caches

We need to support video!

(image)

!!!

## Production ~~Image~~ Everything Caches

And don't we need to cache our model data?

Do we need to re-build everything again?

(image)

!!!

## Abstraction

Okay, let's try again

!!!

## Whatever Cache

```swift
class Cache<Key, Value>
```

Note: Then we'll subclass or something?

!!!

## Protocol-oriented Design

```swift
protocol Cache {
```

```swift
  associatedtype Key
  associatedtype Value
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
  func get(key: Key) -> Future<Value>
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
  func set(key: Key, value: Value) -> Future<Unit>
```
<!-- .element: class="fragment" data-fragment-index="3" -->

```swift
}
```

Note: Don't wince at the side-effects. (unrelated) Also, you can do something similar in Rust

!!!

## Instance of a cache

```swift
class RamCache<K, V>: Cache where K: Hashable {
  typealias Key = K
  typealias Value = V
```

```swift
  func get(key: Key) -> Future<Value> { /* ... */ }
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
  func set(key: Key, value: Value) -> Future<Unit> { /* ... */ }
```
<!-- .element: class="fragment" data-fragment-index="3" -->

```swift
}
```

!!!

## Other Instances

```swift
// hash string key
class DiskCache<K>: Cache where K: StringConvertible {
  typealias Key = K
  typealias V = NSData // byte array
}
```

```swift
// does the photo exist in the camera roll on the device (no-op set)
class LocalPhotoKitCache: Cache {
  typealias Key = LocalPhotoKey
  typealias Value = UIImage
}
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
// always hit the network (no-op set)
class NetworkCache<K>: Cache where K: URLConvertible {
  typealias Key = K
  typealias Value = NSData // byte array
}
```
<!-- .element: class="fragment" data-fragment-index="2" -->

!!!

## Not quite there yet

We need a network cache that gives us images not bytes!

!!!

## Transforming Caches

!!!

### Transforming Caches

```swift
// bytesToImage: NSData -> JpegImage
let imageNetCache = netCache.mapValues(bytesToImage)
```

!!!

### Not so simple

* We want to define the _operators_ on _protocols_ not the concrete instances
* In Swift you _cannot_ return some type with _existentials_ (like our `Cache` protocol)
* We need some type _without_ an existential `associatedtype`.

Note: We need some type with no existential `associatedtype`s.

!!!

### Lambda cache

```swift
struct LambdaCache<K, V>: Cache {
  typealias Key = K
  typealias Value = V
```

```swift
  let getFn: K -> Future<V>
  let setFn: (K, V) -> Future<Unit>
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
  func get(key: Key) -> Future<Value> {
    return getFn(key)
  }
  func set(key: Key, value: Value) -> Future<Unit> {
    return setFn(key, value)
  }
}
```
<!-- .element: class="fragment" data-fragment-index="2" -->

Note: Type erasure

!!!

### MapValues

(image)

Note: Let's try to build it

!!!

### Transforming caches

```swift
extension Cache {
  func mapValues<V2>(f: Value -> V2) -> LambdaCache<K, V2> {
```

```swift
    return new LambdaCache(
      get: { k in self.get(k).map(f) }
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
      set: { k, v in self.set(k, ???) }
    )
  }
}
```
<!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### Transforming caches

The `v` appears in _covariant_ and _contravariant_ positions

We need two transformation functions

!!!

### Transforming caches

```swift
extension Cache {
  func mapValues<V2>(
      f: V -> V2,
      _ fInv: V2 -> V) -> LambdaCache<K, V2> {
```

```swift
    return new LambdaCache(
      get: { k in self.get(k).map(f) }
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
      set: { k, v in self.set(k, fInv(v)) }
    )
  }
}
```
<!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### Profunctor w.r.t. values

* Because we need both transformations
* Caches are _profunctors_ w.r.t. values

Note: Plus some other things you need to prove it that I'm hand waving

!!!

### Transforming caches

```swift
// bytesToImage: NSData -> JpegImage
// imageToBytes: JpegImage -> NSData
let imageNetCache = 
    netCache.mapValues(bytesToImage, imageToBytes)
```

```swift
// SLOW!
```
<!-- .element: class="fragment" data-fragment-index="1" -->

Note: Ok cool, now we can get images… but it's slow

!!!

### Transforming caches

```swift
// bytesToImageAsync: NSData -> Future<JpegImage>
// imageToBytes: JpegImage -> Future<NSData>
let imageNetCache =
    netCache.asyncMapValues(bytesToImage, imageToBytes)

// asyncMapValues implementation similar to mapValues
```

!!!

### Key transformations

```swift
  func get(key: Key) -> Future<Value> { /* ... */ }
  func set(key: Key, value: Value) -> Future<Unit> { /* ... */ }
  // key is always in the contravariant input position
  // we only need an inverse transform
```

!!!

### Contravariant-functors w.r.t. keys

* Because we only need the inverse transform
* Caches are _contravariant functors_ w.r.t. values

Note: Plus some other things you need to prove it that I'm hand waving

!!!

### Key transformations

```swift
// diskCache: DiskCache<String>
// urlToString: Url -> String
let urlCache = diskCache.mapKeys(urlToString)
```

!!!

### Transforming caches

Note that these transformed caches are _virtual_.

They provide different _projections_ onto the same underlying cache.

!!!

## Reusing inflight requests

!!!

### Reusing inflight requests

```swift
extension Cache where K: Hashable {
  func reuseInflight(
      dict: [K: Future<V>]) -> LambdaCache<K, V> {
    return new LambdaCache {
      get: { k in dict[k] ??
              (let f = self.get(k); dict[k] = f; f) }
      set: { (k, v) in self.set(k, value: v) }
    }
  }
  // logic for freeing the memory elided
}
```

!!!

## Reusing inflight requests

```swift
let smartNetworkCache = networkCache.reuseInflight(dict)
// still conforms to Cache<Key=Url,Value=NSData>
```

```swift
let f = smartNetworkCache.get(url)
let f2 = smartNetworkCache.get(url)
// only one real network request!
// same reference f === f2
```
<!-- .element: class="fragment" data-fragment-index="1" -->

!!!

## Better request reuse

How do we reuse inflight requests across _disk_ and _network_?

Note: aka if we're in either state, and it's re-requested, we don't want to dispatch a new request

!!!

### Attempt1: multi-step request reuse

```swift
// Note: Don't do this
let dict = [Url: Future<NSData>]
let smartNet = netCache.reuseInflight(dict)
let smartDisk = diskCache.reuseInflight(dict)
```

!!!

### Attempt1: multi-step request reuse

```swift
// Note: Don't do this
func loadDataSafely(url: Url) -> Future<NSData> {
  return smartDisk(url).orElse {
    smartNetwork(url)
  }
}
```

!!!

### Attempt1: multi-step request reuse

```swift
// Note: Don't do this
func loadDataSafely(url: Url) -> Future<NSData> {
  return smartDisk(url).orElse {
    // <-- what if the new request comes here
    smartNetwork(url)
  }
}
```

Note: We wouldn't reuse the request

!!!

### Multi-step request reuse

```swift
// We need something that lets us apply 
// that cache combinator to both at the same time
let safeCache = (netCache and diskCache).reuseInflight(dict)
```

!!!

## Cache Layering

!!!

### Cache Layering

Consider A `on-top-of` B

(diagram)

!!!

### Cache Layering

(same diagram with a circle around it)

!!!

### Cache layering

```swift
extension Cache {
  func onTopOf<K, V, B: Cache>(other: B) -> LambdaCache<K, V>
      where Self.Key == B.Key, B.Key == K,
            Self.Value == B.Value, B.Value == V {
```

```swift
    return LambdaCache(
      get: { k in 
          self.get(k).catchError{ e in
            other.get(k).map{ v in self.set(k, v); return v }
          } }
      }
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
      set: { k, v in
         Future.join(self.set(k, v), other.set(k, v)) }
  }
}
```
<!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### Cache layering

```swift
let c = a.onTopOf(b)
// c is a cache!
```

!!!

### Cache layering

```swift
let safeCache =
    netCache.onTopOf(diskCache)
          .reuseInflight(dict)
// solved!
```

!!!

### Associative -- Cache layering

(diagram)

Note: Proof left as excersise to reader

!!!

### Identity -- Cache layering

(diagram)

Note: Informal proof?

!!!

### Cache layering

Associative binary operator + an identity element = _Monoid_

!!!

### Monoidal Caching

```swift

let imageCache = fold(
  ramCache,
  diskCache,
  networkCache
)
```

!!!

### Monoidal Caching

We're able to forget the provenance of the cache's construction

(image)

!!!

### Monoidal Caching

* In our application logic we only need to care about `imageCache` now!
* <!-- .element: class="fragment" data-fragment-index="1" --> Or <!-- .element: class="fragment" data-fragment-index="1" --> `videoCache` <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> Or <!-- .element: class="fragment" data-fragment-index="2" --> `jsonModelCache` <!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### Monoidal whatever

The ability to _hide "history"_ of the construction anywhere enables the developer to put the information wherever it may make sense to a reader.

(image)

!!!

### Composable Caches

Monoid is math speak for composable

!!!

### Caching is now simple

(image)

!!!

### Caching is now simple?

(image)

!!!

### Caching core

The application logic is great now!

But the core is still imperative and messy.

!!!

## Real Problems

!!!

### Network Cache

Imperative implementation to interface with native iOS APIs

!!!

### Networking

Network cache `get` failed to complete a future in one error case

(image)

!!!

### Networking

The networking cache caps inflight requests

So high priority requests can take over

!!!

### Networking

The network cache eventually becomes unresponsive and requests hang forever!

!!!

### Real problems

A good abstraction doesn't save you from the concrete cruft underneath

!!!

## Real problems

!!!

## Library Bugs

!!!

### Aside: "Carlos" is awesome

* [Carlos](https://github.com/WeltN24/Carlos) is the Swift cache library that implements what we've described
* Vittorio Monaco, creator of Carlos, came up with these ideas
* My contribution was formalizing the cache layering (as a monoid) and some pull requests

Note: Something about being grateful for this awesome library

!!!

### Library bugs

Bugs can go _unnoticed_ if not many are using a library

Let's talk about a _memory leak_

!!!

### Aside: Swift's memory model

Swift uses _atomic reference counting_ instead of garbage collection

!!!

### Aside: Swift's memory model

* A garbage collector can reclaim reference cycles
* Reference counting cannot

!!!

### Aside: Swift's memory model

```swift
class Node { var next: Node }
```

```swift
let a = Node();
a.next = Node();
a.next.next = a;
// cycle!
```
<!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### Aside: Swift's memory model

```swift
class Node { weak var next: Node }
```

```swift
let a = Node();
let b = Node();
a.next = b;
a.next?.next = a;
// a.next *could* be nil (weak ref)
// no cycle!
```
<!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### Core future library

```swift
// Inside Promise.swift (the mutable builder of a future)
```

```swift
/// The Future associated to this Promise
public lazy var future: Future<T> = {
  return Future(promise: self)
}()
```
<!-- .element: class="fragment" data-fragment-index="1" -->

Note: See the bug?

!!!

### Core future library

```swift
public lazy weak var future: Future<T> /*...*/
```

!!!

### Leaks

What did this leak mean?

!!!

### Leaks

Every single future created would retain a strong reference to the data

!!!

### Leaks

`Future<JpegImage>`

(image)

!!!

### Illusion of correctness

* Clean abstractions are provide an _illusion_ that everything is nice
* <!-- .element: class="fragment" data-fragment-index="1" --> But the abstraction abstracts over something! <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> Composition _hides_ the provenance, but sometimes that provenance is _exactly_ where you need to look! <!-- .element: class="fragment" data-fragment-index="2" -->

!!!

## Caching

!!!

### Overall win

The app worked. The caching worked.

!!!

### Purescript Implementation

[Purescript implementation](https://github.com/bkase/purescript-cache/blob/master/src/Main.purs) to help formalize these ideas

!!!

<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

Slide Deck: (todo insert link)

Thanks to [Vittorio Monaco](https://twitter.com/Vittorio_Monaco) for making the [Carlos](https://github.com/WeltN24/Carlos) library


!!!

## Appendix

!!!

### Type erasure

In Swift, type erasure is converting protocol constraints to a concrete struct full of lambdas.
Existential associated types become universally quantified generics.

Type erasure is usually used in Swift to simplify type signatures, but we can also use it to workaround this limitation in the language.
You erase the type information that you had about the specific cache and add a small runtime penalty of an extra function call

```swift
func simplify() -> LambdaCache<K, V> {
  return LambdaCache(getFn: self.get, setFn: self.set)
}
```

