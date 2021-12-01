<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Mina Susbsystem Pathing and Deepdive into Ouroboros Samasika Consensus

By <a href="#">Joseph Spadavecchia</a> and <a href="http://bkase.dev">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

Note: Mina subsystem pathing as a means to work out a way to organize work for the Mina Rust client ... Specifically focusing on an interesting aspect of Ouroboros Samasika

<!--

Task:
  * We at O(1) Labs need to break Mina into subsystems to propose milestones for the project
  * Two goals
    * Optimize the order by stability (as in unchanging) + the prefix needed for a browser node
  * I'm going to talk about the subsystems and how we evaluated this and then Joseph is going to do a deep dive on one of those susbsytems (consensus)
* Mina subsystem breakdown
  * Show all the subsystems chaotically (progressively revealing a diagram)
    * Serialization
    * Block verification
    * Block Selection (Consensus)
    * Transition frontier
    * Networking
    * Ledgers and other state
    * Transactions
    * Provers
    * Snark workers
  * Cross out the bits that aren't needed for a browser node
  * Work backwards from unstable to stable
  * Show the milestones
* And now Joseph will dive deep into block selection and consensus
-->

!!!

### Background

<img src="img/chainsafe.png" width="50%" />

Note: Chainsafe is building an alternate Mina client in Rust

!!!

### Background

<img src="img/o1.png" />

Note: O(1) Labs was tasked with specifying describing and (interestingly) pathing the work such that...

!!!

### Mina Susbsystem Pathing Goals

1. Choose that which will prefix to a secure browser node
2. <!-- .element: class="fragment" data-fragment-index="1" --> Optimize path for most stable subsystems first <!-- .element: class="fragment" data-fragment-index="1" -->

Note: Browser nodes in Mina are not light nodes, they have full security guarantees because we don't need all the blocks in the blockchain merely a tiny bit of succinct state. By stable, I mean O(1) labs is actively developing Mina and adding Snapps our smart contract layer. We don't want to waste time specing or implementing that which will become stale.

!!!

### Mina Specifying

<img src="img/spec.png" />

Note: As part of this work, we're also simultaneously producing a spec -- we want to give you a sense of interesting nuances that come up

!!!

### In this Talk

1. Brandon: Subsystem Pathing
2. <!-- .element: class="fragment" data-fragment-index="1" --> Joseph: Ouroboros Samasika and Min Window Density <!-- .element: class="fragment" data-fragment-index="1" -->

Note: Brandon will talk through the susbsytem pathing in more of a high-level sense; Joseph will do a deep dive into one of those subsystems focusing on an intresting nuance of ouroboros samasika, Mina's consensus algorithm.

!!!

### Susbsystem Pathing

<img src="img/pathing.png" />

!!!

### Subsystem Pathing Goals

1. Choose that which will prefix to a secure browser node
2. <!-- .element: class="fragment" data-fragment-index="1" --> Optimize path for most stable subsystems first <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### OCaml Mina Client

<img src="img/01.png" />

!!!

### OCaml Mina Client

<img src="img/02.png" />

!!!

### OCaml Mina Client

<img src="img/03.png" />

!!!

### OCaml Mina Client

<img src="img/04.png" />

!!!

### OCaml Mina Client

<img src="img/05.png" />

!!!

### OCaml Mina Client

<img src="img/06.png" />

!!!

### OCaml Mina Client

<img src="img/07.png" />

!!!

### OCaml Mina Client

<img src="img/09.png" />

!!!

### OCaml Mina Client

<img src="img/095.png" />

!!!

### OCaml Mina Client

<img src="img/10.png" />

!!!

### OCaml Mina Client

<img src="img/11.png" />

!!!

### OCaml Mina Client

<img src="img/12.png" />

!!!

### OCaml Mina Client

<img src="img/13.png" />

Note: This is only some of the systems, but should give you a sense...

!!!

### Towards a browser node

<img src="img/browser.png" />

!!!

### Towards a browser node

<img src="img/14.png" />

!!!

### Towards a browser node

<img src="img/15.png" />

!!!

### Towards a browser node

<img src="img/16.png" />

!!!

### Towards a browser node

<img src="img/17.png" />

!!!

### Towards a browser node

<img src="img/18.png" />

!!!

### Towards a browser node

<img src="img/19.png" />

!!!

### Towards a browser node

<img src="img/20.png" />

!!!

### Browser Node

<img src="img/205.png" />

Note: ... okay now for the next goal:

!!!

### Stability Optimization

<img src="img/21.png" />

!!!

### Stability Optimization

<img src="img/22.png" />

!!!

### Stability Optimization

<img src="img/24.png" />

!!!

### Stability Optimization

<img src="img/26.png" />

!!!

### Stability Optimization

<img src="img/27.png" />

!!!

### Stability Optimization

<img src="img/28.png" />

!!!

### Stability Optimized

<img src="img/optimized.png" />

Note: We're done -- these are exactly how the milestones are structured on the project:

!!!

### Milestone 1

<img src="img/28.png" />

!!!

### Milestone 2

<img src="img/27.png" />

!!!

### Milestone 3

<img src="img/26.png" />

!!!

### Milestone 4

<img src="img/24.png" />

!!!

### Milestone 5

<img src="img/22.png" />

!!!

### Milestone 6

<img src="img/21.png" />

!!!

### Deep Dive

<img src="img/27.png" />

Note: Handoff to Joseph here

