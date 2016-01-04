---
title: Selecting a broadphase in p2.js
layout: post
meta_description: Choosing the right broadphase for your game can have performance benefits.
twitter_url: https://twitter.com/paulsalaets/status/618833363139469313
---

Choosing the right broadphase for your game can have performance benefits.

### Versions used below

{% highlight javascript %}
{
  "p2": "0.6.1"
}
{% endhighlight %}

## 2D collision detection primer

2D collision detection is typically broken down into two main steps: broadphase and narrowphase. Broadphase happens first in execution but its usefulness is more apparent after talking about narrowphase, so I'll cover narrowphase first.

### Narrowphase

Narrowphase is where body vs body collision checks are done. This involves some fairly expensive math depending on what shapes the bodies have.

If there is a cheaper check to determine that two shapes definitely aren't colliding, we can skip the narrowphase check and get a performance win.

The broadphase is where that cheaper check is done.

### Broadphase

The broadphase determines what pairs *could* be colliding. The definition of could differs by implementation.

Various implementations scale better than others and some are best suited for certain arrangements of game objects. It's easy to swap them out since they follow the same interface.

### Simplified collision detection overview

Broadphase is cheap but imprecise. Narrowphase is precise but expensive.

Here's how they work together:

1. Start with a list of bodies. These are the physics objects in the game.
2. Broadphase determines what pairs of bodies are worth a closer look. These pairs are passed on to narrowphase.
3. Given pairs from step 2, narrowphase does body vs body collision checks.
4. Apply physics and run game logic based on collisions.

## Measuring performance

Picking the right broadphase for a game is mostly about which one makes the physics engine run fastest.

### Timing

Worlds have a profiling flag which is off by default. When enabled, p2 will record how long the previous step took in milliseconds.

{% highlight javascript %}
world.doProfiling = true;

// now this will be populated after each step
world.lastStepTime
{% endhighlight %}

A handler for World's `postStep` event is a good place to check `lastStepTime`.

{% highlight javascript %}
var stepTimes = [];

world.on('postStep', function() {
  stepTimes.push(world.lastStepTime);
});
{% endhighlight %}

From here you can print, find average, find median, etc. Lower is better.

That's a decent way to compare implementations. Here are the broadphase options that p2 has out of the box.

## Broadphase implementations in p2

### Naive

[NaiveBroadphase](http://schteppe.github.io/p2.js/docs/classes/NaiveBroadphase.html) is as simple as it gets. It considers every body to be a potential collider with every other body. This results in the maximum number of narrowphase checks (n<sup>2</sup>).

If your game is simple enough this may work the best because the overhead of the other broadphase implementations won't be worth it.

{% highlight javascript %}
world.broadphase = new p2.NaiveBroadphase();
{% endhighlight %}

### Sweep and Prune

[SAPBroadphase](http://schteppe.github.io/p2.js/docs/classes/SAPBroadphase.html) sorts bodies along an axis and then moves down that list finding pairs by looking at body size and position of the next bodies.

Control what axis to sort along by setting the `axisIndex` property.

{% highlight javascript %}
var sap = new p2.SAPBroadphase();
sap.axisIndex = 0; // 0 for x axis, 1 for y axis

world.broadphase = sap;
{% endhighlight %}

If game objects are spread out along the x axis, like in a sidescroller, `axisIndex = 0` would be better. If game objects are spread out along the y axis, `axisIndex = 1` is better.

This is the default broadphase in p2 (with `axisIndex = 0`).

### Grid

*Note: As of version 0.7.0 GridBroadphase is no longer in p2.js*

[GridBroadphase](http://schteppe.github.io/p2.js/docs/classes/GridBroadphase.html) divides space into a grid of cells. Bodies are placed into the cells they overlap and bodies in the same cell are paired.

GridBroadphase needs to know the size of the space ahead of time so the game must have [predetermined bounds](/posts/planes-in-p2/).

{% highlight javascript %}
var grid = new p2.GridBroadphase({
  // Left edge of grid
  xmin: -100,
  // right edge of grid
  xmax:  100,
  // bottom edge of grid
  ymin: -100,
  // top edge of grid
  ymax:  100,
  // number of cells along x axis
  nx:    10,
  // number of cells along y axis
  ny:    10
});

world.broadphase = grid;
{% endhighlight %}

Cell width will be `(xmax - xmin) / nx` and cell height will be `(ymax - ymin) / ny`.

Ratio between cell size and the average size of game objects is important when trying to tune performance. I typically find that having cells 1-4 times the size of my game objects gives the best performance.

Every game is different so don't take my word for it. Run benchmarks.

## Another thing to tweak

When approximating a body's size, the p2 broadphase implementations use a bounding shape which can be simpler than the body's actual shape.

{% highlight javascript %}
// axis-aligned bounding box, a.k.a. rectangle with no rotation
broadphase.boundingVolumeType = p2.Broadphase.AABB; // this is the default

// or

broadphase.boundingVolumeType = p2.Broadphase.BOUNDING_CIRCLE;
{% endhighlight %}

Maybe if your bodies are mostly circles `BOUNDING_CIRCLE` performs better?

## Takeaways

Rules I follow when selecting a broadphase are

1. Start with the simplest broadphase and change it when it's not performing well enough.
2. Run benchmarks relevant to _your_ game to compare broadphase implementations.