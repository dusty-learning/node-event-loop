# `setImmediate()` vs `setTimeout()`

`setImmediate()` and `setTimeout()` are similar, but behave in different ways depending on when they're called.

- `setImmediate()` is designed to execute a script once the current `poll` phase compeletes
- `setTimeout()` schedules a script to run after a minimum theshold in miliseconds has elapsed

The timer execution order will vary depending on the context they are called in. If both are called from within the main module, then timing wil be bound by the performance of the process. This can be impacted by other applications/processes that are running on the machine.

So if we run a script which is not within a `I/O Cycle` (The main module), the order in which the two timers are executed is non-deterministic. Since it's bound by the performance of the process.

Example:

```js
// timeout-no-io.js
setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('immediate');
});

```

Our output would look something like this:

```
$ node timeout_vs_immediate.js
timeout
immediate

$ node timeout_vs_immediate.js
immediate
timeout
```

However if we move the two calls within an `I/O Cycle`, the immediate callback is **always** executed first.

```js
// timeout-io.js
const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});

```
Our output from the above would be:

```
$ node timeout_vs_immediate.js
immediate
timeout

$ node timeout_vs_immediate.js
immediate
timeout
```

The main Advantage to using `setImmediate()` over a `setTimeout()` is that `setImmediate()` will always be executed **before** any timers if scheduled within an `I/O Cycle`, independently of how many other timers are present.