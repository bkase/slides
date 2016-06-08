<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Don't block
###	Asynchronous swift
						
By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

!!!

## Overview

* What are the async primitives in Swift at Highlight
* When to use what primitives

(my opinions are my own) <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

## Primitives

* Callback
* Delegate
* Future
* NSOperation
* NSNotification
* Event bus
* Observable

!!!

## Callback

Call a function when some task you've called is completed

Get it -- callback to you. <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

## Callback

```swift
func foo(k: (Int) -> ()) {
  NSIDoACallback { x in k(x) }
}
```

!!!

## Callback

Callbacks can be called multiple times from any execution context.

The async work decides the execution context.

* main thread
* this thread later
* this thread immediately

!!!

## Callback

Callbacks are flexible

Flexibility is the root of all evil

!!!

## Callback

What does `func getPhoto(k: (Photo) -> ())` do?

!!!

## Callback

```swift
func getPhoto(k: (Photo) -> ()) {
  NSGivePhoto { p in
    if p.isSuperSpecial() { k(p) }
    k(p)
  }
}
```

!!!

### Aside: Principle of Least Power

Strive for the least powerful interface to solve your problem.

!!!

## Delegate

A delegate conforms to a protocol with one or more methods

!!!

### Using a delegate

```swift
protocol PhotoDelegate: class {
  func newPhoto(photo: Photo)
  func allDone()
}
```

!!!

### Using a delegate

```swift
class IHaveMutableState {
  weak var delegate: PhotoDelegate?
}
IHaveMutableState().delegate = self
```

!!!

### Using a delegate

Don't forget _weak_!

Usually, the delegate will have a reference to the delegator

!!!

### Two delegates?

```swift
let ih = IHaveMutableState()
ih.delegate = self
ih.delegate = other
```

Too bad

!!!

## Delegate

What does `func getPhoto()` do?

This is why delegates are bad <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

## Future

A `Future<T,E>`* eventually yeilds a `T` or fails with an error of `E`.

&#42; <!-- .element: class="fragment" data-fragment-index="1" --> [BrightFutures](https://github.com/Thomvis/BrightFutures) <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

## Make a Future

A Future is either:

* A value now
* An error now
* An asynchronous computation
* Two or more computations combined

!!!

## Make a Future

Preferred:

```swift
f: Future<Int, NoError> 
f = Future(value: 5)
 
f2: Future<Zero, JustError> 
f2 = Future(error: JustError())
```

!!!

## Make a Future

Preferred:

```swift
future(Queue.global.context) {
  slow(100)
}
```

!!!

## Make a Future

Preferred:

```swift
// assuming you have
typealias N = NoError
let f1: Future<Int, N>
let f2: Future<Char, N>

let f: Future<(Int, Char), N> = f1.zip(f2)
```

!!!

## Make a Future

When you have to:

```swift
let p = Promise<Int, NoError>()
NSDoSomething{ i in p.trySuccess(i) }
return p.future
```

!!!

## Make a Future

Don't forget to complete your promise in _all_ branches

```swift
let p = Promise<Int, JustError>()
if something {
  p.trySuccess()
} else {
  if whatever {
    p.tryFailure(JustError())
  }
}
return p.future
```

!!!

## Future

What does `func getPhoto(): Future<Photo, N>` do?

!!!

## Combining Futures

```swift
f1: Future<Int,N>
f2: Future<String,N>

// map lines up the values

let f: Future<String, N> =
    f1.map{ x in "\(x)" }
```

!!!

## Combining Futures

```swift
f1: Future<T, E1>
f2: Future<T, E2>

// mapError lines up the errors

let f: Future<T, E2> =
    f1.mapError{ e in E2(e) }
```

!!!

## Combining Futures

Sequence

```swift
// do f1 and afterwards f2
f1.flatMap{ x in f2 }
```

!!!

## Combining Futures

Parallel

```swift
// do f1 and simultaneously f2
let f: Future<(T,U), N> =
    f1.zip(f2)
```

!!!

## Combining Futures

Parallel

```swift
// do f1,f2...fN simultaneously
let f: Future<[T], N> =
    [f1,f2...fN].sequence()
```

!!!

## Handle failures

```swift
f1: Future<Int, E>
let f: Future<Int, NoError> =
    f1.recover{ e in 5 }
```

!!!

## Use a future

```swift
f.onSuccess(context) { x in /*...*/ }
 .onFailure(context) { e in /*...*/ }

// context is the execution context
```

!!!

## Use a future

The execution context of your callback is determined at the _callsite_ not where the async work occurs

!!!

## NSOperation

Model a unit of work (state, priority, dependencies) with cancellation

Not composable (we should fix that)

!!!

## NSOperation

Must be explicitly started

This means returning an NSOperation is side-effect free <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

## NSOperation

See Ben's talk for more details

!!!

## NSNotification

A notification is a _global_ event associated with _untyped_ data

!!!

## NSNotification

One place fires an event

One place listens for the event

!!!

## NSNotification

Is worse than a mutable global variable

At least mutable global variables have types

!!!

## NSNotification

Couples X to Y implicitly if X fires an event and Y listens

!!!

## Event Bus

