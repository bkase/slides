<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Reader or Weep: Mock-free, Compile-time Checked, Dependency Injection with Reader

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

!!!

### This Code Sucks

```swift

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

* TODO
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
class Cache {
  let disk: Disk
  let network: Network

  init(disk: Disk, network: Network) {
    self.disk = disk
    self.network = network
  }
}
```

Note: This is called dependency injection

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
struct Config {
  let data: Data
  let network: Network
}
func get(config: Config, key: Key) -> Value
func set(config: Config, key: Key, value: Value) -> ()
```

!!!

### Rearrange the parameters

```swift
func get(key: Key, config: Config) -> Value
func set(key: Key, value: Value, config: Config) -> ()
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
func get(key: Key, config: Config) -> Value
func set(key: Key, value: Value, config: Config) -> ()
```

!!!

### We can curry them

```swift
func get(key: Key) -> (Config) -> Value
func set(key: Key, value: Value) -> (Config) -> ()
```

!!!

### Take a second and think what these operations mean

```swift
func get(key: Key) -> (Config) -> Value
```

```swift
func set(key: Key, value: Value) -> (Config) -> ()
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
func get(key: Key) -> Reader<Config, Value>
func set(key: Key, value: Value) -> Reader<Config, ()>
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
func increment(key: Key) -> Reader<Config, ()>
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
func increment(key: Key) -> Reader<Config, ()> {
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
func incrementTwice(key: Key) -> Reader<Config, ()> {
  return increment(key: key).flatMap{ () in
    increment(key: key)
  }
}
```

!!!

# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

Slide Deck: [https://is.gd/mjJamZ](https://is.gd/mjJamZ)

Thanks to [Vittorio Monaco](https://twitter.com/Vittorio_Monaco) for making the [Carlos](https://github.com/WeltN24/Carlos) library

