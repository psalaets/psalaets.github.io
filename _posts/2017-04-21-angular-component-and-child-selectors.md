---
title: Using Angular components without breaking child selectors
layout: post
meta_description: Angular components without breaking child selectors
---

Angular components leave custom elements in the dom and that breaks styles which rely on child selectors. Here's a workaround.

### Versions used below

{% highlight javascript %}
{
  "@angular/core": "4.0.3"
}
{% endhighlight %}

## Scenario

You've got some Angular components that look like

{% highlight javascript %}
/* parent */
import { Component } from '@angular/core';

@Component({
  selector: 'my-list',
  template: `
    <ul>
      <my-list-item></my-list-item>
    </ul>
  `
})
export class ListComponent {}
{% endhighlight %}

and

{% highlight javascript %}
/* child */
import { Component } from '@angular/core';

@Component({
  selector: 'my-list-item',
  template: `
    <li>
      <span>1</span>
    </li>
  `
})
export class ListItemComponent {}
{% endhighlight %}

Since Angular components leave their custom elements in the dom, you get this dom nesting

{% highlight html %}
<my-list>
  <ul>
    <my-list-item>
      <li>
        <span>1</span>
      </li>
    </my-list-item>
  </ul>
</my-list>
{% endhighlight %}

## Problem

The nesting seen above breaks styles that use [child selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/Child_selectors):

{% highlight css %}
/* selector no longer matches */
ul > li {

}
{% endhighlight %}

## A fix

Change the child's selector to something else, like an [attribute selector](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors).

{% highlight javascript %}
@Component({
  // changed to attribute selector
  selector: '[my-list-item]',

  // or even
  // selector: 'li[my-list-item]',

  // outer li is removed because it'll be in parent template now
  template: `
    <span>1</span>
  `
})
{% endhighlight %}

Parent template changes to

{% highlight html %}
<ul>
  <li my-list-item></li>
</ul>
{% endhighlight %}

the dom becomes

{% highlight html %}
<my-list>
  <ul>
    <li my-list-item="">
      <span>1</span>
    </li>
  </ul>
</my-list>
{% endhighlight %}

and now the child selector matches. Sweet.

## What about binding things on child's root element?

We lost the ability to bind things on the `<li>` because that moved out of the child template.

`@HostBinding()` and `@HostListener()` can let us achieve that.

{% highlight javascript %}
import { Component, HostBinding, HostListener } from '@angular/core';

@Component({
  selector: '[my-list-item]',
  template: `
    <span>1</span>
  `,
})
export class ListItemComponent {
  // Equivalent to <li [class.active]="hasActiveClass">
  @HostBinding('class.active') hasActiveClass: boolean = false;

  // Equivalent to <li (click)="handleClick()">
  @HostListener('click') handleClick() {
    this.hasActiveClass = !this.hasActiveClass;
  }
}
{% endhighlight %}

Well, that wasn't ideal but it worked.
