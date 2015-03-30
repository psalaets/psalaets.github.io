---
title: Injecting Angular services into your own code
layout: post
meta_description: How to get Angular dependencies injected into arbitrary functions
twitter_url: https://twitter.com/paulsalaets/status/582544936366682112
---

Define and run an arbitrary function that can have Angular services injected into it.

### Versions used below

{% highlight javascript %}
{
  "angular": "1.3.14",
  "ng-annotate": "0.15.4"
}
{% endhighlight %}

## Why not just use closures?

The easy solution is using a closure to access dependencies instead of injecting them as parameters.

{% highlight javascript %}
app.factory('myService', function(service1) {

  function func() {
    // service1 is available here
  }
});
{% endhighlight %}

In my case, the function was created in a module config block and passed to a provider to be invoked later. You can only inject providers and Angular constants into config blocks so the closure trick won't work.

{% highlight javascript %}
// DOES NOT WORK
app.config(function(service1) { // services don't exist during config phase so can't inject here

  function func() {

  }
});
{% endhighlight %}

The above results in error:

{% highlight bash %}
Error: [$injector:modulerr] Failed to instantiate module app due to:
[$injector:unpr] Unknown provider: service1
{% endhighlight %}

## Using $injector

Angular makes this pretty easy. Use [$injector](https://code.angularjs.org/1.3.14/docs/api/auto/service/$injector) to invoke the function.

{% highlight javascript %}
$injector.invoke(func);
{% endhighlight %}

`$injector.invoke` invokes `func` with all its dependencies as parameters and returns `func`'s return value.

`func` must follow one of the three Angular "injectable function" patterns:

### 1. inferred

Note: this style won't work if the code is minified

{% highlight javascript %}
function func(dep1, dep2) {

}
{% endhighlight %}

### 2. inline

{% highlight javascript %}
['dep1', 'dep2', function func(dep1, dep2) {

}]
{% endhighlight %}

### 3. annotated

{% highlight javascript %}
function func(dep1, dep2) {

}

func.$inject = ['dep1', 'dep2']
{% endhighlight %}

## ng-annotate compatibility

This injection can happen anywhere. It's not in the Angular API like all other injections we're used to. Will [ng-annotate](https://github.com/olov/ng-annotate) still see it?

The answer is... maybe.

There is only so much ng-annotate can detect through static code analysis. It has options for [explicitly marking code to be annotated](https://github.com/olov/ng-annotate#explicit-annotations-with-nginject) but at that point you might as well just annotate the code yourself.