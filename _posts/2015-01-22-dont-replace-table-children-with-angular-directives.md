---
title: Don't replace table children with Angular directives
layout: post
---

Browsers react in a bad way when there are unexpected elements, like directives, inside a table.

### Versions used below

{% highlight bash %}
angular 1.3.8
firefox 34
chrome  39
safari  8.0.2
{% endhighlight %}

## Replacing table children with Angular directives

I wrote a `restrict: 'E'`-style directive intending to do something like this

{% highlight html %}
<table>
  <my-header-row></my-header-row>
  <my-row ng-repeat="..."></my-row>
</table>
{% endhighlight %}

## How it broke

My tr-replacing directive ended up above the table.

This is [not an Angular bug](https://github.com/angular/angular.js/issues/1459). It's how browsers handle invalid html. Elements other than [those allowed to be table children](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/table) will be moved out. This seems to be a best-effort attempt by the browser to make the html parseable.

You'll see the same browser behavior with

{% highlight html %}
<table>
  <b>to boldly go</b>
</table>
{% endhighlight %}

## What works

Use `restrict: 'A'` in the directive definition object instead of `restrict: 'E'`.

The original idea becomes

{% highlight html %}
<table>
  <tr my-header-row></tr>
  <tr my-row ng-repeat="..."></tr>
</table>
{% endhighlight %}
