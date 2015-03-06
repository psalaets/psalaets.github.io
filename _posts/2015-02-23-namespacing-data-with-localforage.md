---
title: Namespacing data with localForage
layout: post
twitter_url: https://twitter.com/paulsalaets/status/569863144421650432
meta_description: I run multiple static web apps at the same domain, so their offline storage is in the same bucket. Luckily localForage has a way to keep the data separated.
---

Through the magic of GitHub Pages I've got multiple static web apps hosted at the same domain. That means offline storage (IndexedDB, WebSQL, LocalStorage) for these apps is in the same bucket. Luckily localForage provides a way to keep the data separated.

### Versions used below

{% highlight javascript %}
{
  "localforage": "1.2.2"
}
{% endhighlight %}

Edited on 27 Feb 2015: Previously I said you need to set `name` *and* `storeName` but actually only `name` is required for namespacing data.

## Set a custom name

Override the [default config](http://mozilla.github.io/localForage/#config) by calling `localforage.config()`. This *must* be done before any [data API methods](http://mozilla.github.io/localForage/#data-api) are called.

{% highlight javascript %}
localforage.config({
  name: 'My App'
});
{% endhighlight %}

The value for `name` should be unique across all other localForage instances on the domain. The value can be anything reasonable, I usually use my app's name.

If you just need to keep your app's localForage data separate from other apps on the same domain, you're done.

## Creating multiple instances of localForage

Since 0.9.3 you can make more localForage instances using `localforage.createInstance()`. The [options](http://mozilla.github.io/localForage/#config) accepted by `localforage.config()` are also accepted here.

{% highlight javascript %}
var otherStore = localforage.createInstance({
  name: 'Other Store'
});
{% endhighlight %}

At this point you'd have two localForage instances: `localforage` and `otherStore`.

## Full example with multiple instances

I use native `Promise` below so it works without polyfills in [anything but IE](http://caniuse.com/#feat=promises).

{% highlight javascript %}
var store1 = localforage.createInstance({
  name: 'Store 1'
});

var store2 = localforage.createInstance({
  name: 'Store 2'
});

var key = 'key';

// set value in each store using same key (but they're namespaced so it's all good)
Promise.all([
  store1.setItem(key, 'value1'),
  store2.setItem(key, 'value2')
]).then(function() {
  // now get values
  return Promise.all([
    store1.getItem(key),
    store2.getItem(key)
  ]);
}).then(function(values) {
  console.log('value from store1 is ' + values[0]);
  console.log('value from store2 is ' + values[1]);
});
{% endhighlight %}

will print

{% highlight text %}
value from store1 is value1
value from store2 is value2
{% endhighlight %}