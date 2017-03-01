<!-- .slide: data-background="#2aa198" -->
<!-- .slide: data-state="terminal" -->
## Bringing Swift Enums to Objective-C

By <a href="http://bkase.com">Brandon Kase</a> / <a href="https://www.pinterest.com/brandernan/"><i class="fa fa-pinterest" aria-hidden="true"></i>brandernan</a> / <a href="http://twitter.com/bkase_">@bkase_</a> 

!!!

# Why

!!!

## Swift Enums are good

(pinterest feed picture)

Note: for many cases of information modelling (taking the spec and putting it into code); I'm using the feed as an example of where this type of thing would fit: Right now we have Pin, User, Board, what if we add another variant

!!!

## Swift Enums are good

```swift
// bad
struct BadFeedItem {
  let userData: UserData?
  let pinData: PinData?
  /* … */
}
```

!!!

## Swift Enums are good

```swift
let user = BadFeedItem(userData: data, pinData: nil)
// forced to deal with optionals
user.userData!
```

!!!

## Swift Enums are good

```swift
let oops = BadFeedItem(userData: nil, pinData: nil)
// oops I messed up my constructor
oops.userData!
```

!!!

## Swift Enums are good

```swift
let wat = BadFeedItem(userData: data1, pinData: data2)
// weird object, should I render a pin or a user?
```

Note: Let's compare that to a good feeditem

!!!

## Swift Enums are good

```swift
enum GoodFeedItem {
  case user(data: UserData)
  case pin(data: PinData)
  /* … */
}
```

!!!

## Swift Enums are good

```swift
let user = .user(data: userData)
let pin = .pin(data: pinData)
```

Note: a pin is a pin, it can't have nil data. A user is a user. You can't construct mysterious things.

!!!

## Swift Enums are good

With Swift enums, impossible states are *impossible by construction*

Note: In other words, your code won't compile if you or your teammates forget some constraint

!!!

## Using the feed items

!!!

## Swift Enums are good

```swift
func render(item: BadFeedItem) {
  if let userData = item.userData {
    // draw user
  } else if let pinData = item.pinData {
    // draw pin
  } /* … */
}
// what if we add a new type of feeditem, without updating this??
```

Note: Answer: we would just not draw -- and see a blank space; Consider what would happen if you had this sort of if-elseif tree in lots of places.

!!!

## Swift enums are good

