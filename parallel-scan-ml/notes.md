* TINY LEAF DOESN"T MAKE SESNE
* Blockchain snark discussion confusing and (uneccesary maybe)
* People don't know what a witness means
* Signature of knowledge bit is confusing
* Too Fast on the overlay of the data (and moving stuff around on the tree)
* Pick a specific telescope (...)


* Entire motivation section (everything before sequences of transactions)
  * What is the blockchain?
  * And what is snarks?
  * Only one sentence on argument of knowledge
* Metaphor of recursive composition is too fast
* Put all information of latex on one slide, animation introducing the bits
  * Not enough information to piece everything together and how it composes
* Show an example of the notation in the tree...

* The assumption of this is available distributed work pool that scales with R
* ...
* So cut out signature of knowledge

* Mention the performance 

* Name dropped the implicit heap.
* Explain the diagram, which shows child and parent computable from the just the index into the array

* Just use Haskell not ocaml
* Don't say alpha and beta








WHAT IS THIS TALK:

On one hand, a case study in jumping between the concrete and the generalized to solve a problem

On the other hand, in the beginning there's an interesting problem and in the end there's an interesting data structure that can solve it.

===

* Framing around functional programming / parametricity leading to it

* Back and forth between concrete and abstract

* A: First show the concrete
  * Too much prose
  * Symbols let us see the picture better
  * Now we can squint
* B: Abstract to show the fold/scan

So we want a fast scan.
We'll need to take advantage of more propeerties of our concrete problem

* A: Concrete PROPERTIES of problem
*    Generalized properties

* B: Concrete requiremnts
* B: Generalized requirements

That drives our investigation -- I'll spare you all the parts where you just bang your head on the whiteboard wondering why things aren't working...

* A: Do the things





Title?

* Intro
* in a cryptocurrency there are lots of cool technical challenges
* specifically this one (the transaction / proof problem)
* it's kind of a scan on an Rx-style stream
* what's a scan?
* actual problem
* okay to sacrifice latency
* abstraction!
* give the signature in swift
* trees
* log trees
* layer them
* succinct data structures (implicit data structures)
* Evaluate solution



* Red to purple
* Apply application to the tree, parallelism is in those steps
* Chunking
* Associativity not clear


* Show the visual representation of the scan before abstracting
* Graphics instead of latex equations (trees) (see picture)
* Requirements / properties instead of assumptions
    * Maximize throughput
    * Then latency
    * And the associativity and the reduction that
    * Make clear that there are two operations
    * Work being available is constant ; why do you want as much work available as possible


* Knowledge aquisition
* Imagine it consumes
* Put a link to the Wikipedia (mention heap)

* More clear about the invariant


