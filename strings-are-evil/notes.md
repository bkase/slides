
Strings Are Evil

Using strings in your code invites runtime errors and worse — logic errors — into your programs! Why? Because strings can always be combined with other strings, regardless of whether they immediately should be. Think about file paths — it’s easy to mix up absolute and relative paths. Same with files and directories. Moreover, they often lack the rich information we need to write good programs. Consider text displayed to users — it’s easy to forget to localize or to mix it around with garbage like a database identifier. What about rendering large blocks text? Strings aren’t flexible enough to capture the structure at different container widths.

Fortunately, Swift’s type system can rescue us. We can properly handle files and directories with absolute and relative paths all at the type-level (i.e. no runtime overhead). No more path mistakes in our codebase. We can wrap text we show in views in a small wrapper struct and require localization on these structs. Additionally, instead of rendering large blocks of texts directly to a string, we can use layers of enums to efficiently pretty-print output different screen widths. In this talk I will show how we can conquer the evil strings in our program with Swift’s type system.


Three anecdotes that show why you should think twice before using a string:

1. By wrapping string types that you use internally
  1. Making separate wrappers for strings you display to the user can enforce that you have localized all of them
  2. Making separate wrappers for database ids means you won’t accidentally pass a bad string to something expecting a dbid and a good string
2. For complex structures use a data structure!
  1. Paths using phantom generics (modeled after Purescript-Pathy library)
3. For rendering large blocks of text, using layers of DSLs gives you flexibility in your rendering
  1. DoctorPretty library for pretty printing


1. Strings are evil
2. Sneaky; hide information
3. Parametricity is bad; it can be anything
4. Okay so we try type-aliases
5. Now we communicate our intent better
6. But we have no compile-time safety
7. Let's do better, we can wrap our strings up in structs
8. Now we have compile time safety
9. In an _ideal_ world, this struct wrapper would have no overhead
10. But even this isn't perfect
11. We're not protected against misusing this kind of object
12. Phantom types
13. Paths!

2. Paths

3. By the way for rendering strings I made a doctor pretty



Paths:
* Explain why strings are sneaky and omnipotent
  * People know that strings are a bad example; relying on strings for things is bad if you make typos
* Not understanding what the strings mean isn't as important
* Important thing is using strings to build things causes unpredictable behavior
* typealiases deprioritize
* Make the bonuses less asides -- the struct gives you a namespace for method behavior
    * the join method in it's own class
* Need the transition between strings are evil; and paths for the rest of the talk
* Introduce the problem (we want to look at how to compose a file path; I'm going to walk you through a type safe thing)
    * Show all the possible ways to mess things up
* Spend more time talking about swift language features (expressible; k and t in phantom type; explain the expression; add another canonicalizing paths)

* Levels of safety
   * Use a typealias (no safety, just understanding of strings)
   * Use a struct wrapper (safe mixing up various string types)
   * Enums (safe construction)
   * Use phantom types (impossible to xxx) (safe operations)

=====

* Speed the beginning, slow down the end
* Examples of sneaky strings on the slide
* Text for omnipotent is confusing, show an example
* WindowsXP files floating across
* Nominal vs Structural: make sure it's
* Liked the expressiblebystringliteral
* Constrain our K and T slide. Keep the example `Path<K, T>` at the top
* Do a markdown table
* Bring up all the problems in the beginning

=====

* Use google maps
* Too build a correct filepath we need all of that
* Do a slide on why it would be impossible to do that with strings
* Always refer back to the strings (nods back)
* This is what we need to build what the user wants
* Pull two english language strings, if it showed up on a perscription
* The context the string lives, and you don't want to have to read the code around where you're reading
    * Strings get meaning from their context
* Be prepared to answer, "This is not overarchitected..."; don't assume everyone sees the value

====

* Bring the example earlier
* How are we safe keep track (note what changed)
* Show applications (generics without an initializable runtime value)
* Phantom types coming from so-and-so
    * A bit more about what phantom types before showing the construction
* Cut slides
* Show the code snippets where it doesn't work
* Spend longer at the breaker slides (let's think about this...)
* Show the phantom example before the shout-out
* Just say "phantom types"

===

* Call Phantom types out more
* Slide on the final implementation
* Need to mention why
* Uses URL fails
* Phantom types
* Why and what phantom types are (more slides)
* This is a phantom type (definition)
* What is a phantom type at a high level
* Move the lets before the function and use those
* Keep track of which FilePath we're using
* Don't animate the "How Safe Are We"
* Unreachable directive in swift


