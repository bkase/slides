
Title:
Control Flow: Past, Present, and Future
Control Flow retrospective and prospective?

Goto/Jumps
* At first there was nothing
* and then there was jumps
* With jumps you had a lot of power...
* Essentially gotos and `if`s?
* But then in 1968: Goto Considered Harmful by Dijkstra
* Dijkstra argued that we should only used for,if (and their variants), and recursion with structured procedures
* Many people thought this was crazy!
* They gave many examples of programs that _could not be expressed_ with merely for and if
* For example, you cannot implement X (yield?)
* Slowly over the next few years (a decade?)
* People started to see the light
* The only reason anyone would still be writing code today like this is if you have to dip into assembly (for optimizations maybe) or you're maintaing _really_ old code

At this point 100% of you should agree with me

"Structured" control flow
* For, if, recursion
* This is powerful enough (eventually people said)
* By giving up some power as developers (the ability to X), we give more power to our compiler or runtime. Optimizations are more obvious
* Then X argued that we should only use map, filter, fold (and friends) and recursion
* Many people thought this was crazy!
* They gave many examples of programs that _could not be expressed_ with merely map, filter, fold and friends
* For example, you cannot implement X (yield?)
* Slowly over the next few years (a decade?)
* People started to see the light
* The only reason anyone should still be writing code today like this is if you have to maintaing old code

If I had started here, about 50% of you would agree with me. Hopefully you see the parallels with the Goto -> structured jump and at least you're considering what I'm saying

Map, filter, fold (and friends) (on functors, monads, and foldables)
* The present (w/ sufficiently powerful languages)
* At first just lists
* Then any collection
* Then any functor (not just collections of data, but Futures/Promises)
* You do NOT sacrifice performance
* By giving up some power as developers (the ability to X), we give more power to our compiler or runtime. Optimizations are more obvious
* You may say, "but when I use map, filter, fold and friends my code is slower"
* Your compiler or runtime is not sufficiently advanced enough to optimize your code. Rust proves this is possible: (benchmarks)
* ...
* Then "Bananas, Lenses, and Barbed Wires" by X argued that we should not use general recursion, we should express all recursion in terms of one of many primitives (known as recursion schemes)
* Many people thought this was crazy!
* They gave many examples of programs that _could not be expressed_ with merely these recursion schemes. It's not even _Turing complete_
* For example, you cannot implement X (yield?)
* Slowly over the next few years (a decade?)
* People started to see the light
* The only reason anyone should still be writing code today like this is if you have to maintaing old code

If I had started here, 99% of you would agree with me. Hopefully you see the parallels and at least you're considering what I'm saying

Recursion Schemes
* The future (but you _can_ do it today in a few languages, some people are using this in production)
* Not turing complete, but you can still write programs that run forever (codata vs data)
* If your compiler/runtime knows your program will halt (with inductive data) or will make progress (with co-inductive data) _insane_ optimization passes can occur on your code
* You can write code that looks like you are doing multiple passes over your structure and fuse them all together?
* ...
