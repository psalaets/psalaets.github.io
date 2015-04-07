---
title: Manually removing a package from Sublime Text 2
layout: post
meta_description: Removing a package from Sublime Text 2 when the command palette is not available.
twitter_url: https://twitter.com/paulsalaets/status/585429203631398912
---

If a package causes Sublime to hang as soon as it starts, you can't remove it with Package Control.

### Versions used below

{% highlight bash %}
Sublime Text 2.0.2 Build 2221
{% endhighlight %}

[Package Control](https://packagecontrol.io/) makes managing Sublime Text packages shockingly easy but it's not available when you can't get to the Sublime command palette. I installed a package that caused a [spinning pinwheel of death](https://en.wikipedia.org/wiki/Spinning_pinwheel) shortly after starting Sublime Text.

After a few failed attempts of racing to the command palette to remove the package, I had to look for another way. The steps below assume you're on OS X but the paths should be similar for other operating systems.

## 1. Close Sublime

This might not be necessary but there's a reason why you're unconscious during major surgery.

## 2. Delete the package's directory

Delete the package's directory in Sublime's packages folder.

{% highlight bash %}
$ rm -rf ~/Library/Application Support/Sublime Text 2/Packages/<package to remove>
{% endhighlight %}

If the package was installed manually, e.g. using `git clone`, you're done. If you installed using Package Control there is one more crucial step.

## 3. Make Package Control forget about the package

Open `~/Library/Application Support/Sublime Text 2/Packages/User/Package Control.sublime-settings` in *some other editor* and remove the package name from the `installed_packages` array.

Without this step Package Control will reinstall the package when you start Sublime. This was really confusing the first few times.

## The offending package

The package causing this trouble was [CursorRuler](https://github.com/icylace/CursorRuler). It claims to be fine in Sublime Text 3 but has [known](https://github.com/icylace/CursorRuler/issues/6) [issues](https://github.com/icylace/CursorRuler/issues/7) in Sublime 2.

Avoid CursorRuler if you use Sublime Text 2.