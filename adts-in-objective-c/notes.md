* Rework slide on "Intuition" after type algebra
* Make sure that all code compiles
* Macro section needs work (actually use before and after the macro evaluates as examples)
* The for-each stuff is very confusing; try to simplify


Abstract:

Abusing the C Preprocessor to bring algebraic data types to iOS development with Objective-C


Talk gist:
* Why ADTs are good
  * prevents illogical data
* No sum-of-product types in Objective-C (or Java)
* You can hack in OO in a few ways (visitor, inheritance+lambdas)
* It's too much boilerplate to write this out
* Macros in Objective-C let you reduce boilerplate with string substitution
* There are a few macro widgets you can use to accomplish a clean API
* Why Pinterest adopted this or not?
"We'll talk about whether or not Pinterest decided that this was worth pursuing"

2-3 paragraphs
abstract
+ "we would also love to hear what the motivation behind the talk or workshop is"
+ Feel free to include a link (github etc) if this is about a project


Bringing Swift enums to Objective-C with macros

Swift enums allow us to clearly, concisely, and correctly model information. Without these algebraic data types, it's common to resort to poorly representing information with nullables. Extra nullables lead to unnecessary error checking and crashes, and crashes lead to apps that no one wants to use.
Swift enums, otherwise known as algebraic data types, are present in all self-respecting _modern_ programming languages. 
// we're stuck with legacy code, but doesn't have to be unsafe
Unfortunately, Objective-C has been around for a while, and does not have a tool like Swift's enum.
In Objective-C we can achieve similar clear and correct expressive power if we sacrifice concision -- with a mixture of inheritance and blocks, or using the visitor pattern. Due to the massive amount of boilerplate we need, we don't reach for these patterns nearly as often as we should.
However, we have a tool to manage boilerplate in Objective-C -- preprocessor macros!
Despite the limitations of the macro substitution system, with a few macro tricks, we can build a massive macro that can give us algebraic data types with none of the boilerplate. We can declare our types just as we would with Swift enums!

```
ONE_OF(PIBarCode,
  CASE(QrCode, NSString *, code),
  CASE(Upc, NSNumber *, num1, NSNumber *, num2, NSNumber *, num3, NSNumber *, num4)
)
```

In this talk, I will cover why these enums matter, how it's possible to represent them in Objective-C (albeit verbosely), how we can remove the verbosity with macros, and, in addition, we'll talk about how we're using it at Pinterest.

// too long
Brandon Kase brings typed functional programming to weird plaes. He has shipped production code on Android with Kotlin, iOS with Swift and React Native, and Web with JS/Flow/React. Brandon is an iOS core-experience engineer at Pinterest. He first came across functional programming while pursuing a B.S. in Computer Science from Carnegie Mellon University. Brandon is excited that strong static typing and functional programming are becoming mainstream, and believes that techniques once reserved for academia will help industry produce more reliable software. He is in general fascinated by anything and everything related to well-designed programming languages and libraries.


