<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Strings Are Evil

An exploration through structural, nominal, and phantom types

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

!!!

## Strings are sneaky and omnipotent

(picture)

!!!

## Conquer the evil strings!

![explorer kid in jungle](img/explorer-kid.jpg)

(todo: Add labels nominal, structural, phantom)

> https://www.flickr.com/photos/mypubliclands/28487604443

!!!

## What does this function do?

```swift
func mystery(p1: String, p2: String) -> String
```

!!!

## Now what does it do?

```swift
func joinPaths(p1: String, p2: String) -> String
```

Note: Why was this better? We exposed more information with names

!!!

## Now what does it do?

```swift
func joinPaths(p1: String, p2: String) -> String {
  return p1 + "/" + p2
}
```

!!!

## Unveil the sneaky String

![sneaky monkey](img/sneaky-monkey.jpg)

> https://www.flickr.com/photos/mctrent/4279043790

!!!

## Names provide context

```swift
typealias FilePath = String

func joinPaths(p1: FilePath, p2: FilePath) -> FilePath
```

!!!

## Parameters can have names

```swift
func join(
    filePathChunk: String,
    otherFilePathChunk: String
) -> String
```

Note: But in Swift, we can name our parameters!

!!!

## Problem: Names don't provide safety

```swift
// oops
let p = joinPaths(p1: user.name, p2: user.password)
```

!!!

## Structually typed typealias

```swift
typealias FilePath = String
```

Note: These "FilePath" and "String" can be interchanged

!!!

## Wrong things should be hard

