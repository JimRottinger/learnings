# Chapter 5 - Scope Events

We are at the point now where we have implemented almost everything that Angular scopes do, including the watches and digest cycle. Now, we are going to start talking about events. The event system actually has very little to do with the digest system, so why is it implemented on the scope? As it turns out, the scope is a great place to implement the event system because it gives it automatic access to the scope hierarchy. This opens up natural channels of communication within our application.

## Publish-Subscribe Messaing

The scope event system in Angular is an implementation of the widely used publish-subscribe messaging pattern. When something significant happens, you can publish that information on the scope as an event. Then, other parts of the application can subscribe to that event and respond accordindly. In this situation, the scope acts as a mediator, decoupling publishers from subscribers. Many other frameworks implement pub-sub, including jQuery and Backbone, and with those, all events flow through a single point. Angular is different in that events are baked into the scope, and the events propogate either up or down the scope.

That being said, when you publish an event on a scope, you choose between two propogation modes - up the scope hierarchy or down the scope hierarchy. Going up the scope is called emitting an event and going down is called broadcasting.

## Registering Event Listeners: $on

The first step in setting up a pub-sub interaction is registering a listener. In Angular, we do this by calling $on on a scope object. It is a function that takes in two arguments: the name of the event of interest and the listener function that will get called when that event occurs. Listeners registered through $on will receive both emitted and broadcasted events. With that in mind, the $on function should store the listener somewhere on the scope so that it can find it later when the events are fired. In our case, we will store it on a $$listeners attribute on the scope. The object keys will be the event names and the values will be arrays holding the listener functions. We also have to ensure that the listeners collection is never shared between scopes via prototypal inheritence.

## The basics of $emit and $broadcast

Storing the listeners on the scope was fairly easy. Obviously, the second part of the equations is broadcasting and emitting scopes in the correct direction and getting the listener functions ot fire off. For this, we will implement two functions: $emit and $broadcast. The basic functionality of both funcitons is that when you call them with an event name as an argument, they will call all of the listeners that have been registered for that event name. Here is our basic functionality:

```javascript
this.$emit = function(event) {
  this.$$fireEventOnScope(event)
};

this.$broadcast = function(event) {
  this.$$fireEventOnScope(event)
};

this.$$fireEventOnScope = function(event) {
  var listeners = this.$$listeners[event] || [];
  _.forEach(listeners, function(listener){
    listener();
  });
}
```

## Event Objects

We're currently calling the listeners without any arguments, however, events wouldn't be nearly as useful if they were not able to carry information. Event objects are regular JavaScript objects that carry information and behavior related to the event. We will attach several attributes to the event object, but to begin with, the name of the event with be attached to the `name` attribute.

We also want to be able to attach information that the broadcaster or emitter may have sent along with the event name, like this:

```javascript
aScope.$emit('eventName', 'and', 'additional', 'arguments');
```

$emit and $broadcast will also return the event object that we have built up.

## Deregistering Event Listeners

Our mechanism for deregistering an event listener is the same as deregistering a watch - the $on function will return a function to destroy that listener.

## Emitting Up the Scope Hierarchy

Up until this point, only events emitted and broadcast from the current scope will be received by events on that same scope. But we just talked about wanting to be able to emit/broadcast events up or down the scope hierarchy. This is the part that is different between $emit and $broadcast. $emit goes up the chain and $broadcast go down the chain. So in this part, we are going to be implemented the $emit function.

Recall that we previously located listeners by checking to see if the current scope's listeners object contained a key with the event name. That is no longer enough. Now, we need to be able to check multiple parents event scopes for listeners. This is done by iterating up the parent chain and emitting the event until there are no more parents.

```javascript
this.$emit = function(eventName) {
  var eventObject = {name: eventName};
  var listenerArgs = [eventObject].concat(_.tail(arguments));
  var scope = this;
  do {
    scope.$$fireEventOnScope(eventName, listenerArgs);
    scope = scope.$parent;
  } while (scope);
  return eventObject;
};
```

## Broadcasting Down the Scope Hierarchy

Broadcast is the mirrior image of $emit, excep that it invokes the listener in the children instead of a parent. This gets a little bit more tricky because while a scope can have only one parent, it can have many child, and those children could have many child. We have to ensure that we broadcast to all of them, included isolated childeren.

This might sound familiar. That is because we already implemented a child tree traversal function for everyScope function that it uses.

```javascript
this.$broadcast = function(eventName) {
  var eventObject = {name: eventName};
  var listenerArgs = [eventObject].concat(_.tail(arguments));
  this.$$everyScope(function(scope){
    scope.$$fireEventOnScope(eventName, listenerArgs);
    return true;
  });
  return eventObject;
};
```

Based on this code and the description, it should be obvious that emitting an event is much more performant than broadcasting one.

## Additional Information on the Event Object

At the moment, our event object only contains one attribute - the event name. That can only do so much for us. Now, we are going to bundle some more information in it, especially now that we are broadcasting and emitting events up and down the scope hierarchy. The first thing we can add is a link to the target scope \(the scope that the event came from\) and the current scope \(the scope of the listener function we are executing in\). We will call these `currentScope` and `targetScope`.

## Stopping Event Propogation and Preventing Default Behavior

Much like the way we often want to stop the propagation on javascript DOM events, it would be nice if we had a way to stop the propogation of scope events in Angular. Broadcasted events cannot be stopped, however, we want to have a `stopPropagation` function for emitted events to prevent them from bubbling up if its already been handled. We want to ensure that even if an event is stopped, it still gets handled by all listeners on the current scope.

Another DOM behavior that we want to mimic is prevent the default behavior on events such as pereventing links from changing the url. It does this through `preventDefault`. But what default behavior do we want to prevent on scope events? Browsers have default behaviors, our scope does not. As it turns out, our preventDefault function will not affect the scope event system behavior. It simply meant to be a carrier of boolean information to be used later on. An example of when it can be used is by a custom directive to determine if it should trigger some default browser behavior.

## Summary

We have now implemented the Angular scope system in full. In this chapter specifically, we covered:

* How Angular implements its scope system using pub/sub
* How it is tied to the scope hierarchy
* The difference between broadcasting an event and emitting one
* What information is contained within the event object
* How the scope attributes are modelled after the DOM event model
* When and how the scope events can be stopped.

In a vacuum, what we have created thus far is an object creation and management system. Remember that the scope started off as a plain-old JavaScript object that we added a lot of functionality to. This functionality includes registering watchers to detect changes in the object properties, running a digest to see if anything has changed, executing the listener functions, and an event system to broadcast or emit an event. We also implemented ways to run functions in the context of a scope, including eval, apply, evalAsync, and applyAsync.

Even though scope is a core part of how Angular works, we are still a far cry from having a full framework. The biggest point of JavaScript frameworks is to keep the UI in the same state as your application state. Where we are really going to start benefitting from the scope system we have created is when we implement it on top of the DOM

