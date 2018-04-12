# 002\_scope-methods

## Chapter 2 - Scope Methods

In Chapter 1, we implemented basic dirty-checking, $watch, and $digest. Now, we are going to turn our attention to expanding the methods on the scope. We'll add several ways one can access scopes and evaluate scope to cause dirty checking to be triggered.

### `$eval` - Evaluating Code in the Context of a Scope

In Angular, `$eval` is a method on the scope that allows you to execute a function in the context of that scope. It takes a function as an argument and immediately executes it, giving it the scope itself as an argument.

```javascript
scope.$eval(function(scope){
    //do something with the scope
});
```

The implementation of `$eval` is incredibly simple, so what's the point of it? First of all, using `$eval` makes it very explicit that a piece of code is dealing specifically with the contents of the scope. In addition, it is a building block for `$apply`, which we will see next. Also later on, we will see the most interesting use of `$eval` which is when we can give it an expression and compile it into a function that executes in the context of the scope.

### `$apply` - Integrating External Code with the Digest Cycle

`$apply` takes a function as an argument. It executes that function using `$eval`, and then kickstarts the digest cycle by invoking $digest. It is considered the standard way to integrate external libraries to Angular.

### `$evalAsync` - Deferred Execution and Scope Phases

Often time in JavaScript, we want to defer the execution of a function to some point in the future when the current execution context has finished. The usual way to do this is by calling `setTimeout` with a zero delay parameter. This same pattern applies in Angular as well, though the preferred way to do it is by using the $timeout service which integrates the delayed function to the digest cycle with $apply.

The other way to defer code in Angular is through the $evalAsync function on scopes. It takes a function and schedules it to run later but still during the ongoing digest. This is often preferable to $timeour with zero delay because of the browser event loop. $timeout inherently relinquishes control to the browser and lets it decide when to schedule work. $evalAsync, on the other hand, is gauranteed to execute during the ongoing digest cycle which happens in the same process on the event loop.

Another thing that `$evalAsync` does it to schedule a `$digest` if one isn't already ongoing. This assures you that whenever you call this function that the work you're deferring will be invoked very soon. However, even though `$evalAsync` does schedule a digest, the preferred way to asynchronously execute code within a digest is with `$applAsync` which we will see soon.

It was just stated that evalAsync schedules a digest if one isn't already going. That implies that we need some sort of way to know whether or not a digest is currently running. For this purpose, Angular scopes implement something called a _phase_, which is simply a string attribute on the scope that stores information about what is going on. These phases are going to be `$digest`, `$apply`, and `null`. To help us test asynchronous code, we can use Jasmine's `done` function which can be passed into `beforeEach`, `it`, and `afterEach` as a single argument and hten called when the async work is complete. Here is what our new function looks like:

```javascript
this.$evalAsync = function(fn) {
  if (!this.$$phase && !this.$$asyncQueue.length) {
    setTimeout(function() {
      if (this.$$asyncQueue.length) this.$digest();
    }.bind(this), 0);
  }
  this.$$asyncQueue.push({
    scope: this,
    fn: fn
  });
};
```

Note that we check the length of the current async queue in two places here:

1. Before calling setTimeout because we don't want ot call setTimeout more than we need to. If there's already something in the queue, we already have a timeout set and it will eventually drain the queue
2. Inside the setTimeout function we make sure that the queue is not empty. That is because we don't want ot kick off a digest unnecessarily if we have nothing to do.

If you call $evalAsync when a digest is already running, your function will be evaluated during that digest. If there is no digest running, one is started.

### Coalescing $apply Invocations - $applyAsync

Recall that the difference between $eval and $apply is that $apply executes the provided function in the context of the scope and then kicks off a digest cycle. We just saw $evalAsync, which is designed to defer work from inside of the digest cycle. Now, we have $applyAsync, which is used to integrate asynchronous code that is not aware of the Angular digest cycle. Unlike apply, it does not execute the provided function immediately nor does it launch a digest immediately. Instead, it schedules both of these things to happen after a short period of time.

> The original motivation for adding $applyAsync to the framework was handling HTTP responses: Whenever the $http service receives a response, any response handlers are invoked and a digest is launched. That means a digest is run for each HTTP response. This may cause performance problems with applications that have a lot of HTTP traffic \(as many apps do when they start up\) and/or an expensive digest cycle. The $http service can now be configured to use $applyAsync instead, in which case HTTP responses arriving very close to each other will be coalesced into a single digest. However, $applyAsync is in no way tied to the $http service, and you can use it for anything that might benefit from coalescing digests.

The biggest difference between $evalAsync and $applyAsync is that $evalAsync is guaranteed to call the function during the same digest cycle, while $applyAsync always defers the invocation. Consider the following test case:

```javascript
it('never executes $applyAsynced function in the same cycle', function(done){
  scope.value = 1;
  scope.asyncApplied = false;

  scope.$watch(
    function(scope) { return scope.value; },
    function(newValue, oldValue, scope) {
      scope.$applyAsync(function(scope){
        scope.asyncApplied = true;
      });
    }
  );

  scope.$digest();
  expect(scope.asyncApplied).toBe(false);
  setTimeout(function(){
    expect(scope.asyncApplied).toBe(true);
    done();
  }, 50)
});
```

