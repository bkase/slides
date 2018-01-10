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

### What about changing what a view means?

(picture)

!!!

### We can't from outside UIKit

(picture of AppKit, Website, Serialization to disk)

Note: But what if we wanted to...

!!!

### How about an Enum?

(picture)

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
  func renderSomething() -> Something {
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
```

```swift
    switch self {
    case let .textField(hint):
      let tv = UITextField()
      /* ... */
      return tv
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
    case let .stack(axis, children):
      let sv = UIStackView()
      let subViews = children.map{ $0.renderSomething() }
      subViews.forEach{ sv.addSubView($0) }
      /* ... */
      return sv
    }
  }
}
```
<!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### We can render to NSViews

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

* <!-- .element: class="fragment" data-fragment-index="1" --> Add new kinds of items <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> Add new interpretations for those items <!-- .element: class="fragment" data-fragment-index="2" -->
* <!-- .element: class="fragment" data-fragment-index="3" --> From outside the scope of the initial definitions of items/interpretations <!-- .element: class="fragment" data-fragment-index="3" -->
* <!-- .element: class="fragment" data-fragment-index="4" --> Keeping type-safety <!-- .element: class="fragment" data-fragment-index="4" -->

Note: MAKE SURE TO REFER BACK TO VIEWS. If we cast things and forget about type safety it's not too hard. This is not a talk about a new view framework, this is a talk about solving the expression problem. So let's talk about more...

!!!

### Another Expression Problem Instance

(picture)

!!!

### Drawing Shapes

(picture)

!!!

### Drawing Shapes: New Interpretations

* <!-- .element: class="fragment" data-fragment-index="1" --> CoreGraphics <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> CoreAnimation <!-- .element: class="fragment" data-fragment-index="2" -->
* <!-- .element: class="fragment" data-fragment-index="3" --> Svg image <!-- .element: class="fragment" data-fragment-index="3" -->

!!!

### Drawing Shapes: New Operations

* <!-- .element: class="fragment" data-fragment-index="1" --> Ellipse <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> Rectangle <!-- .element: class="fragment" data-fragment-index="2" -->
* <!-- .element: class="fragment" data-fragment-index="3" --> Gradients <!-- .element: class="fragment" data-fragment-index="3" -->
* <!-- .element: class="fragment" data-fragment-index="4" --> Layer many shapes <!-- .element: class="fragment" data-fragment-index="4" -->

!!!

### Drawing Shapes

(gif of drawing shapes app)

!!!

### Expression Problem

* Add new kinds of items
* Add new interpretations for those items
* From outside the scope of the initial definitions of items/interpretations
* Keeping type-safety

!!!

### Can we solve the Expression Problem?

(picture of high-dimensional legos)

Note: Wouldn't it be great to take the same view once and render it in multiple extensible places

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
```

```swift
func renderWeb(_ v: HTML) -> HTML { v }
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```swift
func renderPdf(_ v: Pdf) -> Pdf { v }
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```swift
func serialize(_ v: NSData) -> NSData { v }
```
<!-- .element: class="fragment" data-fragment-index="3" -->

!!!

### Use it

```swift
// inside view-controller
self.view = renderUIKit(complex2())
```

```swift
// ...
showPdf(renderPdf(complex2()))
```
<!-- .element: class="fragment" data-fragment-index="1" -->

Note: But what is this approach called?

!!!

### Final Tagless Style

Note: A protocol is a final tagless dsl, and instance is a final tagless interpreter.

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

1. <!-- .element: class="fragment" data-fragment-index="1" --> Expression Problem <!-- .element: class="fragment" data-fragment-index="1" -->
  * <!-- .element: class="fragment" data-fragment-index="2" --> Add new items <!-- .element: class="fragment" data-fragment-index="2" -->
  * <!-- .element: class="fragment" data-fragment-index="3" --> Add new interpretations <!-- .element: class="fragment" data-fragment-index="3" -->
  * <!-- .element: class="fragment" data-fragment-index="4" --> Without modifying original code <!-- .element: class="fragment" data-fragment-index="4" -->
  * <!-- .element: class="fragment" data-fragment-index="5" --> Maintaining typesafety <!-- .element: class="fragment" data-fragment-index="5" -->
4. <!-- .element: class="fragment" data-fragment-index="6" --> Final Tagless Approach: <!-- .element: class="fragment" data-fragment-index="6" -->
  * <!-- .element: class="fragment" data-fragment-index="7" --> Items -- Protocols: Methods return Self <!-- .element: class="fragment" data-fragment-index="7" -->
  * <!-- .element: class="fragment" data-fragment-index="8" --> Interpretations -- Protocol Instances <!-- .element: class="fragment" data-fragment-index="8" -->

!!!

### Resources:

* [Oleg's Tagless Final Lecture Notes](http://okmij.org/ftp/tagless-final/course/lecture.pdf)
* [Chris's Side-effects](https://gist.github.com/chriseidhof/a542d0a074cbf8d418d25a8b8253ff33)
* [Tagless Graphics]()
* Forms?
* Check out upcoming Swift Talk episodes!

!!!

# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

Slide Deck: [https://is.gd/mjJamZ](https://is.gd/mjJamZ)