![A scary rubik's-type toy; mixed up](img/magic-cube.jpg)

> https://pixabay.com/p-2399883/?no_redirect

Note: Make it hard to do the wrong thing

!!!

## Make it hard by making Path different from String

```swift
struct Path {
  let s: String
  init(_ s: String) {
    self.s = s
  }
}

func joinPaths(p1: Path, p2: Path) -> Path
```

Note: A sufficiently smart compiler should optimize away the wrapper. Unfortunately, Swift isn't sufficiently smart yet; but it is smart enough to catch the simple mistakes

!!!

## Compiler saves you from simple mistakes

```swift
// Compile error!
// cannot convert value of type 'Username' to expected argument type 'FilePath'
let p = joinPaths(p1: user.name, p2: user.password)
```

!!!

## Nominal typing

```swift
struct FilePath
```

Note: String and FilePath are different now

!!!

## Bonus: Nominal types give us a method namespace

```swift
extension FilePath {
  func join(_ other: FilePath) -> FilePath {
    return FilePath(self.s + "/" + other.s)
  }
}
```

Note: We get a space to put domain specific functionality

!!!

## Bonus: Struct wrapper is ergonomically good

```swift
extension FilePath:
    ExpressibleByStringLiteral { /* ... */ }

let p: FilePath = "Sources/Hello.swift"
```

!!!

## We still have a problem...

```swift
let p = FilePath("Sources/Hello.swift")
    .join("/Users/bkase/project")
```

Note: Nothing stops us from joining an absolute path on the right; or a file on the left

!!!

## Phantoms can vanquish our foe

(image of a phantom fighting the evil string)

!!!

## Make it hard(er) to do the wrong thing

* You can't join with an absolute path on the right
* You can't join with a file on the left

!!!

## Whip out the enums

```swift
struct FileName { /* ... */ }
struct DirName { /* ... */ }
indirect enum Path {
  case root // /
  case current // .
  case dirIn(Path, DirName)
  case fileIn(Path, FileName)
  // you'll also want `parentIn` for ..
}
```

Note: This doesn't guarantee us the safety we wanted above

!!!

## Phantom Path

```swift
indirect enum Path<K: PathKind, T: FileType> {
```

```swift
  case root // /
  case current // .
  case dirIn(Path<K, T>, DirName)
  case fileIn(Path<K, T>, FileName)
}
```
<!-- .element: class="fragment" data-fragment-index="1" -->

Note: Path is now a phantom type because we don't use K and T in any cases

!!!

## Constrain our K and T

```swift
protocol PathKind {}
```

```swift
enum Absolute: PathKind {}
enum Relative: PathKind {}

```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
protocol FileType {}
enum Directory: FileType {}
enum File: FileType {}

```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
indirect enum Path<K: PathKind, T: FileType>
```
<!-- .element: class="fragment" data-fragment-index="3" -->

Note: You cannot instantiate any PathKind or FileType!

!!!

## We need "private" Path enum constructors

```swift
indirect enum Path<K: PathKind, T: FileType> {
```

```swift
  case _root // /
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
  case _current // .
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
  case _dirIn(Path, DirName)
```
<!-- .element: class="fragment" data-fragment-index="3" -->

```swift
  case _fileIn(Path, FileName)
}
```
<!-- .element: class="fragment" data-fragment-index="4" -->

!!!

## We expose constructors that specialize the generics

```swift
func file(_ name: FileName) -> Path<Relative, File> {
  return ._fileIn(._current, name)
}
```

```swift
func dir(_ name: DirName) -> Path<Relative, Directory> {
  return ._dirIn(._current, name)
}
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
let root: Path<Absolute, Directory> = ._root
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
let current: Path<Relative, Directory> = ._current
```
<!-- .element: class="fragment" data-fragment-index="3" -->

!!!

## Phantom Parameters for Constructors

![phantom parameter table](img/phantomtable.png)

Note: Fixed Values for our Phantom Parameters

!!!

## Safe join

```swift
extension Path where T == Directory {
```

```swift
  func join<T2>(_ other: Path<Relative, T2>) -> Path<K, T2> {
    switch (self, other) {
      /* ... */
    }
  }
```

!!!

## Safe join

```swift
    case (._current, ._current):
      return ._current
```

```swift
    case (._root, ._current):
      return ._root
```

```swift
    case let (._fileIn(p1, f1), ._current):
      return ._fileIn(p1 <%> ._current, f1)
```

```swift
    case let (._dirIn(p1, d1), ._current):
      return ._dirIn(p1 <%> ._current, d1)
```

```swift
    case let (p1, ._fileIn(p2, f2)):
      return ._fileIn(p1 <%> p2, f2)
```

```swift
    case let (p1, ._dirIn(p2, d2)):
      return ._dirIn(p1 <%> p2, d2)
```

!!!

## Safe join (the rest)

```swift
    // nonsense (assuming you don't use private constructors)
    case (._current, ._root),
      (._root, ._root),
      (._fileIn(_), ._root),
      (._dirIn(_), ._root):
      fatalError("Unreachable join cases")
    }
  }
}
```

!!!

## Operator for join

```swift
infix operator <%>: TernaryPrecedence
func <%><K, T>(lhs: Path<K, Directory>, rhs: Path<Relative, T2>) -> Path<K, T2> {
  return lhs.join(rhs)
}
```

!!!

## Typesafe Paths

```swift
let p = root <%>
    dir("Users") <%>
    dir("bkase") <%>
    file("Hello.swift")
```

!!!

## Typesafe Paths

```swift
// Compile Error!
file("Hello.swift") <%> dir("Sources")

// Compile Error!
dir("Sources") <%> root <%> dir("Hello.swift")
```

Note: Invariants of file on the left, or absoute on the right unbroken!

!!!

## Vanquished

(picture of explorer again)

!!!

## Shout-out

[Purescript pathy library](https://github.com/slamdata/purescript-pathy) invented this representation of file paths

Note: I just succeeded in porting the ideas to Swift

!!!

## What did we explore?

* Raw strings are sneaky and too powerful
* Typealiases (structural typing) makes them less sneaky
* Struct wrappers (nominal typing) weakens them
* Phantom types can restrict operations on certain states

!!!

<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

Slide Deck: [https://is.gd/edLKW7](https://is.gd/edLKW7)

