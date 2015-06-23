---
title: Setting Angular config with Gulp
layout: post
meta_description: A way to set an Angular app config from a Gulp build
twitter_url: https://twitter.com/paulsalaets/status/612996167475957760
---

Exploring a way to set an Angular app's config from a Gulp build.

### Versions used below

{% highlight javascript %}
{
  "angular": "1.4.0",
  "gulp": "3.9.0",
  "gulp-ng-config": "1.2.1",
  "add-stream": "1.0.0",
  "gulp-concat": "2.4.2",
  "buffer-to-vinyl": "1.0.0"
}
{% endhighlight %}

## What is Angular config?

For the purposes of this post, Angular config is the [constants defined on a module](https://code.angularjs.org/1.4.0/docs/api/ng/type/angular.Module#constant).

{% highlight javascript %}
angular.module('app')
  .constant('name', 'bob')
  .constant('url', 'http://example.org');
{% endhighlight %}

Apps typically use those constants in [config blocks](https://code.angularjs.org/1.4.0/docs/api/ng/type/angular.Module#config) to [turn off debug helpers](/posts/disable-debug-info-to-increase-angular-performance/), set urls, toggle app flags, etc.

{% highlight javascript %}
angular.module('app').config(function(url, someServiceProvider) {
  someServiceProvider.setUrl(url);
});
{% endhighlight %}

## Using gulp-ng-config

Given a json file, [gulp-ng-config](https://www.npmjs.com/package/gulp-ng-config) generates JavaScript that defines constants.

{% highlight javascript %}
var gulpNgConfig = require('gulp-ng-config');

function makeConfig() {
  return gulp.src('./app-config.json')
    .pipe(gulpNgConfig('app.config'));
}
{% endhighlight %}

Assuming `app-config.json` looks like

{% highlight json %}
{
  "name": "bob",
  "url": "http://example.org"
}
{% endhighlight %}

`gulp-ng-config` generates a js file that looks like

{% highlight javascript %}
angular.module('app.config', [])
.constant('name', "bob")
.constant('url', "http://example.org");
{% endhighlight %}

### Appending to app js

Use [add-stream](https://www.npmjs.com/package/add-stream) and [gulp-concat](https://www.npmjs.com/package/gulp-concat) to append the generated config to your main app js.

{% highlight javascript %}
var addStream = require('add-stream');
var concat = require('gulp-concat');

gulp.task('js', function() {
  return gulp.src('./app/js/**/*')
    .pipe(addStream.obj(makeConfig())) // makeConfig is defined a few code blocks up
    .pipe(concat('app.js'))
    .pipe(gulp.dest('...'));
});
{% endhighlight %}

Also be sure your app module depends on the config module.

{% highlight javascript %}
angular.module('app', ['app.config']);
{% endhighlight %}

### Environment specific config

The second parameter to `gulp-ng-config` is an [options object](https://github.com/ajwhite/gulp-ng-config#configuration).

If the `environment` option is set, `gulp-ng-config` looks for a top-level property in the json matching that name and uses its value as the config. This is an easy way to have different settings for development, test and production.

{% highlight javascript %}
function makeConfig() {
  return gulp.src('./app-config.json')
    .pipe(gulpNgConfig('app.config', {
      environment: 'production' // or maybe use process.env.NODE_ENV here
    }));
}
{% endhighlight %}

`app-config.json`

{% highlight json %}
{
  "production": {
    "url": "http://prod.com"
  },
  "development": {
    "url": "http://dev.com"
  }
}
{% endhighlight %}

Result:

{% highlight javascript %}
angular.module('app.config', [])
.constant('url', "http://prod.com");
{% endhighlight %}

### Without a json file on disk

If your config values come from somewhere *other* than a file on disk, you can use [buffer-to-vinyl](https://www.npmjs.com/package/buffer-to-vinyl) to create and stream a vinyl file into `gulp-ng-config`.

{% highlight javascript %}
var b2v = require('buffer-to-vinyl');
var gulpNgConfig = require('gulp-ng-config');

function makeConfig() {
  var json = JSON.stringify({
    // your config here
  });

  return b2v.stream(new Buffer(json), 'config.js')
    .pipe(gulpNgConfig('app.config'));
}
{% endhighlight %}
