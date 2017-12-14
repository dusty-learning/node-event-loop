# Timers

A timer specifies the **threshold** after which a provided callback **may be executed** rather than the **exact** time a person wants it to be executed.

**Important Note**: Technically, the `poll` phase controls when timers are executed

## Example

Let's say you schedule a timeout to execute after a 100ms threshold, then your scripts starts `async` reading a file which takes 95ms.

```js
const fs = require('fs');

function someAsyncOperation(callback) {
  // Assume this takes 95ms to complete
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);


// do someAsyncOperation which takes 95 ms to complete
someAsyncOperation(() => {
  const startCallback = Date.now();

  // do something that will take 10ms...
  while (Date.now() - startCallback < 10) {
    // do nothing
  }
});

```

When the event loop enters the `poll` phase, it has an empty queue since `fs.readFile()` has not completed. It will wait for the number of miliseconds remaining until the sonnest `timer` threshold is reached. While it is waiting for the 95ms to pass `fs.readFile()` finishes reading the file and it's callback which takes 10ms to complete is added to the `poll` queue and executed.

When the callback finishes, there are no more callbacks in the queue, so the event loop will see that the threshold of the soonest timer has been reached then wrap back to the timers phase to execute the timer's callback.

In this example, you will see that the total delay between the timer being scheduled and its callback being executed will be 105ms.

In order to prevent the poll phase from starving the event loop, the C library [(libuv)](http://libuv.org/) also has a hard maximum (which is system dependent) before it stops polling for more events.
