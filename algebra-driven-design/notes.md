ABSTRACT
It's not immediately clear how to build "nice" libraries. Let's consider animations: Think of a box flying across an iPhone screen before fading out. How might we design this library? Writing the type first, before the implementation, is known and common in the functional programming community as a method for writing correct code, but is it the right way to build an API? Authors should consider something else even before the type: Composability. How can we make animations compose? We can combine two animations in sequence to produce an animation. And two animations in parallel is an animation, as well. But we shouldn't stop here. Documenting the laws to which these binary operations conform leads to a better understanding of the domain both for consumers and producers of a library. In this talk, we will introduce algebraic properties: associativity, commutativity, identity, idempotency, distributivity, etc. We will talk about the algebraic structures they induce on a binary operation: semigroup, monoid, commutative monoid, semiring, etc. At every step, we will make the abstract concrete by bringing it back to our animation example.

Only after we fully understand the laws around our animations' composition do we think about the type and the implementation of our animations. This is abstract-algebra-driven design. In Swift, a language without higher-kinded-types, we can still reify our knowledge of algebraic structures into software! Here, we'll look at an implementation of the animation library we designed earlier. By the end of the talk, it will be clear how to build complex animations out of our simple, lawful, composable primitives. More importantly, we'll understand basic abstract algebra and its application to API design!



ROUGH:

Open with an animation playing
How should we define a library that lets us build these things?
Goals : then the sequence...
Composable APIs are a good way to get closer to the goal
We can compose animations in parallel or in sequence
Composition is great, but how can we clearly describe the behaviors
We know we want composition, but we want the composition to behave...
What's a law? Laws can help.
Let's let algebra guide our design!

Commutativity
Identity
Associativity
...

Now we have our abstract model in our head. We know how we want it to behave. Now we can start implementing it.

Values change over time:

```swift
public struct Animation<A> {
  public let duration: CFAbsoluteTime
  private let _value: (CFAbsoluteTime) -> A
}
```

Animation is a functor and applicative functor, this means we can `map` and `zip` animations.

Now we can turn an int changing over time into an x coordinate of a view. We can combine two floats and then map to change x and y positions. Moreover we can combine and map to describe complex primitives...

But we want to compose multiple animations:

Sequence two animations

```swift
/// Sequences two animations.
public static func * (lhs: Animation, rhs: Animation) -> Animation {
  let sum = lhs.duration + rhs.duration
  let ratio = lhs.duration / sum

  return Animation(duration: sum) { t in
    (t <= ratio && lhs.duration != 0)
      ? lhs.value(t / ratio)
      : rhs.value((t - ratio) / (1 - ratio))
  }
}
```

Animations in parallel means we need to be able to combine the two animating values

```swift
extension Animation where A: Semigroup {
  /// Runs two animations in paralllel and combines the results. If one is longer than the other, the shorter one will stop at it's
  /// last value until the longer one finishes.
  public static func + (lhs: Animation, rhs: Animation) -> Animation {
    let newDuration = max(lhs.duration, rhs.duration)
  
    return .init(duration: newDuration) { t in
      let a1 = lhs.value(min(1, t * newDuration / lhs.duration))
      let a2 = rhs.value(min(1, t * newDuration / rhs.duration))
      return a1 <> a2
    }
  }
}
```

Let's verify that our implementation matches our spec in our head

