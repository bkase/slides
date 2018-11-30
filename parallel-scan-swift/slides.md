<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Scanning for scans

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

!!!

### Coda

<img src="img/coda-logo.svg"></img>

!!!

### Succinct blockchain

(picture -- from izzy)

Note: .. powered zksnarks

!!!

### ZkSnark definition

* Zero
* Knowledge
* Succinct
* Non-interactive
* Argument of
* Knowledge

Note: There exists DATA s.t. PROPERTY

!!!

### Simple SNARK

_There exists_ a pdf file _such that_ the hash of the file is ab3df33...

Note: And the fact that I am able to construct this means I KNOW such a PDF file

!!!

### Recursive composition

<img src="img/zksnark.svg" style="height:60%;width:60%"></img>

!!!

### Blockchain SNARK

_There exists_ a proof from genesis to now _and_ some new transactions _and_ consensus metadata _such that_ the proof verifies _and_ all transactions are valid _and_ the metadata is consistent.

!!!

### SNARKs are slow to construct

(picture)

Note: Not about snarks, it's about dealing with the slowness of SNARK proof construction. So we actually do:

!!!

### Blockchain SNARK

_There exists_ a proof from genesis to now _and_ a proof that the transactions changed our account state _and_ consensus metadata _such that_ both proofs verifies _and_ the metadata is consistent.

Note: There are (at least) two different proofs floating around

!!!

### Latex it up

![blockchain state](img/blockchain-state.png)

Note: Blockchain state is big sigma

!!!

### Latex it up

![blockchain state detailed](img/blockchain-state-detailed.png)

Note: Part of that state is the state of everyone's accounts

!!!

### Latex it up

![blockchain state proof](img/blockchain-state-proof.png)

!!!

### Latex it up

![transaction data](img/single-transaction.png)

!!!

### Latex it up

![transaction proof](img/transaction-proof.png)

!!!

### Process Transactions Serially

![serial transaction processing](img/serial-proofs-concrete.png)

!!!

### Scan

```swift
extension Array {
  func scan<U>(init: U, f: (U, Element) -> U) -> [U]
}
```

Note: It's almost like reduce, but you get the intermediate results

!!!

### Scan on a stream

```swift
extension Stream {
  func scan<U>(
    init: U,
    f: (U, Element) -> U
  ) -> Stream<U>
}
```

Note: In Rx this is called: `X` in ReactiveSwift it's called `Y`

!!!

### Process Transactions Serially

![serial transaction processing](img/serial-proofs-concrete.png)

Note: Look again now that you know scan

!!!

### Low throughput

![lean book think](img/lean-book-think2.jpg)

> https://c1.staticflickr.com/1/740/31689460193_18d613a3d8_b.jpg

Note: and then we have a bad cryptocurrency

!!!

### Plan to scan for a scan

1. <s>Concrete problem</s>
2. <!-- .element: class="fragment" data-fragment-index="1" --> Properties and requirements <!-- .element: class="fragment" data-fragment-index="1" -->
3. <!-- .element: class="fragment" data-fragment-index="2" --> Iterate <!-- .element: class="fragment" data-fragment-index="2" -->
4. <!-- .element: class="fragment" data-fragment-index="3" --> Instantiate <!-- .element: class="fragment" data-fragment-index="3" -->

!!!

### Properties

![book stack](img/book-stack2.jpg)

>  http://dl.maxpixel.freegreatpicture.com/?f=books-1082949_1280.jpg&type=Download&token=0089c38f1e35d52df822cbbcd97116fc&pid=1082949

Note: In order to derive requirements

!!!

### Transaction proofs and blockchain proofs are slow

!!!

### Transactions arrive at some rate R

!!!

### Proof work can be done by others

!!!

### Merge proofs

![merge proofs exist](img/merging-exists.png)

!!!

### Merge Associative

![merge proofs associative](img/merging-associative.png)

!!!

### Process Transactions

![merge proofs concrete](img/merge-proofs-concrete.png)

!!!

### Periodic Scan

```swift
extension Stream {
  func periodicScan<P>(
    init: P,
    lift: Element -> P,
    merge: (P, P) -> P
  ) -> Stream<P>
}
```

Note: A scan-like operation that is more optimizable

!!!

### Requirements

![timeless book](img/timeless-book2.jpg)

> https://upload.wikimedia.org/wikipedia/commons/thumb/d/d6/Timeless_Books.jpg/1024px-Timeless_Books.jpg

!!!

### Requirements

* Maximize throughput
* <!-- .element: class="fragment" data-fragment-index="1" --> Minimize latency <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> Minimize state size <!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### Abstract

![abstract](img/abstract.jpg)

> https://wallpaperstudio10.com/wallpaper-abstract_colorful-67718.html

Note: There are just too many sigmas

!!!

### Data

![D = s1 T1 s2](img/define-data.png)

!!!

### Base

![B = s0 -> s1](img/define-proof.png)

!!!

### Merges

![M12 = s0 -> s2](img/define-merge.png)

!!!

### Outter value

![A = proof S](img/define-a.png)

!!!

### Scanning

![naive with scan](img/naive-with-scan.png)

!!!

### Plan to scan for a scan

1. <s>Concrete problem</s>
2. <s>Properties and requirements</s>
3. Iterate
4. Instantiate

!!!

### Iterate

![so many books](img/so-many-books2.jpg)

> https://pixabay.com/en/books-library-knowledge-tunnel-21849/

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

### Naive Solution

![naive with scan](img/naive-with-scan.png)

!!!

### Analysis

* Throughput up
* <!-- .element: class="fragment" data-fragment-index="1" --> Latency increased <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> State size larger <!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### Problem?

<img alt="thinking face" src="img/thinking-face2.jpg" width="75%" height="75%"/>

> http://maxpixel.freegreatpicture.com/Face-Female-Girl-Looking-Adult-Isolated-Cute-15814

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

* Throughput up more
* <!-- .element: class="fragment" data-fragment-index="1" --> Latency same <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> State size larger <!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### Let's do better!

![great it works](img/great-it-works2.jpg)

> https://static.pexels.com/photos/269474/pexels-photo-269474.jpeg

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

* Throughput same
* <!-- .element: class="fragment" data-fragment-index="1" --> Latency same <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> State size smaller <!-- .element: class="fragment" data-fragment-index="2" -->

Note: Same throughput and latency, but now we drastically reduced size

!!!

### More size shrinking!

![great it works](img/great-it-works2.jpg)

> https://static.pexels.com/photos/269474/pexels-photo-269474.jpeg

!!!

### Packing information

<img src="img/job-view.png" style="height:60%;width:60%"></img>

!!!

### Succinct data structures

![succinct data structures](img/succinct.png)

Note: "Implicit Heap"; I'll post a link to wikipedia at the end

!!!

### Instantiate

![instantiate](img/instantiated.png)

Note: Now we can do the snark proof work for transactions faster, and have higher throughput on the cryptocurrency

!!!

### We scanned for a scan

1. Concrete problem
2. <!-- .element: class="fragment" data-fragment-index="1" --> Properties and requirements <!-- .element: class="fragment" data-fragment-index="1" -->
3. <!-- .element: class="fragment" data-fragment-index="2" --> Iterate <!-- .element: class="fragment" data-fragment-index="2" -->
4. <!-- .element: class="fragment" data-fragment-index="3" --> Instantiate <!-- .element: class="fragment" data-fragment-index="3" -->

!!!

<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

Slide Deck: [https://is.gd/bWsLhD](https://is.gd/bWsLhD)

Succinct Datastructures: [https://is.gd/1q22MX](https://is.gd/1q22MX)

!!!

## Appendix

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


