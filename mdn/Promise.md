# Promise

The `Promise` object represents the eventual completion (or failure) of an asynchronous operation and its resulting value.

To learn about the way promises work and how you can use them, we advise you to read Using promises first.

## Description

A `Promise` is a proxy for a value not necessarily known when the promise is created. It allows you to associated handlers
with an asynchronous action's eventual success value or failure reason.
This lets asynchronous methods return value like synchronous methods: instead of immediately returning the final value,
the asynchronous method returns a promise to supply the value at some point in the future.

A `Promise` is in one of these states:

- *pendint*: initial state, neither fulfilled nor rejected.
- *fulfilled*: meaning that the operation was completed successfully.
- *rejected*: meaning that the operation failed.

The *eventual state* of a pending promise can either be *fulfilled* with a value or *rejected* with a reason (error).
When either of these options occur, the associated handlers queued up by a promise's `then` method are called.
If the promise has already been fulfilled or rejected when a corresponding handler is attached,
the handler will be called, so there is no race condition between an asynchronous operation completing and its handlers
being attached.

A promise is said to be settled if it is either fulfilled or rejected, but not pending.

You will also hear the term resolved used with promises - this mean that the promises is settled or "locked-in" to match
the eventual state of another promise, and further resolving or rejecting it has no effect.
The State and fates document from the original Promise proposal contains more details about promise terminology.
Colloquially, "resolved" promises are often equivalent to "fulfilled" promises, but as illustrated in "State and fates",
resolved promises can be pending or rejected as well. For example:

```js
new Promise((resolveOuter) => {
  resolveOuter(
    new Promise((resolveInner) => {
      setTimeout(resolveInner, 1000);
    }),
  );
});
```

The promise is already resolved at the time when it's created (because the `resolveOuter` is called synchronously),
but it is resolved with another promise, and therefore won't be fulfilled until 1 second later,
when the inner promise fulfills.
In practice, the "resolution" is often done behind the scenes and not observable,
and only its fulfillment or rejection are.

Note: Several other languages have mechanisms for lazy evaluation and deferring a computation,
which they also call "promises", e.g., Scheme.
Promises in JavaScript represent processes that are already happening, which can be chained with callback functions.
If you are looking to lazily evaluate an expression, consider using a function with no arguments e.g., `f = () => expression`
to create the lazily-evaluated expression, and `f()` to evaluate the expression immediately.

`Promise` itself has no first-class protocol for cancellation,
but you may be able to directly cancel the underlying asynchronous operation, typically using `AbortController`.

### Chained Promises

The promises methods `then()`, `catch()`, and `finally()` are used to associate further action
with a promise that become settled.
The `then()` method takes up to two arguments; the first argument is a callback function for the fulfilled case of the
promise, and the second arguement is a callback function for the rejected case.
The `catch()` and `finally()` methods call `then()` internally and make error handling less verbose.
For example, a `catch()` is really just a `then()` without passing the fulfillment handler.
As these methods return promises, they can be chained. For example:

```js
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("foo");
  }, 300);
});

myPromise
  .then(handleFulfilledA, handleRejectedA)
  .then(handleFulfilledB, handleRejectedB)
  .then(handleFulfilledC, handleRejectedC);
```

We will use the following terminology: initial promise is the promise on which `then` called;
new promises is the promise returned by `then`.
The two callbacks passed to `then` are called fulfillment handler and rejection handler, respectively.

The settled state of the initial promise determines which handler to execute.

- If the initial promise is fulfilled, the fulfillment handler is called with the fulfillment value.
- If the initial promise is rejected, the rejection handler is called with the rejection reason.

The completion of the handler determines the settled state of the new promise.

- If the handler returns a thenable value, the new promise settles in the same state as the returned value.
- If the handler returns a non-thenable value, the new promise is fulfilled with the returned value.
- If the handler thwos an error, the new promises is rejected with the thrown error.
- If the initial promise has no corresponding handler attached,
the new promise will settle to the same state as the initial promise - that is, without a rejection handler,
a rejected promise stays rejected with the same reason.

