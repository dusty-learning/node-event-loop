# Close Callbacks

If a socket or handle is closed abruptly (e.g.`socket.destroy()`), the `'close'` event will be emitted in this phase. Otherwise it will be emitted via `process.nextTick()`.


