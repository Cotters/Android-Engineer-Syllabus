Dispatchers route the work of your coroutines. The `Default` dispatcher uses the device's CPU cores. The `IO` dispatcher has a pool of 64 threads minimum, or number of cores. The `Main` dispatchers routes things to the UI thread, usually a single thread.

## Default

Use for CPU intensive tasks, such as:
- Matrix multiplication.
- Big in-memory list manipulation, like sorting, searching, or filtering.
- Parsing or filtering things in memory (NOT reading from disk), such as Bitmaps or JSON.

## IO

Many threads; useful for network operations or disk read/writes.
Example use cases include:
- Making a network request.
- Reading and writing files to disk.
- Making a database query (this is basically reading from disk).
- Using SharedPreferences (again, reading from disk).

## Main

Use this dispatcher to switch back to the main thread.

## Unconfined

Useful for tests. This dispatcher tells the Coroutine that it may run on any thread.
It can start on one thread and be resumed on another.