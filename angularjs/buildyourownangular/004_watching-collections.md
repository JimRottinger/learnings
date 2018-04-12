# Chapter 4 - Watching Collections

In Chapter 1 we implemented two different strategies for identifying changes in watches: By reference and by value. We saw that the way you choose between the two is by passing a boolean flag to the $watch function. In this chapter, we will be implementing a third and final strategy for identifying a change - $watchCollection. We will use this when we need to know if something in an array or object has changed, including additions, deletions, and reorderings.

Of course, this is already possible with a watch expression when comparing by value, however, this is essentially an optimized version of that because it only cares about a single level of nesting. It does not need to watch the entire object graph. From the author:

> This chapter is all about $watchCollection. While conceptually simple, the function packs a punch. By knowing how it works you’ll be able to use it to full effect. Implementing $watchCollection also serves as a case study for writing watches that specialize in certain kinds of data structures, which is something you may need to do when building Angular applications.

The function signature of $watchCollection is going to be nearly identical to that of the $watch function. Within, it will actually delegate to the $watch function, but we will supply our own, locally created versions of the watch and listener function.

```javascript
this.$watchCollection = function(watchFn, listenFn) {
  var internalWatchFn = function(scope) {
  };

  var internalListenerFn = function() {
  };

  return this.$watch(internalWatchFn, internalListenerFn);
};
```

Even though $watchCollection is designed for watching arays and objects, it should also work for watching primitives just like $watch does.

## Disecting the $watchCollection function

I am not going to spend a lot of time taking too many notes on this section because it isn't super difficult to understand conceptually. Here is our final function:

```javascript
this.$watchCollection = function(watchFn, listenerFn) {
  var newValue, oldValue, oldLength, veryOldValue;
  var trackVeryOldValue = (listenerFn.length > 1);
  var changeCount = 0;
  var firstRun = true;

  var internalWatchFn = function(scope) {
    var newLength;

    newValue = watchFn(scope);
    if (_.isObject(newValue)) {
      if (isArrayLike(newValue)) {
        if (!_.isArray(oldValue)) {
          changeCount++;
          oldValue = [];
        }
        if (newValue.length !== oldValue.length) {
          changeCount++;
          oldValue.length = newValue.length;
        }
        _.forEach(newValue, function(newItem, i) {
          var bothNaN = _.isNaN(newItem) && _.isNaN(oldValue[i]);
          if (!bothNaN && newItem !== oldValue[i]) {
            changeCount++;
            oldValue[i] = newItem;
          }
        });
      } else {
        if (!_.isObject(oldValue) || isArrayLike(oldValue)) {
          changeCount++;
          oldValue = {};
          oldLength = 0;
        }
        newLength = 0;
        _.forOwn(newValue, function(newVal, key) {
          newLength++;
          if (oldValue.hasOwnProperty(key)) {
            var bothNaN = _.isNaN(newVal) && _.isNaN(oldValue[key]);
            if (!bothNaN && oldValue[key] !== newVal) {
              changeCount++;
              oldValue[key] = newVal;
            }
          } else {
            changeCount++;
            oldLength++;
            oldValue[key] = newVal;
          }
        });
        if (oldLength > newLength) {
          changeCount++;
          _.forOwn(oldValue, function(oldVal, key) {
            if (!newValue.hasOwnProperty(key)) {
              oldLength--;
              delete oldValue[key];
            }
          });
        }
      }
    } else {
      if (!this.$$areEqual(newValue, oldValue, false)) {
        changeCount++;
      }
      oldValue = newValue;
    }

    return changeCount;
  }.bind(this);

  var internalListenerFn = function() {
    if (firstRun) {
      listenerFn(newValue, newValue, this);
      firstRun = false;
    } else {
      listenerFn(newValue, veryOldValue, this);
    }

    if (trackVeryOldValue) {
      veryOldValue = _.clone(newValue);
    }
  }.bind(this);

  return this.$watch(internalWatchFn, internalListenerFn);
};
```

Looks like a huge function, but it is really just setting up a $watch expression. Remember that a watch function should return a value to be watched. It doesn't particlarly matter what it is returned, so long as it is different than the previous time it was executed, then the listener function will run. In this case, we keep a simple counter that we increment every time a change is detected.

There are three different cases for detecting a change - the array case, the object case, and the primitive case. For the array, it compares the size and each individual value in order. Objects also check the length, but they doesn't care about order. Therefore, it compares each property value to the same value at the past iteration, and checks to see if that property was there before. The primitive case simply checks the equality of the new value and the old value. If any of the these detect a change, the listener function will run. The listener function we have created is exactly like the one we created for watchGroup. If it is the first run, it will use the same parameter for the newValue and oldValue.

