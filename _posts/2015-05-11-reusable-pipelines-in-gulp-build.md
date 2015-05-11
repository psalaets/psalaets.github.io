---
title: Making reusable pipelines in a Gulp build
layout: post
meta_description: How to make reusable processing pipelines in a Gulp build
twitter_url: https://twitter.com/paulsalaets/status/597711855616978944
---

Since a gulp build is just JavaScript code, you can make small processing pipelines and reuse them.

### Versions used below

{% highlight javascript %}
{
  "gulp": "3.8.11",
  "lazypipe": "0.2.3",
  "stream-combiner2": "1.0.2",
  "bun": "0.0.11"
}
{% endhighlight %}

## Why reusable pipelines?

A typical pattern for gulp tasks is

{% highlight javascript %}
gulp.task('taskName', function() {
  return gulp.src('...')
    .pipe(stepA())
    .pipe(stepB())
    .pipe(stepC())
    .pipe(gulp.dest('...'));
});
{% endhighlight %}

If some other task needs to run `stepB` and `stepC` you could copy/paste the code or move it out into something reusable. The latter will result in a more maintainable build.

In this post I'll make a reusable pipeline that will [uglify](https://www.npmjs.com/package/gulp-uglify) and [rev](https://www.npmjs.com/package/gulp-rev) JavaScript files.

## The manual way

Here's a way to do it with no help from outside modules.

{% highlight javascript %}
// make reusable pipeline
function processJs(inputStream) {
  return inputStream
    .pipe(uglify())
    .pipe(rev());
}

// use it
gulp.task('taskName', function() {
  return processJs(gulp.src('...'))
    .pipe(gulp.dest('...'));
});
{% endhighlight %}

This works but it reads out of order -- the call to `gulp.src` is not first. In gulp builds I'm used to seeing code in the same order that the file processing happens in.

It looks worse when there are steps before the reusable pipeline.

{% highlight javascript %}
gulp.task('taskName', function() {
  var before = gulp.src('...')
    .pipe(stepA())
    .pipe(stepB());

  return processJs(before)
    .pipe(gulp.dest('...'));
});
{% endhighlight %}

### Why pass inputStream in?

It seems like the previous examples could be cleaner by not passing `inputStream` into `processJs`. We could just use `processJs()` directly in the gulp task.

{% highlight javascript %}
// DOES NOT WORK

// make reusable pipeline
function processJs() {
  return uglify()
    .pipe(rev());
}

// use it
gulp.task('taskName', function() {
  return gulp.src('...')
    .pipe(processJs())
    .pipe(gulp.dest('...'));
});
{% endhighlight %}

This runs but it doesn't give the desired effect.

In node streams `a.pipe(b)` returns `b`. That means `processJs` returns a `rev` stream and `gulp.src` will pipe directly into it. Even though `uglify` is also piped into `rev`, nothing pipes into `uglify` so it has no effect.

Okay, on to some better ideas.

## Using lazypipe

[lazypipe](https://www.npmjs.com/package/lazypipe) creates a function that lazily creates your pipeline.

{% highlight javascript %}
var lazypipe = require('lazypipe');

// make reusable pipeline
var processJs = lazypipe()
  .pipe(uglify)
  .pipe(rev);

// use it
gulp.task('taskName', function() {
  return gulp.src('...')
    .pipe(processJs())
    .pipe(gulp.dest('...'));
}
{% endhighlight %}

This is nicer because the code is in the same order as the file processing.

### Passing parameters to gulp plugins

The previous code example uses `.pipe(uglify)` instead of `.pipe(uglify())` because lazypipe will invoke `uglify` for us when needed. We lost our chance to pass an options object to `uglify`.

lazypipe's `.pipe()` accepts extra parameters which are passed directly to the plugin.

{% highlight javascript %}
var uglifyOptions = {
  mangle: false
};

var processJs = lazypipe()
  .pipe(uglify, uglifyOptions)
  .pipe(rev);
{% endhighlight %}

### Gotcha with gulp-if

With lazypipe it's tricky to use streams that take other streams, e.g. [gulp-if](https://www.npmjs.com/package/gulp-if).

Common gulp-if usage looks like

{% highlight javascript %}
.pipe(gulpif(condition, thenThisStream(), otherwiseThisStream()))
{% endhighlight %}

If we do that when building the lazypipe it's no longer lazy and no longer reusable. There is a [way around that](https://github.com/OverZealous/lazypipe#using-with-more-complex-function-arguments-such-as-gulp-if) but this is more special case handling than I'd like in a build file.

## Using stream-combiner2

[stream-combiner2](https://www.npmjs.com/package/stream-combiner2) wraps a series of streams into a single duplex stream.

{% highlight javascript %}
var combiner = require('stream-combiner2');

var wrapper = combiner(a, b, c);
{% endhighlight %}

Things to note:

1. `a`, `b` and `c` are piped together inside `wrapper`
2. Input to `wrapper` is the input to `a`
3. Output of `c` is the output of `wrapper`
4. Errors from `a`, `b` and `c` become errors of `wrapper`

With that in mind, we can do

{% highlight javascript %}
var combiner = require('stream-combiner2');

// make reusable pipeline
function processJs() {
  return combiner(
    uglify(),
    rev()
  );
}

// use it
gulp.task('taskName', function() {
  return gulp.src('...')
    .pipe(processJs())
    .pipe(gulp.dest('...'));
}
{% endhighlight %}

This is my preferred approach for creating reusable pipelines in a gulp build.

### Similar module

[bun](https://www.npmjs.com/package/bun) works like stream-combiner2 except it takes an *array* of streams.