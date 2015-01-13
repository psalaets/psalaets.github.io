---
title: Testing with Promises in Mocha
layout: post
---

As of Mocha [1.18.0](https://github.com/mochajs/mocha/blob/master/HISTORY.md#1180--2014-03-13) you can return a promise from a test and the test will pass if the promise is fulfilled or fail if the promise is rejected.

### Versions used below

{% highlight javascript %}
{
  "mocha": "1.18.0"
}
{% endhighlight %}

## Overview

The basic pattern is

{% highlight javascript %}
it('does something async with promises', function() {
  return somethingAsync();
});
{% endhighlight %}

Things to note here

- Test function *does not* declare a `done` parameter like it would if this used callbacks
- The promise from `somethingAsync` is returned

## Verify a fulfilled promise

You can assert all you want inside the fulfill handler. Errors thrown by the fulfill handler cause the resulting promise to be rejected. That's fine, the test should fail if any of its asserts fail.

{% highlight javascript %}
it('does something async with promises', function() {
  return somethingAsync().then(function fulfilled(result) {
    assert(...)
  });
});
{% endhighlight %}

## Verify a rejected promise

When the code-under-test is expected to return a rejected promise, the promise can't be returned directly to Mocha or the test will fail.

By specifying a reject handler in the `then` call, the error is "caught", the resulting promise is fulfilled and the test passes.

{% highlight javascript %}
it('returns rejected promise on bad input', function() {
  return somethingAsync('bad input').then(function fulfilled(result) {
    throw new Error('Promise was unexpectedly fulfilled. Result: ' + result);
  }, function rejected(error) {
    assert(...)
  });
});
{% endhighlight %}

Asserts are fine in the reject handler because if they fail, an Error is thrown and the resulting promise will be rejected.
