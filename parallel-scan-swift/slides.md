<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Parallel Periodic Scan

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

!!!

### Coda

(picture)

!!!

### Succinct blockchain

(picture)

Note: .. powered zksnarks

!!!

## zk-SNARKs

(picture)

!!!

## ZkSnark definition

* Zero
* Knowledge
* Succinct
* Non-interactive
* Argument of
* Knowledge

Note: There exists DATA s.t. PROPERTY

!!!

## SNARKs are slow to construct

(picture)

Note: Not about snarks, it's about dealing with the slowness of SNARK proof construction

!!!

### Plan

1. Concrete problem
2. Abstract
3. Goal
4. Iterate
5. Instantiate

!!!

## Process Transactions

(picture of state transition (singular))

!!!

## Process Transactions

(picture of state transitions (plural))

!!!

### Aside: Scan

```swift
extension Array {
  func scan<U>(init: U, f: (U, Element) -> U) -> [U]
}
```

Note: It's almost like reduce, but you get the intermediate results

!!!

### Aside: Scan on a stream

```swift
extension Stream {
  func scan<U>(init: U, f: (U, Element) -> U) -> Stream<U>
}
```

Note: In Rx this is called: `X` in ReactiveSwift it's called `Y`

!!!

## Throughput is bad

(picture)

Note: and then we have a bad cryptocurrency

!!!

## But what if we hold back some transactions

(picture of batching transactions but not doing anything with them)

!!!

## And then handle them all at once somehow?

(picture of question mark on the tree-like thing)

!!!

## Merge SNARKs

(picture)

!!!

## Aside: Periodic scan

```swift
extension Stream {
  func periodicScan<A>(
    init: A,
    lift: Element -> A,
    merge: (A, A) -> A
  ) -> Stream<A>
}
```

Note: It's kind of like this but it helps if we model it differently

!!!

## Okay enough already, let's abstract

(picture)

!!!

### Finding the problem

1. Assume things
2. Abstract things
3. State the goal

!!!

### Let's make some assumptions

(picture)

!!!

### Assumption 1: Fixed rate of transactions R

(picture)

Note: We always have more transactions available than we can include in a transition

!!!

### Assumption 2: Infinite cores

(picture)

Note: Since we can farm out work to the network, let's just assume we have an infinite core machine (we want to max out parallelism)

!!!

### Finding the problem

1. Assume things
2. Abstract things
3. State the goal

!!!

### Abstract away details

(picture)

!!!

### Just data

(picture)

Note: No more transactions

!!!

### Expensive computation

(picture)

Note: No more proofs

!!!

### Associative merge operation

(picture)

Note: No merge proofs

!!!

### Finding the problem

1. Assume things
2. Abstract things
3. State the goal

!!!

### State the goal

_Efficiently compute a periodic scan on an infinite stream pumping at some target rate, prefer maximizing throughput, then minimizing latency, then minimizing size of state on an infinite core machine._

!!!

### Shared Model

(picture showing stepping)

Note: Work happens out of band -- one step is taken at a time

!!!

### Swift!

(picture)

!!!

### Jobs

```swift
enum Job {
  // a namespace
  enum Incomplete<Data, A> { /* ... */ }
  enum Available<Data, A> { /* ... */ }
  enum Complete<A> { /* ... */ }
}
```

!!!

### Jobs: Incomplete

```swift
enum Incomplete<Data, A> {
  case base(Data?)
  case merge(A?,A?)
}
```

!!!

### Jobs: Available

```swift
enum Available<Data, A> {
  case base(Data)
  case merge(A,A)
}
```


!!!

### Jobs: Complete

```swift
enum Complete<A> {
  case lifted(A)
  case merged(A)
}
```

!!!

### State

```swift
protocol ScanState {
  associatedtype Data
  associatedtype A

  /* ... */
}
```

!!!

### State

```swift
  func nextJobs() -> [Job.Available<Data,A>]
```

!!!

### State

```swift
  func step(
    completedJobs: [Job.Complete<Data,A>]
  ) -> Result<A?>
}
```

!!!

### Compute time

All units of work take the same amount of time

!!!

### More use-cases

(picture)

!!!

### Problem

_Efficiently compute a periodic scan on an infinite stream pumping at some target rate, prefer maximizing throughput, then minimizing latency, then minimizing size of state on an infinite core machine._

!!!

### Astronomic Data Processing

(picture)

Note: Non-parametric models, huge firehose of data with expensive compute on it

!!!

### Livestream Analysis

(picture)

Note: Firehose of data; some associative combine

!!!

### Naive Solution

(all the pics needed to explain things)

!!!

### Naive Solution

(nextJobs)

!!!

### Naive Solution

(step)

!!!

### Analysis

_Efficiently compute a periodic scan on an infinite stream pumping at some target rate, prefer maximizing throughput, then minimizing latency, then minimizing size of state on an infinite core machine._

!!!

### Analysis

1. We are computing a periodic scan on an infinite stream pumping at some rate R
2. We have increased throughput at the cost of some latency when compared with the serial approach

!!!

### Problem?

(picture)

Note: Parallelism is halved every layer!

!!!

### More trees

(picture)

Note: Let's take the R data pieces that are available at every step

!!!

### More trees

(pictures...)

Note: Trace a run through

!!!

### More trees

(picture log(n))

!!!

### Analysis

We increased throughput again at the cost of some latency

!!!

### Let's do better!

Let's try to attack the size

!!!

### Waste of space

(picture)

Note: Once we process some layer of the tree, it becomes useless, higher layers are useless. Let's just not store that.

!!!

### Overlay the trees

(picture)

Note: We have the "frontiers" of the log n trees at each layer

!!!

### Analysis

(picture)

Note: Same throughput and latency, but now we drastically reduced size

!!!

### More size shrinking!

!!!

### Succinct data structures

(picture of stuffing in an array)

Note: But no time for this in details

!!!

### Instantiate

(picture)

Note: Now we can do the snark proof work for transactions faster, and have higher throughput on the cryptocurrency

!!!

### Future Work

(picture)

!!!

### Take aways

1. Concrete problem
2. Abstract
3. Goal
4. Iterate
5. Instantiate

!!!

<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

Slide Deck: [https://is.gd/bWsLhD](https://is.gd/bWsLhD)

