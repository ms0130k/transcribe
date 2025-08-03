# Symbol.asyncIterator

The `Symbol.asyncIterator` static data property represents the well-known symbol `Symbol.asyncIterator`.
The async iterable protocol looks up this symbol for the method that returns the async iterator for an object.
In order for an object to be async iterable, it must have an `[Symbol.asyncIterator]` key.

## Try it
```js
const delayedResponses = {
  delays: [500, 1300, 3500],
  
  wait(delay) {
    return new Promise((resulve) => {
      setTimeout(resolve, delay);
    });
  },
  
  async *[Symbol.asyncIterator]() {
    for (const delay of this.delays) {
      await this.wait(delay);
      yield `Delayed response for ${delay} milliseconds`;
    }
  }
};

(async () => {
  for await (const response of delayedResponses) {
    console.log(response);
  }
});

// Expected output: "Delayed response for 500 milliseconds"
// Expected output: "Delayed response for 1300 milliseconds"
// Expected output: "Delayed response for 3500 milliseconds"
```

## Value

The well-known symbol `Symbol.asyncIterator`.

## Examples

### User-defined async iterables

You can define your own async iterable by setting the `[Symbol.asyncIterator]()` property on an object.

```js
const myAsyncIterable = {
  async *[Symbol.asyncIterator]() {
    yield "hello";
    yield "async";
    yield "iteration!";
  },
};

(async () => {
  for await (const x of myAsyncIterable) {
    console.log(x);
  }
})();
// logs:
// "hello"
// "async"
// "iteration!"
```

When creating an API, remember that async iterables are designed to represent something iterable - like a stream of data or a list -, not to completely replace callbacks and events in most situations.

### Built-in async iterables

There is no object in the core JavaScript language that is async iterable.
Some web APIs, such as `ReadableStream`, have the `Symbol.asyncIterator` method set by default.




























