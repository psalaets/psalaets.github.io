---
title: Pre-caching Angular templates with Gulp
layout: post
---

In production deployments, include directive and route templates as part of your initial js load to reduce http requests made when using your app.

### Versions used below

{% highlight javascript %}
{
  "angular": "1.3.8",
  "add-stream": "1.0.0",
  "gulp": "3.8.10",
  "gulp-angular-templatecache": "1.5.0",
  "gulp-concat": "2.4.2"
}
{% endhighlight %}

## 1. Get gulp-angular-templatecache

I like using [gulp-angular-templatecache](https://www.npmjs.com/package/gulp-angular-templatecache) to generate js code that puts templates into `$templateCache`.

{% highlight bash %}
npm install gulp-angular-templatecache --save-dev
{% endhighlight %}

## 2. Create Angular module to drop templates into

By default `gulp-angular-templatecache` uses the Angular module `templates` to populate `$templateCache` and assumes the `templates` module is already defined.

{% highlight javascript %}
// Include a comment about why this seemingly unused module exists
angular.module('templates', []);
{% endhighlight %}

Don't forget to depend on `templates` in your main app module.

{% highlight javascript %}
angular.module('app', [
  'templates'
]);
{% endhighlight %}

## 3. Use gulp-angular-templatecache in gulpfile.js

Since `gulpfile.js` is just js, I'm wrapping the template pre-caching in a function. The resulting stream will be combined with the app js file in next step.

{% highlight javascript %}
var angularTemplateCache = require('gulp-angular-templatecache');

function prepareTemplates() {
  return gulp.src('app/templates/**/*.html')
    //.pipe(minify and preprocess templates here)
    .pipe(angularTemplateCache());
}
{% endhighlight%}

You can tweak the behavior of `gulp-angular-templatecache`. See its [README](https://github.com/miickel/gulp-angular-templatecache#gulp-angular-templatecache) for more info.

## 4. Append resulting js into your app.js

I use [add-stream](https://www.npmjs.com/package/add-stream) and [gulp-concat](https://www.npmjs.com/package/gulp-concat) to append the js generated in previous step with my app js.

{% highlight javascript %}
var concat = require('gulp-concat');
var addStream = require('add-stream');

gulp.task('build-app.js', function() {
  return gulp.src('app/scripts/**/*.js')
    //.pipe(concat your app js files somehow)

    // append the template js onto app.js
    .pipe(addStream.obj(prepareTemplates()))
    .pipe(concat('app.js'))

    //.pipe(ng annotate, minify, etc)
    .pipe(gulp.dest('build/scripts'));
});
{% endhighlight %}

## 5. Verify

- Look at resulting app.js file and you should see the templates at the bottom.
- Open the Network tab in devtools and click around in your app. You should not see any ajax requests loading templates.