```swift
func render(item: GoodFeedItem) {
  switch item {
    case let .user(userData):
      // draw user
    case let .pin(pinData):
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

### Why Swift Enums are good

* Impossible states are impossible by construction
* Compiler enforces all cases are handled 

Note: For modelling something like the Pinterest home feed

!!!

## Intuition

![intuition](img/intuition.jpg)

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

Note: It's insane! It is fundamental to modeling information!

!!!

## We want it. Let's do it

Okay so how can we get a sum-of-product in Objective-C?

* Impossible states are impossible by construction
* Compiler enforces all cases are handled 

!!!

### Step 1: Impossible states are impossible by construction

* No way to make a Pin and user at the same time
* No way to create some object that has neither a Pin nor a user.

!!!

### Step 1: Impossible states are impossible by construction

```objective-c
// inheritance and constructor specialization
@interface FeedItem : NSObject
-(instancetype)init NS_UNAVAILABLE;
@end
@interface UserFeedItem 
-(FeedItem)init withUserData:(UserData *)
@end
@interface PinFeedItem // etc
```

Note: Omitting namespace for clarity

!!!

### Step 2: Compiler enforces all cases are handled

![enforce](img/enforce.jpg)

> http://www.justingary.com/wp-content/uploads/2016/07/Enforcement.jpg

Note: we can use a method with parameters for each case

!!!

### Step 2: Compiler enforces all cases are handled

```objective-c
// take 1
-(void)matchWithUser:^(UserData *)user pin:^(PinData *)pin;
```

!!!

### Step 2: Compiler enforces all cases are handled

```objective-c
// take 1
-(void)render {
  [feedItem matchWithUser:^(UserData * userData){
    // draw user
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
            user:^ValueType(UserData *)user
              pin:^ValueType(PinData *)pin;
```

!!!

### Step 2: Compiler enforces all cases are handled

```objective-c
// take 1
-(NSNumber *)render {
  return [FeedItem<NSNumber *> match:item
                         user:^NSNumber *(UserData *userData){
    // draw user
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

> http://www.myfreewallpapers.net/abstract/wallpapers/code-matrix.jpg

Note: So why is this not good enough?

!!!

## So much code

```objective-c
// compare to this
@interface FeedItem
@property (nonatomic, strong) user;
@property (nonatomic, strong) pin;
@end
// that's it!
```

Note: It's so much easier to do it this way

!!!

### Implementation of match

```objective-c
+(ValueType)match:(FeedItem *)item
            user:^ValueType(UserData *)user
              pin:^ValueType(PinData *)pin {
  if ([item isKindOfClass: [UserFeedItem class]]) {
    return user((UserData *)item);
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

![manager](img/manager.jpg)

> http://www.eylean.com/blog/wp-content/uploads/2014/06/project-manager-multitasking.jpg

!!!

## ONE_OF

```objective-c
ONE_OF(FeedItem,
  CASE(Pin, PinData *, pinData)
  CASE(user, UserData *, userData /*, … */)
  /* … */
)
```

Note: We can make Swift enums! One of either a pin or a user

!!!

## Swift enum in Objective-C

```
// if you squint they look similar
enum FeedItem {
  case pin(PinData)
  case user(UserData)
}
```

```
// if you squint they look similar
ONE_OF(FeedItem,
  CASE(Pin, PinData *, pinData),
  CASE(user, UserData *, userData)
)
```

Note: explain it in detail here

!!!

## How can we build it?

![builder](img/builder.jpg)

> http://clipart-library.com/clipart/385426.htm

Note: Let's talk about macros

!!!

## Macros in Objective-C

```objective-c
#define FOO @"bar"

NSLog(FOO); // logs "bar"
```

Note: Replace with a string

!!!

## Macros in Objective-C

```objective-c
#define FOO(x, y) @"x bar y"

NSLog(FOO(before, after)); // logs "before bar after"
```

Note: Macro functions

!!!

## Macros in Objective-C

```objective-c
#define FOO(x, y) @"x##y"

NSLog(FOO(before, after)); // logs "beforeafter"
```

Note: Token pasting

!!!

## Variadic macros

```objective-c
#define BAZ(a, b, c, ...) a##b##c
#define BAR(a, b, ...) a##b
#define FOO(...) BAR(__VA_ARGS__) BAZ(__VA_ARGS__)

FOO(1,2,3,4) // replaced with "12 123"
```

!!!

## How can we build it?

![crazy toothpick city street](img/toothpicks.jpg)

> http://s535395661.websitehome.co.uk/wp-content/uploads/2011/12/San-Fran-Toothpick-Model-41.jpg

Note: This seems like it's not enough

!!!

## What do we need?

```
ONE_OF(FeedItem,
  CASE(Pin, PinData *, pinData),
  CASE(user, UserData *, userData)
)
```

Note: variadic within the case, variadic across the cases, we need to communicate between the macros

!!!

## What do we need?

1. Variadic within the case
2. Variadic across the cases
3. Send the FeedItem prefix down to the cases
4. Send the case info up to the parent to put in match and constructors

!!!

## CONCATENATE

```
// This macro lets you x##y but x and/or y
// can be nested macro calls.
#define CONCATENATE(x, y) CONCATENATE1(x, y)
#define CONCATENATE1(x, y) CONCATENATE2(x, y)
#define CONCATENATE2(x, y) x##y
```

TODO attribute to stack overflow

!!!

## FOR_EACH

![for each](img/foreach.png)

Note:

!!!

## FOR_EACH

```objective-c
#define FOR_EACH_0(...)
#define FOR_EACH_1(what, x, ...) what(x)
#define FOR_EACH_2(what, x, ...)\
  what(x) \
  FOR_EACH_1(what, __VA_ARGS__)
#define FOR_EACH_3(what, x, ...)\
  what(x) \
  FOR_EACH_2(what, __VA_ARGS__)
// etc
```

TODO attribute the stack overflow

Note: one of these will be called, and cascades downwards

!!!

## FOR_EACH

```objective-c
#define FOR_EACH_NARG(...) \
  FOR_EACH_NARG_(__VA_ARGS__, FOR_EACH_RSEQ_N())
#define FOR_EACH_NARG_(...) \
  FOR_EACH_ARG_N(__VA_ARGS__)
#define FOR_EACH_ARG_N(_0, _1, /* … */, _7, N, ...) N
#define FOR_EACH_RSEQ_N() 8, 7, 6, 5, 4, 3, 2, 1, 0
```

Note: NARG will be invoked to select the suffix of the correct spot in the waterfall, the trick is that the numbers push back the others

!!!

## FOR_EACH

```
#define ARG_N(_0, _1, _2, _3, N, ...) N
ARG_N(4, 3, 2, 1, 0) // yeilds 0
ARG_N(x, 4, 3, 2, 1, 0) // yeilds 1
ARG_N(x, y, 4, 3, 2, 1, 0) // yeilds 2
```

!!!

## FOR_EACH

```objective-c
#define FOR_EACH_(N, what, ...) \
  CONCATENATE(FOR_EACH_, N)(what, __VA_ARGS__)
#define FOR_EACH(what, ...) \
  FOR_EACH_(FOR_EACH_NARG(__VA_ARGS__), what, __VA_ARGS__)
```

Note: Invoke the proper FOR_EACH_N using FOR_EACH_NARG

!!!

## FOR_DOUBLE_EACH

```objective-c
CASE(Pin, PinData *, pinData, NSNumber *, otherStuff)
// we want to run a macro on the type and value in a pairs
// FOO(type, value)
```

!!!

## FOR_DOUBLE_EACH

```
#define FOR_DOUBLE_EACH_0(...)
#define FOR_DOUBLE_EACH_1(what, x, y, ...) what(x, y)
#define FOR_DOUBLE_EACH_2(what, x, y, ...)\
  what(x, y) \
  FOR_DOUBLE_EACH_1(what, __VA_ARGS__)
```

Note: Similar but we're invoking the inner macro with two params

!!!

## FOR_DOUBLE_EACH

```
#define FDE_ARG_N(_0, /* … */, _1, F1, /* … */, _7, F7, N, ...) N
#define FDE_RSEQ_N() 8, 8, 7, 7, /* … */, 1, 1, 0
```

Note: Here's how we adjust the arg pushing

!!!

## Handling Case

```objective-c
CASE(Pin, PinData *, pinData, NSNumber *, otherStuff)
```

Note: Now we can run through the type and values and expand that to what we need

!!!

## Passing data upwards

```objective-c
#define CHILD(a, b) a , b , using##a##b 
#define PARENT(secret, c) RUN (c, secret)
#define RUN(x, y, z, secret) x woo secret y woo z
PARENT(password, CHILD(a, b))
```

Note: we return a list of arguments to be evaulated by another macro

!!!

## Passing data upwards

```objective-c
// name, (typ, arg)...
// returns:
//    (name, ifaceChunk, implChunk, privateChunk)...
//    where chunk =
//        starting at the line after @interface
//        and the line after @implementation
//        INCLUDES @end
#define CASE(name, typ, var, ...) \
/* … */
```

Note: Case returns it's name so the parent can use it, and a few chunks of code that rely on the knowledge of the parameters inside case

!!!

## Passing data down

We can do this by including an extra parameter inside our `FOR_EACH`

!!!

## Passing data

```objective-c
// returns:
//    (name, ifaceChunk, implChunk, privateChunk)...
#define CASE(name, typ, var, ...) \
/* … */
#define ONE_OF(prefix, ...) \
```

Note: We pass up 4 things from each case; that means we want to run a macro over groups of 4 things + we need the prefix (a constant)

!!!

## FOR_QUAD_CONST_EACH

```
#define ONE_OF(prefix, ...) \
  FOR_QUAD_CONST_EACH(FOO, prefix, __VA_ARGS__) \
  /* … */
```

Note: Invoke ONE_FORWARD_DECLARATION macro with the constant `prefix` parameter over every 4 arguments in va args

!!!

## FOR_QUAD_CONST_EACH

```objective-c
#define FOR_QUAD_CONST_EACH_0(...)
#define FOR_QUAD_CONST_EACH_1(what, c, x, y, z, w, ...)\
  what(c, x, y, z, w)
#define FOR_QUAD_CONST_EACH_2(what, c, x, y, z, w, ...)\
  what(c, x, y, z, w) \
  FOR_QUAD_CONST_EACH_1(what, c, __VA_ARGS__)

#define FQCE_ARG_N(/* … */, _1, F1, F11, F111, /* … */
#define FQCE_RSEQ_N() 8, 8, 8, 8, 7, 7, 7, 7, 6, 6, 6, 6, /* … */
```

Note: Same story as before

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

