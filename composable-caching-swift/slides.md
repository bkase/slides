<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Composable Caching in Swift

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

!!!

Story:

Complexity of client-side caching ->
Let's manage the complexity ->

Composability ->
Identity? (always fail) ->
Monoids! -> // life is amazing with monoids
Materialized? projections of a few true imperative caches ->
Contravariant Functors and Profunctors ->
    (as an exsercise try and prove the laws)
Legos ->

Transformations ons legos (for any legos) ->
Simple(r) client-side caching ->

Simpler?, you still need the imperative core! ->
Bugs interfacing with native libs ->
Bugs in the core of the library ->


!!!

## Photos

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

## Transforming Caches

!!!

### Transforming Caches

We need a network cache that gives us images not bytes!

!!!

### Transforming Caches

```swift
// bytesToImage: NSData -> JpegImage
let imageNetCache = netCache.mapValues(bytesToImage)
```

!!!

### How do we return a cache?

We want to define the operators on protocols not the concrete instances
So what do we return?

In Swift you cannot return some type with existentials (like our `Cache` protocol)

We need some type with no existential `associatedtype`s.

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
  func get(key: Key) -> Future<Value> { return getFn(key) }
  func set(key: Key, value: Value) -> Future<Unit> { return setFn(key, value) }
}
```
<!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### Type erasure

(maybe)

In Swift, type erasure is converting protocol constraints to a concrete struct full of lambdas.
Existential associated types become universally quantified generics.

Type erasure is usually used in Swift to simplify type signatures, but we can also use it to workaround this limitation in the language.
You erase the type information that you had about the specific cache and add a small runtime penalty of an extra function call

```swift
func simplify() -> LambdaCache<K, V> {
  return LambdaCache(getFn: self.get, setFn: self.set)
}
```

!!!

### Transforming caches

```swift
extension Cache {
  func mapValues<V2>(f: V -> V2) -> LambdaCache<K, V2> {
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

The `v` appears in covariant and contravariant positions so we need two transformation functions

!!!

### Transforming caches

```swift
extension Cache {
  func mapValues<V2>(f: V -> V2, _ fInv: V2 -> V) -> LambdaCache<K, V2> {
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

### Transforming caches

```swift
// bytesToImage: NSData -> JpegImage
// imageToBytes: JpegImage -> NSData
let imageNetCache = netCache.mapValues(bytesToImage, imageToBytes)
```

```swift
// SLOW!
```
<!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### Transforming caches

```swift
// bytesToImageAsync: NSData -> Future<JpegImage>
// imageToBytes: JpegImage -> Future<NSData>
let imageNetCache = netCache.asyncMapValues(bytesToImage, imageToBytes)

// asyncMapValues implementation similar to mapValues
```

!!!

### Key transformations

```swift
  func get(key: Key) -> Future<Value> { /* ... */ }
  func set(key: Key, value: Value) -> Future<Unit> { /* ... */ }
  // key is always in the contravariant input position
  // we only need an reverse transform
```

!!!

### Key transformations

```swift
// diskCache: DiskCache<String>
// urlToString: Url -> String
let urlCache = diskCache.mapKeys(urlToString)
```

!!!

### Transforming caches

Note that these transformed caches are _virtual_. They provide different _projections_ onto the same underlying cache.

!!!

## Reusing inflight requests

```swift
extension Cache where K: Hashable {
  func reuseInflight(dict: [K: Future<V>]) -> BasicCache<K, V> {
    return new BasicCache {
      get: { k in dict[k] ?? (let f = self.get(k); dict[k] = f; f) }
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
// We need something that let's us apply 
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
            other.get(k).map{ v in
              self.set(k, v)
              return v
            }
          }
      }
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
      set: { k, v in Future.zip(self.set(k, v), other.set(k, v)) }
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
let safeCache = (netCache.onTopOf(diskCache)).reuseInflight(dict)
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

Associative binary operator + an identity element = Monoid

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
* Or `videoCache` <!-- .element: class="fragment" data-fragment-index="1" -->
* Or `jsonModelCache` <!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### Monoidal whatever

The ability to hide the "history" of the construction anywhere enables the developer to put the information wherever it may make sense to a reader.

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

### Networking

Network cache `get` failed to complete a future in one branch of an error case

(image)

!!!

### Networking

The networking cache keeps track of how many requests are inflight to not oversaturate the bandwidth for higher priority things

!!!

### Networking

The network cache eventually becomes unresponsive and requests hang forever!

!!!

### Real problems

A good abstraction doesn't save you from the concrete cruft underneath

!!!

## Real problems

!!!

### Library bugs

Don't blidly trust code you have never seen, esp. with few consumers

!!!

### Aside: Swift's memmory model

Swift uses atomic reference counting instead of garbage collection

!!!

### Aside: Swift's memmory model

* A garbage collector can reclaim reference cycles
* Reference counting cannot

!!!

### Aside: Swift's memmory model

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

### Aside: Swift's memmory model

```swift
class Node { weak var next: Node }
```

```swift
let a = Node();
let b = Node();
a.next = b;
a.next?.next = a;
// a.next *could* be nil
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

* A clean abstraction seems to imply an illusion that everything will work
* But the abstraction abstracts over something! <!-- .element: class="fragment" data-fragment-index="1" -->
* Composition hides the provenance of some thing, but sometimes that provenance is where you need to look! <!-- .element: class="fragment" data-fragment-index="2" -->

!!!

## Caching

!!!

### Overall win

The app worked. The caching worked.



(extra)

## Similarities between our get / set

The calls inside `get` and `set` need to be in the same order

* `checkRam` -> `setRam`
* `checkDisk` -> `setDisk`

!!!

More generally...

Take two things of some type `T` and make one thing that same type `T`.

Implies => The shape of one thing informs the other

!!!



!!!


## TODO

!!!

[Purescript implementation?](https://github.com/bkase/purescript-cache/blob/master/src/Main.purs)

!!!

<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

Slide Deck: (todo insert link)

