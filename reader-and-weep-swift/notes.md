1. Problem slide: Addressed only 2 out of 3 points. Need to address compile time checking.
2. Mock implicit deps -- you can't do it in Swift is a bad reason. You forgot to mock a nested dependency
  * Accidentally transitively depend on stuff.
  * Keeping mocks in sink with production code. Are mocks strict, to catch issues
3. This is our current cache get and set, here's the curried version
  * Use get and set on Cache in the example of currying
4. Use long descriptive variables (everywhere)
  * Play with the names of parameters
5. Put comments or type annotations everywhere
6. Not clear why we want option reader
  * Hide the fact Value is option (let's show why this is better)
7. You can just see flatMap

8. `annotateWithServer` or something for `maybeReverse`
9. Break up that loadReversedKeyIFSupported to more lines
10. Put the footnote of the URLs
11. Put the references at the end

12. Excersize to the reader (look at coreader)

13. Call out benefits of pure

14. Add increment N slide

15. FlatMap is a way to sequence two things. It's flatmap because this is what we call it.

16. Break up do you need to understand monads to use reader. No

17. Add really good summary slides (after inspire slide)



1. Ignore some of the implementation detail
2. Probably just show flatmap
3. Do cake pattern
4. Consider a better example (inc)
5. 


1. Singletons aren't gotten rid of it's the MANNER in which they are injected that we make explicit
2. A single struct containing every dependency doesn't scale because you can't have separate compilation units. Reader is nice because any arbitrary sub-unit of computation can have all it's deps mocked and it's typesafe!!!



Abstract:


Reader or Weep: Mock-free, Compile-time Checked, Dependency Injection with Reader

Global variables are bad. Singletons are globals. So why do we use singletons to depend on different components? Some of us don’t — some use “dependency injection” solutions that rely on runtime checks and assertions for correctness. But Swift has a wonderful type-system; runtime errors are unacceptable!

Fundamentally, depending on X is just wrapping logic in a function that takes X as input. So we can use that. Reader is a struct wrapper around our dependency function that provides convenient methods for extending our dependency context as we compose modules. Since Reader is a scary M word (a Monad) we can combine it with asynchrony (via Futures) or failures (via Option) without changing our usage in our application logic! In this talk, I’ll cover the pros and cons of Reader and touch on other type-safe DI solutions in Swift.


- Problem: We need to access the network, hit the db, or access other modules in our project for any interesting app.
  - If you’re not careful this induces tight coupling
    - You need to mock things (if you forget to mock something, oh well)
    - It becomes so annoying that many people just don’t do it
- Dependency Injection
  - Inject dependencies
  - ss
- Why traditional DI solutions are bad: Not typechecked!
- Show Reader Monad usage
- Touch on the internals
- Show how Reader can compose with Future
- All these flatmaps are annoying (look into the Cake Pattern)






Story:

1. We need to access the network, hit the db, or do some sort of side-effects
2. If you're not careful you get tight coupling; hard to test; hard to swap out
3. 

