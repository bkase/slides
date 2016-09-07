<!-- .slide: data-background="#DAA852" -->
<!-- .slide: data-state="terminal" -->

# Composable Caching in Swift

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

!!!

### Photos

![photo](img/photos2.jpg)

> https://pixabay.com/p-1150076

Note: We want to have an app with photos in it

!!!

### Photos

Review these numbers

| Action                   | Milliseconds |
| ------------------------ | ------- |
| Downloading over 3g | 5000 |
| Reading from disk | 500 |
| Reading from memory | 1 |

!!!

### Load an image

1. Does this url exist in my hashmap (url to bitmap)?
2. If so, return the bitmap <!-- .element: class="fragment" data-fragment-index="1" -->
3. If not, does the jpeg exist on disk? <!-- .element: class="fragment" data-fragment-index="2" -->
4. If so, convert it to a bitmap, save it to the hashmap, return it <!-- .element: class="fragment" data-fragment-index="3" -->
5. If not, does a network request for the jpeg succeed? <!-- .element: class="fragment" data-fragment-index="4" -->
6. If so, store it to disk, convert it to a bitmap, save it to the hashmap, return it <!-- .element: class="fragment" data-fragment-index="5" -->
7. If not, fail <!-- .element: class="fragment" data-fragment-index="6" -->

Note: Ad-hoc caching logic makes code messy fast

!!!

### Image Cache

```swift
class ImageCache {
```

```swift
  func get(url: Url) -> Future<UIImage>
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
  func set(url: Url, image: UIImage) -> Future<Unit>
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
}
```

!!!

### Image Cache (get)

```swift
func get(url: Url) -> Future<UIImage> {
  return checkRam(url).orElse{
    checkDiskAndFallbackToRam(url)
  }.orElse{
    hitNetworkAndFallbackToAll(url)
  }
}
```

Note: We're shoving a lot of domain-specific details into our helper functions here, but this should work

!!!

### Image Cache (set)

```swift
func set(url: Url, image: UIImage) -> Future<Unit> {
  return Futures.join(
    setRam(url, image),
    setDisk(url, image)
  )
}
```

Note: Okay so this sort of makes sense

!!!

### Image Cache

This isn't what your real code will look like though...

!!!

### Production Image Caches

1. Does this url exist in my hashmap (url to bitmap)?
2. If so, return the bitmap
3. If not, _lookup url in a hashmap to reuse inflight request if possible_ 
4. Does the jpeg exist on disk?
4. If so, convert it to a bitmap, save it to the hashmap, _remove from inflight-map_, return it
5. If not, does a network request for the jpeg succeed?
6. If so, store it to disk, convert it to a bitmap, save it to the hashmap, _remove from inflight-map_, return it
7. If not, fail and _remove from inflight-map_

Note: But then your PM says we need videos!

!!!

### Production ~~Image~~ Video Caches

1. Does this url exist in my hashmap (url to _video data_)?
2. If so, return the _video data_
3. If not, lookup url in a hashmap to reuse inflight request if possible
4. Does the _mp4_ exist on disk?
4. If so, _read the video data_, save it to the hashmap, remove from inflight-map, return it
5. If not, does a network request for the _mp4_ succeed?
6. If so, store it to disk, _read h264 frames_, save it to the hashmap, remove from inflight-map, return it
7. If not, fail

!!!

### Production ~~Image~~ Everything Caches

1. Does this url exist in my hashmap (url to _user profile_)?
2. If so, return the _profile_
3. If not, lookup url in a hashmap to reuse inflight request if possible
4. Does the _json data_ exist on disk?
4. If so, _parse the data_, save _the user profile_ to the hashmap, remove from inflight-map, return it
5. If not, does a network request for the _json data_ succeed?
6. If so, store it to disk, _parse the data_, save it to the hashmap, remove from inflight-map, return it
7. If not, fail

Note: And don't we need to cache our model data?; Do we need to re-build everything again?

!!!

## Abstraction

![abstract](img/abstract.jpg)

> https://pixabay.com/p-1066182

Note: Okay, let's try again

!!!

### Whatever Cache

```swift
class Cache<Key, Value>
```

Note: Then we'll subclass or something?

!!!

### Protocol-oriented Design

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

