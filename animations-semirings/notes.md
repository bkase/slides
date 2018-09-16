
ROUGH:

* Consider choreographing animations
  * "well-designed" applications have these all over the place
  * consistency and design language
* This is small enough to fit in our presentation but has enough going on that it will illustrate my points
* Constrained problem, should be easy
    * describe animations
    * Easy to read, easy to maintain, reuse, and reason about

* Basic.mp4

* Idiomatic iOS way:
  * UIView.animate(.....)

      ```
          UIView.animate(withDuration: 2.0,delay:0, options:.curveEaseInOut, animations: {
        resize(v1)(CGFloatAverage(10))
    }, completion: nil)
    ```

      * okay so if we need to tweak the curves, we need to wrap it in side-effecting CATransactions

      * How do we reverse this??
     * We can't "build an animation" this way, it just RUNS
       * We need callback hell for sequencing things
      * We can mitigate callback hell if we make reuse even more difficult with keyframes

```
      UIView.animateKeyframes(withDuration: 1, delay: 0, options: .calculationModeCubic, animations: {
    UIView.addKeyframe(withRelativeStartTime: 0, relativeDuration: 0.5, animations: {
        self.circleView.transform = .identity
    })
    UIView.addKeyframe(withRelativeStartTime: 0.5, relativeDuration: 0.5, animations: {
        self.circleView.transform = CGAffineTransform(scaleX: 0, y: 0)
    })
}, completion: nil)
```

        * Moreover we can't nest keyframes either so it's not truly composable
    * People claim this is "nice and declarative"
      * (as I was preparing for this talk)
      * Is it really declarative?
        * Side-effects / mutation
        * Composability
        * Reuse

  * Okay okay, coreanimation
      * We can use `CABasicAnimation` to describe the primitives and luckily they don't run automatically
      * we can even do the looped thing
    * If we want to compose animations, we sort-of kind-of can with `CAAnimationGroup`, but don't animate two different layers in the same group!
        * From what I can tell, the only way to compose animations that affect different layers is adhoc by setting beginTime to duration*i in a for-loop
      * From what I can tell, there is no way to compose animations in parallel, you're just supposed to side-effectly start them at the same time, again hurting reuse


* Let's use coreanimation
* Why don't I like this
    * Composability
    * Side-effects
    * reuse?
* Let's do it using uiview-anmiate
* Why don't I like this
    * People claim this is "nice and declarative"
      * (as I was preparing for this talk)
      * Is it really declarative?
        * Side-effects / mutation
        * Composability
        * Reuse

* Declarativity is a noble goal
* Back to description
* Rich algebraic structure:
* Duality
    * Parallel and sequence
* Orthoganility
* Algebraicness -- point to other talk
* Show the concrete representation
* Show how stuff is built
* Go over the viewer
* You could compile this into CoreAnimation instructions with little modification to the representation

