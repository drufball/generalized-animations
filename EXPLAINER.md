# General structure
This proposal suggests that animations be decomposed into 3 parts:

- __Inputs__ from the 'real world' or the web app that control the animation.
- An __animation model__ that maps inputs onto a *t value* from 0 to 1.
- An __output model__ that converts a *t value* into properties, which create the visual change of the animation.

## Inputs
Inputs can take many forms including clock time, scroll position, or physical parameters. Generally, inputs can be separated into various classes:

__Timelines__
Timelines are essentially continuous ranges. Timelines can be *unbounded* (span to infinity in both directions) or *bounded*. The system clock is an example of an unbounded timeline, while scroll position is bounded.

__Triggers__
Triggers represent instantaneous changes in state for the animation. Triggers are signals like `play()` or `pause()` from the current web animations API, but they can also be derived from a timeline. For example, an animation could be set to trigger once scroll position has passed a certain point.

__Value inputs__
Value inputs can be thought of as configuration or internal state for the animation. These are things like `currentTime` from the current web animations API, but could also include constants used for physical animation models. Values can be __passive__, where they do not change frame-to-frame, or __active__ where they do change.
