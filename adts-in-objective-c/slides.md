<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
## Bringing Swift Enums to Objective-C

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

!!!

## Swift Enums are good

(pinterest feed picture)

Note: for many cases of information modelling (taking the spec and putting it into code); I'm using the feed as an example of where this type of thing would fit: Right now we have Pin, Story, Board (and now Tries?), maybe food reviews?, it seems like we're going to keep adding new variants

!!!

## Swift Enums are good

```swift
// bad
struct BadFeedItem {
  let storyData: StoryData?
  let pinData: PinData?
  /* … */
}
```

!!!

## Swift Enums are good

```swift
let story = BadFeedItem(storyData: data, pinData: nil)
// forced to deal with optionals
story.storyData!
```

!!!

## Swift Enums are good

```swift
let oops = BadFeedItem(storyData: nil, pinData: nil)
// oops I messed up my constructor
oops.storyData!
```

!!!

## Swift Enums are good

```swift
let wat = BadFeedItem(storyData: data1, pinData: data2)
// weird object, should I render a pin or a story?
```

Note: Let's compare that to a good feeditem

!!!

## Swift Enums are good

```swift
enum GoodFeedItem {
  case Story(data: StoryData)
  case Pin(data: PinData)
  /* … */
}
```

!!!

## Swift Enums are good

```swift
let story = .Story(data: storyData)
let pin = .Pin(data: pinData)
```

Note: a pin is a pin, it can't have nil data. A story is a story. You can't construct mysterious things.

!!!

## Swift Enums are good

With Swift enums, impossible states are *impossible by construction*

Note: In other words, your code won't compile if you or your teammates forget some constraint

!!!

## Swift Enums are good

```swift
func render(item: BadFeedItem) {
  if let storyData = item.storyData {
    // draw story
  } else if let pinData = item.pinData {
    // draw pin
  } /* … */
}
// what if we add a new type of feeditem??
```

Note: Answer: we would just not draw -- and see a blank space; Consider what would happen if you had this sort of if-elseif tree in lots of places.

!!!

## Swift enums are good

```swift
func render(item: GoodFeedItem) {
  switch item {
    case let .Story(storyData):
      // draw story
    case let .Pin(pinData):
      // draw pin
  }
  // if you add a new case
  // the compiler will *save* you
}
```

!!!

## Swift Enums are good

Swift enums *protect* us by requiring that all cases are handled

Note: In other words, your code won't compile if you mess up

!!!

## Algebraic Data Type

`Swift Enums` == `Algebraic Data Types`

`Swift Enums` == `Sum of Products`

Note: This construct is not just in Swift, it's in several modern languages

!!!

## Type Algebra?

