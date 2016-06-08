<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Protocol Oriented Programming Part 1
### Trait composition AKA abstract classes + multiple inheritance (sort of)

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

!!!

## Code reuse + modularity

!!!

### Code reuse + modularity

Is good

The dream is that product wants you to make a change, and you write the new code and don't touch anything else

!!!

### Code reuse + modularity

In Swift, _protocols_ give you that most power

!!!

# Protocols

!!!

## Protocols

```swift
protocol Helloable {
  func helloWorld() -> String
}
```

!!!

## Protocols

```swift
struct BrandonsWorld: Helloable {
  let myName = "Brandon"
  func helloWorld() -> String {
    return "Hello, \(myName)"
  }
}

let b = BrandonsWorld()
print(b.helloWorld())
```

!!!

## Protocols

Let's do better

!!!

## Protocols

```swift
protocol Helloable {
  var myName: String
  func helloWorld() -> String
}
```

!!!

### Defualt implementations

```swift
// you can use the var from the protocol
extension Helloable {
  func helloWorld() -> String {
    return "Hello, \(myName)"
  }
}
```

!!!

### Default implementations

```swift
struct BrandonsWorld: Helloable {
  let myName = "Brandon"
}

// that's it!
let b = BrandonsWorld()
print(b.helloWorld())
```

!!!

### Partial implementations

```swift
protocol Helloable {
  func helloWithVCName() -> String
}
```

!!!

### Partial implementations

```swift
extension Helloable where Self: UIViewController {
  func helloWithVCName() {
    // pretend this is a local method in UIViewController
    return "Hello \(nameFromXibOrWhatever())"
  }
}
```

!!!

### Partial implementations

```swift
class FooViewController: UIViewController, Helloable {
  func viewWillAppear() {
    // you can pretend it's a local method
    print(helloWithVCName())
    // but it's defined in a modular way!
  }
}
```

!!!

## Protocols

Protocols have
* required methods (or computed properties)
* optionally with default implementations
* or _partial_ default implementations
* BUT cannot contain state

!!!

## Protocols

Protocols are _traits_ (from a programming language theory perspective)

Since there is no state, they are not _mixins_

!!!

### Validation example

We validate usernames during onboarding and in settings

!!!

### Validation example

```swift
enum InvalidUsername: ErrorType {
  case TooShort
  case TooLong
  case PoopRelated(String)
}
protocol ValidatesUsername {
  func usernameValid(uname: String) -> InvalidUsername?
}
// implement it once!
extension ValidatesUsername { /* ... */ }
```

!!!

## Validation example

```swift
class OnboardingVC: UIViewController, ValidatesUsername {
  /* ... */
  handleError(usernameValid(self.textView.text))
}
class SettingsVC: UIViewController, ValidatesUsername {
  /* ... */
  handleError(usernameValid(self.textView.text))
}
```

!!!

### Depend on data

What if we want to depend on some piece of data, like a `validator` that hits the network

!!!

### Depend on data

```swift
protocol ValidatesUsername {
  // this thing hits the network
  var validator: Validator
  func usernameValid(uname: String) -> InvalidUsername?
}
extension ValidatesUsername {
  func usernameValid(uname: String) -> InvalidUsername? {
    // we can pretend we have validator in scope
    return validator.valid(uname)
  }
}
```

!!!

### Depend on data

```swift
class OnboardingVC: UIViewController, ValidatesUsername {
  let validator = AppModules.sharedInstance.validator
  /* ... */
  handleError(usernameValid(self.textView.text))
}
```

!!!

### Depend on other protocols

!!!

### Depend on other protocols

```swift
protocol Validates {
  func validate(data: String) -> ValidationError?
}
protocol ValidatesUsername: Validates {
  func usernameValid(uname: String) -> InvalidUsername?
}

// here we can call anything from Validates
extension ValidatesUsername { /* ... */ }
```

!!!

### Pseudo-multiple inheritance

```swift
class OnboardingVC: UIViewController, ValidatesUsername, Helloable {
  /* ... */
}
```

!!!

### Pseudo-multiple inheritance

In reality, this is just protocol-oriented composition, but it _feels_ like _inheritance_ without the downside of a rigid hierarchy

!!!

## Protocols

Later: associated types

AKA generalize your protocols with type variables.

!!!

<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Protocols


