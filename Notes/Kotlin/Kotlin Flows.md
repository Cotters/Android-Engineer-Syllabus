Docs: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/

To represent a stream of values we can use Kotlin's `flow`s.
Flows typically work with asynchronously computed data. As such they emit values.

In Android, we can use Flow with popular libraries like CoreData, Room, Retrofit, and WorkManager. For example with Room, we can wrap the return type in a Flow type:

```kotlin
@Dao
interface MessagesDao {
	@Query('SELECT * FROM messages')
	fun getAllMessages(): Flow<List<Message>>
}
```

## Creating a flow

The `flow { ... }` builder creates a flow, which runs in a coroutine instance.
Because of this, flows are sequential, just like `lauch { ... }`.

Let's say we want to query our news API. We probably want to check it frequently to ensure we have breaking news. To achieve this, we can create a flow with an infinite loop and a delay.

```kotlin
class NewsDataSource(
	private val newsApi: NewsApi,
	private val refreshIntervalMs: Long = 5000,
) {
	val newsItems: Flow<List<NewsItem>> = flow {
		while(true) {
			val latestNewsItems = newsApi.fetchLatestNews()
			emit(latestNewsItems)
			delay(refreshIntervalMs)
		}
	}
}
```

The `flow {}` block above is known as a 'producer block' as it is producing values via the `emit` method. Whatever consumes this block - by using something like the `collect` method - is known as a 'consumer'. (More on this below.)

> The `collect` method is known as a 'terminal operator'.
## Intermediate Operators

Functions like `map` and `onEach` receive a flow and do some *intermediary* work on it, before returning the result. For example, using `map` to return news items of a certain topic:

```kotlin
newsApi.fetchLatestNews()
	.map { newsItems -> newsItems.filter { newsItem.topic == favouriteTopic } }
```

These intermediate operators are lazily constructed chain of operations; they do not create a flow, but operate on the flow when it is created. On the other hand, when a terminal operator is applied to a flow, the flow is created on demand and starts emitting values.
## Terminal Operators

We have seen intermediate operators like `map` that applies an operation to a flow, so now let's look at observing these flows with terminal operators like `collect`.
Each time you call `collect`, a new flow is created. The flow will then start emitting values.

Other useful operators include `first` and `toList`.
### Cold Flow vs Hot Flow

Cold Flows are like a function call whereas Hot Flows are like broadcasts; Cold Flows execute independently whereas Hot Flows are ongoing events you can tune into.

Cold Flows only emit values when the flow is collected. Hot Flows emit values as soon as the flow is created.

Cold Flows will re-execute the producer code (e.g. fetching data from an API) for each collector whereas Hot Flows do not re-execute (as collectors share the same data stream).

| Aspect           | Cold Flow                      | Hot Flow                |
| ---------------- | ------------------------------ | ----------------------- |
| **Execution**    | Starts when collected          | Starts immediately      |
| **Data Sharing** | Independent per collector      | Shared among collectors |
| **Producer**     | Re-executes for each collector | Single active instance  |
## StateFlow

StateFlow is a type of Hot Flow and is useful for persisting and re-using values produced by a flow, such as UI state.

For example, if we rotate our device, we don't want the flow to call our API again to fetch the same data we had before. Instead, we can use StateFlow which acts as a buffer and stores the values from our API, re-emitting the same values once our view is recreated.