Bool has 2 values: `true` and `false` (let's just call it 2)

Void has 1 value: `()` (let's just call it 1)

Note: The algebra in Algebraic data type

!!!

## Type Algebra?

```
struct Product{
  let two: Bool
  let one: Void
}
// two possible instances
let a = Product(two: false, one: ())
let b = Product(two: true, one: ())
// 2 * 1 = 2
```

!!!

## Type Algebra?

```
enum Sum {
case two(Bool)
case one(Void)
}
// three possible instances
let (a, b, c) = (.two(false), .two(true), .one(()))
// 2 + 1 = 3
```

!!!

## Type Algebra

We can add an multiply types or do *algebra* just like numbers! 

!!!

## Sum of Products

```
enum SumOfProducts {
case twoAndTwo(Bool, Bool)
case one(Void)
}
// 5 possible values
// (2 * 2) + 1
```

!!!

## Intuition about ADTs

Not having a `sum` type (aka a Swift `enum`) is like not being able to add numbers.

It's insane! It is fundamental to modeling information!

!!!

## We want it. Let's do it

Okay so how can we get a sum-of-product in Objective-C?

* Impossible states are impossible by construction
* Compiler enforces all cases are handled 

!!!

### Step 1: Impossible states are impossible by construction

There is no way to make a Pin and Story at the same time. There is no way to create some object that has neither a Pin nor a Story.

!!!

### Step 1: Impossible states are impossible by construction

```objective-c
// inheritance and constructor specialization
@interface FeedItem : NSObject
-(instancetype)init NS_NOT_ALLOWED; (lookup)
@end
@interface StoryFeedItem 
-(FeedItem)init withStoryData:(StoryData *)
@end
@interface PinFeedItem // etc
```

Note: Omitting namespace for clarity

!!!

### Step 2: Compiler enforces all cases are handled

(picture)

Note: we can use a method with parameters for each case

!!!

### Step 2: Compiler enforces all cases are handled

```objective-c
// take 1
-(void)matchWithStory:^(StoryData *)story pin:^(PinData *)pin;
```

!!!

### Step 2: Compiler enforces all cases are handled

```objective-c
// take 1
-(void)render {
  [feedItem matchWithStory:^(StoryData * storyData){
    // draw story
  }, pin:^(PinData * pin) {
    // draw pin
  }];
  // if we add another case, this will no longer compile
}
```

!!!

### Step 2: Compiler enforces all cases are handled

We can one-up `Swift enums`!

!!!

### Step 2: Compiler enforces all cases are handled

```objective-c
// take 2
FeedItem<ValueType>
+(ValueType)match:(FeedItem *)item
            story:^ValueType(StoryData *)story
              pin:^ValueType(PinData *)pin;
```

!!!

### Step 2: Compiler enforces all cases are handled

```objective-c
// take 1
-(NSNumber *)render {
  return [FeedItem<NSNumber *> match:item
                         story:^NSNumber *(StoryData *storyData){
    // draw story
    return @1
  }, pin:^NSNumber *(PinData *pin) {
    // draw pin
    return @2
  }];
  // we can return something as long as type
  // is the same in all branches
}
```

!!!

## We did it?

![matrix](img/code-matrix.jpg)

Note: So why is this not good enough?

!!!

## So much code

```objective-c
// compare to this
@interface FeedItem
@property (nonatomic, strong) story;
@property (nonatomic, strong) pin;
@end
// that's it!
```

Note: It's so much easier to do it this way

!!!

## Implementation of match

```objective-c
+(ValueType)match:(FeedItem *)item
            story:^ValueType(StoryData *)story
              pin:^ValueType(PinData *)pin {
  if ([item isKindOfClass: [StoryFeedItem class]]) {
    return story((StoryData *)item);
  } else if ([item isKindOfClass: [PinFeedItem class]]) {
    return pin((PinData *)item);
  } /* … */
  // we have the if-else tree!
  // We can't forget to update this!
}
```

Note: And we still have the same sort of problem as before to manage this boilerplate

!!!

## Macros can manage boilerplate

(picture)

!!!

## ONE_OF

```objective-c
ONE_OF(FeedItem,
  CASE(Pin, PinData *, pinData)
  CASE(Story, StoryData *, storyData /*, … */)
  /* … */
)
```

Note: We can make Swift enums! One of either a pin or a story

!!!

## Swift enum in Objective-C

```
// if you squint they look similar
enum FeedItem {
  case pin(PinData)
  case story(StoryData)
}
```

```
// if you squint they look similar
ONE_OF(FeedItem,
  CASE(Pin, PinData *, pinData),
  CASE(Story, StoryData *, storyData)
)
```

Note: explain it in detail here

!!!

## How can we build it?

(picture)

Note: Let's talk about macros

!!!

## Macros in Objective-C

```objective-c
#define FOO @"bar"

NSLog(FOO); // logs "bar"
```

Note: Replace a string

!!!

## Macros in Objective-C

```objective-c
#define FOO(x, y) @"x bar y"

NSLog(FOO(before, after)); // logs "before bar after"
```

Note: Macro functoins? (read this more)

!!!

## Macros in Objective-C

```objective-c
#define FOO(x, y) @"x##y"

NSLog(FOO(before, after)); // logs "beforeafter"
```

Note: Token pasting

!!!

## How can we build it?

(picture building a complex thing out of nothing)

Note: This seems like it's not enough

!!!

## What do we need?

```
// if you squint they look similar
ONE_OF(FeedItem,
  CASE(Pin, PinData *, pinData),
  CASE(Story, StoryData *, storyData)
)
```

Note: variadic within the case, variadic across the cases, we need the 

!!!

## What do we need?

1. Variadic within the case
2. Variadic across the cases
3. Send the FeedItem prefix down to the cases
4. Send the case info up to the parent to put in match and constructors

!!!

## Variadic macros

```objective-c
#define BAZ(a, b, c, ...) a##b##c
#define BAR(a, b, ...) a##b
#define FOO(...) BAR(__VA_ARGS__) BAZ(__VA_ARGS__)

FOO(1,2,3,4) // replaced with "12 123"
```

Note: What if we want to map over the args

!!!

## FOR_EACH

(picture of the crazy)

!!!

## FOR_EACH

(explain each bit)

!!!

## FOR_DOUBLE_EACH

```objective-c
CASE(Pin, PinData *, pinData, NSNumber *, otherStuff)
// we want to run a macro on the type and value in a pairs
// FOO(type, value)
```

!!!

## FOR_DOUBLE_EACH

(secret sauce)

!!!

## Passing data upwards (case -> one_of)

(secret sauce)

!!!

## Passing data

Secret is: only pass upwards

!!!

## Passing data

We pass up 4 things from case; that means we want to run a macro over groups of 4 things + we need the prefix (a constant)

!!!

## FOR_QUAD_CONST_EACH

(secret sauce)

!!!

## Macro

The rest is simple!

!!!

## Downsides

Debugging?

!!!

## Downsides

```objective-c
// Macros can't capitalize
[PinFeedItem initWithpinData:(PinData *)data]
```

!!!

## Pinterest usage

!!!


