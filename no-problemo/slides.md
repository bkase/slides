<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# No Problemo: Finally, an Expression Problem solution

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

!!!

### Let's think about views

(picture)

!!!

### UIKit

!!!

### Base view + preset views

```swift
class UIView { /*...*/ }
```

```swift
class UIButton : UIView { /*...*/ }
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
class UIStackView : UIView { /*...*/ }
//...
```
<!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### Add views without modifying UIKit (subclassing)

```swift
class MyMapView: UIView {
  /* ... */
}
```

!!!

### Add new renderer?

(picture)

!!!

### Can't render UIKit views to AppKit, Website, Serialize to disk etc

!!!

### What if we want to render to multiple places?

!!!

### How about an Enum?

!!!

### Pretend we had EnumKit

```swift
indirect enum View {
```

```swift
  case button(text: String, onTap: () -> (), /*...*/)
  case textField(hint: String)
  /* ... */
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
  case stack(axis: Axis, children: [View])
  /* ... */
}
```
<!-- .element: class="fragment" data-fragment-index="2" -->

Note: TableViews etc...

!!!

### Each recursive traversal is a renderer

```swift
extension View {
  func renderSomething() {
```

```swift
    switch self {
    case let .textField(hint):
      /* ... */
    case let .stack(axis, children):
      let subviews = children.map { $0.renderSomething() }
      /* ... */
    }
  }
}
```
<!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### We can render to a UIView

```swift
extension View {
  func renderUIView() -> UIView {
    switch self {
    case let .textField(hint):
      let tv = UITextField()
      /* ... */
      return tv
    case let .stack(axis, children):
      let sv = UIStackView()
      let subViews = children.map{ $0.renderSomething() }
      /* ... */
      return sv
    }
  }
}
```

!!!

### We can render to a NSView

```swift
extension View {
  func renderAppKit() -> NSView {
    switch self {
    /* ... */
    }
  }
}
```

!!!

### How do we add new view types?

(picture)

!!!

### We have to modify EnumKit inernals!

```swift
enum View {
  /*...*/
  case newView(/*...*/)
}
```

!!!

### How can we make UIKit + EnumKit?

(picture)

!!!

### Expression Problem

* Add new kinds of items
* Add new interpretations for those items
* From outside the scope of the initial definitions of items/interpretations
* Keeping type-safety

Note: If we cast things and forget about type safety it's not too hard. This is not a talk about a new view framework, this is a talk about solving the expression problem. So let's talk about more...

!!!

### More Expression Problem instances

!!!

### Arithmetic

```
// 1 + (2 - 3)
```

!!!

### Arithmetic: New interpretations

(picture)

Note: We could want to render to an integer, or serialize to a string

!!!

### Arithmetic: New operations

* Multiplication
* Division

!!!

### Side-effects

(picture)

!!!

### Side-effects: New interpreations

* Execute for the effects
* Log to the console
* Show in a debug view

!!!

### Side-effects: New operations

* Make a network request
* Pick a random number

!!!

### Drawing Shapes

(picture)

!!!

### Drawing Shapes: New Interpretations

* CoreGraphics
* CoreAnimation
* Svg image

!!!

### Drawing Shapes: New Operations

* Ellipse
* Rectangle
* Gradients
* Layer many shapes

!!!

### Expression Problem

* Add new kinds of items
* Add new interpretations for those items
* From outside the scope of the initial definitions of items/interpretations
* Keeping type-safety

!!!

### Expression Problem Power

(epitome of modularity picture)

Note: This is the epitome of modularity. Libraries compatible with expression problem are encourage ecosystems of libraries that don't need to depend on each other

!!!

### Solve the expression problem?

(picture of high-dimensional legos)

Note: Super extensible, composable, modular library. You can swap out the renderer, add new cases, you don't need to fork anything. Healthy ecosystem of helper libraries around yours. Lego pieces. Legos in different dimensions.

!!!

### Subclassing (UIKit) doesn't work

(picture)

Note: We can add new views, but we can't change what it menas to be a view

!!!