If this were $evalAsync, the async would have happened before the $digest finished. However, as we just said, with $applyAsync, we want to defer the invocation until after the digest cycle. Then, when it is done, it will kick off another digest cycle since it calls $apply which calls $digest. So how do we prevent multiple $applyAsyncs from calling many digests? We need some way to know if a queue dump has been scheduled or not. To do this, we will use a property called `$$applyAsyncId`. If an ID is set, we will not scheudle an additional $apply knowing that the current $apply will drain the queue.

Another aspect of $applyAsync that we should consider is to not do the digest if one happens to be launched for some other reason before the timeout triggers. In that case, the digest should should the queue and the applyAsync timeout should be cancelled.

### Running Code After a Digest - $$postDigest

Although $applyAsync, which we just learned about, invokes its function after the current $digest cycle has concluded, it is not correct to use it for doing post-digest work, and that is through scheduling a postDigest is an internal Angular construct not meant to be used by developers directly and does not cause a digest to be scheduled. It just performs work after the digest has been completed.

### Watching Several Changes With One Listener: $watchGroup

So far we have been looking at watchers as simple cause-and-effect pairs, but it is not unusual to want to watch several pieces of state and execute some code when any of them change.

To do this, recall that each digest iterates over all of the watchers and executes the listener function when the return value of the watcher has changed from the previous iteration. With that in mind, to craft $watchGroup, we can just unpack an array of watch functions to all invoke the same listener function. Then, instead of passing in the new value and the old value into the listen function, it will pass in all of the values as an array. Here is our first pass at an implementation:

```javascript
this.$watchGroup = function(watcherFnList, listenerFn) {
  var newValues = new Array(watcherFnList.length);
  var oldValues = new Array(watcherFnList.length);
  _.forEach(watcherFnList, function(watcherFn, i){
    this.$watch(watcherFn, function(newValue, oldValue){
      newValues[i] = newValue;
      oldValues[i] = oldValue;
      listenerFn(newValues, oldValues, this);
    });
  }.bind(this));
};
```

This gets us close to what we are trying to do, however, it is calling the listenerFn for every single watcher in the list of them that we passed in. We ideally only want it to run once if there is any change. The way we can implement this is through scheduling an $evalAsync in the watch group listener, but only if one has not been scheduled yet. Here is our new implementation:

```javascript
this.$watchGroup = function(watcherFnList, listenerFn) {
  var self = this;
  var newValues = new Array(watcherFnList.length);
  var oldValues = new Array(watcherFnList.length);
  var changeReactionScheduled = false;
  var firstRun = true;

  function watchGroupListener() {
    if (firstRun) {
      firstRun = false;
      listenerFn(newValues, newValues, self);
    } else {
      listenerFn(newValues, oldValues, self);
    }
    changeReactionScheduled = false;
  }

  _.forEach(watcherFnList, function(watcherFn, i){
    self.$watch(watcherFn, function(newValue, oldValue){
      newValues[i] = newValue;
      oldValues[i] = oldValue;

      if (!changeReactionScheduled) {
        changeReactionScheduled = true;
        self.$evalAsync(watchGroupListener);
      }
    });
  });
};
```

One small thing to note that is on the first run of the watch group, the old values and the new values array will always be the same. Therefore, in order to make the comparison pass, we have to ensure that we are comparing the same arrays.

Another small case to handle is what happens when an empty array of watch functions is passed in. You would think that this is pointless and that the listener never runs, however, what Angular actually does is ensure that the listener gets called exactly one time, which empty values as the arrays.

And now, after all of that, there is one final thing we have to do - and that is be able to deregister a watch group the same way we were able to deregister a single watch before. Since each individual watch returns a function to destroy that watch, we can store up those destroy functions as we register them. Then, the return value of the $watchGroup function returns a function that invokes each individual destroy function.

```javascript
var destroyFunctions = _.map(watcherFnList, function(watcherFn, i){
  return self.$watch(watcherFn, function(newValue, oldValue){
    newValues[i] = newValue;
    oldValues[i] = oldValue;

    if (!changeReactionScheduled) {
      changeReactionScheduled = true;
      self.$evalAsync(watchGroupListener);
    }
  });
});

return function() {
  _.forEach(destroyFunctions, function(fn){
    fn();
  });
};
```

For the special case of an empty array of watch functions, the deregistration function should prevent that listener from ever being called.

## Summary

This chapter was all about fleshing out the functions attached to the scope and how they impact the digest cycle.

* $eval and $apply both execute a function within the context of the scope, however, $apply kicks off a new digest and $eval does not
* $evalAsync and $applyAsync both schedule work to be done at a later point in time, however, $evalAsync guarantees that the work will be done during the same digest cycle while $applyAsync won't run until after it has completed.
* $watchGroup allows you to watch multiple values at once with the same listener function, although it will only be called once per digest cycle for each change in the watch group.

