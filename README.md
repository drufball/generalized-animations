# generalized-animations
This proposal modifies the current [web animation API](https://developer.mozilla.org/en-US/docs/Web/API/Animation) to support a broader range of animations, including physics-style effects, scroll triggering, and springs. More information is available in the [EXPLAINER](EXPLAINER.md).

# Motivations
UI design has become an advanced science with increasingly nuanced applications. Animations have proven to be a particularly powerful tool, making interfaces more pleasing and intuitive to the user. As usage of animations has increased, designers have adopted best practices that have moved beyond simple time based animations.

Two paradigms common on the web (and in general) are animations controlled by scroll position and animations that [model physics](https://iamralpht.github.io/physics/). Currently, developers must map these models onto time based animations. This requires specifying a cumbersome number of [keyframes](https://developer.mozilla.org/en-US/docs/Web/CSS/@keyframes) or risking performance by modifying [`currentTime`](https://developer.mozilla.org/en-US/docs/Web/API/Animation/currentTime) using script.

# Examples
Two widely used UX paradigms on the web that are not currently well supported are shrinking headers and spring animations.

```javascript
TODO: example code for shrinking header
```

```javascript
TODO: example code for spring function
```

