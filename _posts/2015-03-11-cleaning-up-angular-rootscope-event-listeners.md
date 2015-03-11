---
title: Cleaning up Angular $rootScope event listeners
layout: post
meta_description: How to clean up rootScope event listeners and what happens when you forget
twitter_url: https://twitter.com/paulsalaets/status/575668042119667712
---

When I started using ngIf I noticed some `$rootScope` event listeners firing more than once per event.

### Versions used below

{% highlight javascript %}
{
  "angular": "1.3.14"
}
{% endhighlight %}

## Lifespan of $rootScope vs other scopes

`$rootScope` lives as long as your app, which is usually the duration of the page. Other scopes are not guaranteed to last that long, like when their directive is nested in ngIf or ngRepeat.

## Forgetting to clean up the listener

When a directive like this

{% highlight js %}
angular.module('app').directive('myDirective', myDirective);

function myDirective() {
  return {
    restrict: 'E',
    scope: {
      foo: '='
    },
    controller: function($rootScope) {
      // problem: registered a listener but never cleans it up
      $rootScope.$on('someEvent', function(event) {

      });
    }
  };
}
{% endhighlight %}

is used like this

{% highlight html %}
<div ng-if="flag">
  <my-directive></my-directive>
</div>
{% endhighlight %}

an event listener will be left behind every time `flag` changes from `true` to `false`. A symptom of this is an event being handled more than once. The extra listeners are sometimes called "zombie" listeners.

## Cleaning up the listener

`$rootScope.$on()` returns a function that will deregister the listener. That function should be invoked when the directive's scope is destroyed.

With this in mind, the directive changes to

{% highlight js %}
angular.module('app').directive('myDirective', myDirective);

function myDirective() {
  return {
    restrict: 'E',
    scope: {
      foo: '='
    },
    controller: function($scope, $rootScope) {
        // add listener and hold on to deregister function
        var deregister = $rootScope.$on('someEvent', function(event) {

        });

        // clean up listener when directive's scope is destroyed
        $scope.$on('$destroy', deregister);
    }
  };
}
{% endhighlight %}