---
title: localForage migrations in an Angular app
layout: post
meta_description: A module to migrate localForage data in an Angular app
twitter_url: https://twitter.com/paulsalaets/status/577832690940030977
---

My Angular app's offline data schema needed to change and I had to preserve existing data.

### Versions used below

{% highlight javascript %}
{
  "angular": "1.3.14",
  "localforage": "1.2.2",
  "angular-localforage": "1.2.2"
}
{% endhighlight %}

## What is data migration?

When your data schema changes, existing data needs to be converted to the new format. By "schema" I just mean whatever structure your data is in.

For example, in version 2 of your app a friends array was added to the person object. Any people created *before* version 2 need to be given a friends array.

## Basic idea

Put pending migrations behind a promise and chain all data access off of that promise. The promise acts as a gatekeeper ensuring no data access occurs until the migrations are finished.

{% highlight javascript %}
function migrate() {
  var migrationPromise = <data migrations here>
  return migrationPromise;
}

function loadData() {
  // run data access only after data migrations are done
  return migrate().then(function() {
    return $localForage.getItem('data');
  });
}
{% endhighlight %}

## Migrations service

The logic to create `migrationPromise` tends to be a little messy so it moves to its own service.

{% highlight javascript %}
angular.module('app').factory('migrations', migrations);

function migrations($localForage) {
  var migrationPromise;

  return {
    migrate: function() {
      // if promise is not set yet, migration needs to be run
      if (!migrationPromise) {
        migrationPromise = $localForage.getItem('people').then(function(people) {
          // add friends array to each existing person
          people.forEach(function(person) {
            person.friends = [];
          });
          return $localForage.setItem('people', people);
        })
      }

      return migrationPromise;
    }
  };
}
{% endhighlight %}

## Multiple migrations

With migrations in subsequent versions of an app, a user's data could be anywhere along the migration timeline. Depending on what the migrations do, you might not be able to run all migrations every time the app is loaded.

To know how far a user's data has been migrated, give migrations increasing id numbers and keep track of the highest migration id that has been run.

## Angular provider recipe

The only interesting parts are the actual migrations. The framework around migration execution doesn't change.

We could use the [Angular provider recipe](https://code.angularjs.org/1.3.14/docs/guide/providers#provider-recipe) to package up migration execution code. Each migration would be added to the provider in a [module config block](https://code.angularjs.org/1.3.14/docs/api/ng/type/angular.Module#config).

## angular-localforage-migrations

All of the ideas above have been put into a module called [angular-localforage-migrations](https://github.com/psalaets/angular-localforage-migrations) which is available through npm and bower.