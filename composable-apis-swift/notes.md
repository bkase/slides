Title?
Making the subjective objective: Composable algebraic APIs are elegant?
Algebraic APIs
Applying Abstract Algebra at APIs

You don't need HKTs to get use out of thinking algebraically

Composition lets us amplify the amount of expressive power we have relative to the number of primitives in our library.

We are all familiar with composable arithmatic APIs
5 + 10 + 400 = 415
What if our numbers couldn't compose with addition if they had a different amount of digits
/// 1,2,3,4,5 etc
/// NOT 10 or 100 or 1000
protocol OnesNumber {
  /// throws if you add something that causes a HundredsNumber
  func addWithTens(other: TensNumber) -> TensNumber throws
  /// throws if you add something that causes a ThousandsNumber
  func addWithHundreds(other: HundredsNumber) -> HundredsNumber throws
  /*...*/
}
protocol TensNumber
protocol HundredsNumber

This is the status quo with most libraries we use.

Given some amount of expressive power we want our APIs to have, the simplest design is one with the fewest number of types, functions, and values to solve that problem safely

How do we do better?

Let's look at numbers: Composition gives us the freedom to have very few types and methods and still express all sorts of computations.
But there's more hidden in our numbers:
Arithmetic obeys many algebraic laws -- these laws can be applied to values other than just numbers!

While composition helps us minimize the surface area of our types, function, values
Various laws over the composition helps us reason over our code and give us more expressive power

Building blocks:
  * Closed binary operation: (T, T) -> T
  * Associativity
    * No need for parentheses; run different parts in parallel
    * x + y + z = (x + y) + z = x + (y + z)
  * Commutativity
    * order doesn't matter; non-deterministic concurrency okay
    * x + y = y + x
  * Distributivity
    * factor out common chunks of code in business logic to reduce code and compute needed to solve a problem
    * x*y + x*z = x * (y + z)
  * Idempotence
    * freedom to do things more than once; out-of-order execution in a distributed system
    * max(5, 6) = 6 and max(max(5, 6), 6) = 6
  * Identity
    * not doing something is still a value of the same type (no need for option etc)
    * 0 + x = 0
  * Inverse
    * I can undo anything
    * x + (-x) = 0
  * Annihilation
    * I can stop anything
    * 0*x = 0

Mathmeticians in abstract algebra have already talked about various combinations of these properties. The names may sound scary, but all it is is a mapping to a group of these properties:

semigroup = one operator + associativity
monoid = one operator + associativity + identity
group = one operator + associativity + identity + inverse
abelian group = one operator + associativity + identity + inverse + commutativity
right semiring = two operators: (+, 0) a monoid; (*, 1) a monoid; 0*x annihalates; * distributes over + rightwards

You may ask, why do we care about these scary names. The reason is you can read about them on the wikipedia and talk to mathmaticians to discover power in your APIs. Theorems on the algebraic structure are generic methods in APIs.
Also there has been lots of academic research lately on applying these algebraic structures to software. There is something to be learned there.

3 problems:

* Caching large pieces of data
* Managing the cancellation of various asynchronous tasks
    Semigroup
    Monoid
    Commutative Monoid
    Idempotent
    Bounded semilattice

```swift
//  Write some awesome Swift code, or import libraries like "Foundation",
//  "Dispatch", or "Glibc"
struct Monoid<T> {
    let empty: T
    let append: (T, T) -> T
    
    func fold(_ arr: [T]) -> T {
        return arr.reduce(empty, append)
    }
    
    func foldMap<R>(_ arr: [R], _ f: (R) -> T) -> T {
        return fold(arr.map(f))
    }
}

// TODO: Atomic this
func once(_ f: @escaping () -> ()) -> () -> () {
    var ran = false
    return {
        if !ran {
            f()
            ran = true
        }
    }
}

struct CancelToken {
    static let monoid: Monoid<CancelToken> =
        Monoid(empty: CancelToken(cancel: {}), append: { f, g in 
            CancelToken(cancel: { f.cancel(); g.cancel() }) 
        })

    private let fn: () -> ()
    init(cancel: @escaping () -> ()) {
        self.fn = once(cancel)
    }
    
    func cancel() {
        fn()
    }
}

let c1 = CancelToken(cancel: { print("cancel 1") })
let c2 = CancelToken(cancel: { print("cancel 2") })
CancelToken.monoid.fold([c1, c2, c1, c2]).cancel()
```

* Choreographed Animations (non-interactive) FORMS: Right idempotent semiring
    An animation is just a curve function with a duration
    It can be cancelled
    If I have animations A and B, I should be able to construct C = A + B in parallel
    Parallel composition can be:
      associative
      commutative
      idempotent
    If I have animations A and B, I should be able to construct C = A * B in sequence
    Sequential composition can be:
      associative
    A parallel identity is a cancelled animation
      * we can't have invertible parallel composition now so we are a semiring, not a ring
    A sequential identity is an animation of duration length zero

