## remember

Use `remember` whenever we need a local state in our view that should be calculated once and cached for reuse. For example, `remember { randomColor }` will produce and store a `Color` once when our view loads will be reused between recompositions.

> Avoid 'backward writes', where you update a state that's already been read. Writing to state that's already been read means that our view will recompose, because our view sees new state and deems itself out of date. This causes a recomposition loop.

Best way to ensure our views are using state correctly is to use unidirectional state flow with a view model. If this is unavoidable, use `remember` with the parameter as the data that should cause our state to update. You can think of these parameters as keys to a local cache.

For example, if we have `transactions` and we want to show balances as well, we could write the following:

```kotlin
val balances = remember(transactions) {
	transactions.runningReduce { a, b -> a + b }
}

// Show transactions and balances in list of Text views.

```

This will run our composable once, and will only consider itself out of date when `transactions` change, avoiding unnecessary recompositions and avoiding writing to any state entirely!

Better yet would be to do this work in the ViewModel and providing an immutable state.

To summarise, in Jetpack Compose views, if we have expensive or impure function calculations with input data we generally want to wrap them in a `remember` function to ensure we re-use their values. If we are just presenting simple data then there's no need.

## mutableStateOf

Using `mutableStateOf` allows us to create state within our views which is subscribed to whenever it's value is read. The mutability is in the fact that this value can be written to as well as read. Whenever the value updates, the view is recomposed.

```kotlin
var text by remember { mutableStateOf("Sounds good!") }
TextField(
	value = text,
	onValueChange = { text = it }
)
```

> Note on 'property delegates': Using state with the *by* keyword allows us to treat the state object's value as the value itself. Meaning the `getValue` and `setValue` methods of `MutableState` are implicit when we use the state object.

