---
title: $q promise chains need a digest cycle to go
layout: post
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
  var gameService, persistence, $rootScope;

  // This uses Angular's underscore-wrap naming convention to avoid collisions
  // between the variables above and function argument names below
  // https://docs.angularjs.org/api/ngMock/function/angular.mock.inject
  beforeEach(inject(function(_gameService_, _persistence_, $q, _$rootScope_) {
    gameService = _gameService_;
    persistence = _persistence_;
    $rootScope = _$rootScope_;

    // This uses jasmine's built-in spy/mock/stub functionality
    // http://jasmine.github.io/2.1/introduction.html#section-Spies
    spyOn(gameService, 'start');

    // Stub method so it doesn't save anything, just return a resolved promise.
    // The controller being tested uses the persistence service.
    spyOn(persistence, 'savePlayers').and.callFake(function(players) {
      return $q(function(resolve, reject) {
        resolve(players);
      });
    });
  }));

  it('starts a new game with given players', function(done) {
    controller.addPlayer('jill');
    controller.addPlayer('joe');
    controller.addPlayer('jen');

    // here's the $q promise chain
    controller.beginGame()
      .then(assertGameStarted)
      .finally(done);

    function assertGameStarted() {
      expect(gameService.start).toHaveBeenCalledWith(jasmine.objectContaining({
        playerNames: ['jill', 'joe', 'jen']
      }));
    }

    $rootScope.$apply(); // manually trigger digest cycle
  });
});
{% endhighlight %}

Without line 41 the promise chain doesn't propagate, `done()` is never called and the test fails due to timeout.

## Why $q works fine in app code

Digest cycles are triggered for you behind the scenes. Angular built-ins (ngClick, $timeout, etc) call `$rootScope.$apply()` which triggers a digest cycle.