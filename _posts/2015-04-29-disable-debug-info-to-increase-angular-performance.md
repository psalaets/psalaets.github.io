---
title: Disable debug info to increase Angular performance
layout: post
meta_description: Disable Angular debug info for a performance boost
twitter_url: https://twitter.com/paulsalaets/status/593371511525871616
---

Angular adds metadata to the DOM to aid development tools. Turning this off in production can give a performance boost.

### Versions used below

{% highlight javascript %}
{
  "angular": "1.3.14"
}
{% endhighlight %}

## What debug info?

Angular adds info to elements related to binding and scopes. If you've ever been poking around in devtools and noticed some elements have extra classes like

{% highlight html %}
<div class="ng-binding">
  ...
</div>
<div class="ng-scope">
  ...
</div>
{% endhighlight %}

that's part of it.

The debug info is added for tools like [Protractor](http://angular.github.io/protractor) and [Batarang](https://chrome.google.com/webstore/detail/angularjs-batarang/ighdmehidhipcmcojjgiloacoafjmpfk). You probably won't be running them in production so it's safe to disable it.

## Disabling debug info

{% highlight js %}
angular.module('app').config(['$compileProvider', function($compileProvider) {
  $compileProvider.debugInfoEnabled(false);
}]);
{% endhighlight %}

There's a [way to get it back](https://code.angularjs.org/1.3.14/docs/api/ng/function/angular.reloadWithDebugInfo) in a live app.

## What does this save?

Angular will hit the DOM less.

Four lines starting [here](https://github.com/angular/angular.js/blob/v1.3.14/src/ng/compile.js#L1145) become no-ops with debug info disabled. Depending on the directive, some or all of those run roughly once per directive *usage*.