Note: Blueprint that can be adopted

!!!

### Aside: Carlos Caching

* [Carlos](https://github.com/WeltN24/Carlos) is an open source Swift library
* Created by [@Vittorio_Monaco](https://twitter.com/Vittorio_Monaco) and [@esad](https://twitter.com/esad)
* My contribution is some formalization and a few pull-requests

!!!

### Instance of a cache

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

### Other Instances

```swift
// md5 string key
class DiskCache<K>: Cache where K: StringConvertible {
  typealias Key = K
  typealias Value = NSData // byte array
}
```

```swift
// does the photo exist in the camera roll (no-op set)
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

### Not quite there yet

![thinking](img/thinking2.jpg)

> https://upload.wikimedia.org/wikipedia/commons/8/81/A_woman_thinking.jpg

Note: We need a network cache that gives us images not bytes!

!!!

## Transforming Caches

!!!

### Transforming Caches

```swift
// bytesToImage: NSData -> UIImage
// netCache: Cache<Value=NSData>
let imageNetCache: Cache<Value=UIImage> =
    netCache.mapValues(bytesToImage)
```

!!!

### MapValues

![mapvalues](img/mapvalues.png)

Note: Let's try to build it

!!!

### Transforming caches

```swift
extension Cache {
  func mapValues<V2>(f: Value -> V2) -> BasicCache<K, V2> {
```

```swift
    return new BasicCache(
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

### MapValues

![mapvalues2](img/mapvalues2.png)

!!!

### Transforming caches

* The `v` appears in _covariant output_ and _contravariant input_ positions
* We need two transformation functions
* Caches are _profunctors_ w.r.t. values

Note: Plus some other things you need to prove it that I'm hand waving

!!!

### Transforming caches

```swift
extension Cache {
  func mapValues<V2>(
      f: V -> V2,
      _ fInv: V2 -> V) -> BasicCache<K, V2> {
```

```swift
    return new BasicCache(
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
// bytesToImage: NSData -> UIImage
// imageToBytes: UIImage -> NSData
let imageNetCache: Cache<Value=UIImage> = 
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
// bytesToImageAsync: NSData -> Future<UIImage>
// imageToBytes: UIImage -> Future<NSData>
let imageNetCache: Cache<Value=UIImage> =
    netCache.asyncMapValues(
        bytesToImageAsync,
        imageToBytesAsync
    )

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

Note: Caches are contravariant functors w.r.t. the keys

!!!

### Map Keys

![mapkeys](img/mapkeys.png)

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

### Virtual caches

![virtualcache](img/virtualcache.png)

!!!

### Caches

* Network cache now returns images if necessary
* What about re-using inflight requests?

!!!

## Reusing inflight requests

!!!

### Reusing inflight requests

```swift
extension Cache where K: Hashable {
  func reuseInflight(
      dict: [K: Future<V>]) -> BasicCache<K, V> {
```

```swift
    return new BasicCache {
      get: { k in dict[k] ??
              (let f = self.get(k); dict[k] = f; f) }
      set: { (k, v) in self.set(k, value: v) }
    }
  }
  // logic for freeing the memory elided
}
```
<!-- .element: class="fragment" data-fragment-index="1" -->

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
  return smartDisk.get(url).orElse {
    smartNetwork.get(url)
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
let safeCache = (diskCache and netCache).reuseInflight(dict)
```

!!!

## Cache Layering

!!!

### Cache Layering

![layers](img/layers.png)

Note: Consider A on-top-of B


!!!

### Cache Layering

![layers2](img/layers2.png)

!!!

### Cache layering

```swift
extension Cache {
  func onTopOf<B: Cache>(other: B) ->
          BasicCache<Self.Key, Self.Value>
      where Self.Key == B.Key
            Self.Value == B.Value {
```

```swift
    return BasicCache(
      get: { k in 
          self.get(k).orElse{
            other.get(k)
              .map{ v in self.set(k, v); return v }
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
    diskCache.onTopOf(netCache)
          .reuseInflight(dict)
// solved!
```

!!!

### Associative -- Cache layering

![assoc](img/assoc.png)

Note: Proof left as excersise to reader

!!!

### Identity -- Cache layering

![identity](img/identity2.png)

Note: Informal proof?

!!!

### Cache layering

_Associative binary_ operator + an _identity_ element = _Monoid_

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

![noprovenance](img/noprovenance.png)

Note: We're able to forget the provenance of the cache's construction

(image)

!!!

### Monoidal Caching

* In our application, we only need to care about `imageCache`!
* <!-- .element: class="fragment" data-fragment-index="1" --> Or <!-- .element: class="fragment" data-fragment-index="1" --> `videoCache` <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> Or <!-- .element: class="fragment" data-fragment-index="2" --> `jsonModelCache` <!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### Caching is now simple

```swift
let diskAndNet =
    (diskCache <> netCache).reuseInflight()
let diskAndNetJpeg =
    diskAndNet.mapValues(bytesToImg, imgToBytes)
return ramCache <> diskAndNetJpeg
```

!!!

### Caching is now simple?

![whatcore](img/whatcore.png)

Note: Application logic great! The core is imperative and potentially messy

!!!

## Real Problem #1

#### Imperative Core

!!!

### Network Cache

Imperative implementation to interface with native iOS APIs

!!!

### Networking

```swift
if {
  /* … */
} else if {
  // forgot to complete promise!!
  /* … */
} else if {
  /* … */
} /* … */
```

Note: Forgot to complete a promise (mutable builder of future) in a branch

!!!

### Networking

* The networking cache caps inflight requests
* So high priority requests can take over

!!!

### Networking

![freezingrain](img/freezingrain2.jpg)

> https://upload.wikimedia.org/wikipedia/commons/0/0c/Freezing_Rain_on_Tree_Branch.jpg

Note: The network cache eventually becomes unresponsive and requests hang forever!

!!!

### Real problems

A good abstraction doesn't save you from cruft underneath

!!!

## Real problem #2

#### Library bugs

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
// Inside Promise.swift
// (the mutable builder of a future)
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

![deadbattery](img/deadbattery2.png)

> https://pixabay.com/p-1623377/

Note: What did this leak mean?

!!!

### Leaks

Every future created retained a strong reference to the data

!!!

### Leak UIImage

![bigpoint](img/bigpoint2.png)

!!!

### Illusion of correctness

* Clean abstractions provide an _illusion_ that everything is nice
* <!-- .element: class="fragment" data-fragment-index="1" --> When using black box `.magic()`, you can't assume perfection <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> Composition _hides_ the provenance, but sometimes that provenance is _exactly_ where you need to look! <!-- .element: class="fragment" data-fragment-index="2" -->

Note: Now that we've fixed the bugs...

!!!

## Overall win

!!!

### Overall win

1. Build virtual cache out of primitives
2. Call _get_ on the appropriate cache

Note: That's it. Not seven steps

!!!

### Purescript Implementation

[Purescript implementation](https://github.com/bkase/purescript-cache/blob/master/src/Main.purs) created to help formalize these ideas

!!!

<!-- .slide: data-background="#DAA852" -->
<!-- .slide: data-state="terminal" -->

# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

Slide Deck: (todo insert link)

Thanks to [Vittorio Monaco](https://twitter.com/Vittorio_Monaco) for making the [Carlos](https://github.com/WeltN24/Carlos) library

!!!

## Appendix

!!!

### Not so simple

* We want to define the _operators_ on _protocols_ not the concrete instances
* In Swift you _cannot_ return some type with _existentials_ (like our `Cache` protocol)
* We need some type _without_ an existential `associatedtype`.

Note: We need some type with no existential `associatedtype`s.

!!!

### Type erasure

In Swift, type erasure is converting protocol constraints to a concrete struct full of lambdas.
Existential associated types become universally quantified generics.

Type erasure is usually used in Swift to simplify type signatures, but we can also use it to workaround this limitation in the language.
You erase the type information that you had about the specific cache and add a small runtime penalty of an extra function call

```swift
func simplify() -> BasicCache<K, V> {
  return BasicCache(getFn: self.get, setFn: self.set)
}
```

!!!

### Type-erased cache

```swift
struct BasicCache<K, V>: Cache {
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

### Monoidal whatever

The ability to _hide "history"_ of the construction anywhere enables the developer to put the information wherever it may make sense to a reader.

(image)

!!!


