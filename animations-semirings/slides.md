
<div class="vidcode">
  <video playsinline autoplay muted loop>
    <source src="vids/title.mp4" type="video/mp4">
  </video>
</div>

!!!

### Natural description

```ideal
let left =
  grow circle from radius 10 to 100 with an ease-in-out
```

```ideal
let right =
  rotate square from 0 to 2*pi degrees with a high ease-out
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```ideal
let animations = left + right
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```ideal
let final = fadeIn all views * animations
```
<!-- .element: class="fragment" data-fragment-index="3" -->

!!!

### Animations are important

(image)

Note: Delightful animations make your users happy, and it makes your app standout.

!!!

### Principled Use of Animations More important

(image)

Note: Being consistent is extremely important. Large organizations even build out an official design language: these are the colors you should use, these are the curves, etc. All buttons should do the following...

!!!

### Let's try it

<div class="vidcode">
  <video playsinline autoplay muted loop>
    <source src="vids/basic.mp4" type="video/mp4">
  </video>
</div>

Note: A very simple animation. Break it down.

!!!

### Natural description

```ideal
let left =
  grow circle from radius 10 to 100 with an ease-in-out
```

```ideal
let right =
  rotate square from 0 to 2*pi degrees with a high ease-out
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```ideal
let animations = (left + right) looped once
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```ideal
let final = fadeIn all views * animations
```
<!-- .element: class="fragment" data-fragment-index="3" -->

!!!

### Why this description

* Think about pieces separately
* Combine the pieces later to build more complex things
* Notion of composability (see my other talks on this)
TODO

!!!

### Simple Animation

<div class="vidcode">
  <video playsinline autoplay muted loop>
    <source src="vids/basic.mp4" type="video/mp4">
  </video>
</div>

Note: A very simple animation. Break it down.

!!!

### Should be easy

(image)

!!!

### Idiomatic iOS Approach

(image)


Note: UIView.animate

!!!

### Left (description)

```ideal
let left =
  grow circle from radius 10 to 100 with an ease-in-out
```

!!!

### Left (code)

```swift
func left(_ v: UIView) {
```
<!-- .element: class="fragment" data-fragment-index="3" -->

```swift
  UIView.animate(
```

```swift
    withDuration: 2.0,
    delay:0,
    options:.curveEaseInOut,
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
    animations: {
    resize(v, to: 10)
  }, completion: nil)
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
}
```
<!-- .element: class="fragment" data-fragment-index="3" -->

Note: But this code isn't quite right: we need a function; This code isn't quite right...

!!!

### Left (code)

```swift
func left(_ v: UIView, completion: (() -> ())?) {
  UIView.animate(
    withDuration: 2.0,
    delay:0,
    options:.curveEaseInOut,
    animations: {
    resize(v, to: 10)
  }, completion: completion)
}
```

!!!

### Right (description)

```ideal
let right =
  rotate square from 0 to 2*pi degrees with a high ease-out
```

!!!

### Right (code)

```swift
func right(_ v: UIView, completion: (() -> ())?) {
```

```swift
  let timingFunction = CAMediaTimingFunction(controlPoints: 0, 0, 0.2, 1.0)

  CATransaction.begin()
  CATransaction.setAnimationTimingFunction(timingFunction)
```
<!-- .element: class="fragment" data-fragment-index="3" -->

```swift
  UIView.animate(
    withDuration: 2.0,
    delay:0,
    options:.curveEaseOut,
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
    animations: {
    rotate(v, to: 2*CGFloat.pi)
  }, completion: completion)
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
  CATransaction.commit()
```
<!-- .element: class="fragment" data-fragment-index="3" -->

```swift
}
```

Note: But: We want a particular easeOut curve... Wait a minute! Side-effects, global state changes, we haven't even tried to combine these yet! To reiterate:

!!!

### Parallel composition

(video of simple parallel composition)

!!!

### Parallel animations with UIView.animate

```swift
UIView.animate(/*...*/ {
  self.alpha = 1.0
  self.backgroundColor = .green
} /*...*/)
```

Note: This is what they tell you

!!!

### Parallel composition (description)

```ideal
let animations = (left + right) looped
```

!!!

### Parallel composition (code)

```swift
UIView.animate(/*...*/ {
  left(?)
  right(?)
} /*...*/)
```

Note: This completely doesn't work

!!!

### Parallel composition (code)

```swift
func animations(completion: (() -> ())?) {
```

```swift
  left(circle, completion: nil)
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
  right(square, completion: completion)
}
```
<!-- .element: class="fragment" data-fragment-index="2" -->

Note: We had to know apriori which completion fires second, otherwise this would get really messy. Doesn't handle the looping

!!!

### Stuck

(image)

Note: We can't do the looping because we can't inspect or manipulate the animations, they're trapped in a function.

!!!

### Sequential composition

