# class *Promise.Stream*

- [new Stream()](#new-streamsource)
- [Stream.from()](#fromiterable---stream)
- [Stream#map()](#mapconcurrency-callback---stream)
- [Stream#forEach()](#foreachconcurrency-callback---stream)
- [Stream#filter()](#filterconcurrency-callback---stream)
- [Stream#takeUntil()](#takeuntilpromise---stream)

### new Stream(*source*)

Creates a new stream from the given *source*. Streams are used to handle many values (or promises of values) concurrently. Unlike many styles of streams, this stream does not maintain the same sequence of items as they were supplied. It will output the items as soon as they are resolved.

Streams are also instances of `jellypromise`. If an error occurs in a stream, the stream will be rejected, along with all streams that originate from it. If no error occurs, most streams will be fulfilled with `undefined` when all of its items have been passed on. However, some methods are available to change the fulfillment value of a stream ([.merge()](), [.reduce()]()).

The provided *source* must be some style of `EventEmitter` with the `.addListener()` and `.removeListener()` methods. The stream will listen for the `data`, `end`, and `error` events. The `data` event is used to push an item into the stream; this can be any type of data (or a promise of the data). The `end` event is used to indicate that no more data will be pushed into the stream. After the `end` event, the stream will continue processing the rest of its current data before fulfilling its promise interface. The `error` event is used to indicate an error. Errors will propagate down the pipeline to all streams that originated from this one.


### .map([*concurrency*], *callback*) -> *Stream*

Transforms the stream's data through the provided `callback` function, and passes the resulting data to a new stream. If the function returns a promise, its value will be awaited before being passed on to the destination stream (which is returned by this method).

If a `concurrency` number is provided, only that many items will be processed at a time. The default is `Infinity`.

If a promise is rejected, or if `callback` throws an exception, processing will stop and the stream will be rejected with the same error.

`callback` has the following signature: `function callback(value, index)`

The `index` parameter is the item's *original* index from when it entered the first stream in the pipeline.

### .forEach([*concurrency*], *callback*) -> *Stream*

Similar to [.map()](#mapconcurrency-callback---stream), except the resulting data will not be changed. If the callback returns a promise, it will still be awaited, but it will not determine the data that is passed on to the destination stream. This method is primarily used for side effects.

`callback` has the following signature: `function callback(value, index)`

### .filter([*concurrency*], *callback*) -> *Stream*

Similar to [.forEach()](#foreachconcurrency-callback---stream), but the items will be filtered out by the provided function (just like `Array#filter`). The function may return both booleans or promises of booleans.

`callback` has the following signature: `function callback(value, index)`

### .takeUntil(*promise*) -> *Stream*

Forwards the stream's data to a new stream that is returned by this method. The new stream will continue to pull data until the given `promise` is fulfilled. If the `promise` is rejected, the returned stream itself will also be rejected with the same error.