Is an immutable object that sends and receives _typed_ data

Buses cannot be composed

!!!

## Event Bus

Channels in Go are a good example of an event bus

In Swift, [EmitterKit](https://github.com/aleclarson/emitter-kit)

!!!

## Event Bus

In order to send or receive data you must have an event bus in scope

Therefore, no globals <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

## Observable

An `Observable<T>`* outputs many `T`s over time and eventually fails or completes.

How does it fail?

&#42; [RxSwift](https://github.com/ReactiveX/RxSwift)

!!!

## Observable

An Observable is a push-based stream of data

!!!

## Making an Observable

```swift
class Observable {
  class func create<T>(f: Observer<T> -> Disposable)
}
```

!!!

## Making an Observable

Disposable has `dispose()` method.

If you call `dispose()` free all memory and stop the stream.

If the stream completes it is automatically disposed.

!!!

## Making an Observable

```swift
Observable.create{ observer in
  observer.on(.Next(1))
  observer.on(.Next(2))
  observer.on(.Completed)
  return NopDisposable.instance
}
```

!!!

## Making an Observable

See [docs](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md#creating-your-own-observable-aka-observable-sequence) for more

!!!

## Observable

An Observable only starts (and always starts) when `subscribe()`ed

If you don't call `subscribe` nothing executes.

!!!

## Observable

The execution context of your subscriber callback is determined at the _callsite_ not where the async work occurs

!!!

## Observable

What does `func getPhotos() -> Observable<Photo>` do?

!!!

## Observable

Observables compose with `map`, `filter`, `zip` with sane behavior

Other [operators](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/API.md) with [diagrams](http://rxmarbles.com/)

!!!

## Observable

We need a whole hour or more to actually go into this

!!!

## When to use what

!!!

## When to use what

Always prefer _local_ information.

!!!

## When to use what

Function _signature_ should describe the behavior.

!!!

## When to use what

Not the function name.

!!!

## When to use what

* When you want to do one thing _once_
* When you want to do _n_ things _once_
* When you want to do one thing _alot_
* When the _OS_ does something
* When you need to do something _big_ and _cancellable_
* When you don't want to restructure things and you acknowledge that you are doing bad things

Note: Without loss of generality

!!!

#### When you want to do one thing _once_

Aka read data from a file

!!!

#### When you want to do one thing _once_

Use a future.

!!!

#### When you want to do one thing _once_

Don't use a callback.

Callbacks can be called multiple times.

!!!

#### When you want to do one thing _once_

Callbacks lead to callback hell

```swift
doA {
  doB {
    doC {
    }
  }
}
```

!!!

#### When you want to do one thing _once_

Futures don't

```swift
doA.flatMap{
  doB
}.flatMap{
  doC
}
```

!!!

#### When you want to do one thing _once_

Don't use delegates

!!!

#### When you want to do one thing _once_

Delegates require mutable state

Delegates require you to remember what is hooked up to what

!!!

#### When you want to do one thing _once_

Delegates require your coworkers to read your mind

!!!

#### When you want to do one thing _once_

What does `func getPhotos()` do?

How about `func getPhotos() -> Future<Photo>`?

!!!

#### When you want to do _n_ things _once_:

Aka read `n` files.

!!!

#### When you want to do _n_ things _once_:

Return `n` Futures.

!!!

#### When you want to do _n_ things _once_:

Don't use delegates.

Don't use callbacks.

!!!

#### When you want to do _n_ things _once_:

If you want to name the things, use a struct

```swift
struct LoadingFutures {
  profileLoad: Future<Profile>
  channelLoad: Future<Channel>
}

func loadStuff() -> LoadingFutures
```

!!!

#### When you want to do one thing _alot_:

Aka get progress on some network request

!!!

#### When you want to do one thing _alot_:

If we all grok `Observable`, use that.

Otherwise, use a callback.

!!!

#### When you want to do one thing _alot_:

Callbacks can be called more than once.

Therefore, let's just use them for that.

!!!

#### When you want to do one thing _alot_:

Don't use delegates.

Don't use events (event bus or NSNotfication).

!!!

#### When you want to do one thing _alot_:

NSNotifications are untyped globals

!!!

#### When the _OS_ does something:

Aka when a network request finished (after your app was killed)

!!!

#### When the _OS_ does something:

Use delegates.

!!!

#### When the _OS_ does something:

You have nothing in RAM to track your intent to perform an action

!!!

#### When the _OS_ does something:

Don't use events.

If you are about to fire an event, just handle it there.

!!!

#### When you need to do something _big_ and _cancellable_:

Use NSOperation

!!!

#### When you need to do something _big_ and _cancellable_:

Cancellation being built-in is a big win.

!!!

#### When you don't want to restructure things and you acknowledge that you are doing bad things

Use events

I still think we should use something like EmitterKit instead of NSNotifications, however. <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

## The dream

A simple, cancellable, functional pure (only executes when you start it) abstract Task (like a future but more functional)

aka wrap NSOperation to make it composable and feel nicer

!!!

## Summary

Prefer being explicit.

Prefer being stateless.

State is the enemy.

!!!

## Summary

Futures are usually fine

!!!

<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Discussion