```swift
let step1 = linear(from: 0, to: 200, in: 1)
let step2 = linear(from: 50, to: 200, in: 3)
let step3 = linear(from: 200, to: 300, in: 1)
let sequenced = step1 * step2 * step3

sequenced.value(0)
sequenced.value(0.2)
sequenced.value(0.4)
sequenced.value(0.6)
sequenced.value(0.8)
sequenced.value(1)

let paralleled = step1 + step2

paralleled.value(0)
paralleled.value(0.2)
paralleled.value(0.4)
paralleled.value(0.6)
paralleled.value(0.8)
paralleled.value(1)

// verification of associativity:
let assoc1 = step1 * (step2 * step3)
let assoc2 = (step1 * step2) * step3
"\(assoc1.value(0.5).avg) == \(assoc2.value(0.5).avg)"
"\(assoc1.value(0.25).avg) == \(assoc2.value(0.25).avg)"
"\(assoc1.value(0.75).avg) == \(assoc2.value(0.75).avg)"

// verification of associativity, commutativity of +
let assocP1 = step1 + (step2 + step3)
let assocP2 = (step1 + step2) + step3
let commuteP = (step2 + step1) + step3
"\(assocP1.value(0.5).avg) == \(assocP2.value(0.5).avg) == \(commuteP.value(0.5).avg)"
"\(assocP1.value(0.25).avg) == \(assocP2.value(0.25).avg) == \(commuteP.value(0.25).avg)"
"\(assocP1.value(0.75).avg) == \(assocP2.value(0.75).avg) == \(commuteP.value(0.75).avg)"

// (almost) proof of (one-side) of distributivity
// let A, B, C be animations
// WTS: A * (B + C) = A * B + A * C
// ->
//  A * (B + C)
//  A * (B <> C) (since + forms a semigroup now)
//  A and then (B <> C) (by definition)
//  (A and-then B) <> (A and-then C) (I can't prove this step but it feels right Please help here)
//  (A * B) + (A * C) by defition

// verification of distributivity:

let distLhs = step1 * (step2 + step3)
let distRhs = (step1 * step2) + (step1 * step3)
"\(distLhs.value(0.1).avg) == \(distRhs.value(0.1).avg)"
"\(distLhs.value(0.2).avg) == \(distRhs.value(0.2).avg)"
"\(distLhs.value(0.3).avg) == \(distRhs.value(0.3).avg)"
"\(distLhs.value(0.4).avg) == \(distRhs.value(0.4).avg)"
"\(distLhs.value(0.5).avg) == \(distRhs.value(0.5).avg)"
"\(distLhs.value(0.6).avg) == \(distRhs.value(0.6).avg)"

// verification of multiplicative identity
let onLeft = step1 * .one
let onRight = .one * step1
"\(step1.value(0.5).avg) == \(onLeft.value(0.5).avg) == \(onRight.value(0.5).avg)"
"\(step1.value(0.25).avg) == \(onLeft.value(0.25).avg) == \(onRight.value(0.25).avg)"
"\(step1.value(0.75).avg) == \(onLeft.value(0.75).avg) == \(onRight.value(0.75).avg)"

// Therefore we have a semiring (with no additive identity)!
// it's also morally an idempotent semiring if you consider two things equivalent that have the same averages

typealias AAtoBBtoAB<A: Semigroup,B: Semigroup> = (Tuple2<A, A>) -> (Tuple2<B, B>) -> Tuple2<A, B>
```

Scaling time!



NOTES:
1. Make title animate title
2. Why are iOS and Android bad -- explain it ;; introduce the goals then analyze the two
3. Why should we use Math -- motivate it ;; objectivity and more princippled approach guide you. Don't let your feelings or your gut drive
4. Show the code for the title slide somewhere -- beautiful idealized form.
5. Watch my old videos "shared language across mathematicians and programmers"
6. On the `x <> (y <> z)` slide you can show it maybe
7. Maybe do animations for the digrams
8. Remove "Cancelled Animations" slide
9. Sequence distributes over parallel
10. Don't forget sequence is *
11. Make the distribute less confusion
12. Show sequence and parallel together; use the times and addition. I can introduce Delay
13. Now that that we've done this principled reasoning ... there's no confusion
14. See it visually then show the code
15. POWER: Drop the optional -- (fade or empty) instead
16. I'll do another slide on the min part of parallel
17. Here's an example of me moving numbers around. I won't even call out the category theory, we could keep going but I'm going to stop here