(video of simple sequential composition)

!!!

### Sequential composition (description)

```ideal
let final = fadeIn all views * animations
```

!!!

### Sequential composition (code)

```swift
func final() {
  fadeInAll {
    animations(completion: nil)
  }
}
```

!!!

### Keyframes

```swift
UIView.animateKeyframes(/*...*/, animations: {
```

```swift
    UIView.addKeyframe(
      withRelativeStartTime: 0,
      relativeDuration: 0.5,
      animations: {
      /* put your properties here */
      })
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
    UIView.addKeyframe(
      withRelativeStartTime: 0.5,
      relativeDuration: 0.5,
      animations: {
      /* put your properties here */
      })
}, completion: nil)
```
<!-- .element: class="fragment" data-fragment-index="2" -->

Note: To be fair, I want to call out that you could use keyframes as well; but (1) you have to keep track of relative timing in your head and (2) you can't nest keyframes, so it's not really much better for composability.

!!!

### Simple Animation?

<div class="vidcode">
  <video playsinline autoplay muted loop>
    <source src="vids/basic.mp4" type="video/mp4">
  </video>
</div>

Note: Shouldn't this be simpler?

!!!

### Problems

* Side-effects
* Callback-based sequencing (leads to right-indentation hell)
* Animations aren't values, can't manipulate them

!!!

### Core Animation

(image)

Note: More advanced, the implied tradeoff "power" vs "easy" (I say we can have both), but maybe the workarounds we had to do made the easy thing unneccesarily more confusing

!!!

### Left (description)

```ideal
let left =
  grow circle from radius 10 to 100 with an ease-in-out
```

!!!

### Left (code)

```swift
let a = CABasicAnimation(keyPath: "transform.scale")
```

```swift
a.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInOut)
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
a.fromValue = 10
a.toValue = 100
a.duration = 2.0
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
animation.fillMode = kCAFillModeForwards
// Note: You have to set this also
animation.isRemovedOnCompletion = false
```
<!-- .element: class="fragment" data-fragment-index="3" -->

```swift
circle.layer.add(animation, forKey: nil)
```
<!-- .element: class="fragment" data-fragment-index="4" -->

Note: Animations are values now, but at what cost!

!!!

### Right

(omitted for brevity)

!!!

### Parallel composition

```swift
func parallel(a1: CAAnimation, a2: CAAnimation) -> CAAnimation {
  fatalError("?")
}
```

!!!

### Parallel composition (description)

```ideal
let animations = (left + right) looped
```

!!!

### Parallel

```swift
func animations() {
  left.autoreverses = true
  circle.layer.add(left, forKey: nil)

  right.autoreverses = true
  square.layer.add(right, forKey: nil)
}
```

Note: Back to side-effect land; (1) we destructively modify left/right (2) we can't say that the composition as a whole loops, we have dig down.

!!!

### Sequential Composition

(vid)

!!!

### Sequential composition (description)

```ideal
let final = fadeIn all views * animations
```

!!!

### Sequential composition (code)

```swift
let final = CAAnimationGroup()
```

```swift
final.animations = [fadeInAll, animations]
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
final.duration = 4
```
<!-- .element: class="fragment" data-fragment-index="2" -->

Note: (1) assume `animations` is actually a CAAnimation (2) duration has to be manually calculated (3) curves being set overwrite the children's. AND this doesn't even work because the layers are different.

!!!

### Sequential composition (code)

```swift
func final() {
  left.beginTime = CACurrentMediaTime() + fadeInAll.duration
  right.beginTime = CACurrentMediaTime() + fadeInAll.duration
```

```swift
  [circle, square].forEach{ v in v.layer.add(fadeIn) }
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
  animations()
}
```
<!-- .element: class="fragment" data-fragment-index="2" -->

