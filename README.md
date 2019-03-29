# JavaScript-Promises
Understanding Javascript Promise API in depth


## Important Promise Rules

#### Promises following the spec must follow a specific set of rules:
1. A promise or “thenable” is an object that supplies a standard-compliant .then() method.
2. A pending promise may transition into a fulfilled or rejected state.
3. A fulfilled or rejected promise is settled, and must not transition into any other state.
4. Once a promise is settled, it must have a value (which may be undefined). That value must not change.

Every promise must supply a .then() method with the following signature:
```
promise.then(
  onFulfilled?: Function,
  onRejected?: Function
) => Promise
```

#### The .then() method must comply with these rules:
1. Both onFulfilled() and onRejected() are optional.
2. If the arguments supplied are not functions, they must be ignored.
3. onFulfilled() will be called after the promise is fulfilled, with the promise’s value as the first argument.
4. onRejected() will be called after the promise is rejected, with the reason for rejection as the first argument. The reason      may be any valid JavaScript value, but because rejections are essentially synonymous with exceptions, I recommend using        Error objects.
5. Neither onFulfilled() nor onRejected() may be called more than once.
6. .then() may be called many times on the same promise. In other words, a promise can be used to aggregate callbacks.
7. .then() must return a new promise, promise2.
8. If onFulfilled() or onRejected() return a value x, and x is a promise, promise2 will lock in with (assume the same state and    value as) x. Otherwise, promise2 will be fulfilled with the value of x.
9. If either onFulfilled or onRejected throws an exception e, promise2 must be rejected with e as the reason.
10. If onFulfilled is not a function and promise1 is fulfilled, promise2 must be fulfilled with the same value as promise1.
11. If onRejected is not a function and promise1 is rejected, promise2 must be rejected with the same reason as promise1.

## Promise Chaining
Because .then() always returns a new promise, it’s possible to chain promises with precise control over how and where errors are handled. Promises allow you to mimic normal synchronous code’s try/catch behavior.
```
fetch(url)
  .then(process)
  .then(save)
  .catch(handleErrors)
;
```

example of a complex promise chain with multiple rejections:
```
const wait = time => new Promise(
  res => setTimeout(() => res(), time)
);

wait(200)
  // onFulfilled() can return a new promise, `x`
  .then(() => new Promise(res => res('foo')))
  // the next promise will assume the state of `x`
  .then(a => a)
  // Above we returned the unwrapped value of `x`
  // so `.then()` above returns a fulfilled promise
  // with that value:
  .then(b => console.log(b)) // 'foo'
  // Note that `null` is a valid promise value:
  .then(() => null)
  .then(c => console.log(c)) // null
  // The following error is not reported yet:
  .then(() => {throw new Error('foo');})
  // Instead, the returned promise is rejected
  // with the error as the reason:
  .then(
    // Nothing is logged here due to the error above:
    d => console.log(`d: ${ d }`),
    // Now we handle the error (rejection reason)
    e => console.log(e)) // [Error: foo]
  // With the previous exception handled, we can continue:
  .then(f => console.log(`f: ${ f }`)) // f: undefined
  // The following doesn't log. e was already handled,
  // so this handler doesn't get called:
  .catch(e => console.log(e))
  .then(() => { throw new Error('bar'); })
  // When a promise is rejected, success handlers get skipped.
  // Nothing logs here because of the 'bar' exception:
  .then(g => console.log(`g: ${ g }`))
  .catch(h => console.log(h)) // [Error: bar]
;
```

## Error Handling
```
save().then(
  handleSuccess,
  handleError
);
```
But what happens if handleSuccess() throws an error? The promise returned from .then() will be rejected, but there’s nothing there to catch the rejection — meaning that an error in your app gets swallowed. Oops!

For that reason, some people consider the code above to be an anti-pattern, and recommend the following, instead:
```
save()
  .then(handleSuccess)
  .catch(handleError)
;
```

What if you want to handle them differently? You could opt to handle them both:
```
save()
  .then(
    handleSuccess,
    handleNetworkError
  )
  .catch(handleProgrammerError)
;
```

## Cancel a Promise?
### Adding .cancel() to the promise
Adding .cancel() makes the promise non-standard, but it also violates another rule of promises: Only the function that creates the promise should be able to resolve, reject, or cancel the promise. Exposing it breaks that encapsulation, and encourages people to write code that manipulates the promise in places that shouldn't know about it. Avoid spaghetti and broken promises.

### Rethinking Promise Cancellation
```
const wait = (
  time,
  cancel = Promise.reject()
) => new Promise((resolve, reject) => {
  const timer = setTimeout(resolve, time);
  const noop = () => {};

  cancel.then(() => {
    clearTimeout(timer);
    reject(new Error('Cancelled'));
  }, noop);
});

const shouldCancel = Promise.resolve(); // Yes, cancel
// const shouldCancel = Promise.reject(); // No cancel

wait(2000, shouldCancel).then(
  () => console.log('Hello!'),
  (e) => console.log(e) // [Error: Cancelled]
); 
```

We’re using default parameter assignment to tell it not to cancel by default. That makes the cancel parameter conveniently optional. Then we set the timeout as we did before, but this time we capture the timeout’s ID so that we can clear it later.

We use the cancel.then() method to handle the cancellation and resource cleanup. This will only run if the promise gets cancelled before it has a chance to resolve. If you cancel too late, you’ve missed your chance. That train has left the station.

### Abstracting Promise Cancellation
1. Reject the cancel promise by default — we don’t want to cancel or throw errors if no cancel promise gets passed in.
2. Remember to perform cleanup when you reject for cancellations.
3. Remember that the onCancel cleanup might itself throw an error, and that error will need handling, too. (Note that error      handling is omitted in the wait example above — it’s easy to forget!)


## Source
This information is taken from the blog post [Master the JavaScript Interview](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261) by  Eric Elliot

