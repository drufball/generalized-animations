# Timeline animations
Timeline animations are driven by a single timeline input. The `play()` trigger starts an animation playing. The animation model scales the input value to a *t* of 0 at the beginning of the animation and 1 at the end. Animation duration is extracted from the output model.

A `startTime` passive value input registers the start of the animation into the timeline. Animation playback position is provided by the `currentTime` active value input. This can be set to seek the animation.

Timeline animation behavior is quite complex, for example:
- Output model duration is scaled by a playback rate
- Timeline animation models define an ‘active region’ that maps to the duration of the output model. Triggering `play()` when a timeline animation model’s `currentTime` is outside the active region causes the animation to start from the beginning; otherwise the `currentTime` doesn’t change
- Timeline animations can be paused - the `currentTime` (and output *t value*) is fixed while an animation is paused.

Timeline animation models can be used for scroll-linked animations by using the scroll region as a timeline.

Timeline animation models can be used for scroll-triggered animations, by registering an appropriate trigger on the `play()` slot. If the animation should be retriggered every time the trigger point is passed, this can be achieved by registering the trigger point to both the `cancel()` and `play()` triggers, in that order.

# State based animation models  
State based models provide more complex reactions to triggers. Rather than being driven by a timeline, these models are driven towards a default state. For example, a spring animation model animates  its *t value* towards a rest state. The animation is modeled by a force proportional to the distance from this rest target.

There is an important caveat to physics animations like this one: they operate in *t space*, not in the output space. This means that non-linear mappings in the output model (e.g. via the use of keyframes or timing functions) will produce unexpected results. It also means that developers need to be mindful of the mapping between *t space* and the output space to produce sensible effects.

## Example
The spring animation model exposes the following value slots:
- target *t* value (passive)
- current *t* value (active)
- springiness value (stiffness / mass) (passive)
- damping value (friction) (passive)
- duration value (passive)
- *dt* value (active)

Setting springiness, velocity or damping values will cause the duration to update, while setting the duration will adjust the damping value. The model provides two triggers: play() and cancel().

The collection of state based animation models will grow over time.
