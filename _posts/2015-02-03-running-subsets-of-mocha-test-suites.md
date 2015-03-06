---
title: Running subsets of Mocha test suites
layout: post
twitter_url: https://twitter.com/paulsalaets/status/562606862395523072
meta_description: Sometimes you do not want to run your entire Mocha test suite. Here are five ways to do it.
---

Sometimes you don't want to run your entire test suite. Here are five ways to do it.

### Versions used below

{% highlight javascript %}
{
  "mocha": "2.1.0"
}
{% endhighlight %}

## 1. File pattern

This is an easy but coarse-grained way to control what gets run. It may be all you need depending on how your tests are divided up.

You can run specific test files

{% highlight bash %}
$ mocha test/login-required.js
{% endhighlight %}

When not given a pattern, Mocha runs `./test/*.js`.

## 2. grep

Passing `--grep <pattern>` on the command line will run tests and suites with names matching the pattern.

{% highlight bash %}
$ mocha --grep auth
{% endhighlight %}

## 3. grep with invert

Adding `--invert` to a command line with `--grep <pattern>` runs tests and suites with names that *do not* match the pattern.

{% highlight bash %}
$ mocha --grep auth --invert
{% endhighlight %}

## 4. only

Chain an `it` with `.only()`. Mocha will only run the test with the `.only()`.

{% highlight javascript %}
describe("using only", function() {
  it.only("this is the only test to be run", function() {

  });

  it("this is not run", function() {

  });
});
{% endhighlight %}

### describe blocks

You can also chain `.only()` on a `describe` block to run that block exclusively.

### Gotcha

`.only()` is implemented using the grep option. Similar naming could cause extra tests to run ([issue 1481](https://github.com/mochajs/mocha/issues/1481)).

## 5. skip

This is like the reverse of `.only()`. Mocha will run all tests *except* the ones with `.skip()`.

{% highlight javascript %}
describe("using skip", function() {
  it.skip("this is not run", function() {

  });

  it("this runs", function() {

  });

  it("this runs too", function() {

  });
});
{% endhighlight %}

### describe blocks

You can also chain `.skip()` on `describe` blocks to ignore the block.

## 6. tagging (unreleased as of Mocha 2.1.0)

A new [tagging api](https://github.com/mochajs/mocha/pull/1445) could be coming to Mocha. This seems like it'd trump all other methods of test filtering.
