---
title: Looking at node's require()
layout: post
meta_description: Exploring various ways to load files using require.
twitter_url: https://twitter.com/paulsalaets/status/631113764369268736
---

Exploring various ways to load files using `require()`.

### Versions used below

{% highlight bash %}
node 0.10.36
{% endhighlight %}

## Common usages

If you've used node before these are familiar.

### Core Module

Load one of node's [built-in modules](https://nodejs.org/api/index.html).

{% highlight javascript %}
var http = require('http');
{% endhighlight %}

### Dependency from npm

Load something that's in `node_modules/`, which was most likely installed via [npm](https://www.npmjs.com/).

{% highlight bash %}
$ npm install faye -S
{% endhighlight %}

{% highlight javascript %}
var faye = require('faye');
{% endhighlight %}

### Relative path

Load a file relative to the current file.

{% highlight javascript %}
var helper = require('./util/helper.js');
{% endhighlight %}

The file extension is optional.

### Absolute path

I've never used this but it's possible to load files with an absolute path.

{% highlight javascript %}
var helper = require('/Users/paul/js/helper.js');
{% endhighlight %}

Again, the file extension is optional.

## Other usages

### Directory

Given this directory structure

<pre>
├── stuff/
│   ├── foo.js
│   ├── bar.js
│   └── index.js
└── main.js
</pre>

In `main.js` you could

{% highlight javascript %}
var stuff = require('./stuff');
{% endhighlight %}

and it'd load `./stuff/index.js`.

`index.js` can require the other files in `stuff/`. This is handy for making a directory sort of act as an embedded module.

One benefit to omitting the file extension in relative path requires is that you can break a large file down into a directory of smaller files without having to update all the `require` calls.

{% highlight javascript %}
// this code doesn't know if it's getting ./stuff.js or ./stuff/index.js
var stuff = require('./stuff');
{% endhighlight %}

### Json file

When you require a json file you get the parse result.

{% highlight javascript %}
{
  "colors": ["red", "green", "blue"]
}
{% endhighlight %}

{% highlight javascript %}
var colors = require('./data.json').colors;
{% endhighlight %}

### Any file from a dependency

If you're okay with relying on a module's internal file structure, you can require any file from it.

{% highlight bash %}
$ npm install <module> -S
{% endhighlight %}

{% highlight javascript %}
var something = require('<module>/lib/something');
{% endhighlight %}

This allows you to programmatically determine a module's version.

{% highlight javascript %}
var gulpVersion = require('gulp/package.json').version;
{% endhighlight %}
