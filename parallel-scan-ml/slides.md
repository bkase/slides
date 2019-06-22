<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Fast Accumulation on Streams

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

Note: Motivated by a concrete problem

!!!

### Coda

<img src="img/CodaLogo.svg" width="200%" height="200%"></img>

!!!

### Blockchain

<img src="img/blockchain.png"></img>

!!!

### Succinct blockchain

<img src="img/tiny-leaf.jpg" width="70%" height="70%"></img>

> https://www.flickr.com/photos/75001512@N00/4450040533

Note: .. powered zksnarks

!!!

### ZkSnark Acronym

* Zero
* Knowledge
* Succinct
* Non-interactive
* Argument of
* Knowledge

!!!

### SNARK statements

_There exists_ data _such that_ predicates hold on that data

Note: There exists DATA s.t. PROPERTY

!!!

### Simple SNARK

_There exists_ a pdf file _such that_ the hash of the file is ab3df33...

Note: And the fact that I am able to construct this means I KNOW such a PDF file

!!!

### Recursive composition

_There exists_ a proof of this statement _such that_ that proof verifies

Note: Recursion!

!!!

### Recursive composition

<img src="img/zksnark.svg" style="height:60%;width:60%"></img>

!!!

### SNARKs are slow to construct

<img src="img/snail.jpg" width="70%" height="70%"></img>

> https://pixabay.com/en/snail-slow-shell-mollusk-close-up-1844618/

Note: Not about snarks, it's about dealing with the slowness of SNARK proof construction. So we actually do:

!!!

### Latex it up

![account state latex](img/account-state.png)

![transaction data](img/single-transaction.png)
<!-- .element: class="fragment" data-fragment-index="1" -->

![transaction proof](img/transaction-proof.png)
<!-- .element: class="fragment" data-fragment-index="2" -->

Note: Part of that state is the state of everyone's accounts

!!!

### Process Transactions Serially

![serial transaction processing](img/merge-tree3.png)

Note: Squint

!!!

### Fold

```haskell
-- foldl (+) 0 [1,2,3,4] => 10

foldl :: (b -> a -> b) -> b -> [a] -> b
```

Note: This is likely review for most of this audience

!!!

### Scan

```haskell
-- scanl (+) 0 [1,2,3,4] => [1,3,6,10]

scanl :: (b -> a -> b) -> b -> [a] -> [b]
```

Note: It's almost like reduce, but you get the intermediate results

!!!

### Scan on a stream

```haskell
scan :: (b -> a -> b) -> b -> Stream a -> Stream b
```

Note: A push-based async stream

!!!

### Process Transactions Serially

![serial transaction processing](img/merge-tree3.png)

Note: Look again now that you know scan

!!!

### Low throughput

![lean book think](img/lean-book-think2.jpg)

> https://c1.staticflickr.com/1/740/31689460193_18d613a3d8_b.jpg

Note: Now we understand the real problem, great.

!!!

### This talk

1. <s>Concrete problem</s>
2. <!-- .element: class="fragment" data-fragment-index="1" --> Properties and requirements <!-- .element: class="fragment" data-fragment-index="1" -->
3. <!-- .element: class="fragment" data-fragment-index="2" --> Iterate <!-- .element: class="fragment" data-fragment-index="2" -->

Note: Generalize it, and then iterate improving on various performance aspects..

!!!

### Properties

![book stack](img/book-stack2.jpg)

>  http://dl.maxpixel.freegreatpicture.com/?f=books-1082949_1280.jpg&type=Download&token=0089c38f1e35d52df822cbbcd97116fc&pid=1082949

Note: In order to derive requirements

!!!

### Proofs are slow to construct

!!!

### Transactions arrive at some rate R

!!!

### Exists a distributed work pool

Note: That scales with R

!!!

### Merge proofs

![merge proofs exist](img/merging-exists.png)

!!!

### Merge Associative

![merge proofs associative](img/merging-associative.png)

!!!

### Requirements

![timeless book](img/timeless-book2.jpg)

> https://upload.wikimedia.org/wikipedia/commons/thumb/d/d6/Timeless_Books.jpg/1024px-Timeless_Books.jpg

!!!

### Requirements

* Maximize data throughput
* <!-- .element: class="fragment" data-fragment-index="1" --> Minimize latency <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> Minimize state size <!-- .element: class="fragment" data-fragment-index="2" -->

