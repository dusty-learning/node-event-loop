# Check

This phase allows a person to execute calbacks immediately after the `poll` phase has completed. If the `poll` phase becomes idle and scripts have been queued with `setImmediate()`, the event loop may continue to the `check` phase rather than waiting.

`setImmediate()` is actually a special `timer` that runs in a separate phase of the event loop. It uses a [(libuv)](http://libuv.org/) API that schedules callbacks to execute after the `poll` phase has completed.

Generally, as code is executed the event loop will eventually hit the `poll` phase where it will wait for an incoming connection, request, etc. However, if a callback has been scheduled with `setImmediate()` and the `poll` phase becomes idle, it will end and continue to the `check` phase rather than waiting for `poll` events.
