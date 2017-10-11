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

