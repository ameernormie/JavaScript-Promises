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
