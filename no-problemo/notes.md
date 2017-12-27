

No Problemo: Finally, A Nice Solution to the Expression Problem

How can we define some data type (for example the input to a calculator) such that (1) we can later add new variants (the sqrt for our calculator) and (2) we can add new behavior (for example rendering the calculator expression to a UIView rather in addition to producing a number) without losing type safety or touching the first bit of code. This is known as the Expression Problem. 

Subclassing solves (1), but we can’t add new methods to a base-class later. Swift’s enum solves (2), but we can’t add new cases later without modifying the enum. The Final Tagless Interpreter approach solves (1) and (2) taking advantage of Swift’s powerful Self type inside protocols. Data is defined with a protocol, new variants are added with protocol inheritance, and new behavior is added by providing a new protocol instance. Programs are declared generically and evaluated with identity functions. It’s wild! What’s unique about the Final Tagless approach among practical “Expresion Problem”-solutions in Swift is the separation between declaration and evaluation. This leads to a more easily testable and reusable code. I believe Final Tagless Interpreters are a hidden gem as they are simple to understand and extremely powerful for solving all sorts of problems in our Swift code!

NOTES: Why do we care about expression problem?
We see it a lot but don’t know how to solve. Before reading it, you may have never known it, but you have probably struggled with it. You can try subclassing, you can try enums, ….
You may not have a name for this problem ….
Don’t use calculator in abstract olny talk.  You see this problem in X use-case ← find something if you’ve dealt with x you’ll understand



- An Expression Problem instance
- The Expression Problem (general)
- Why Expression Problem is hard to solve (show all the things that don’t solve it)
- A simpler Expression Problem instance (calculator)
- Finally Tagless solution:
- Protocols all the way down
- Interpreters are monomorphic identity functions!
- More examples
  - Console
  - Network
  - Composition of them
- Multiple interpretations at once
- Conclusion: Expression Problem…


