---
layout: post
title: transitionend Promise
image: https://vaggrippino.github.io/blog/images/20180424_041449_transitionend_Promise_demo.gif
---
<img style="display: block; margin: auto;" alt="demo" src="https://vaggrippino.github.io/blog/images/20180424_041449_transitionend_Promise_demo.gif">
I'm working on a page that displays thumbnail images. When I click on a thumbnail I want it to show an info box containing more image details. I'm using a CSS transition on the info box and I want to populate the image details while the info box is hidden. If the info box is already visible from clicking on a different thumbnail, I need to hide it first and populate the image details after the transition completes.

Here's some pseudocode that shows what I want to happen...

{% highlight javascript %}
thumbnail.addEventListener('click', e => {
  if (infoBox.isVisible()) {
    // hide the info box (CSS transition)
    // after the transition ends, populate the image details
  } else {
    // populate the image details immediately
  }

  // show the info box (CSS transition)
})
{% endhighlight %}

The browser triggers a [`transitionend` event](https://developer.mozilla.org/en-US/docs/Web/Events/transitionend) when a transition finishes, but it's not fired at all if the info box is already hidden. A function that hides the element and returns a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises) would be ideal. The Promise would be resolved after the transition completed or immediately if the info box was already hidden, but I don't use JavaScript to hide or show an element. I typically just toggle a `hidden` class and let the CSS animation do the work, so we can't determine when the animation has completed.

I can't just return a Promise from the event handler for `transitionend` because I don't call the event handler. I just pass it as an argument to [`addEventListener()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener) and the browser calls it. But there's always a way...

I searched for *promises for CSS transition events*, but the higher ranking search result has a **fatal flaw**. In a *pen* (codepen.io) entitled "CSS Transition End with a Promise", a Promise *is* resolved when the transition completes, but **the event handler is never removed from the element**. The code attempts to remove the handler, but [`removeEventListener()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/removeEventListener) is passed a function that wasn't attached as the event handler. **The actual event handler is an anonymous function** that calls the function which the author tries to remove. If you use this code it could eventually cause a problem (What kind of problem? How soon? ... this needs testing) as new event handlers are infinitely attached to the element. 

I found a good solution on my second attempt at [this *Gist*](https://gist.github.com/davej/44e3bbec414ed4665220). The trick is a function that returns a Promise and makes a CSS (or CSS class) change that causes a transition, then immediately attaches an event handler for the `transitionend` event in which it removes itself and resolves the promise.

Here's my demo inspired by the Gist:

<p data-height="431" data-theme-id="dark" data-slug-hash="pVgjjj" data-default-tab="js,result" data-user="VAggrippino" data-embed-version="2" data-pen-title="CSS `transitionend` event with a Promise" class="codepen">See the Pen <a href="https://codepen.io/VAggrippino/pen/pVgjjj/">CSS `transitionend` event with a Promise</a> by Vince Aggrippino (<a href="https://codepen.io/VAggrippino">@VAggrippino</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>
