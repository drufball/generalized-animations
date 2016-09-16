# generalized-animations
This proposal extends the current [web animation API](https://developer.mozilla.org/en-US/docs/Web/API/Animation) to support a broader range of animations, including physics-style effects, scroll triggering, and springs. More information is available in the [EXPLAINER](EXPLAINER.md).

# Examples
Two widely used UX paradigms on the web that are not currently well supported are shrinking headers and spring animations.

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

Applying a spring function using JavaScript:
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
  animation: spring 1s;
}
```
