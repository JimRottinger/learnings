# Chapter 3 - Scope Inheritance

So far, we have learned that scope manages state and how watchers and the digest cycle can react to changes in the state. We have also seen a number of difference methods on the scope that can be used to perform work either immediately, sometime within the same digest cycle, or sometime after the digest cycle has concluded. Now, in this chapter, we'll look into the way scopes connect to each other using inheritance. This is the mechanism that allows an Angular scope to access properties on its parents, all of the way up to the root scope. This inheritance is implemented on top of regular JavaScript prototypal object inheritance, and only adds a few additional things on top of it.

To this point, we have only been working with a single scope. A scope created like we have been doing so is called a root scope because it has no parent. In an Angular application, there is exactly one root scope that is available by injecting `$rootScope`.

## Making a Child Scope

Technically, you can make as many root scopes as you want, however, what normally happens is that you create a child scope for an existing scope, or let Angular do it for you. This is done by invoking a function called $new on an existing scope. The relationship between the parent and the child should have the following characteristics:

 - The child should inherit the parent's properties
 - The parent should not have the child's properties
 - The sharing of properties between the parent and child does not care when the properties are defined.
 - You can manipulate a parent's scope from the child's scope since they actually point ot the same value
 - You can watch a parent scope property from a watcher on the child scope

All of these properties should hold true regardless of the depth of the inheritance. This sounds like a lot of stuff, but it's really all things that JavaScript's prototype chain takes care of for us. Here is the implementation of the $new function on the scope:

```js
this.$new = function() {
  var ChildScope = function() { };
  ChildScope.prototype = this;

  return new ChildScope();
};
```

## Attribute Shadowing

Attribute Shadowing is a direct consequence of the prototype chain in JavaScript. Specifically, it is when the parent and child scope both have an attribute by the same name, but they are different values/references. Consider the following test case:

```js
it('shadows a parents property with the same name', function(){
  var parent = new Scope();
  var child = parent.$new();

  parent.name = 'John';
  child.name = 'Jim';

  expect(child.name).toBe('Jim');
  expect(parent.name).toBe('John');
});
```

When we assign an attribute on a child that already exists on a parent, it does not change the parent. However, we now have two difference attributes on the scope chain. This is commonly referred to as shadowing in that the parents's name attribute is shadowed by the child's name attribute.

If this behavior is not desirable, then one way around this is to wrap the attribute in an object. The contents of that object can then be mutated without shadowing out the parent's attribute.

```js
it('should not shadow members of the parent scopes attributes', function (){
  var parent = new Scope();
  var child = parent.$new();

  parent.user = {name: 'John'};
  child.user.name = 'Jim';

  expect(child.user.name).toBe('Jim');
  expect(parent.user.name).toBe('Jim');
});
```

This pattern is sometimes called the dot rule, which refers to the amount of property access dots you have in an expression that makes changes to the scope. If you don't have a dot, then you're doing it wrong.

## Separated Watches

We have already seen that we can watch for a change on the parent scope from the child scope, however, where are those watchers actually stored? Because of the prootype chain, they are actually stored on the parent's $$watchers attribute because that does not exist on the child so it bubbles up into the parent's zone. This has the consequence of all watches being executed in the root scope, regardless of which scope calls $digest. What we really want is for the scope that called $digest to be used as the context of the $digest. Therefore, we need to assign each child its own watchers array. In other words, we need to shadow the $$watchers property on purpose.

## Recursive Digestion

We just saw that $digest should not run watches up the hierarchy and instead, each child should have its own array of watchers. While we don't want watchers running up the chain, we do want them to check for changes down the chain in its children. In order to do this, we need to keep track of the children that each scope has and recursively digest through all of them to check for changes in scope.

Angular's implementation involves of this process involves a linked-list implementation of the scope hierarchy and it iterates over each node recursively until the linked list ends at the lowest-level child. Instead of a $$children array like we are implementing, they have $$nextSibling, $$prevSibling, $$childHead, and $$childTail. This is a performance optimization that we will not be worrying about in our writing. Instead, we will right a helper function to iterate recursively through the $$children array. Functionally, this does the same exact thing.

Our function is going to be called $$everyScope. It is based on JavaScript's array every function which returns true if all values in the array pass some provided conditional test.

```js
this.$$everyScope = function(fn) {
  if (fn(this)) {
    return this.$$children.every(function(child) {
      return child.$$everyScope(fn);
    });
  } else {
    return false;
  }
};
```

Our conditional test is a simple if-else. If our provided function returns true in the context it is executed in, we recursively call it on its children. The function we are going to be providing is a dirty check. If any of the watchers on the children are dirty, we will keep running the digest loop. Here is our new $$digestOnce function.

```js
this.$$digestOnce = function() {
  var dirty;
  var continueLoop = true;
  this.$$everyScope(function(scope) {
    var newValue, oldValue;
    _.forEachRight(scope.$$watchers, function(watcher){
      if (watcher) {
        try {
          newValue = watcher.watcherFn(scope);
          oldValue = watcher.last;
          if (!scope.$$areEqual(newValue, oldValue, watcher.compareByValue)) {
            dirty = true;
            scope.$$lastDirtyWatcher = watcher;
            watcher.last = watcher.compareByValue ? _.cloneDeep(newValue) : newValue;
            watcher.listenerFn(newValue, oldValue, scope);
          } else if (scope.$$lastDirtyWatcher === watcher) {
            continueLoop = false;
            return false; //return false in a lodash loop causes it to break
          }
        } catch (e) {
          console.error(e);
        }
      }
    }.bind(this));
    return continueLoop;
  });

  return dirty;
};
```

There is quite a bit to unpack here. Recall that before our scopes had children, $$digestOnce would iterate through all of its watchers and return true if any of them were dirty which signaled the calling $digest function to iterate again until none of the watchers were dirty. Now, we want that same behavior of $$digestOnce returning true if any of the watchers are dirty, however, now we are checking it recursively. It looks like a lot has changed, however, we are simply calling $$digestOnce on each array of children recursively, each of which will eventually return false when it is no longer dirty. The `every` test ensures that the entire digestOnce will be true if any of the children have a dirty watcher.

## Digesting the Whole Tree from $apply, $evalAsync, and $applyAsync

As we just saw, $digest only works from the current scope down and that is intentional. This is not the case with $apply, which goes directly to the root and digests the entire scope hierarchy. Therefore, it is not enough to call $digest on the same scope that $apply was called on because it may have been called on a child.

To solve this is actually pretty simple. Because the hierarchy chain is based on prototypal inheritance, all we have to do is define a property on the root scope and never shadow it and then all of the children will automatically have access to that property.

```js
this.$root = this;
```

Before, $apply was just a function that ran a function and then called $digest. But now we have just learned an important difference. $digest only runs from the current scope down and apply runs digest from the root scope.

We also learned previously that $evalAsync also kicked off a $digest. So what scope do we run this on? As it happens, it works exactly like $apply does in that it digests from the root scope.