Note: In this order

!!!

### Abstract

![abstract](img/abstract.jpg)

> https://wallpaperstudio10.com/wallpaper-abstract_colorful-67718.html

Note: There are just too many sigmas

!!!

### Data

![A_1^2 = s1 T1 s2](img/define-a.png)

![B_0^1 = s0 -> s1](img/define-b.png)
<!-- .element: class="fragment" data-fragment-index="1" -->

Note: You'll see how the notation fits together after our first example

!!!

### Plan

1. <s>Concrete problem</s>
2. <s>Properties and requirements</s>
3. Iterate

!!!

### Iterate

![so many books](img/so-many-books2.jpg)

> https://pixabay.com/en/books-library-knowledge-tunnel-21849/

!!!

### Naive Solution

![naive data](img/naive-1.png)

!!!

### Naive Solution

![naive base](img/naive-2.png)

!!!

### Naive Solution

![naive merge](img/naive-3.png)

!!!

### Naive Solution

![naive almost all](img/naive-4.png)

!!!

### Naive Solution

<img src="img/naive-5.png" width="70%" height="70%"></img>

!!!

### Periodic Scan

```haskell
-- periodicScan (+) id 0 1,2,3,4,5,6,7,8
--    => 10,36

periodicScan ::
  (b -> b -> b) ->
  (a -> b) ->
  b ->
  Stream a ->
  Stream b
```

Note: Some of you may be thinking oh this is like a parallel reduce, and yeah we'll start with that..

!!!

### Naive Solution

<img src="img/naive-5.png" width="70%" height="70%"></img>

Note: Discuss throughput / latency

!!!

### Analysis

* Throughput increased
* <!-- .element: class="fragment" data-fragment-index="1" --> Latency increased <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> State size larger <!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### Problem?

<img src="img/traffic_jam.jpeg" width="70%" height="70%"></img>

> https://upload.wikimedia.org/wikipedia/commons/a/a4/Moscow_traffic_congestion.JPG

Note: Parallelism is halved every layer!

!!!

### More trees

![better data](img/more-1.png)

Note: Let's take the R data pieces that are available at every step

!!!

### More trees

![better base](img/more-2.png)

Note: Trace a run through

!!!

### More trees

![better merge1](img/more-3.png)

!!!

### More trees

![better merge2](img/more-4.png)

!!!

### More trees

![better merge3](img/more-5.png)

!!!

### More trees

![better merge4](img/more-6.png)

!!!

### Analysis

* Throughput is perfect
* <!-- .element: class="fragment" data-fragment-index="1" --> Latency same <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> State size larger <!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### Let's do better!

![great it works](img/great-it-works2.jpg)

> https://static.pexels.com/photos/269474/pexels-photo-269474.jpeg

!!!

### Waste of space

<img src="img/waste.png" width="150%" height="50%"></img>

Note: Once we process some layer of the tree, it becomes useless, higher layers are useless. Let's just not store that.

!!!

### Overlay the trees

<img src="img/compressed.png" width="70%" height="70%"></img>

Note: We have the "frontiers" of the log n trees at each layer

!!!

### Overlay the trees

<img src="img/compressed-2.png" width="70%" height="70%"></img>

Note: We have the "frontiers" of the log n trees at each layer

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

<img src="img/rearranged.png" style="height:60%;width:60%"></img>

!!!

### Succinct data structures

![succinct data structures](img/succinct.png)

Note: Explain child/parent computable from just the index into the array "Implicit Heap"; I'll post a link to wikipedia at the end

!!!

### Other Applications

* Astronomical Data Processing
* <!-- .element: class="fragment" data-fragment-index="1" --> Livestream Analysis <!-- .element: class="fragment" data-fragment-index="1" -->

Note: Square killometre array in australia.. non-parametric machine learning.. More generally, any online-map-reduce type workloads

!!!

### Fast Accumulation on Streams

1. Concrete problem
2. <!-- .element: class="fragment" data-fragment-index="1" --> Properties and requirements <!-- .element: class="fragment" data-fragment-index="1" -->
4. <!-- .element: class="fragment" data-fragment-index="2" --> Iterate <!-- .element: class="fragment" data-fragment-index="2" -->

!!!

<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

Slide Deck: [https://is.gd/E69TRA](https://is.gd/6shi8b)

Coda: [https://codaprotocol.com](https://codaprotocol.com)

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


