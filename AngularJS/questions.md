# Angular Interview-Style Questions

From the author of Build Your Own Angular:

>>> 	AngularJS is not a small framework. It has a large surface area with many new concepts to grasp.
Its codebase is also substantial, with 35K lines of JavaScript in it. While all of those new concepts
and all of those lines of code give you powerful tools to build the apps you need, they also come
with a steep learning curve.
	I hate working with technologies I don’t quite understand. Too often, it leads to code that just happens
to work, not because you truly understand what it does, but because you went through a lot
of trial and error to make it work. Code like that is difficult to change and debug. You can’t reason
your way through problems. You just poke at the code until it all seems to align.
	Frameworks like AngularJS, powerful as they are, are prone to this kind of code. Do you understand
how Angular does dependency injection? Do you know the mechanics of scope inheritance?
What exactly happens during directive transclusion? When you don’t know how these things work,
as I didn’t when I started working with Angular, you just have to go by what the documentation
tells you or what people have said on Stack Overflow. When that isn’t enough, you try different
things until you get the results you need.

## Questions

#### Scopes

 1. What is the purpose of the `$watch` function on the scope object?
 2. Explain the proccesses that are kicked off when calling `$digest` on a scope.
 3. What is the difference between value-based dirty checking and reference-based?
 4. Each scope stores its watchers in an array, however, it possible to remove watchers during the digest cycle. How does Angular ensure that each watcher is executed when one is removed?
 5. What is the difference between $eval and $apply? $evalAsync and $applyAsync?
 6. When would you use $watchGroup instead of creating multiple watchers?
 7. How is the scope hiararchy tied to JavaScript's prototypal inheritance?
 8. When calling digest on a child scope, what watchers should be checked?
 9. Explain how a scope can have a different protypal parent and hierarchical parent
 10. What is an isolated scope?

## Answers

#### Scopes

 1. $watch is the first half of the bread and butter of Angualar scopes with $digest being the second half. $watch takes in a watch function or expression and a listener function. The watch function returns a value to be watched and, if that value has changed since the last cycle, the listener function is executed.
 2. The main work that the $digest function does is recursively iterate over all of the watcher functions until none of the values come back dirty. It is recursive because it has to iterate over all of its child watchers as well. While doing this, if a value is dirty, it executes the listener function for that watcher. In addition, it ensures that everything on the async queue is exeuted before the digest is complete.
 3. Most of the time, Angular performs reference checking during the digestion cycle. This simply checks if a value has been reassigned since the last cycle. Obviously, this does not detect changes in object properties or nested within an array. To check that, we can pass a boolean into the watch function to check by value instead, which compares the new value and the old value at a nested level.
 4. First of all, it should always check that a watcher function still exists before executing it. However, there is still a case in which a watcher could be skipped because the array gets downshifted beyond the current index. To prevent this, Angular always adds new watchers to the front of the array. Then, when iterating over all of the watchers, it iterates from right-to-left. That way, even if a watcher is removed, the array gets shifted to the left and the index will never skip over one.
 5. $eval and $apply both execute a block of code within the context of a scope and can be used to incorporate third party code into your angular application, however, $apply is preferred because it kicks off a digest cycle after it executes. As for $evalAsync and $applyAsync, both schedules some work to be done later, however, $evalAsync is guaranteed to execute within the same digest cycle while $applyAsync schedules work to be done sometime after the current digest cycle.
 6. The difference between $watchGroup and having multiple $watchers is that, during the digest cycle, the listener is only called once when any of the values in the watch grouo changes.
 7. A child scope is created by calling $new on an existing scope. Within this function, a new object is created with the parent scope as the prototype of child. This gives the child access to all of the same functions and properties that exist on the parent. It shadows values such as the watchers array in order to not digest the parents watchers.
 8. When digest is called on a scope, it should iterate over all of its own watchers and those of its children down the hierarchy chain. It should not iterate over any of its parents' watchers.
 9. When calling $new on a scope, it is possible to pass in a parent. This is an interesting feature because the original purpose of the $new function was to be able to create a child on an existing scope. So if the $new function is already adding the new child scope to its list of children, then why would we ever need to pass in a parent? It is possible because it allows you to set up a biforcated parent chain in which one parent scope is used for prototypal inheritance and another is used for hierarchical digestions. The one that is passed in is the hierarchical parent.
 10. An isolated scope is one that exists on the parent's hierarchy chain in terms of digestion but is cut off from the prototype chain and therefore does not have access to the values on the parent scope.

