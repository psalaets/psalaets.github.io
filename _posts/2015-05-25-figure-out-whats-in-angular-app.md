---
title: Figure out what's in an Angular app
layout: post
meta_description: Introducing a tool to see what injectables are in an Angular app and what modules they are from.
twitter_url: https://twitter.com/paulsalaets/status/603207641540599809
---

When an Angular app gets big it can be time consuming to figure out what injectables are available and what modules they are from.

### Versions used below

{% highlight javascript %}
{
  "angular": "^1.0.8",
  "ps-which": "1.2.2",
  "gulp": "3.8.11",
  "gulp-filter": "2.0.2",
  "browserify": "10.2.1"
}
{% endhighlight %}

How could you not know what's in your Angular app? It's *your* app.

Well, maybe you

- wake up one day and realize your app is a monolith
- have inherited an app due to turnover in the development team
- are the new developer on a project

## Introducing ps-which

[ps-which](https://github.com/psalaets/ps-which) is a tool that watches as things are defined in an Angular app. It can tell you which module a directive or service (or value or factory, etc) is from.

It can also list out types by module.

## See what module a type is from

Let's say you have a directive `my-directive` in a template and want to know what module it's from.

Run

{% highlight javascript %}
psWhich.ask('my-directive')
{% endhighlight %}

and it will respond

{% highlight javascript %}
'directive in myModule'
{% endhighlight %}

## List types by module

ps-which can list out everything defined in an Angular app using a markdown format.

{% highlight javascript %}
psWhich.report()
{% endhighlight %}

will print to console

{% highlight text %}
# myModule
## directive
- myDirective
- anotherDirective
## value
- someValue

# myOtherModule
## service
- someService
{% endhighlight %}

### Filtering report output

`psWhich.report()` accepts a `RegExp` to filter what modules are listed.

{% highlight javascript %}
psWhich.report(/^ui\.bootstrap/)
{% endhighlight %}

## Keeping it out of production builds

Since ps-which is a development helper you probably don't want it in your production build.

### gulp when using bower

Filter ps-which out of the js file stream.

{% highlight javascript %}
var filter = require('gulp-filter');

// then later in some task:

var files = <js files included on html page>

return gulp.src(files)
  // "allow everything through except ps-which"
  .pipe(filter(['*', '!ps-which.js']))
  .pipe(concat())
  .pipe(minify())
  .pipe(gulp.dest('...'));
{% endhighlight %}

### browserify

The lazy way is to have browserify [ignore](https://github.com/substack/browserify-handbook#ignoring-and-excluding) ps-which, turning its export into `{}`, essentially a no-op. Since ps-which's export is not useful (ps-which is *always* accessed via global variable), no app code should be using it and ignoring it should be fine.

{% highlight bash %}
$ browserify app.js --ignore ps-which
{% endhighlight %}

A non-lazy way could be using [envify](https://www.npmjs.com/package/envify) with [unreachable-branch-transform](https://www.npmjs.com/package/unreachable-branch-transform) to completely remove `require('ps-which')` from the code before browserify makes the bundle.