Note: (1) side effects :(, cannot compose sequentially, need to reach in to the underlying animations

!!!

### Problems CAAnimation

* Side-effects / hidden state
* Composition is adhoc
* Poor reuse

Note: (1) Animations are values? Sort of, but not really (2) Poor reuse BOTH through destructive modification of animations when combining, and adhoc side-effecty composition.

!!!

### Simple Animation?

<div class="vidcode">
  <video playsinline autoplay muted loop>
    <source src="vids/basic.mp4" type="video/mp4">
  </video>
</div>

Note: Shouldn't this be simpler?

!!!

### Left (description)

```ideal
let left =
  grow circle from radius 10 to 100 with an ease-in-out
```

!!!

### Left (code)

```swift
let left =
  linear(from: 10, to: 100, in: 2.0).time(easeInOut)
    .do(resize(circle))
```

!!!

### Right (description)

```ideal
let right =
  rotate square from 0 to 2*pi degrees with a high ease-out
```

!!!

### Right (code)

```swift
let right =
  linear(from: 0, to: 2*CGFloat.pi, in: 2.0).time(easeOut(1.5))
    .do(rotate(square))
```

!!!

### FadeIn (description)

```ideal
fadeIn all views
```

!!!

### FadeIn (code)

```swift
let fadeIn =
  linear(from: 0, to: 1, in: 1.0).time(easeInOut)
    .bind(container, with: \.alpha)
```

!!!

### Parallel Composition (description)

```ideal
let animations = (left + right) looped once
```

!!!

### Parallel Composition (code)

```swift
let animations = (left + right).looped
```

Note: Not a typo, this is the code

!!!

### Sequential Composition (description)

```ideal
let final = fadeIn * animations
```

!!!

### Sequential Composition (code)

```swift
let final = fadeIn * animations
```

Note: Literally the same

!!!

### All the code

```swift
let left =
  linear(from: 10, to: 100, in: 2.0).time(easeInOut)
    .do(resize(circle))
```

```swift
let right =
  linear(from: 0, to: 2*CGFloat.pi, in: 2.0).time(easeOut(1.5))
    .do(rotate(square))
```

```swift
let fadeIn =
  linear(from: 0, to: 1, in: 1.0).time(easeInOut)
    .bind(container, with: \.alpha)
```

```swift
let animations = left + right
let final = fadeIn * animations
```

Note: How would we go about designing a library for this?

!!!

### Benefits

* Animations are values
* Animations compose cleanly
* Rich algebraic structure

Note: Let me speak a bit about it

!!!

### Library Design

(image)

Note: Algebra-driven

!!!

### Algebra-driven design

(see my other talk)

!!!

### Algebraic properties

(image)

(TODO: Paste all the algebra stuff here and blast through it)

!!!

### Semiring

```swift
// + forms a commutative monoid (empty = 0)
// * forms a monoid (empty = 1)
// * distributes over + (left and right)
// 0* annihalates (0 * x = x * 0 = 0)
protocol Semiring {
  func +(lhs: Self, rhs: Self) -> Self
  func *(lhs: Self, rhs: Self) -> Self
  var zero: Self { get }
  var one: Self { get }
}
```

Note: Now we just need to find a value with which we can implement this protocol

!!!

### Creativity with guide-rails

Note: I sat down with very smart people and we played around with different representations for a bit until we found one that works

!!!

### What is an animation

A value changing over time

!!!

### Animation at a high-level

```swift
/*
Animation :=
  | Value changing over time AND a non-zero duration
  | A cancelled animation (additive identity)
  | A trivial animation with no duration (multiplicative identity)
*/
```

Note: Remember, we need to keep the duration to get all our laws to work

!!!

### We use the algebra of datatypes and parametricity

```swift
public enum Animation<A> {
```

```swift
  case cancelled
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
  case trivial
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
  case runnable(
    duration: CFAbsoluteTime,
    value: (Progress) -> A)
}
```
<!-- .element: class="fragment" data-fragment-index="3" -->

!!!

### Animation Semiring

```swift
protocol Semiring {
  var one: Self { get }
  var zero: Self { get }
  func *(lhs: Self, rhs: Self) -> Self
  func +(lhs: Self, rhs: Self) -> Self
}

```

```swift
extension Animation : Semiring
```
<!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### Identities

```swift
  /// A multiplicative identity
  public static var one: Animation {
    return .trivial
  }
```

```swift
  /// An additive identity
  public static var zero: Animation {
    return .cancelled
  }
```
<!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### Sequence composition (mulitiplication)

```swift
static func *(lhs: Animation, rhs: Animation) -> Animation {
```

```swift
   switch (lhs, rhs) {
   case (.cancelled, _),
          (_, .cancelled):
         return .cancelled
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
   case (.trivial, let x),
        (let x, .trivial):
       return x
   /* ... */
}
```
<!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### Sequence composition (multiplication)

```swift
static func *(lhs: Animation, rhs: Animation) -> Animation {
    switch (lhs, rhs) {
    /* ... */
    case (._runnable(let duration1, let value1),
      ._runnable(let duration2, let value2)):
```

```swift
        let sum = duration1 + duration2
        let ratio = duration1 / sum
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
        return Animation.runnable(duration: sum) { t in
```
<!-- .element: class="fragment" data-fragment-index="3" -->

```swift
            t <= ratio
                ? value1(t / ratio)
                : value2((t - ratio) / (1 - ratio))
        }
    }
}
```
<!-- .element: class="fragment" data-fragment-index="4" -->

Note: SWIFTISM: Recall that we have this trailing closure syntax. This function is part of the .runnable constructor

!!!

### Parallel Composition

```swift
extension Animation where A: CommutativeSemigroup {
```

```swift
public static func +(lhs: Animation, rhs: Animation) -> Animation {
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
   switch (lhs, rhs) {
   case (.cancelled, .trivial),
        (.trivial, .cancelled),
        (.trivial, .trivial):
       return .trivial
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
   case (.cancelled, .cancelled):
       return .cancelled
   /* ... */
   }
}
```
<!-- .element: class="fragment" data-fragment-index="3" -->

!!!

### Parallel Composition

```swift
public static func +(lhs: Animation, rhs: Animation) -> Animation {
    /* ... */
    case (._runnable(let duration1, let value1),
    ._runnable(let duration2, let value2)):
```

```swift
        let newDuration = max(duration1, duration2)
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift

        return .runnable(duration: newDuration) { t in
            let a1 = value1(min(1, t * newDuration / duration1))
            let a2 = value2(min(1, t * newDuration / duration2))
            return a1 * a2
        }
```
<!-- .element: class="fragment" data-fragment-index="3" -->

```swift
    case (_, ._runnable(let d, let v)),
         (._runnable(let d, let v), _):
        return .runnable(duration:d, value:v)
    }
}
```
<!-- .element: class="fragment" data-fragment-index="4" -->

Note: This is the almost part (the semigroup constraint on the A)

!!!

### Validate laws!

![verify laws](img/verification.png)

Note: You can use quickcheck for the this, here is a not proof, but something at least.. Some extra bits

!!!

### Map

<div class="vidcode">
  <video playsinline autoplay muted loop style="width:60%;">
    <source src="vids/map.mp4" type="video/mp4">
  </video>
  ```
  nums.map{x in setRadius(x)}
  ```
</div>

!!!

### Layer2 primitives

```swift
public var reversed: Animation

```

```swift
public var looped: Animation

```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
public func delayed(by delay: CFAbsoluteTime) -> Animation

```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
public static func linear(
    from a: CGFloat,
    to b: CGFloat,
    in duration: CFAbsoluteTime
) -> Animation<CGFloatAverage>
```
<!-- .element: class="fragment" data-fragment-index="3" -->

!!!

### Layer3 primitives

```swift
public static var fadeOut: Animation

```

```swift
public static var translate: Animation

```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
public static var rotate: Animation

```
<!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### API Consumers are empowered

<div class="vidcode">
  <video playsinline autoplay muted loop style="width:60%;">
    <source src="vids/actors.mp4" type="video/mp4">
  </video>
  ```
  scene = .times(/*...*/, .plus(/*...*/))
  ```
</div>

!!!

### Secret Sauce

```swift
func intervals(/*...*/) -> [(String, Double, Double)]
```

Note: Think of "FreeSemiring" as: taking advantage of the algebraic structure of animations to analyze a big animation before evaluating it

!!!

### Scenes

<div class="vidcode">
  <video playsinline autoplay muted loop style="width:60%;">
    <source src="vids/actors.mp4" type="video/mp4">
  </video>
  ```
  scene = .times(/*...*/, .plus(/*...*/))
  ```
</div>

!!!

### Take-away

TODO

* <!-- .element: class="fragment" data-fragment-index="1" --> API design is about maximizing simplicity and expressivity
<!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> Composable APIs maximize these properties
<!-- .element: class="fragment" data-fragment-index="2" -->
* <!-- .element: class="fragment" data-fragment-index="3" --> We can find composition in animations
<!-- .element: class="fragment" data-fragment-index="3" -->
* <!-- .element: class="fragment" data-fragment-index="4" --> Don't stop at composition. More laws = More expressivity
<!-- .element: class="fragment" data-fragment-index="4" -->
* <!-- .element: class="fragment" data-fragment-index="5" --> These libraries empower API consumers to do amazing things!
<!-- .element: class="fragment" data-fragment-index="5" -->

Note: 5. Remember the layers

!!!

<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

Library: [https://github.com/bkase/swift-fp-animations](https://github.com/bkase/swift-fp-animations)

Slide Deck: [https://is.gd/x11G7Q](https://is.gd/x11G7Q)

!!!

## Appendix

!!!

## The interval function

```swift
    private static func intervals(_ frags: FreeSemiring<SceneFragment>) -> [(String, Double, Double)] {

        func helper(_ a: FreeSemiring<SceneFragment>, _ currDuration: CFAbsoluteTime, _ build: [(String, CFAbsoluteTime, CFAbsoluteTime)]) -> [(String, CFAbsoluteTime, CFAbsoluteTime)] {
            switch a {
            case ._one: return build
            case ._zero: return []
            case .single(let a): return build + [(a.name, currDuration, currDuration+a.animation.duration)]
            case .plus(let l, let r):
                let lBuild = helper(l, currDuration, build)
                return helper(r, currDuration, lBuild)
            case .times(let l, let r):
                let lBuild = helper(l, currDuration, build)
                let next = longestTime(lBuild)
                return helper(r, currDuration+next, lBuild)
            }
        }

        return helper(frags, 0, [])
    }
```

