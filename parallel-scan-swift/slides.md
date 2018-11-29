<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Solving a Cool Problem

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

!!!

### Coda

<img src="img/coda-logo.svg"></img>

!!!

### Succinct blockchain

(picture -- from izzy)

Note: .. powered zksnarks

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

## Simple SNARK

_There exists_ a pdf file _such that_ the hash of the file is x.

Note: And the fact that I am able to construct this means I KNOW such PDF

!!!

## zk-SNARKs

<img src="img/zksnark.svg" style="height:60%;width:60%"></img>

!!!

### Blockchain SNARK

_There exists_ a proof from genesis to now _and_ some new transactions _and_ consensus metadata _such that_ the proof verifies _and_ all transactions are valid _and_ the metadata is consistent.

!!!

## SNARKs are slow to construct

(picture)

Note: Not about snarks, it's about dealing with the slowness of SNARK proof construction. So we actually do:

!!!

### Blockchain SNARK

_There exists_ a proof from genesis to now _and_ a proof that the transactions changed our account state _and_ consensus metadata _such that_ both proofs verifies _and_ the metadata is consistent.

Note: There are (at least) two different proofs floating around

!!!

### Latex it up

(Sigma)

Note: Blockchain state is big sigma

!!!

### Latex it up

(Sigma with little sigma)

Note: Part of that state is the state of everyone's accounts

!!!

### Latex it up

(A Blockchain SNARK proof)

!!!

### Latex it up

(Transaction data)

!!!

### Latex it up

(Transaction proof)

!!!

## Process Transactions Serially

(TODO show diagram)

!!!

### Aside: Scan

```swift
extension Array {
  func scan<A>(init: A, f: (A, Element) -> A) -> [A]
}
```

Note: It's almost like reduce, but you get the intermediate results

!!!

### Aside: Scan on a stream

```swift
extension Stream {
  func scan<A>(
    init: A,
    f: (A, Element) -> A
  ) -> Stream<A>
}
```

Note: In Rx this is called: `X` in ReactiveSwift it's called `Y`

!!!

## Throughput is bad

(picture)

Note: and then we have a bad cryptocurrency

!!!

### Plan

1. <s>Concrete problem</s>
2. Requirements
3. Iterate
4. Instantiate

!!!

### Properties

(picture)

Note: In order to derive requirements

!!!

### Transaction proofs and blockchain proofs are slow

(picture)

!!!

### Merge proofs

(a series of pictures explaining merge proofs are associative)

!!!

### Proof work can be done by others

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

## Process Transactions

(TODO show the merge diagram more)

!!!

### Requirements

(picture)

!!!

### Maximize throughput

(picture)

!!!

### Minimize latency

!!!

### Minimize size of state

!!!

### State the goal

_Efficiently compute a periodic scan on an infinite stream pumping at some target rate, prefer maximizing throughput, then minimizing latency, then minimizing size of state on an infinite core machine._

!!!

### Abstract

(A := foo, D := bar, etc)

Note: Work happens out of band -- one step is taken at a time

!!!

### More use-cases

(picture)

!!!

### Astronomical Telescope Data

(picture)

Note: Non-parametric models, huge firehose of data with expensive compute on it

!!!

### Livestream Analysis

(picture)

Note: Firehose of data; some associative combine

!!!

### Naive Solution

![naive data](img/naive-data.png)

!!!

### Naive Solution

![naive base](img/naive-base.png)

!!!

### Naive Solution

![naive merge](img/naive-merge.png)

!!!

### Naive Solution

![naive almost all](img/naive-almost-all.png)

!!!

### Naive Solution

![naive all](img/naive-all.png)

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

![better data](img/better-data.png)

Note: Let's take the R data pieces that are available at every step

!!!

### More trees

![better base](img/better-base.png)

Note: Trace a run through

!!!

### More trees

![better merge1](img/better-merge1.png)

!!!

### More trees

![better merge2](img/better-merge2.png)

!!!

### More trees

![better merge3](img/better-merge3.png)

!!!

### More trees

![better merge4](img/better-merge4.png)

!!!

### More trees

![better merge5](img/better-merge5.png)

!!!

### More trees

![better merge6](img/better-merge6.png)

!!!

### Analysis

We increased throughput again at the cost of some latency

!!!

### Let's do better!

(picture)

!!!

### Waste of space

![waste of space](img/better-waste.png)

Note: Once we process some layer of the tree, it becomes useless, higher layers are useless. Let's just not store that.

!!!

### Overlay the trees

<img src="img/compress1.png" style="height:60%;width:60%"></img>

Note: We have the "frontiers" of the log n trees at each layer

!!!

### Overlay the trees

<img src="img/compress2.png" style="height:60%;width:60%"></img>

!!!

### Analysis

(picture)

Note: Same throughput and latency, but now we drastically reduced size

!!!

### Reveal the simplification

<img src="img/job-view.png" style="height:60%;width:60%"></img>

!!!

### More size shrinking!

!!!

### Succinct data structures

![succinct data structures](img/succinct.png)

Note: "Implicit Heap"; I'll post a link to wikipedia at the end

!!!

### Instantiate

![instantiate](img/instantiation.png)

Note: Now we can do the snark proof work for transactions faster, and have higher throughput on the cryptocurrency

!!!

### Take aways

1. Concrete problem
2. Requirements
3. Iterate
4. Instantiate

!!!

<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

Slide Deck: [https://is.gd/bWsLhD](https://is.gd/bWsLhD)

Succinct Datastructures: [https://is.gd/1q22MX](https://is.gd/1q22MX)

