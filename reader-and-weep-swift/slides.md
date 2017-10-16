<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Reader or Weep: Mock-free, Compile-time Checked, Dependency Injection with Reader

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

!!!

### This Code Sucks

```swift
class Cache {
  func get(key: Key) -> Value {
    return Data.readFile(key.name) ??
      Network.sharedInstance.get("/key", key) ??
      Value.default
  }
  func set(key: Key, value: Value) { /*...*/ }
}
```

```swift
func increment(key: Key) {
  let value = Cache.sharedInstance.get(key: key)
  Cache.sharedInstance.set(key: key, value: inc(value))
}
```

!!!

### Why does it suck?

* Tight coupling to core modules
* Requires mocking to test

!!!

### Why is it bad to be coupled?

* Can't reason about code in small pieces
* Modularity is good for software (motivate this more)

!!!

### Why is it bad to mock?

* You could easily forget a nested dependency (accidentally hit network in unit tests)
* Note: It's so bad, that Swift doesn't even allow it (really)

!!!

### Let's fix it

(picture)

Note: Instead of implicitly depending on our modules...

!!!

### What is this talk?

I'm not pitching one particular library, I'm trying to teach you a different way to think about dependencies to hopefully inspire you to create patterns that work for your Swift app/program

What I want to do is show you that the state of the world is bad. We can do better.

!!!

### Depend only on that which you say

```swift
// let's say disk and network are protocols
class Cache {
  let disk: Disk
  let network: Network

  init(disk: Disk, network: Network) {
    self.disk = disk
    self.network = network
  }
}
```

