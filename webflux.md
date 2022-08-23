## Why reactive programming?
- Embedded Tomcat server's default thread pool is 200
- Each thread is expensive and can take up to 1MB of heap memory
- `Callbacks`: 
  - Async methods that accept a callback as parameter and invoke it when the blocking calls complete
  - Writting code with Callbacks is hard, `Callbackhell`
- `Future`:
  - Write async code 
  - Not easy to combine the results of multi futures
  - Future.get() is a blocking call
- `CompletableFuture`
  - Write async code in functional style
  - Easier to compose/combine muilti futures
  - Not easy when future returns many elements like List<Item> will need to wait till all items to complete
  - Cannot handle infinite values (stream)
- `Reactive programming` is async, all calls are non blocking
- Push/pull based data model
- `Netty` is a non blocking server using Event Loop Model
- `Publisher`: can be a DB or API
- `Subscriber`: onSubscribe()
- `Subscription`: can request how much data is sent
- Errors are sent as events just like success events
- `Processor` extends both `Publisher` and `Subscriber`

## Project Reactor
- Flux - 0 to N elements
- Mono - 0 or 1 element
- Use `StepVerifier` to test publishers
- Reactive streams are immutable, calling transform functions won't modify the streams but only returns new streams
- Transform Flux<Mono<Object>> or Flux<Flux<Object>> to Flux<Object> or Mono<Object> in stream using `flatMap()`
- For Mono, `flatMap()` can also transform Mono<List<Object>> to Mono<Object>
- `map()` is synchronous, transform one to one
- `flatMap()` is asynchronous, elements are not transformed orderly, it subscribes to either Flux or Mono then flatten it to send to downstream
- `flatMapSequential()` is similar to `flatMap()` but it subscribes to all elements and queue them in order, it only subscribes to the next stream when the previous stream is already subscribed
- `concatMap()` is similar to `flatMap()` but it wait for the previous substream to complete before subscribing to the next substream, thus preserve the original order but slower than `flatMap()`
- `delayElements()` by default publishes data on `Schedulers.parallel()` thus introduces multi threads beside main thread.
- When there are many threads running, `subscribe()` as an asynchronous method does not block main thread, thus main thread can complete before other threads emit any data
- If we want to wait for all threads to complete and capture all data, we need to use `.collectList().block()` or can implement waiting in main thread
- `collectList()` transforms Flux<Object> to Mono<List> while `block()` waits for all threads to complete
- `flatMapMany()` of Mono is to transform a Mono<Flux<Object>> to Flux<Object>, can use `collectList()` to transform this flux to Mono<List> again
- `concat()` and `concatWith()` combine 2 publishers but 1 publisher will wait for the other to complete first then it subscribes
- `merge()` and `mergeWith()` also combine 2 publishers but asynchronously and do not wait for 1 to complete before starting 2
- `mergeSequential()` also subscribes to 2 publishers eagerly like `merge()` but unlike `merge()`, the results of both publishers after complete are merged into a subscription order
- `zip()` and `zipWith()` subscribe eagerly to 2 publishers, wait for all publishers to finish and then combine them into 1 result

## WebFlux
- To test webflux API, use `@WebFluxTest`
- To call the reactive API, use `WebTestClient`
- To test http status, can use `expectStatus().is2xxSuccessful()`
- To capture the body, can use `returnResult(String.class).getResponseBody()` then use `StepVerifier`
- Or can use `expectBody(String.class).consumeWith(Consumer<EntityExchangeResult<String>>)`
- To test SSE API, first we need to capture the response body of the flux, then use `StepVerifier`, use `expectNext()` to get part of the data, then use `thenCancel().verify()` to stop the stream and verify it
- When calling repository to interact with the database in tests or when init the app, need to call `block()` or `subscribe()` to wait for the insert or delete to happen otherwise the method may end before 
- To test MongoDB repository, can use embedded MongoDB `de.flapdoodle.embed.mongo`, need `@DataMongoTest` and `@ActiveProfile("test")`, also need to specify the version property in yml file `spring.mongodb.embedded.version=3.5.5`
- To assert the result, use StepVerifier with `assertNext()` and `assertComplete()`
- When running embedded MonggoDB, it's best to have `@AfterEach` where we `deleteAll()` the embedded database
- While adding new item to DB, if we check if this item already exists by using `findById()`, it could return empty Mono thus cancel the whole pipeline and `save()` is never called, it's best to use `switchIfEmpty()` to return an empty object instead and check null this object before calling `save()`
- Since `JUnit5`, we need to use annotation `@ExtendWith(MockitoExtension.class)` to do unit tests

## Bean Validation
- To use Bean validation in Spring, use `spring-boot-starter-validation`
- There are 2 ways to validate a bean:
  - Manual by creating a validator
  - Using `@Valid` in parameter of the method
- When the validation fails, it should throw an `WebExchangeBindException`, to get the complete list of violation, use `e.getBindingResult().getAllErrors().stream()`