For example, in the code above, if `myPromise` rejects, `handleRejectedA` will be called, and if `handleRejectedA`
completes normally (without throwing or returning a rejected promise),
the promise returned by the first `then` will be fulfilled instead of staying rejected.
Therefore, if an error must be handled immediately, but we want to maintain the error state down the chain,
we must throw an error of some type in the rejection handler.
On the other hand, in the absence of an immediate need, we can leave out error handling until the final `catch()` handler.

```js
myPromise
  .then(handleFulfilledA)
  .then(handlefulfilledB)
  .then(handleFulfilledC)
  .catch(handleRejectedAny);
```

Using arrow functions for the callback functions, implementation of the promise chain might look something like this:

```js
myPromise
  .then((value) => `${value} and bar`)
  .then((value) => `${value} and bar again`)
  .then((value) => `${value} and again`)
  .then((value) => `${value} and again`)
  .then((value) => {
    console.log(value);
  })
  .catch((err) => {
    console.error(err);
  });
```

Note: For faster execution, all synchronous actions should preferably be done within one handler,
otherwise it would take several ticks to execute all handlers in sequence.

JavaScript maintains a job queue.
Each time, JavaScript picks a job from the queue and execution it to completion.
The jobs are defined by the executor of the `Promise()` constructor,
the handler passed to `then`, or any platform API that returns a promise.
The promise in a chain represent the dependency relationship between these jobs.
When a promise settles, the respective handlers associated with it are added to the back of the job queue.

A promise can participate in more than one chain.
For the following code, the fulfillment of `promiseA` will cause both `handleFulfilled1` and `handleFulfilled2`
to be added to the job queue.
Because `handleFulfilled1` is registered first, it will be invoked first.

```js
const promiseA = new Promise(myExecutorFunc);
const promiseB = promiseA.then(handleFulfilled1, handleRejected1);
const promiseC = promiseA.then(handleFulfilled2, handleRejected2);
```

An action can be assigned to an already settled promise. In this case, the action is added
immediately to the back of the job queue and will be performed when all existing jobs are
completed. Therefore, an action for an already "settled" promise will occur only after the
current synchronous code completes and at least one loop-tick has passed. This
guarantees that promise actions are synchronous.

```js
const promiseA = new Promise((resolve, reject) => {
  resolve(777);
});
// At the point, "promiseA" is already settled.
promiseA.then((val) => console.log("asynchronous logging has val:", val));
console.log("immediate logging");

// produces output in this order:
// immediate logging
// asynchronous logging has val: 777
```

Thenables

The JavaScript ecosystem had made multiple Promise implementations long before it
became part of the language. Despite being represented differently internally, at the
minimum, all Promise-like objects implement the Thenable interface. A thenable
implements the `.then()` method, which is called with two callbacks: one for when the
promise is fullfilled, one for when it's rejected. Promises are thenables as well.

to interoperate with the existing Promise implementations, the language allows using
thenables in place of promises. For example, `Promise.resolve` will not only resolve
promises, but also trace thenables.

```js
const thenable = {
  then(onFulfilled, onRejected) {
    onFulfilled({
      // The thenable is fulfilled with another thenable
      then(onFulfilled, onRejected) {
        onFulfilled(42);
      },
    });
  },
};

Promise.resolve(thenable); // A promise fulfilled with 42
```

### Promise concurrency

The `Promise` class offers four static methods to facilitate async task concurrency:

`Promise.all()`

Fulfills when all of the promises fulfill; rejects when any of the promises rejects.

`Promise.allSettled()`

Fulfills when all promises settle.

`Promise.any()`

Fulfills when any of the promises fulfills; rejects when all of the pormises reject

`Promise.race()`

Settles when any of the promises settles. In other words, fulfills when any of the 
promises fulfills; rejects when any of the promises rejects.

