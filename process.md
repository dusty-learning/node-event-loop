# `process.nextTick()`

Understanding `process.nextTick()`, you may have noticed that it was was not displayed in the diagram in the [README](https://github.com/dusty-learning/node-event-loop/blob/master/README.md) even though it's part of the async API.

Well that's because `process.nextTick()` is not technically part of the event loop. **Gasp**

Instead, the `nextTickQueue` will be processed after the current operation completes, regardless of the current phase of the event loop. Looking back at our diagram, any time you call process.nextTick() in a given phase, all callbacks passed to process.nextTick() will be resolved before the event loop continues.

This can create some bad situations because **it allows you to "starve" your `I/O` by making recursive `process.nextTick()` calls**, which prevents the event loop from reaching the `poll` phase.

## Why is that allowed?

That seems like something that we wouldn't want ever right? So why the heck can we do it?

Well part of it is a design philosophy where an API should always be asynchronous even where it doesn't have to be

For example:

```js
function apiCall(arg, callback) {
  if (typeof arg !== 'string')
    return process.nextTick(callback, new TypeError('argument should be string'));
}

```

The above does an argument check, and if it;s not correct, it will pas the error to the callback.

The API updated fairly recently to allow passing arguments to `process.nextTick()` allowing it to take any arguments passed after the callback to be propagated as the arguments to the callback so you don't have to nest functions.

What we're doing is passing an error back to the user but only **after** we have allowed the rest of the user's code to execute. So by using `process.nextTick()` we can guarantee that `apiCall()` always runs its callback **after** the rest of the user's code and **before** the event loop is allowed to proceed.


## `process.nextTick()` vs `setImmediate()`

We have two calls that are similar as far as users are concerned, but their names are confusing.

- `process.nextTick()` fires immediately on the same phase
- `setImmediate()` fires on the following iteration or `tick` of the event loop

In reality, the names should actually be swapped. `process.nextTick()` fires more immediately than `setImmediate()` does, but this is an artifact of the past which is unlikely to change. Doing this switch would break a large percentage of packages and projects across npm.

It is recommended that developers use `setImmediate()` in all cases because it's easier to reason about.

## So Why Use `process.nextTick()`?

There are two main reasons:

- Allow users to handle errors, cleanup any unneeded resources, or perhaps ttry the request again before the event loop continues
- At times it's necessary to allow a callback to run after the call stack has unwound but before the event loop continues

An example might look something like this:

```js
const server = net.createServer();
server.on('connection', (conn) => { });

server.listen(8080);
server.on('listening', () => { });

```

So say that `listen()` is run at the beginning of the event loop, but however the `listening` callback is placed in a `setImmediate()`. Unless a hostname is passed, binding to the port will happen **immediately**. For the event loop to proceed, it must hit the `poll` phase, which means there is a non-zero chance that a connection could have been recieved allowing the `connection` event to be fired before the `listening` event.

Another example could be running a function constructor that was to inherit from `EventEmitter` and it wanted to call an event within the constructor.

```js
const EventEmitter = require('events');
const util = require('util');

function MyEmitter() {
  EventEmitter.call(this);
  this.emit('event');
}
util.inherits(MyEmitter, EventEmitter);

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});

```

You can't emit an event from the constructor immediately because the script will not have processed to the point where the user assigns a callback to that event.

So, within the constructor itself, you can use `process.nextTick()` to set a callback to emit the event **after** the constructor has finished, which should provide the expected outcome.

Example:

```js
const EventEmitter = require('events');
const util = require('util');

function MyEmitter() {
  EventEmitter.call(this);

  // use nextTick to emit the event once a handler is assigned
  process.nextTick(() => {
    this.emit('event');
  });
}
util.inherits(MyEmitter, EventEmitter);

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});

```