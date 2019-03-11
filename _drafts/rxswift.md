---
title: "RxSwift and RxCocoa basics"
categories: [swift, ios]
---
### What is Rx?

Rx stands for Reactive Extensions (aka ReactiveX), it is a set of tools which helps to write a code acording to the reactive programming pattern. Using the reactive programming we can write our code in a more declarative way and using Rx library we don't have to think about threads and all the infrastructure.  
RxSwift is an implementation of Rx in Swift. RxCocoa is a library which adds Rx functionality to Cocoa classes and Rx types which can better cooperate with Cocoa. To start with RxSwift and RxCocoa just add the libraries ([project on GitHub](https://github.com/ReactiveX/RxSwift)) to your project and import them (`import ...`) in the file where you want to use Rx.

### Observable sequences

Almost everything in RxSwift is an observable sequence (`Observable`) and almost every Swift collection can be transformed to `Observable`.

{% highlight swift %}
let sequence = Observable.from([0, 1, 2, 3, 4, 5])
{% endhighlight %}

### *Hot* and *cold* observables

There are two main types of observables - hot and cold. A hot observable emits events no matter if there is an object that subscribes to the observable, an example of a hot observable is `Variable` type. The observable is usually stateful and computation resources are usually shared between all of the subscribed observers.  
A cold observable doesn't emit events until an observer subscribes to it, is used i.e. for HTTP connections. The observable is usually stateless and computation resources are usually per observer.

### Subscribing to observables

It is natural that if something produces events, an other thing wants to listent to them. We can listent to events from observables using `subscribe(on: (Event<T>) -> Void)` method.

{% highlight swift %}
let sequence = Observable.just("Hello")
let subscription = sequence.subscribe { event in
    print(event)
}
{% endhighlight %}

The code above produces

```
next(Hello)
completed
```

`event` is `enum` value that can be one of the following values:

* next(value: T)
* error(error: Error)
* completed