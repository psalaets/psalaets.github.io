---
title: Friction and bounce in p2.js
layout: post
twitter_url: https://twitter.com/paulsalaets/status/565141470278348802
meta_description: Controlling friction and bounce of bodies in the p2 physics engine
---

p2 is a JavaScript 2D physics library that works in the browser and node. Setting an object's friction and bounce is not obvious at first glance.

### Versions used below

{% highlight bash %}
{
  "p2": "0.6.0"
}
{% endhighlight %}

## Body doesn't have friction and bounce

[Body](http://schteppe.github.io/p2.js/docs/classes/Body.html) seems like the most sensible place to have friction and bounce properties but they're not there.

p2 handles it in a more flexible way at the cost of a little more initial set up. Here's how you do it...

## Assign Materials to Shapes

Bodies are made up of one or more Shapes and each Shape can have a [Material](http://schteppe.github.io/p2.js/docs/classes/Material.html). Materials have no physical properties. They act as identifiers representing whatever the Shape is made out of.

{% highlight javascript %}
// create body
var boxBody = new p2.Body({
  mass: 1
});

// add shape to body
var boxShape = new p2.Rectangle(width, height);
boxBody.addShape(boxShape);

// assign material to shape
var boxMaterial = new p2.Material();
boxShape.material = boxMaterial;
{% endhighlight %}

## Create ContactMaterials for pairs of Materials

A [ContactMaterial](http://schteppe.github.io/p2.js/docs/classes/ContactMaterial.html) is a pairing of two Materials. ContactMaterials have properties that control the physics when Shapes with those Materials collide. This is where friction and bounce, among other things, can be set.

{% highlight javascript %}
var boxVsBall = new p2.ContactMaterial(boxMaterial, ballMaterial, {
    // friction
    friction: 0.5,
    // bounce
    restitution: 1
    // see p2 docs for other options allowed here
});
{% endhighlight %}

The values to use is not an exact science. I tweak and check repeatedly until it feels right for my game. For inspiration see [p2's demos and examples](http://schteppe.github.io/p2.js/).

ContactMaterials must be added to the World so p2 knows about them.

{% highlight javascript %}
world.addContactMaterial(boxVsBall);
{% endhighlight %}

## Default ContactMaterial

If both colliding Shapes have a Material and the World has a ContactMaterial for that pair, it will be used. Otherwise the World's default ContactMaterial is used. The default ContactMaterial has reasonable defaults and can be modified.

{% highlight javascript %}
world.defaultContactMaterial.friction = 0;
{% endhighlight %}

If your objects will all interact with the same physics, this is a quick way to get going.