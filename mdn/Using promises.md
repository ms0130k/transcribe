# Using promises

A promise is an object representing the eventual completion or failure of an asynchronous operation.
Since most people are consumers of already-created promises, this guide will explain consumption of returned promises before explaining how to create them.

Essentially, a promise is a returned object to which you attach callbacks, instead of passing callbacks into a function.
Imagin a function, `createAduioFileAsync()`, which asynchronoulsy generates a sound file given a configuration record and two callback functions:
one callback if the audio file is successfully created, and the other called if an error occurs.

Here's come code that uses `createAudioFileAsync()`:

```js
function successCallback(result) {
  console.log(`Audio file ready at URL: ${result}`);
}

function failureCallback(error) {
  console.log(`Error generating audio file: ${error}`);
}

createAudioFileAsync(audioSettings, successCallback, failureCallback);
```

If `createAudioFileAsync()` were rewritten to return a promise, you would attach your callbacks to it instead:

```js
createAudioFileAsync(audioSettings).then(successCallback, failureCallback);
```

The convention has serveral advantages. We will explore each one.

## Chaining

A common need is to execute two or more asynchronous operations back to back,
where each subsequent operation starts when the previous operation succeeds,
with the result from the previous step.
In the old day, doing several asynchronous operations in a row would lead to the classic callback hell:

```js
doSomething(function (result) {
  doSomethingElse(result, function (newResult) {
    doThirdThing(newResult, function (finalResult) {
      console.log(`Got the final result: ${finalResult}`);
    }, failureCallback);
  }, failureCallback);
}, failureCallback);
```

With promises, we accomplish this by creating a promise chain.
The API design of promises makes this great, because callbacks are attached to the returned promise object, instead of being passed into a function.

Here's the magic: the `then()` function returns a **new promise**, different from the original:

```js
const promise = doSomething();
const promise2 = promise.then(successCallback, failureCallback);
```

This second promise (`promise2`) represents the completion not just of `doSomething()`, but also of the `successCallback` or `failureCallback` you passed in — which can be other asynchronous functions returning a promise.
When that's the case, any callbacks added to `promise2` get queued behind the promise returned by either `successCallback` or `failureCallback`.

Note: if you want a working example to play with, you can use the following template to create any function returning a promise:
```js
function doSomething() {
  return new Promise((resolve) => {
    setTimeout(() => {
      // Other things to do before completion of the promise
      console.log("Did something");
      // The fulfillment value of the promise
      resolve("https://example.com/");
    }, 200)
  });
}
```
The implementation is discussed in the Creating a Promise around an old callback API section below.

With this pattern, you can create longer chains of processing, where each promise representing the completion of one asynchronous step in the chain.
In addition, the arguments to `then` are optional, and `catch(failureCallback)` is short for `then(null, failurecallback)` - so if your error handling code is the same for all steps, you can attach it to the end of the chain:

```js
doSomething()
  .then(function (result) {
    return doSomethingElse(result);
  })
  .then(function (newResult) {
    return doThirdThing(newResult);
  })
  .then(function (finalResult) {
    console.log(`God the final result: ${finalResult}`);
  })
  .catch(failureCallback);
```

You might see this expressed with arrow functions instead:

```js
doSomething()
  .then((result) => doSomethingElse(result))
  .then((newResult) => doThirdThing(newResult))
  .then((finalResult) => {
    console.log(`God the final result: ${fianlResult}`)
  });
```

Note: Arrow function expressions can have an implicit return; so, `() => x` is short for `() => { return x; }`.

`doSomethingElse` and `doThirdThing` can return any value — if they return promises,
that promise is first waited until it settles
































