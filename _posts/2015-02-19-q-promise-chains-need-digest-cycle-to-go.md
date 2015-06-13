---
title: $q promise chains need a digest cycle to go
layout: post
twitter_url: https://twitter.com/paulsalaets/status/568401499707052032
meta_description: $q promise chains will not propagate without a digest cycle
---

In a Jasmine test for an Angular app, I stubbed a method that returns a $q promise. The test kept timing out even though I didn't forget about `done()`.

### Versions used below

{% highlight javascript %}
{
  "angular": "1.3.8",
  "jasmine-core": "2.1.3"
}
{% endhighlight %}

## Digest cycle needed

Just like how a digest cycle is needed for Angular to notice changes to scope properties, a digest is needed for $q promise chain propagation.

When necessary, you can trigger a digest cycle by calling `$rootScope.$apply()`.

## Test with timeout fix

{% highlight javascript linenos %}
describe('beginGame()', function() {
  var gameService, $rootScope;

  beforeEach(inject(function(_gameService_, _$rootScope_, $q) {
    gameService = _gameService_;
    $rootScope = _$rootScope_;

    // Stub method so it just returns a resolved promise.
    // The controller being tested uses gameService.
    spyOn(gameService, 'start').and.callFake(function(game) {
      return $q(function(resolve, reject) {
        resolve(game);
      });
    });
  }));

  it('starts a game with given players', function(done) {
    controller.addPlayers(['jill', 'joe', 'jen']);

    // here's the $q promise chain
    controller.beginGame()
      .then(assertGameStarted)
      .finally(done);

    function assertGameStarted() {
      expect(gameService.start).toHaveBeenCalledWith(jasmine.objectContaining({
        playerNames: ['jill', 'joe', 'jen']
      }));
    }

    // manually trigger digest cycle
    $rootScope.$apply();
  });
});
{% endhighlight %}

Without line 32 the promise chain doesn't propagate, `done()` is never called and the test fails due to timeout.

## Why $q works fine in app code

Digest cycles are triggered for you behind the scenes. Angular built-ins (ngClick, $timeout, etc) call `$rootScope.$apply()` which triggers a digest cycle.