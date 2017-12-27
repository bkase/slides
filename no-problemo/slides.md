<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->

# Reader or Weep: Mock-free, Compile-time Checked, Dependency Injection with Reader

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

!!!

### Let's talk about views

!!!

### UIKit

!!!

### Base view + preset views

!!!

### Add views without modifying UIKit (subclassing)

!!!

### Add new renderer?

!!!

### Can't render UIKit views to AppKit, Website, Serialize to disk etc

!!!

### What if we want to render to multiple places?

!!!

### How about an Enum?

!!!

### Pretend we had EnumKit

!!!

### Each recursive traversal is a renderer

!!!

### We can render to UIKit

!!!

### We can render to a WebView

!!!

### How do we add new view types?

!!!

### We have to modify EnumKit!

!!!

### How can we make UIKit + EnumKit?

!!!

### It turns out this is even more universal

Note: Anytime you have some situation where you want 

!!!

### Expression Problem

* Add new kinds of items
* Add new interpretations for those items
* Without modifying the underlying defition of the item
* Keeping type-safety

Note: If we cast things and forget about type safety it's not too hard

!!!

### Other examples:

* A calcluator expression
* Commands in your app's redux-like store?

!!!

### Subclassing (UIKit) doesn't work

!!!

### Enums (EnumKit) doesn't work

!!!

### We need a new solution

!!!

### Protocols!

(it's always protocols isn't it)

Note: But not just "protocols" with hand-waving

!!!

### Final Tagless DSLs

!!!

### Provided set of items

```
protocol PowerView {
  static func text(text: String) -> Self
  static func linear(children: [Self]) -> Self
  func something() -> Self
}
```

Note: These are your subclasses of UIView

!!!

### Adding new items with protocol inheritance

```
protocol WithMapView: PowerView {
  static func map(initialLatLong: Gps.Point) -> Self
}
```

!!!

### Build up complex view hierarchies with a generic function

```swift
func complex1<V: PowerView>() -> V {/* ... */ }

func complex2<V: WithMapView>() -> V {
  WithMapView.linear(children: [
    .complex1(),
    .map(initialLatLong: Gps.Point.here())
  ])
}
```

Note: It's composable btw

!!!

### Interpretations are protocol instances

```swift
extension UIView : PowerView {
  static func text(text: String) -> UIView {
    let tv = UITextView()
    tv.text = text
    return tv
  }
  /* etc */
}
```

!!!

### Choose an interpretation with an identity function

```swift
func renderUikit(_ v: UIView) -> UIView { v }

func renderWeb(_ v: HTML) -> HTML { v }

func renderAppKit(_ v: NSView) -> NSView { v }
```

!!!

### Use it

!!!

### Bigger example

!!!

### Expression Problem

Note: Expression problem comes up everywhere, we focused

!!!

### Summarize

!!!

# Thanks!

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

Slide Deck: [https://is.gd/mjJamZ](https://is.gd/mjJamZ)

[DI Explorations](https://github.com/bkase/swift-di-explorations), [Corridor](https://github.com/symentis/Corridor), [Monads (effect stacks with codegen)](https://github.com/facile-it/Monads)