All the methods take an iterable of promises (thenables, to be exact) and return a new
promise. They all support subclassing, which means they can be called on subclasses of
`Promise`, and the result will be a promise of the subclass type. To do so, the subclass's
constructor must implement the same signature as the `Promise()` constructor - accepting
a single `executor` function that can be called with the `resolve` and `reject` callbacks as
parameters. The subclass must also have a `resolve` static method taht can be called like
`Promise.resolve()` to resolve values to promises.

Note that JavaScript is single-threaded by nature, so at a given instant, only one task will
be executing, although control can shift between different promises, making execution of 
the promises appear concurrent. Parallel execution in JavaScript can only be achieved
through worker threads.

## Constructor

`Promise()`

Creates a new `Promise` object. The constructor is primarily used to wrap functions that
do not already support promises.

## Static properties

`Promise[Symbole.species]`

Returns the constructor used to construct return values from promises methods.

## Static methods

`Promise.all()`

Takes an iterable of promises as input and returns a single `Promise`. This returned
promise fulfills when all the input's promises fulfill (including when an empty iterable
is passed), with an array of the fulfillment values. It rejects when any of the input's
promises reject, with this first rejection reason.

`Promise.allSettled()`

Takes an iterable of promises as input and returns a single `Promise`. This returned
promise fulfills when all of the input's promises settle (including when an empty iterable
is passed), when an array of objects that describe the outcome of each promise.

`Promise.any()`

Takes an iterable of promises as input and returns a single `Promise`. This returned
promise fulfills when any of the input's promise fulfill, with first fulfillment value. It
rejects when all of the input's promises reject (including when an empty iterable is 
passed), with an `AggregateError` containing an array of rejection reasons.

`Promise.race()`

Takes an iterable of promises as input and returns a single `Promise`. This returned
promise settles with the eventual state of the first promise that settles.

`Promise.reject()`

Returns a new `Promise` object that is rejected with the given reason.

`Promise.resolve()`

Returns a `Promise` object that is resolved with the given value. If the value is a thenable
(i.e., has a `then` method), the returned promise will "follow" that thenable, adopting its
eventual state; otherwise, the returned promise will be fulfilled with the value.

`Promise.try()`

Takes a callback of any kind (returns or throws, synchronously or asynchronously) and
wraps its result in a `Promise`.

`Promise.withResolvers()`

Returns an object containing a new `Promise` object and two functions to resolve or reject
it, corresponding to the two parameters passed to the executor of the `Promise()`
constructor.

## Instance properties

These properties are defined on `Promise.prototype` and shared all `Promise` instances.

`Promise.prototype.constructor`

The constructor function that created the instance object. For `Promise` instances, the
initial value is the `Promise` constructor.

`Promise.prototype[Symbole.toStringTag]`

The initial value of the `[Symbol.toStringTag]` property is the string `"Promise"`. This
property is used in `Object.prototype.toString()`

## Instance methods

`Promise.prototype.catch()`

Appends a rejection handler cabllack to the promise, and returns a new promise
resolving to the return value of the callback if it is called, or to its original fulfillment
value if the promise is instead fulfilled.

`Promise.prototype.finally()`

Appends a handler to the promise, and returns a new promise that is resolved when the
original promise is resolved. The handler is called when the promise is settled, whether
fulfilled or rejected.

`Promise.prototype.then()`

Appends fulfillment and rejection handlers to the promise, and returns a new promise
resolving to the return value of the called handler, or to its original settled value if the
promise was not handled (i.e., if the relevant handler `onFulfilled` or `onRejected` is not a 
function).

## Examples

### Basic example

In this example, we use `setTimeout(...)` to simulate async code. In reality, you will
probably be using something like XHR or an HTML API.

```js
const myFirstPromise = new Promise((resolve, reject) => {
  // We call resolve(...) when what we were doing asynchronously
  // was successful, and reject(...) when it failed.
  setTimeout(() => {
    resolve("Success!"); // Yay! Everything went well!
  }, 250);
});

myFirstPromise.then((successMessage) => {
  // successMessage is whatever we passed in the resolve(...) function above.
  // It doesn't have to be a string, but if it is only a succeed message, it probably will be.
  console.log(`Yay! ${successMessage}`);
});
```




















































































































































