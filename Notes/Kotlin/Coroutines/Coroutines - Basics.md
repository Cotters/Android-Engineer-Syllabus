Coroutines are light-weight and run on threads. 
They can be suspended and resumed - and not necessarily resumed on the same thread.
They can also be blocking, though there isn't much use case for this in Android as the app should be asynchronous and especially not block the main thread.

## Suspend Functions

A function marked with `suspend` will 'suspend' the coroutine until it completes. It also ensures that this function is called from within a coroutine context.

## Coroutine Scope

There are a few builders available to launch a coroutine scope.
Common one is `viewModelScope` which can be used in ViewModels to call suspend functions elsewhere, a common pattern in Clean Architecture that uses Coroutines.

```
class AuthViewModel(
	private val registerAccount: RegisterAccountUseCase,
) {
	fun register(email: String, password: String) = viewModelScope.launch {
		// Check email & password are not empty.
		registerAccountUseCase.invoke(email, password)
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

Continuing from our `RegisterAccountUseCase` above, our repository will look something like this:

```
class AuthRepository(
	private val ioDispatcher: Dispatcher = Dispathers.IO
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
