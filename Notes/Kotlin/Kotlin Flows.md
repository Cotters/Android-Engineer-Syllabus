## Creating a flow

The `flow { ... }` builder creates a flow, which runs in a coroutine instance.
Because of this, flows are sequential, just like `lauch { ... }`.

## Intermediate Operators

Functions like `map` and `onEach` receive a flow and do some *intermediary* work on it, before returning the result. 
For example, using `map` to return news items of a certain topic:
```
newsApi.fetchNews()
	.map { newsItems -> newsItems.filter { newsItem.topic == favouriteTopic } }
```

