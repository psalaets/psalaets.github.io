---
title: Speed up your workflow with Quick Searches
layout: post
meta_description: Using browser quick searches to find things faster
twitter_url: https://twitter.com/paulsalaets/status/587925563794415616
---

As a developer I am always looking for ways to get more done in less time.

### Versions used below

{% highlight bash %}
Chrome 41
Firefox 37
{% endhighlight %}

## What is a Quick Search?

Quick Searches are shortcuts in the browser address bar to search any site.

Once they're set up you can type `w dog<enter>` to search Wikipedia for dogs. You can do `Cmd + L` then `w dog<enter>` to stay completely on the keyboard. After a while this becomes a reflex.

## Setting it up in Chrome

1. Go to a site and search for a simple term like dog
2. Select and copy the url of the ensuing results page
3. Right-click the address bar and pick "Edit Search Engines..."
4. In the "Other search engines" section...
5. Enter the site's name in the first field
6. Enter a short but memorable keyword in the second field
7. Paste url from step 2 in the third field
8. Also in the third field, replace dog with %s
9. Click Done

Back in the browser address bar, enter the keyword from step 6 followed by a search term.

Chrome comes with some Quick Searches already set up but with long keywords. They're listed under "Default search settings" in the popup after step 3.

## Setting it up in Firefox

1. Go to a site and search for a simple term like dog
2. Bookmark the ensuing search results page
3. Go to Bookmarks > Show All Bookmarks
4. Click the bookmark you just made (it's probably listed under Unsorted Bookmarks)
5. Edit the Location field, replacing dog with %s
6. Enter something short but memorable in the Keyword field

Back in the browser address bar, enter the keyword from step 6 followed by a search term.

## Some handy searches for web development

### npm

{% highlight bash %}
https://www.npmjs.com/search?q=%s
{% endhighlight %}

### bower

{% highlight bash %}
http://bower.io/search/?q=%s
{% endhighlight %}

### Mozilla developer network

{% highlight bash %}
https://developer.mozilla.org/en-US/search?q=%s
{% endhighlight %}

### stack overflow

{% highlight bash %}
http://stackoverflow.com/search?q=%s
{% endhighlight %}

## Prefilling google searches

Maybe you'd rather use google to search MDN.

{% highlight bash %}
https://www.google.com/search?q=mdn %s
{% endhighlight %}

## Not just for searches

This works anywhere you do a GET request. Move the %s around in the url.