Note: This is called dependency injection (if you do it this way, you're already better than most people)

!!!

### Unfortunately, this gets unwieldy fast

(screenshot of nested dependencies)

!!!

### Dependency Injection Frameworks

```swift
let graph = Node()
graph.register(Foo.self, { _ in Foo() })
graph.register(Bar.self, { graph in Bar(foo: graph[Foo.self])})
// ...
let bar = graph[Bar.self]!
```

Note: This is definitely a step up, but see the problem?

!!!

### The problem

```swift
let bar = graph[Bar.self]!
```

Note: You can forget to register a type in your dependency graph and you won't know until runtime!

!!!

### The solution

Types!

!!!

### Consider our class with a dependency

```swift
class Cache {
  // ...
  init(data: Data, network: Network)
  // ...
  get //...
  set //...
}
```

!!!

### This is morally the same as a collection of functions

```swift
// this is what the cache does
func get(data: Data, network: Network, key: Key) -> Value
func set(data: Data, network: Network, key: Key, value: Value) -> Void
```

!!!

### We can group together our dependencies

```swift
struct CacheConfig {
  let data: Data
  let network: Network
}
func get(config: CacheConfig, key: Key) -> Value
func set(config: CacheConfig, key: Key, value: Value) -> ()
```

!!!

### Rearrange the parameters

```swift
func get(key: Key, config: CacheConfig) -> Value
func set(key: Key, value: Value, config: CacheConfig) -> ()
```

!!!

### Curry the function

(picture)

!!!

### What is currying?

```swift
func uncurriedFoo(a: Int, b: String) -> Int
let x = uncurriedFoo(4, "a")
```

```swift
func curriedFoo(a: Int) -> (String) -> Int
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
let x = curriedFoo(4)("a")
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
let partial = curriedFoo(4)
```
<!-- .element: class="fragment" data-fragment-index="3" -->

```swift
let y = partial("b")
let z = partial("c")
```
<!-- .element: class="fragment" data-fragment-index="4" -->

!!!

### Take another look at our operations

```swift
func get(key: Key, config: CacheConfig) -> Value
func set(key: Key, value: Value, config: CacheConfig) -> ()
```

!!!

### We can curry them

```swift
func get(key: Key) -> (CacheConfig) -> Value
func set(key: Key, value: Value) -> (CacheConfig) -> ()
```

!!!

### Take a second and think what these operations mean

```swift
func get(key: Key) -> (CacheConfig) -> Value
```

```swift
func set(key: Key, value: Value) -> (CacheConfig) -> ()
```
<!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### Capture the second part in a type

```swift
struct Reader<In, Data> {
    let run: (In) -> Data
}
```

```swift
func get(key: Key) -> Reader<CacheConfig, Value>
func set(key: Key, value: Value) -> Reader<CacheConfig, ()>
```
<!-- .element: class="fragment" data-fragment-index="1" -->


!!!

### Reader me a story

(picture)

!!!

### Building programs?

```swift
func increment(key: Key) {
  // get the value at the key
  // set the value back incremented
}
```

!!!

### This program depends on the cache

```swift
func increment(key: Key) -> Reader<CacheConfig, ()>
```

!!!

### We need to combine two readers in sequence

```swift
// get and then set
```

!!!

### Reader flatMap

```swift
func flatMap<B>(
```

```swift
  _ f: @escaping (Data) -> Reader<In, B>
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
) -> Reader<In, B> {
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
  return Reader<In, B> { i in
```
<!-- .element: class="fragment" data-fragment-index="3" -->

```swift
    let data = self.run(i)
```
<!-- .element: class="fragment" data-fragment-index="4" -->

```swift
    let newReader = f(data)
```
<!-- .element: class="fragment" data-fragment-index="5" -->

```swift
    return newReader.run(i)
  }
}
```
<!-- .element: class="fragment" data-fragment-index="6" -->

!!!

### Hooking up increment

```swift
func increment(key: Key) -> Reader<CacheConfig, ()> {
```

```swift
  return get(key: Key).flatMap{ value in
    set(key: Key, value: inc(value))
  }
}
```
<!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### Great it works!

(picture)

!!!

### It's composable!

```swift
func incrementTwice(key: Key) -> Reader<CacheConfig, ()> {
  return increment(key: key).flatMap{ () in
    increment(key: key)
  }
}
```

!!!

### Two different dependencies?

(image of a circle and a triangle arrows)

!!!

### Use cache and calendar

```swift
func todaysKey() -> Reader<TimeConfig, Key>
func increment(key: Key) -> Reader<CacheConfig, ()>
```

```swift
func incTodaysKey() -> Reader<?, ()> {
  return todaysKey().flatMap{ increment($0) }
}
```
<!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### Consider the union of the two

(diagram of both)

!!!

### Lift each one to the global context

(diagram of lifting the little ones into the big)

!!!

### Lift each into the global

```swift
struct TimeAndCache {
  let time: TimeConfig
  let cache: CacheConfig
}
```

```swift
func incTodaysKey() -> Reader<?, ()> {
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
  return todaysKey().local{ both in both.time }
      .flatMap{ key in
        increment(key).local{ both in both.cache }
      }
}
```

!!!

### Local in Reader

```swift
// struct Reader<In, Data>
extension Reader {
```

```swift
  func local<Global>(_ f: (Global) -> In) -> Reader<Global, Data> {
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
    return Reader<Global, Data> { global in
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
      self.run(f(global))
    }
  }
}
```
<!-- .element: class="fragment" data-fragment-index="3" -->

!!!

### Combine pure computations

```swift
func reverseKeyGet() -> Reader<CacheConfig, ()> {
  return Reader.pure(reverse(key))
```

```swift
    .flatMap{ get(key: $0) }
}
```
<!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### Combine pure computations

```swift
// struct Reader<In, Data>
extension Reader {
```

```swift
  static func pure(a: Data) -> Reader<In, Data> {
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
    return Reader<In, Data> { _ in a }

  }
}
```
<!-- .element: class="fragment" data-fragment-index="2" -->

Note: Just like we ignore the extra state with `local`, we can ignore all state with `pure`

!!!

### More effects

(picture)

Note: Our cache maybe will fail to give us a value for example

!!!

### More effects

```swift
func get(key: Key) -> Reader<CacheConfig, Value?>
```

```swift
// we want
func get(key: Key) -> OptionReader<CacheConfig, Value>
```

Note: We want to capture this so we flatMap once!

!!!

### ReaderOption

```swift
struct OptionReader<In, A> {
  let reader: Reader<In, A?>
  init(_ reader: Reader<In, A?>) { self.reader = reader }
}
```

```swift
extension OptionReader {
  static func pure(a: A) -> OptionReader<In, A> {
    return OptionReader(
      Reader<In, A?>.pure(.some(a))
    )
  }
}
```

```swift
extension OptionReader {
  func map<B>(_ f: (A) -> B) -> OptionReader<In, B> {
    return OptionReader<In, B>(
      reader.map{ $0.map(f) }
    )
  }
}
```

```swift
extension OptionReader {
  func flatMap<B>(_ f: (A) -> OptionReader<In, B>) -> OptionReader<In, B> {
    return OptionReader<In, B>(
      Reader<In, B?> { input in
        self.reader.run(input).flatMap{ f($0).reader.run(input) }
      }
    )
  }
}
```

!!!

### OptionReader

```swift
// can fail and needs dep
func get(key: Key) -> OptionReader<CacheConfig, Value>
// needs dep
func cacheInfo() -> Reader<CacheConfig, Info>
// can fail
func maybeReverse(key: Key) -> Key?
```

```swift
typealias O<Value> = OptionReader<CacheConfig, Value>

func loadReversedKeyIfSupported(key: Key) -> O<Value> {
  O(cacheInfo().map{.some($0)}).flatMap{ info in
    O(Reader.pure(info.reversable ? maybeReverse(key: key) : nil))
  }.flatMap(get)
}
```

Note: So noisy!

!!!

### OptionReader reduce noise

```swift
extension OptionReader {
  static func lift(opt: A?) -> OptionReader {
    return OptionReader(Reader.pure(opt))
  }

  static func lift(reader: Reader<In, A>) -> OptionReader {
    return OptionReader(reader.map{.some($0)})
  }
}
```

!!!

### Less noisy

```swift
typealias O<Value> = OptionReader<CacheConfig, Value>

func loadReversedKeyIfSupported(key: Key) -> O<Value> {
  O.liftReader(cacheInfo()).flatMap { info in
    O.liftOpt(info.reversable ? maybeReverse(key: key) : nil)
  }.flatMap(get)
}
```

!!!

### Effects stack

(picture)

Note: This works for any monad (Future, option, reader, nondeterminism, etc). Unfortunately, in Swift you do have to do this work for your bespoke effect stack in your program, we can't abstract it in a library (unless we codegen)

!!!

### Aside: Effect stack with codegen

Check out [Monads](https://github.com/facile-it/Monads) by Elviro Rocca (@_logicist)

!!!

### Monad?

Do you need to "understand monads" to use Reader? No.

Note: I encourage everyone to try and learn as much as they can, so I don't like the excuse of "monads are confusing, I won't talk about them", but even here you can use Reader without know monads

!!!

### Real name: Reader Monad

(by the way this thing I've been talking about the whole time is formally called a Reader Monad)

!!!

### Config me how?

(picture)

(maybe)

Note: This is all great in theory, but how do I get all my dependencies in that neat package from my view controllers

!!!

### Co-readers!

If you're interested, check out the [Corridor](https://github.com/symentis/Corridor) framework by Elmar Kretzer @elmkretzer

Note: I don't have time to talk about it here

!!!

### Other typesafe DI methods

(picture)

Note: I have to mention these, for the interested to explore

!!!

### Cake pattern

```swift
protocol Cache: HasData, HasNetwork {
  func get(key: Key) -> Value
  func set(key: Key, value: Value)
}
```

```swift
extension Cache {
  // implementation for get and set
}
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
protocol HasCache {
  var cache: Cache
}
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
protocol Program: HasCache, HasTime {
  func main()
}
extension Program {
  func main() { /*...*/ }
}
```
<!-- .element: class="fragment" data-fragment-index="3" -->

Note: Layers and layers of protocols

!!!

### Free Monads

```swift
enum CacheCommands<Next> {
  case get(key: Key, next: (Value) -> Next)
  case set(key: Key, value: Value, next: () -> Next)
}
```

```swift
// and machinery to make the callbacks
// turn into pures, maps, and flatmaps
// a monad "for free"
```
<!-- .element: class="fragment" data-fragment-index="1" -->

Note: Lots of boilerplate, codegen necessary for use in Swift

!!!

### Swift DI Explorations

[Swift DI Explorations](https://github.com/bkase/swift-di-explorations) has literate examples of Cake, Reader, and Free DI patterns

!!!

### Inspire

(picture)

Note: There is some elegance here. Maybe you can come up with a nice way to fit it in with Protocols!

!!!

# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

Slide Deck: [https://is.gd/mjJamZ](https://is.gd/mjJamZ)

[DI Explorations](https://github.com/bkase/swift-di-explorations), [Corridor](https://github.com/symentis/Corridor), [Monads (effect stacks with codegen)](https://github.com/facile-it/Monads)

