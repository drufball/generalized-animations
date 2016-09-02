# Conceptual structure
This proposal suggests that animations be decomposed into 3 parts:

- __Inputs__ from the 'real world' or the web app that control the animation.
- An __animation model__ that maps inputs onto a *t value* from 0 to 1.
- An __output model__ that converts a *t value* into properties, which create the visual change of the animation.

## Input types
Inputs can take many forms including clock time, scroll position, or physical parameters. Generally, inputs can be separated into various classes:

__Timelines__  
Timelines are essentially continuous ranges. Timelines can be *unbounded* (span to infinity in both directions) or *bounded*. The system clock is an example of an unbounded timeline, while scroll position is bounded.

__Triggers__  
Triggers represent instantaneous changes in state for the animation. Triggers are signals like `play()` or `pause()` from the current web animations API, but they can also be derived from a timeline. For example, an animation could be set to trigger once scroll position has passed a certain point.

__Value inputs__  
Value inputs can be thought of as configuration or internal state for the animation. These are things like `currentTime` from the current web animations API, but could also include constants used for physical animation models. Values can be __passive__, where they do not change frame-to-frame, or __active__ where they do change.

## Animation models
Animation models control operation of the animation. An animation model defines what input slots are available, and converts inputs to a single *t value* which is fed into the output model.

## Outputs 
Currently the only valid output for an animation is a sequence of keyframes. It is possible that we’ll allow registration of more generic effector functions to handle output in the future.

# API

## Imperative
__Inputs__  
Timeline inputs are modeled by the [`AnimationTimeline`](https://www.w3.org/TR/web-animations/#animationtimeline) interface. A consequence of this is that all inputs of timeline type have an implementation of the getAnimations function. Timelines are provided to the constructor of [`Animation`](https://www.w3.org/TR/web-animations/#animation) objects and are subsequently accessible (and modifiable) via the timeline property.

Trigger slots are exposed as zero-argument functions on [`Animation`](https://www.w3.org/TR/web-animations/#animation) objects, although a separate affordance (see below) will be exposed to register triggers programmatically.

Value inputs are exposed as read/write properties on [`Animation`](https://www.w3.org/TR/web-animations/#animation) objects.

__Models__  
Models are an amalgamation of [`Animation`](https://www.w3.org/TR/web-animations/#animation) and part of the [`AnimationEffect`](https://www.w3.org/TR/web-animations/#animationeffectreadonly) objects (`AnimationEffects` deal with both mapping from timing values to ‘t’ values, and output). There are a couple of options for the new model types:  
 
 - We could create new `Animation` objects. It doesn’t really make much sense to have a full AnimationEffect underlying e.g. the Linear Acceleration Seeking model, but it doesn’t hurt either.
- We could create new `AnimationEffect` subclasses and matching `Animation` objects. So we’d have a `LASAnimationEffect` and matching `LASAnimation` (or something similar).
- We could create new `AnimationEffect` subclasses and accept some complexity when registering triggers on `Animations` (some would be handled at the `Animation` level while others would proxy down to the `AnimationEffect`)
- We could create new `AnimationEffect` subclasses and wrap `Animations` in `Model` classes.

The current preferred option is #3 as trigger registration should probably just take strings anyway.

__New concepts__  
Converting a scroll region to a timeline will require an adapter class, maybe:
```webidl
interface ScrollTimelineAdapter extends AnimationTimeline {
  CSSLengthValue zeroPoint
  CSSLengthValue onePoint
  Element scrollElement
}
```

The zeroPoint is the position on the scroller at which the animation is considered to be at zero milliseconds. The onePoint is the position at which the animation is considered to be at 1000 milliseconds (i.e. one second).

Triggers need to be registered:
```webidl
partial interface Animation {
  registerTrigger(DOMString triggerName, AnimationTrigger trigger);
}
```

Triggers also need to be constructible:
```javascript
AnimationTrigger triggerFromScrollRegion(Element scrollElement, CSSLengthValue triggerPoint)
```

There are likely to be other sources of triggers too - for example it would be useful to be able to trigger a particular element entering or leaving a scroll region’s viewport.

__Examples__  
Trigger an Animation `a` to play when `el` scrolls past 50%:

```javascript
a.registerTrigger(“play”, triggerFromScrollRegion(el, 
    CSSLengthValue.from(“50%”)));
```

__Timeline animations__  
Timeline animations are driven by a single timeline input. The `play()` trigger starts an animation playing. The animation model scales the input value to a *t* of 0 at the beginning of the animation and 1 at the end. Animation duration is extracted from the output model.

A `startTime` passive value input registers the start of the animation into the timeline. Animation playback position is provided by the `currentTime` active value input. This can be set to seek the animation.

Timeline animation behavior is quite complex, for example:
- Output model duration is scaled by a playback rate
- Timeline animation models define an ‘active region’ that maps to the duration of the output model. Triggering `play()` when a timeline animation model’s `currentTime` is outside the active region causes the animation to start from the beginning; otherwise the `currentTime` doesn’t change
- Timeline animations can be paused - the `currentTime` (and output *t value*) is fixed while an animation is paused.

Timeline animation models can be used for scroll-linked animations by using the scroll region as a timeline.

Timeline animation models can be used for scroll-triggered animations, by registering an appropriate trigger on the `play()` slot. If the animation should be retriggered every time the trigger point is passed, this can be achieved by registering the trigger point to both the `cancel()` and `play()` triggers, in that order.

__Physics animation models__  
Physics animations models provide more complex reactions to triggers.  Rather than being driven by a timeline, physics animation models are driven by state perturbations. For example, a spring animation model will animate its *t value* towards a target value, modeled by a force proportional to the distance from the target.

There is an important caveat to physics animations: they operate in *t space*, not in the output space. This means that non-linear mappings in the output model (e.g. via the use of keyframes or timing functions) will produce unexpected results. It also means that developers need to be mindful of the mapping between *t space* and the output space to produce sensible effects.

*Examples*  
The spring animation model exposes the following value slots:
- target *t* value (passive)
- current *t* value (active)
- springiness value (stiffness / mass) (passive)
- damping value (friction) (passive)
- duration value (passive)
- *dt* value (active)

Setting springiness, velocity or damping values will cause the duration to update, while setting the duration will adjust the damping value. The model provides two triggers: play() and cancel().

The linear acceleration seeking model exposes the following value slots:
- target *t* value (passive)
- current *t* value (active)
- duration value (passive)
- target *dt* value (passive)
- current *dt* value (active)
- acceleration value (active)

For a given target *t*, *dt*, and duration, the model will adjust the acceleration in a piecewise linear fashion to bring the current *t* value in line with the target *t* value, with a rate of change specified by the target *dt*.

There may be multiple solutions, or alternatively zero solutions to the seek. If there are zero solutions then the animation cancels itself (reporting via a cancelled promise that this occurred). If there are multiple solutions then the least energetic (lowest area under the acceleration curve) solution is selected.

The collection of physics animation models will grow over time.
