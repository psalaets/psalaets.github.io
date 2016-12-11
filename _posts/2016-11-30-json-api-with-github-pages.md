---
title: Making a JSON API with GitHub Pages
layout: post
meta_description: How to make a static JSON API using GitHub Pages
twitter_url: https://twitter.com/paulsalaets/status/805875228668809217
---

GitHub Pages is an easy, free way to make data available through an API.

## What is GitHub Pages?

Any repo, user and organization on GitHub can have its own static website hosted by GitHub for free.

Many projects use it to host documentation websites but it can host *any* kind of content. Instead of hosting html, you can host JSON files.

### Enable it

1. Go to the homepage of a repo you have admin access to
2. Go to the "Settings" tab
3. Scroll down to the "GitHub Pages" section
4. Set "Source" to "master branch"

Wait a few seconds and then go to `https://<your github username>.github.io/<repo name>`. Any files on the master branch in the repo are now served at that url.

### CORS header

GitHub graciously includes a wildcarded [CORS header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) in the responses.

![screenshot of devtools showing CORS response header](/images/github-pages-cors-header.png)

This is a big win because now any webpage can pull data directly from the API. No backend servers are needed.

## Url and routes

The "base" url is made from your GitHub username (or organization name) and the repo name:

`https://<username or organziation name>.github.io/<repo name>`

Any url path after that is determined by directory structure of the repo.

A repo called "restaurant" with this directory structure

{% highlight bash %}
└── locations
    ├── los-angeles
    │   └── menu.json
    └── new-york
        └── menu.json
{% endhighlight %}

will have these routes

{% highlight bash %}
https://<username>.github.io/restaurant/locations/los-angeles/menu.json
https://<username>.github.io/restaurant/locations/new-york/menu.json
{% endhighlight %}

## What datasets?

This kind of API is best suited for datasets that are read-only and mostly static.

The API is updated by `git push` so highly dynamic data like stock prices would be difficult. Routes that appear to have ids in them are really just directories or files, so all of the data must be known ahead of time.

## Handling 404s

When a non-existent route is accessed, GitHub serves `404.html`, if it exists at the root of the repo. Otherwise it serves a default 404 page. Sending html is not ideal for a JSON API but the response status code is probably the most important thing for clients, and that is set correctly.

I make my `404.html` look like

{% highlight bash %}
404 Not Found

Please see <url to API docs>
{% endhighlight %}

Maybe that will help a confused developer figure out what they're doing wrong.
