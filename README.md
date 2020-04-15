# RxJava

Excerpts from the GitBook by Thomas Nield
https://legacy.gitbook.com/book/thomasnield/rxjavafx-guide/

RxJava has two core types, `Observable` and `Observer`.


An `Obserable` _pushes_ things. A given `Observable<T>` will push items of type `T` through a series of operators that form other Observables, and finally the terminal `Observer` is what consumes the items at the end of the chain.


Each pushed `T` item is known as an _emission_. Emissions can be finite or infinite! An emission can represent either data or an event (or both!). This is where the power of reactive programming differentiates itself from Java 8 Streams and Kotlin Sequences.

It has a notion of _emissions over time_.

Emissions are pushed all the way to a `Observer` where they are finally consumed.

- Create a _source Observable_ where emissions originate, and you can use many factories to create it.
    ```java
    Observable<Integer> source = Observable.just(1,2,3,4,5);
    ```
    Nothing is pushed yet. You need to create an `Observer` before pushing emissions.

- The quickest way is to call `subscribe()` and pass a lambda specifying what to do with each emission.
    ```java
    source.subscribe(System.out::println);
    ```

- The `subscribe()` operation creates an `Observer` for us based on the lambda argument. This effectively pushed the integers 1 through 5, one-at-a-time, to the Observer defined by the lambda in the subscribe() method. The subscribe() method does not have to print items. It could populate them in a JavaFX control, write them to a database, or post it as a server response.

- You can specify up to three lambda arguments on the `subscribe()` method to handle each emission, handle the event of an error, and the action on completion when there are no more emissions.

- _You should always supply an `onError` lambda to your `subscribe()` call so errors don't go unhandled._
    ```java
    source.subscribe(System.out::println, Throwable::printStackTrace, () -> System.out.println("Done!"));
    ```

- You can create your own `Observer` object by extending `ResourceObserver` and implementing its three abstract methods, `onNext()`, `onError()`, `onComplete()`. You can then pass this `Observer` to the `subscribe()` method.

- It is critical to note that `onNext()` can only be called by one thread at a time.

- When you subscribe to a factory like `Observable.just(1,2,3,4,5)` you will always get those emissions serially, in that exact order, and on a _single thread_.

- You can turn any `Iterable<T>` into an `Observable<T>` by using `Observable.fromIterable()`. The observable will then emit items from that iterable and call `onComplete()` when done.

---

## Operators

In RxJava, you can use hundreds of operators to transform emissions and create new Observables with those transformations.

Eg. You can create an `Observable<Integer>` off an `Observable<String>` by using the `map()` operator, and use it to emit each String's length.

Operators behave as both an intermediary `Observer` and an `Observable`, receiving emissions from the upstream source, transforming them, and passing them downstream to the final `Observer`. 

```java
Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")  // calls onNext() on map()
    .map(String::length)  // calls onNext() on Observer
    .subscribe(System.out::println);
```

Map pushes all items through a function that returns a value.

Filter supresses emissions taht fail to meet a certain criteria, and pushes the ones that do forward.

Distinct suppresses emissions whcih have previously been emitted to prevent duplicate emissions based on each emission's hashcode()/equals() implementation.

