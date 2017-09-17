<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Strings Are Evil

An exploration through structural, nominal, and phantom types

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

!!!

## Strings are sneaky and omnipotent

(picture)

!!!

## What does this function do?

```swift
func mystery(lhs: String, rhs: String) -> String
```

!!!

## Now what does it do?

```swift
func joinPaths(lhs: String, rhs: String) -> String
```

Note: Why was this better? We exposed more information with names

!!!

## Now what does it do?

```swift
func joinPaths(lhs: String, rhs: String) -> String {
  return lhs + "/" + rhs
}
```


!!!

## Unveil the sneaky String

(picture)

!!!

## Names provide context

```swift
typealias FilePath = String

func joinPaths(lhs: FilePath, rhs: FilePath) -> FilePath
```

!!!

## But in Swift, we can name our parameters!

```swift
func join(
    filePathChunk: String,
    otherFilePathChunk: String
) -> String
```

!!!

## Problem: Names don't provide safety

```swift
// oops
let p = joinPaths(lhs: user.name, rhs: user.password)
```

!!!

## Structually typed typealias

```swift
typealias FilePath = String
```

Note: These "Path" and "String" can be interchanged

!!!

## Make it hard to do the wrong thing

!!!

## Make it hard by making Path different from String

```swift
struct Path {
  let s: String
  init(_ s: String) {
    self.s = s
  }
}

func joinPaths(a: Path, b: Path) -> Path
```

Note: A sufficiently smart compiler should optimize away the wrapper. Unfortunately, Swift isn't sufficiently smart yet...

!!!

(TODO: Show how struct path makes compile errors where we want)

!!!

## Nominal typing

```swift
struct Path
```

Note: String and Path are different now

!!!

## Bonus: Nominal types give us a method namespace

```swift
extension Path {
  func join(_ other: Path) -> Path {
    return self.s + "/" + other.s
  }
}
```

Note: We get a space to put domain specific functionality

!!!

## Bonus: Struct wrapper is ergonomically good

```swift
extension Path: ExpressibleByStringLiteral { /* ... */ }

let p: Path = "Sources/Hello.swift"
```

!!!

## We still have a problem...

```swift
let p = Path("Sources/Hello.swift")
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
  case root // /
  case current // .
  case dirIn(Path<K, T>, DirName)
  case fileIn(Path<K, T>, FileName)
}
```

Note: Path is now a phantom type because we don't use K and T in any cases

!!!

## Constrain our K and T

```swift
protocol PathKind {}
enum Absolute: PathKind {}
enum Relative: PathKind {}

protocol FileType {}
enum Directory: FileType {}
enum File: FileType {}

indirect enum Path<K: PathKind, T: FileType>
```

Note: You cannot instantiate any PathKind or FileType!

!!!

## We need "private" Path enum constructors

```swift
indirect enum Path<K: PathKind, T: FileType> {
  case _root // /
  case _current // .
  case _dirIn(Path, DirName)
  case _fileIn(Path, FileName)
}
```

!!!

## We expose constructors that specialize the generics

(TODO break these out on separate slides)

```swift
func file(_ name: FileName) -> Path<Relative, File> {
  return ._fileIn(._current, name)
}
func dir(_ name: DirName) -> Path<Relative, Directory> {
  return ._dirIn(._current, name)
}
let root: Path<Absolute, Directory> = ._root
let current: Path<Relative, Directory> = ._current
```

!!!

## State machine limiting

(draw state transition diagram?)

!!!

## Safe join

```swift
extension Path where T == Directory {
  func join<T2>(_ other: Path<Relative, T2>) -> Path<K, T2> {
    switch (self, other) {
    case (._current, ._current):
      return ._current
    case (._root, ._current):
      return ._root
    case let (._fileIn(p1, f1), ._current):
      return ._fileIn(p1 <%> ._current, f1)
    case let (._dirIn(p1, d1), ._current):
      return ._dirIn(p1 <%> ._current, d1)
    case let (p1, ._fileIn(p2, f2)):
      return ._fileIn(p1 <%> p2, f2)
    case let (p1, ._dirIn(p2, d2)):
      return ._dirIn(p1 <%> p2, d2)
    // nonsense (assuming you don't use private constructors)
    case (._current, ._root),
      (._root, ._root),
      (._fileIn(_), ._root),
      (._dirIn(_), ._root):
      fatalError("Unreachable with type safety")
    }
  }
}
```

!!!

## Path fun

```swift
infix operator <%>: TernaryPrecedence
func <%><K, T>(lhs: Path<K, Directory>, rhs: Path<Relative, T2>) -> Path<K, T2> {
  return lhs.join(rhs)
}
```

!!!

## Typesafe Paths

```swift
let p = root <%> dir("Users") <%> dir("bkase") <%> file("Hello.swift")
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

(TODO: Need some sort of conclusion to wrap everything up)

