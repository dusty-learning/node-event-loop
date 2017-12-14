# node-event-loop
The basis of what node works on, and how things work. The Node Event loop is where to start


## What is it?

So the node event loop is central to being able to handle high throughput scenarios. It's the reason Node can essentially be "single threaded" while still allowing an arbitrary number of operations to be handled in the background.

The event loop is what allows Node.js to perform non-blocking I/O operations — despite the fact that JavaScript is single-threaded — by offloading operations to the system kernel whenever possible.

It's the thing that we are always telling you to not block because you're gonna have a bad time.

## What does it do?

When Node.js starts, it initializes the event loop, processes the provided input script (or drops into the REPL, which is not covered in this document) which may make async API calls, schedule timers, or call process.nextTick(), then begins processing the event loop.

The Event loop works in a series of `Phases` as it processes incoming connections, data, etc.

Each phase has a `FIFO` queue of callbacks to execute. While each phase is special in its own way, when the event loop enters a given phase, it will perform any operations specific to that phase, and then execute a callback in that phases queue until the queue has been exhausted or the maximum number of callbacks has executed. Once this happens the event loop will then move on to the next phase, and so on.

Since any of these operations may schedule more operations and new events processed in the poll phase are queued by the kernel, poll events can be queued while polling events are being processed. As a result, long running callbacks can allow the poll phase to run much longer than a timer's threshold.

## Phases Overview

- `timers`: This phase executes callbacks scheduled by `setTimeout()` and `setInterval()`
- `I/O callbacks`: Executes almost all callbacks with the exception of close callbacks, the ones scheduled by timers, and `setImmediate()`
- `idle, prepare`: Only used internally
- `poll`: Retrieve new I/O events; node will block here when appropriate
- `check`: `setImmediate()` callbacks are invoked here
- `close callbacks`: things like `socket.on('close', ...)`

A little diagram of phases layout:

```
   ┌───────────────────────┐
┌─>│        timers         │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │
   └───────────────────────┘
```

## Phases in Detail

You can check out the phases folder, or click on the links below to get more details on each phase

- [Timers](https://github.com/dusty-learning/node-event-loop/blob/master/phases/timers.md)
- [I/O Callbacks](https://github.com/dusty-learning/node-event-loop/blob/master/phases/IO-callbacks.md)
- [Poll](https://github.com/dusty-learning/node-event-loop/blob/master/phases/poll.md)
- [Check](https://github.com/dusty-learning/node-event-loop/blob/master/phases/check.md)
- [Close Callbacks](https://github.com/dusty-learning/node-event-loop/blob/master/phases/close-callbacks.md)

