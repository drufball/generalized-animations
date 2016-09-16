# generalized-animations
This proposal extends the current [web animation API](https://developer.mozilla.org/en-US/docs/Web/API/Animation) to support a broader range of animations, including physics-style effects, scroll triggering, and springs. More information is available in the [EXPLAINER](EXPLAINER.md).

# Examples
Here are a few examples of the APIs (procedural and declarative) in use:

Shrinking headers are headers that shrink in size as a scroll region is
scrolled. When they get to a certain size they scroll out of view.

Shrinking headers using JavaScript:
```
<div id='scroller'>
<div id='header'></div>
</div>
<script>
var animation = header.animate(
  {height: ['100px', '20px'], translate: ['0px 0px', '0px 80px']}, 
  {duration: 1000, fill: 'both'});
animation.timeline = new ScrollTimelineAdapter('0px', '80px', scroller);
</script>
```

Or declaratively:
```CSS
@keyframes scroll-header {
  0% { height: 100px; translate: 0px 0px; }
  100% { height: 20px; translate: 0px 80px; }
}

#header {
  animation-timebase: scroll(0px, 80px);
  animation: scroll-header 1s;
}
```

Springs are a commonly used UI paradigm - an element "springs" from its current
position to a new one. They're controlled by two parameters - a 'zeta' (which
measures whether the spring is underdamped or overdamped) and an omega0 (which
measures the oscillation frequency).

Applying a spring effect using JavaScript:
```javascript
new Animation(new ScrollAnimationEffect(springer, {translate: ["0px" "400px"]}, {zeta: 0.9, omega0: "0.5Hz"}));
```

Or declaratively:
```CSS
@keyframes spring {
  0% { translate: 0px; }
  100% { translate: 400px; }
}

#springer {
  animation-effect: scroll(0.9 0.5Hz);
  animation: spring 1s; /* do we need a duration here? */
}
```

Another commonly requested effect is being able to trigger an animation at a particular point in a scroller's 
scroll.

Triggering an animation when passing 50% scroll (unless the animation is currently playing):
```
<div id='scroller'>
<div id='animation'>
</div>
</div>
<script>
var a = animation.animate(keyframes, timingProperties);
a.registerTrigger("play", triggerFromScrollRegion(scroller, "50%"));
</script>
```

Or declaratively:
```CSS
animation: keyframes var(--timing);
animation-trigger: scroll("play" 50%);
```

Retriggering an animation *every time* 50% scroll is passed:
```
<div id='scroller'>
<div id='animation'>
</div>
</div>
<script>
var a = animation.animate(keyframes, timingProperties);
a.registerTrigger("cancel", triggerFromScrollRegion(scroller, "50%"));
a.registerTrigger("play", triggerFromScrollRegion(scroller, "50%"));
</script>
```

Or declaratively:
```CSS
animation: keyframes var(--timing);
animation-trigger: scroll("cancel" 50%) scroll("play" 50%);
```