### Enums (EnumKit) doesn't work

(picture)

Note: We can change what it menas to be a view, but we can't add new views

!!!

### Non-exhaustive Enums?

(picture of https://github.com/apple/swift-evolution/blob/master/proposals/0192-non-exhaustive-enums.md)

!!!

### Non-exhaustive-EnumKit

```swift
/*non-exhaustive*/ indirect enum View {
  case button(text: String, onTap: () -> (), /*...*/)
  case textField(hint: String)
  case stack(axis: Axis, children: [View])
}
```

Note: Okay so now we can add an enum case, but...

!!!

### Non-exhaustive-EnumKit

```swift
extension View {
  func renderAppKit() -> NSView {
    switch self {
    case button(text: String, onTap: () -> (), /*...*/)
    case textField(hint: String)
    case stack(axis: Axis, children: [View])
```

```swift
    default: // what do I do here???
    }
  }
}
```
<!-- .element: class="fragment" data-fragment-index="1" -->

Note: But what do I do in the default case, how do I adapt my new enum case to the appkit renderer?

!!!

### Protocols?

(picture)

!!!

### Protocols!

It's always protocols isn't it

Note: But not just "protocols" with hand-waving

!!!

### Final Tagless DSLs

!!!

### Provide set of items

```
protocol View {
  static func button(text: String, onTap: () -> ()) -> Self
  static func stack(axis: Axis, children: [Self]) -> Self
  /*...*/
}
```

Note: These are your "UIView subclasses"

!!!

### Adding new items with additional protocols

```
protocol WithMapView: View {
  static func map(initialLatLong: Gps.Point) -> Self
}
```

!!!

### Build up complex view hierarchies with a generic function

```swift
func complex1<V: View>() -> V {/* ... */ }

func complex2<V: WitMapView>() -> V {
  WithMapView.stack(axis: .vertical, children: [
    .complex1(),
    .map(initialLatLong: Gps.Point.here())
  ])
}
```

Note: It's composable btw

!!!

### Interpretations are protocol instances

```swift
extension UIView : View {
  static func text(text: String) -> UIView {
    let tv = UITextView()
    tv.text = text
    return tv
  }
  /* etc */
}
```

!!!

### More Interpretations

```swift
extension HTML : View {
  static func text(text: String) -> HTML {
    // <p>{text}</p>
    return .p([
      .string(text)
    ])
  }
}
```

!!!

### Choose an interpretation with an identity function

```swift
func renderUIKit(_ v: UIView) -> UIView { v }
func renderWeb(_ v: HTML) -> HTML { v }
func renderPdf(_ v: Pdf) -> Pdf { v }
func serialize(_ v: NSData) -> NSData { v }
```

!!!

### Use it

```swift
// inside view-controller
self.view = renderUIKit(complex2())

// ...
showPdf(renderPdf(complex2()))
```

!!!

### That's not all: Little indirection

(diagram of pointers)

Note: We don't need to build up intermediate structure that we immediately destroy

!!!

### That's not all: Recursion done for you!

```swift
extension HTML : View {
  // Self type is replaced recursively
  static func stack(children: [HTML]) -> HTML {
    return .div(style: [.position(absolute)], children)
  }
  /* ... */
}
```

Note: We don't actually do any recursion

!!!

### Recap

1. UIKit flaws
2. EnumKit flaws
3. Expression Problem
  * Ultimate modularity
4. Arithmetic, Side-Effects, Diagrams, and Forms
5. Final Tagless DSL Approach:
  * Items -- Protocols: Methods return Self
  * Interpretations -- Protocol Instances
  * Little indirection + recursively resolved for free

!!!

### Resources:

* Paper
* Gists/repos
* Check out upcoming Swift Talk episodes!

!!!

# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

Slide Deck: [https://is.gd/mjJamZ](https://is.gd/mjJamZ)

[DI Explorations](https://github.com/bkase/swift-di-explorations), [Corridor](https://github.com/symentis/Corridor), [Monads (effect stacks with codegen)](https://github.com/facile-it/Monads)

