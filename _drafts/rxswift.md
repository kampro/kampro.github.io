---
title: "RxSwift and RxCocoa basics"
categories: [swift, ios]
---
## What is Rx?

Rx stands for Reactive Extensions (aka ReactiveX), it is a set of tools which helps to write a code acording to the reactive programming pattern. Using the reactive programming we can write our code in a more declarative way and using Rx library we don't have to think about threads and all the infrastructure.  
RxSwift is an implementation of Rx in Swift. RxCocoa is a library which adds Rx functionality to Cocoa classes and Rx types which can better cooperate with Cocoa. To start with RxSwift and RxCocoa just add the libraries ([project on GitHub](https://github.com/ReactiveX/RxSwift)) to your project and import them (`import ...`) in the file where you want to use Rx.

## Observable sequences

Almost everything in RxSwift is an observable sequence (`Observable`) and almost every Swift collection can be transformed to `Observable`.

{% highlight swift %}
let sequence = Observable.from([0, 1, 2, 3, 4, 5])
{% endhighlight %}

## *Hot* and *cold* observables

There are two main types of `Observable`s - hot and cold. A hot `Observable` emits events no matter if there is an object that subscribes to the `Observable`, an example of a hot `Observable` is `Variable` type. The `Observable` is usually stateful and computation resources are usually shared between all of the subscribed observers.  
A cold `Observable` doesn't emit events until an observer subscribes to it, is used i.e. for HTTP connections. The `Observable` is usually stateless and computation resources are usually per observer.

## Subscribing to Observables

It is natural that if something produces events, an other thing wants to listent to them. We can listent to events from `Observable`s using `subscribe(on: (Event<T>) -> Void)` method.

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

* `next(value: T)` - this event is emitted when a new value is added to an `Observable`
* `error(error: Error)` - if an error occurs, this type of event is emitted. This event terminates the sequence.
* `completed` - if a sequence ends without errors, this event is emitted. It also terminates the sequence.

If you want to subscribe for events of a specific type you can use

{% highlight swift %}
func subscribe(onNext: ((T) -> Void)?, onError: ((Error) -> Void)?, onCompleted: (() -> Void)?, onDisposed: (() -> Void)?) -> Disposable
{% endhighlight %}

## Disposing subscriptions

If you no longer use a subscription you need to dispose it to avoid a memory leak. You can do it manually by calling `dispose()` on the subscription or by adding it to the `DisposeBag`, then, the subscription is disposed automatically when `DisposeBag` is deinitialised.

{% highlight swift %}
class ViewController: UIViewController {
    
    private let disposeBag = DisposeBag()

    (...)
    
    private func Rx() {
        let sequence = Observable.just("Hello")
        
        sequence.subscribe { event in
            print(event)
        }.dispose() // the subscription is disposed immediately
        
        sequence.subscribe(onNext: { value in
            print(value)
        }).disposed(by: disposeBag) // the subscription is disposed when the view controller is deinitialised
    }
    
}
{% endhighlight %}

## Side effects

If you want to do an action based on an emitted event, you can use

{% highlight swift %}
func do(onNext: ((T) throws -> Void)?, onError: ((Error) throws -> Void)?, onCompleted: (() throws -> Void)?, onSubscribe: (() -> Void)?, onSubscribed: (() -> Void)?, onDispose: (() -> Void)?) -> Observable<T>
{% endhighlight %}

It is quite similar to the subscription but it do not register an observer, it means, that a cold `Observable` will not emit any events when you use the method until you register an observer.

## Subjects

While an `Observable` has an output only (it can only emit values), an subject is an `Observable` that can add new values to the squence dynamically. RxSwift has four kinds of subjects

* `BehaviourSubject` - the subject will emit to the observers the most recent element (or an error if it is the last state of the subject) and any other elements emitted later on
* `PublishSubject` - the subject will emit to the observers every element emitted after the observer subscribed
* `ReplaySubject` - the subject will emit to the observers all elements which have been emitted before and after the observer subscribed
* `Variable` - in fact, this is "sugar" for `BehaviourSubject`, this name is more understandable, at least for iOS developers

Let's see how it works in action.

### BehaviourSubject (Variable)

{% highlight swift %}
let subject = BehaviorSubject(value: "One")
subject.onNext("Two")

subject.subscribe(onNext: { value in
    print(value)
}).disposed(by: disposeBag)

subject.onNext("Three")
subject.onNext("Four")
{% endhighlight %}

Output:

```
Two
Three
Four
```

### PublishSubject

{% highlight swift %}
let subject = PublishSubject<String>()
subject.onNext("One")
subject.onNext("Two")

subject.subscribe(onNext: { value in
    print(value)
}).disposed(by: disposeBag)

subject.onNext("Three")
subject.onNext("Four")
{% endhighlight %}

Output:

```
Three
Four
```

### ReplaySubject

{% highlight swift %}
let subject = ReplaySubject<String>.create(bufferSize: 4)
subject.onNext("One")
subject.onNext("Two")

subject.subscribe(onNext: { value in
    print(value)
}).disposed(by: disposeBag)

subject.onNext("Three")
subject.onNext("Four")
{% endhighlight %}

Output:

```
One
Two
Three
Four
```

## Traits

[traits](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Traits.md)

## Operators

[full list](http://reactivex.io/documentation/operators.html)

## Transforming

### map

### flatMap

### scan

### buffer

## Filtering

### debounce (throttle)

### distinctUntilChanged

### filter

### first / last

## Combining

### combineLatest

### merge

### zip

## References

[https://github.com/ReactiveX/RxSwift](https://github.com/ReactiveX/RxSwift)  
[https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md)  
[http://reactivex.io/documentation/subject.html](http://reactivex.io/documentation/subject.html)  
[https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Traits.md](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Traits.md)  
[http://reactivex.io/documentation/operators.html](http://reactivex.io/documentation/operators.html)