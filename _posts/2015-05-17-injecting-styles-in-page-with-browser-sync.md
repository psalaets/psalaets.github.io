---
title: Injecting styles into a page with BrowserSync
layout: post
meta_description: Using BrowserSync to update css on a page without reloading the browser.
twitter_url: https://twitter.com/paulsalaets/status/600300124925919234
---

Every time I open the BrowserSync docs I find something awesome.

### Versions used below

{% highlight javascript %}
{
  "browser-sync": "2.7.1",
  "gulp": "3.8.11",
  "gulp-sass": "2.0.0"
}
{% endhighlight %}

## BrowserSync

[BrowserSync](http://www.browsersync.io/) is a great module that I use for a static web server with auto-reload. It does a lot more than auto-reload but that's my main use for it.

When paired with Gulp, BrowserSync can inject css into a page -- *no page reload needed*.

## Basic Setup

Given this project structure

{% highlight bash %}
.
├── app/
│   ├── index.html
│   └── css/
│       └── app.css
├── gulpfile.js
└── package.json
{% endhighlight %}

Here is a simple gulp build that uses BrowserSync to serve static files and reload browsers when css or html changes.

{% highlight javascript %}
var gulp = require('gulp');
// create new instance of BrowserSync
var browserSync = require('browser-sync').create();

gulp.task('watch', function(gulpCallback) {
  browserSync.init({
    // serve out of app/
    server: './app/',
    // launch default browser as soon as server is up
    open: true
  }, function callback() {
    // (server is now up)

    // set up watch to reload browsers when source changes
    gulp.watch(['app/index.html', 'app/css/*'], browserSync.reload);

    // notify gulp that this task is done
    gulpCallback();
  });
});
{% endhighlight %}

This is nice but the browser reloads the entire page when the css changes.

BrowserSync can do better.

## Injecting Styles into the Page

With a small change to the watches it will push css updates without a page reload.

{% highlight javascript %}
var gulp = require('gulp');
// create new instance of BrowserSync
var browserSync = require('browser-sync').create();

gulp.task('watch', function(gulpCallback) {
  browserSync.init({
    // serve out of app/
    server: './app/',
    // launch default browser as soon as server is up
    open: true
  }, function callback() {
    // (server is now up)

    // watch html and reload browsers when it changes
    gulp.watch('app/index.html', browserSync.reload);

    // watch css and stream to BrowserSync when it changes
    gulp.watch('app/css/*', function() {
      // grab css files and send them into browserSync.stream
      // this injects the css into the page
      gulp.src('app/css/*')
        .pipe(browserSync.stream());
    });

    // notify gulp that this task is done
    gulpCallback();
  });
});
{% endhighlight %}

This is a very simple example but you can move the call to `browserSync.stream()` elsewhere in more complex builds. As long as css files are going in, it should work.

## Using a CSS Preprocessor

We'll use Sass to generate css. Now the project looks like

{% highlight bash %}
.
├── app/
│   ├── css/
│   │   └── app.css  (index.html references this)
│   ├── index.html
│   └── scss/
│       └── app.scss (edit this to change styles)
├── gulpfile.js
└── package.json
{% endhighlight %}

Bringing in a CSS preprocessor requires another small change to the build.

{% highlight javascript %}
var gulp = require('gulp');
var sass = require('gulp-sass');
// create new instance of BrowserSync
var browserSync = require('browser-sync').create();

gulp.task('watch', ['build-styles'], function(gulpCallback) {
  browserSync.init({
    // serve out of app/
    server: './app/',
    // launch default browser as soon as server is up
    open: true
  }, function callback() {
    // (server is now up)

    // watch html and reload browsers when it changes
    gulp.watch('app/index.html', browserSync.reload);

    // when sass files change run specified gulp task
    gulp.watch('app/scss/*', ['build-styles']);

    // notify gulp that this task is done
    gulpCallback();
  });
});

// run sass then stream resulting css to output dir and to BrowserSync
gulp.task('build-styles', function() {
  return gulp.src('app/scss/*')
    .pipe(sass())
    .pipe(gulp.dest('app/css'))
    // if BrowserSync's static server isn't running this stream is a no-op passthrough
    .pipe(browserSync.stream());
});
{% endhighlight %}

You could do something similar for Less and Stylus.

Injecting css like this is nice because it's faster and parts of the UI that depend on JavaScript aren't reset.