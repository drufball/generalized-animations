# Conceptual structure
This proposal suggests that animations be decomposed into 3 parts:

- __Inputs__ from the 'real world' or the web app that control the animation.
- An __animation model__ that maps inputs onto a *t value* from 0 to 1.
- An __output model__ that converts a *t value* into properties, which create the visual change of the animation.
 
Initially, this conceptual deconstruction could be used to support adding scroll position as a potential timeline input (see below). Eventually, it could be used to support a range of [animation types and models](/animation-models.md), including several physical models like springs.

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

## Output models  
Currently the only valid output for an animation is a sequence of keyframes. It is possible that we’ll allow registration of more generic effector functions to handle output in the future.

# Potential API

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

## Declarative
We need to expose:
- `ScrollTimelineAdapter`
- Triggers & Trigger registration
- New AnimationEffect subclasses

Dean’s proposal was for
- `animation-timebase`
- `animation-trigger`
- `animation-behavior`

This actually maps pretty nicely, though `animation-behavior` in Dean’s proposal is the composite operation. Here’s a slightly modified version of Dean’s proposal which leaves out animation-behavior and adds animation-effect:

```
Name: animation-timebase
Value: auto | scroll([<length-percentage> [<length-percentage>]?]?) | url(<media element>)
Initial: auto
Inherited: no
Animatable: no
```

The two optional paramaters in scroll represent zeroPoint and onePoint respectively. By default they are 0% and 100% - i.e. the scroll region is covered by an animation of 1 second duration.

Note that selecting ‘scroll’ causes the immediate containing block to be the container that the animation animates based off. There’s no declarative way to alter this.

```
Name: animation-trigger
Value: none | [scroll(<trigger> <snap point>) ]*
Initial: none
Inherited: no
Animatable: no
```

We need to bikeshed this heavily against the current scroll snap specification, but the basic point (as described by Dean) is that we’re re-using scroll snap’s element reference semantics to refer to points in the scroller.

```
Name: animation-effect
Value: none | spring(<number> <number> <number>) | linear-a-seek(<number> <number>)
Initial: none
Inherited: no
Animatable: no
```

This will also need heavy bikeshedding, but each choice opts the animation in for an alternative effect . We may want this to be list-valued but I’m not sure that makes sense.
