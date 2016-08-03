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

Transformations on legos (for any legos) ->
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

Note: Don't wince at the side-effects

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

