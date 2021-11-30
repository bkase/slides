<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Mina Susbsystem Pathing and Deepdive into Ouroboros Samasika Consensus

By Joseph Spadavecchia and <a href="http://bkase.dev">Brandon Kase</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

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

(chainsafe image)

Note: Chainsafe is building an alternate Mina client in Rust

!!!

### Background

(O(1) Labs image)

Note: O(1) Labs was tasked with specifying describing and (interestingly) pathing the work such that...

!!!

### Mina Susbsystem Pathing Goals

1. Choose that which will prefix to a secure browser node
2. <!-- .element: class="fragment" data-fragment-index="1" --> Optimize path for most stable subsystems first <!-- .element: class="fragment" data-fragment-index="1" -->

Note: By stable, I mean O(1) labs is actively developing Mina and adding Snapps our smart contract layer. We don't want to waste time specing or implementing that which will become stale.

!!!

### Mina Specifying

(specification image)

Note: As part of this work, we're also simultaneously producing a spec -- we want to give you a sense of interesting nuances that come up

!!!

### In this Talk

1. Brandon: Subsystem Pathing
2. <!-- .element: class="fragment" data-fragment-index="1" --> Joseph: Ouroboros Samasika and Min Window Density <!-- .element: class="fragment" data-fragment-index="1" -->

Note: Brandon will talk through the susbsytem pathing in more of a high-level sense; Joseph will do a deep dive into one of those subsystems focusing on an intresting nuance of ouroboros samasika, Mina's consensus algorithm.

!!!

### Susbsystem Pathing

(image) 

!!!

### Subsystem Pathing Goals

1. Choose that which will prefix to a secure browser node
2. <!-- .element: class="fragment" data-fragment-index="1" --> Optimize path for most stable subsystems first <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### OCaml Mina Client

(animate in subsystems chaotically)

* TODO: Show all the subsystems chaotically (progressively revealing a diagram)
  * Serialization
  * Block verification
  * Block Selection (Consensus)
  * Transition frontier
  * Networking
  * Ledgers and other state
  * Transactions
  * Provers
  * Snark workers

Note: Explain all the pieces

!!!

### OCaml Mina Client

* TODO: Show the subystems that we don't need crossed out

!!!

### OCaml Mina Client

* TODO: Talk through the remaining pieces' stability

!!!

### Sort susbsytems

(image)

!!!

### Milestones

1. Serialization
2. <!-- .element: class="fragment" data-fragment-index="1" --> Block Selection (consensus) <!-- .element: class="fragment" data-fragment-index="1" -->
3. <!-- .element: class="fragment" data-fragment-index="2" --> Block Verification <!-- .element: class="fragment" data-fragment-index="2" -->
4. <!-- .element: class="fragment" data-fragment-index="3" --> Ledgers & Additional State <!-- .element: class="fragment" data-fragment-index="3" -->
5. <!-- .element: class="fragment" data-fragment-index="4" --> Networking <!-- .element: class="fragment" data-fragment-index="4" -->
6. <!-- .element: class="fragment" data-fragment-index="5" --> Transaction Application and final client <!-- .element: class="fragment" data-fragment-index="5" -->

Note: And these are the actual milestones on the project

!!!

### Deep Dive

(image)

Note: Handoff to Joseph here

