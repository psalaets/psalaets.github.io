---
title: Exposing a node module as a global variable
layout: post
meta_description: Use browserify or webpack to make a CommonJS-only module available as a global variable
twitter_url: https://twitter.com/paulsalaets/status/605749980461023232
---

I wanted to try a node module in a browser app that uses global variables to share stuff.

### Versions used below

{% highlight javascript %}
{
  "superagent": "1.2.0",
  "browserify": "10.2.3",
  "webpack": "1.9.10"
}
{% endhighlight %}

I was thinking about using [superagent](https://github.com/visionmedia/superagent) to do ajax in a browser app that uses globals to access libraries and modules. The slightly opinionated superagent only supports CommonJS.

Two module bundlers that can make a CommonJS-only module available as a global are [browserify](https://github.com/substack/node-browserify) and [webpack](https://github.com/webpack/webpack).

## 1. Install the node module

Create a package.json if there isn't one already. This will be your record of what version of the module was used.

{% highlight bash %}
$ npm init -f
{% endhighlight %}

`-f` will [accept all the default options](/posts/quickly-accepting-defaults-of-npm-init-and-bower-init/) of npm init.

Then install the module.

{% highlight bash %}
$ npm install superagent -S
{% endhighlight %}

## 2. Create entry file

browserify and webpack will bundle dependencies of the entry file (and their dependencies, and dependencies of their dependencies, etc). In this case a dependency is anything pulled in through a `require()` call.

The entry file is just a CommonJS passthrough -- it exports the module we want to use.

{% highlight javascript %}
// entry.js
module.exports = require('superagent');
{% endhighlight %}

## 3a. Use browserify

browserify's `--standalone` option uses [umd](https://github.com/ForbesLindesay/umd) to make the resulting bundle work with most JavaScript module systems. A global variable is created if no module system is detected.

The argument to `--standalone` is the name of the global variable. It's named `request` below because that's the naming convention used in the [superagent docs](http://visionmedia.github.io/superagent/).

{% highlight bash %}
$ browserify entry.js --standalone request --outfile superagent-global.js
{% endhighlight %}

## 3b. Or use webpack

The command I use for webpack is similar to browserify.

{% highlight bash %}
$ webpack entry.js --output-library request --outfile superagent-global.js
{% endhighlight %}

It will default to creating a global variable named by the argument to `--output-library`. To generate a umd module or any other type, you'd also use `--output-library-target` and [specify the module type](http://webpack.github.io/docs/configuration.html#output-librarytarget).

## Final usage

Now in my browser app I can do

{% highlight html %}
<script src="superagent-global.js"></script>
<script>
  request.get('/foo').end(function(error, response) {
    console.dir(response.body);
  });
</script>
{% endhighlight %}