```swift
//  Write some awesome Swift code, or import libraries like "Foundation",
//  "Dispatch", or "Glibc"
import Dispatch

/*protocol Seeded {
    typealias Seed
    var initialState
}*/
func setInterval(_ interval: Double, block: @escaping ()->Bool) {
    DispatchQueue.main.asyncAfter(deadline: .now() + interval) {
        let shouldContinue = block()
        if shouldContinue {
        print("I want to continue")
            setInterval(interval, block: block)
        }
    }
}

let fixedMax: (Int64, Int64) -> Int64 = { x, y in max(x, y) }

typealias ZeroToOneInclusive = Double
struct Seed {
   let initialState: ZeroToOneInclusive
   let finalState: ZeroToOneInclusive
}
struct Opacity {
    // TODO: Use seed
    static func run(animation: Animation, step: ZeroToOneInclusive) {
        print("Opacity @ \(animation.curve(.opacity, step))")
    }
}
struct XPosition {
    // TODO: Use seed
    static func run(animation: Animation, step: ZeroToOneInclusive) {
        print("XPosition @ \(animation.curve(.opacity, step))")
    }
}
// TODO: Think of some way to make this open to extension
enum SomeProp {
    case opacity
    case xposition
}
struct AllProps {
    let opacity: Seed
    let xposition: Seed
}
enum Time {
    case cancelled
    case millis(Int64)
    
    var isCancelled: Bool {
        switch self {
        case .cancelled: return true
        default: return false
        }
    }
    
    // ignores cancellation (parallel)
    func max(_ other: Time) -> Time {
        switch (self, other) {
            case (.cancelled, .millis(_)), 
                 (.cancelled, .cancelled): return other
            case (.millis(_), .cancelled): return self
            case (.millis(let x), .millis(let y)): 
                return .millis(fixedMax(x, y))
        }
    }
    
    // cancel annihalates rightwards
    func plus(_ other: Time) -> Time {
        switch (self, other) {
            case (.cancelled, .millis(_)),
                 (.cancelled, .cancelled): return .cancelled
            case (.millis(_), .cancelled): return self
            case let (.millis(x), .millis(y)): return .millis(x + y)
        }
    }
    
    func proportion(_ other: Time) -> Double {
        switch (self, other) {
            case (.cancelled, .cancelled),
                 (.cancelled, .millis(_)): return 0
            case (.millis(_), .cancelled): return 1
            case let (.millis(x), .millis(y)): return Double(x) / Double(x + y)
        }
    }
}
struct Animation {
// TODO: Cancellation
    let curve: (_ prop: SomeProp, _ time: ZeroToOneInclusive) -> ZeroToOneInclusive?
    let duration: Time
    
    static let one = Animation(
        curve: { p, t in t },
        duration: .millis(0)
    )

    static let zero = Animation(
        curve: { p, t in 1.0 },
        duration: .cancelled
    )
    
    func parallel(_ other: Animation) -> Animation {
        return Animation(
            curve: { prop, time in 
                let curve1contrib = self.curve(prop, time)
                let curve2contrib = other.curve(prop, time)
        
                switch (curve1contrib, curve2contrib) {
                case (.none, .none): return .none
                case (.none, let x): return x
                case (.some(let x), .none): return x
                case (.some(let x), .some(let y)): return (x + y)/2
                }
            },
            duration: duration.max(other.duration)
        )
    }
    
    func sequence(_ other: Animation) -> Animation {
        if duration.isCancelled {
            return self
        }
        
        let aProportion = duration.proportion(other.duration)
        let bProportion = 1 - aProportion
        return Animation(
            curve: { prop, time in
                if time < aProportion {
                    return self.curve(prop, time * (1 / aProportion))
                } else {
                    return other.curve(prop, time * (1 / bProportion) - 1)
                }
            },
            duration: duration.plus(other.duration)
        )
    }

    func run(_ seed: Seed) {
        func doAnimation(step: ZeroToOneInclusive) {
            Opacity.run(animation: self, step: step)
            XPosition.run(animation: self, step: step)
        }
    
        switch duration {
            case .cancelled:
                print("Cancelled, not running")
            case .millis(let ms):
                let totalTime: Double = Double(ms) * 1000;
                var currentTime = Double(0)
                // HACK: mostly 60fps?
                doAnimation(step: 0)
                print("pre")
                setInterval(0.016) {
                    print("post")
                    currentTime += 0.016
                    let stepPosition = currentTime / totalTime
                    doAnimation(step: min(stepPosition, 1.0))
                    if stepPosition >= 1.0 {
                        return false
                    }
                    return true
                }
        }
    }
}

let linear: (Double) -> Double = { t in t }
let quad: (Double) -> Double = { t in t*t }
// TODO: this could be nicer with prisms I think
let opacityLinear = Animation(curve: { p, t in 
    switch p {
    case .opacity: return linear(t)
    default: return .none
    }
}, duration: .millis(100))
let xpositionQuad = Animation(curve: { p, t in
    switch p {
    case .xposition: return quad(t)
    default: return .none
    }
}, duration: .millis(200))
opacityLinear.parallel(xpositionQuad).run(Seed(initialState: 0, finalState: 0))
```

