
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


