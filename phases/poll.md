# Poll

The `poll` phase has to main functions:

- Executing scripts for timers whose threshold has elapsed
- Processing events in the `poll` queue

When the event loop etners the `poll` phase and **there are no timers scheduled**, one of two things will happen

- If the `poll` queue **is not empty**, the event loop will iterate through its queue of callbacks executing them synchronously until either the queue has been exhausted, or the system-dependent hard limit is reached
- If the `poll` queue **is empty** one of two more things will happen
  - If scripts have been scheduled by `setImmediate()`, the event loop will end the `poll` phase and continue to the `check` phase to execute those scheduled scripts
  - If scripts **have not** been scheduled by `setImmediate()`, the event loop will wait for callbacks to be added to the queue, then execute them immediately

Once the `poll` queue is empty the event loop will check for `timers` whose **time thresholds have been reached**. If one or more timers are ready, the event loop will wrap back to the `timers` phase to execute those timers' callbacks.
