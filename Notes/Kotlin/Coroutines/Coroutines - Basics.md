## Introduction to Coroutines

To implement concurrency in Kotlin, use coroutines. They are formed by building suspendable functions which can be called in a sequential manner. 

Coroutines suspend their operation, freeing up the thread they work on to allow other suspendable operations to run. This makes them lightweight compared to traditional parallel programming that uses multiple threads. 

They are separate from threads: threads are relatively resource intensive and are **managed by the OS**, while coroutines can run on the same thread or on different threads - they are not bound by a thread; when a coroutine suspends it does not block that thread.
They can be suspended and resumed - and not necessarily resumed on the same thread.

They can be blocking, though there isn't much use case for this in an Android as the app should be asynchronous - and especially not block the main thread.
## Suspend Functions

A function marked with `suspend` will suspend the coroutine until it completes. It also ensures that this function is called from within a coroutine context.
## Coroutine Scope

There are a few builders available to launch a coroutine scope.
A common one for Android is `viewModelScope` which can be used in ViewModels to call suspend functions from within the ViewModel itself, or from another class. For example, ViewModels often call suspend functions in UseCases - following a common pattern in Clean Architecture.

```
class AuthViewModel(
	private val registerAccount: RegisterAccountUseCase,
) {
	fun register(email: String, password: String) = viewModelScope.launch {
		// Check email & password are not empty.
		registerAccount.invoke(email, password)
	}
}

class RegisterAccountUseCase(
	private val authRepository: AuthRepository,
) {
	suspend func invoke(email: String, password: String) {
		// Check password is 8+ characters
		// Call repository
	}
}
```

Because `invoke` is marked with `suspend`, it must be called from a coroutine scope. Which is why we use `viewModelScope.launch` in our `AuthViewModel`.

> Using '`func invoke`' allows the use case to be called like: 
> `registerAccount(email, password)` from within the AuthViewModel.
## Moving threads

Use `withContext(CoroutineContext)` to move threads. For example: `withContext(Dispatchers.IO)` will move execution to the IO thread, which is useful for network requests, which generally take longer than 1 second to execute.

Continuing our `RegisterAccountUseCase` above, our repository would look something like this:

```
class AuthRepository(
	private val ioDispatcher: Dispatcher = Dispathers.IO,
) {
	suspend fun registerAccount(email, password) = withContext(ioDispatcher) {
		// Call API.
	}
}
```

`withContext(ioDispatcher)` switches to the IO thread (useful for network request) and off of the main thread which `viewModelScope` uses. When functions like this return a value, we're back on the thread with which we called it, in this case the main thread.

> It is important to highlight that `viewModelScope` uses Dispatchers.Main (main thread) by default. So avoid any long tasks within `viewModelScope`, or switch to the `Default` thread.
## Supervisor Scope

Useful whenever you want to continue performing more than 1 operation, ignoring or handling the failure(s) of other operations.
This is really only useful when you're waiting for all jobs to happen before moving on.

For example, imagine we have a photo editing app and we're applying multiple filters to an image. We should use a supervisor scope to ensure that failing to apply one filter doesn't stop other filters from being applied. We can handle the failures and alert the user which filters weren't applied while presenting the ones that did.

See the docs for more info: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/supervisor-scope.html

## Coroutine Scope and Structure

A coroutine scope can be considered a tree. 

Kotlin coroutines rely on **structured concurrency** which means that coroutines form a tree hierarchy of parent and child tasks with linked lifecycles. With this structure, parent tasks only finish once all child tasks have finished.
If a parent coroutine fails or is cancelled, or it's children get cancelled too.

To maintain this structure, coroutines must be called from a **CoroutineScope**. We saw one earlier - the `viewModelScope` - which defines and manages the lifecycle of coroutines within it. The benefit of using `viewModelScope` is automatic lifecycle management: we don't have to clean up our coroutines, cancel, or worry about memory leaks as these are all automatically cancelled when the ViewModel is destroyed. Contrast this with RxJava which required a `dispose()` call on jobs/disposables.



