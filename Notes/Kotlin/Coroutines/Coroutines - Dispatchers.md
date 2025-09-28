Dispatchers route the work of your coroutines.
Typically, you will use `withContext` to shift the context of an **existing** coroutine. 

For example, within the ViewModel's coroutine - `viewModelScope` - you typically call a UseCase which will call a Repository. In the Repository, you will want to switch to the IO thread by calling `withContext(Dispatchers.IO)`. This is because `viewModelScope` runs on the Main thread, as the ViewModel is usually performing UI-related work.

Below are details on the 4 dispatchers used in Coroutines.
## Default

Use for CPU-intensive tasks, such as:
- Matrix multiplication.
- Big in-memory list manipulation, like sorting, searching, or filtering.
- Parsing or filtering things in memory (NOT reading from disk), such as Bitmaps or JSON.

> `Default` uses the number of CPU cores in the device (minimum of 2). So a minimum of 2 tasks can run in parallel when using this dispatcher.
## IO

Many threads; useful for network operations or disk read/writes.
Example use cases include:
- Making a network request.
- Reading and writing files to disk.
- Making a database query (this is basically reading from disk).
- Using SharedPreferences (again, reading from disk).

## Main

Use this dispatcher to switch back to the main (usually single) thread.

## Unconfined

Useful for tests. This dispatcher tells the Coroutine that it may run on any thread.
It can start on one thread and be resumed on another.