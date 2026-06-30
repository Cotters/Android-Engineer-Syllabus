Docs: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/

To represent a stream of values you can use Kotlin's `flow`s.
Flows typically work with asynchronously computed data. As such they emit values.

In Android, you can use Flow with popular libraries like CoreData, Room, Retrofit, and WorkManager. For example with Room, you can wrap the return type in a Flow type:

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

Let's say you want to query our news API. you probably want to check it frequently to ensure you have breaking news. To achieve this, you can create a flow with an infinite loop and a delay.

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

Each time a terminal operator such as `collect` is used on a Cold Flow, a new flow is created which will only then start emitting values. Hot Flows are created independently of terminal operators, which you'll explore below.

`collectLatest` is a terminal operator similar to `collect` but the key difference is that `collectLatest` cancels the previous action block when new values are received.
So you can quickly discard old values and operate only on the latest.

Other useful operators include `first` and `toList`.
## Cold Flow vs Hot Flow

Cold Flows are like a function call whereas Hot Flows are like broadcasts; Cold Flows execute independently whereas Hot Flows are ongoing events you can tune into.

Cold Flows only emit values when the flow is collected. Hot Flows emit values as soon as the flow is created.

Cold Flows will re-execute the producer code (e.g. fetching data from an API) for each collector whereas Hot Flows do not re-execute (as collectors share the same data stream).

| Aspect           | Cold Flow                      | Hot Flow                |
| ---------------- | ------------------------------ | ----------------------- |
| **Execution**    | Starts when collected          | Starts immediately      |
| **Data Sharing** | Independent per collector      | Shared among collectors |
| **Producer**     | Re-executes for each collector | Single active instance  |
## StateFlow

StateFlow is a type of Hot Flow and is useful for persisting and re-using values produced by a flow, such as UI state. StateFlow's hold the most recently emitted value.

This is useful for avoiding unnecessary fetching of information you already have. For example, if the device is rotated then you don't want the flow to call our API again to fetch the same data you had before. Instead, you can use StateFlow which acts as a buffer and stores the values from our API, re-emitting the same values once our view is recreated.

## ShareIn

An important operator that converts a cold flow into a (hot) Shared Flow.

This operator provides more control over a flow and is useful when there is expensive computation. If you are fetching data from a backend and the connection takes a long time, you can create a cold flow, then use the `shareIn` operator with some additional parameters to control how this connection starts and what values you store and receive.

```kotlin
fun <T> Flow<T>. shareIn(
	scope: CoroutineScope, 
	started: SharingStarted,
	replay: Int = 0
): SharedFlow<T>
```

If you have an expensive call to our backend, and you want to create a single shared upstream source for multiple subscribes, you can define the expensive backend call as a cold flow and then use the `shareIn` operator inside an application level, session level, or repository level scope and have multiple subscribes use that stream. you then use the `SharingStarted`  parameter to control when the cold flow block executes.

`SharingStarted` configures the flow in the following ways:
- `Eagerly` starts the upstream flow before any subscribes.
- `Lazily` starts the upstream flow when it has it's first subscriber.
- `WhileSubscribed` starts the upstream flow lazily and stops it when it has no subscribes. You can configure a timeout delay to prevent restarting flows which can happen when switching betyouen screens, for example.

## Buffers and Replays

Buffers allow subscribers to lag behind the upstream flows emitted values, whereas replay defines the amount of previously emitted values new subscribers receive when they subscribe to an upstream flow.
This means that buffers are non-blocking (allows the upstream flow to continue emitting values) and replays are blocking, as it requires each subscriber to process the values before continuing.

By default, MutableSharedFlow doesn't use a buffer, so emitting is suspended until all subscribers receive the emitted value.

