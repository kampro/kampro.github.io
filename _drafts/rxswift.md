---
title: "RxSwift and RxCocoa basics"
categories: [swift, ios]
---
## What is Rx?

Rx stands for Reactive Extensions (aka ReactiveX), it is a set of tools which helps to write a code according to the reactive programming pattern. Using the reactive programming we can write our code in a more declarative way and using Rx library we don't have to think about threads and all the infrastructure.  
*RxSwift* is an implementation of Rx in Swift. *RxCocoa* is a library which adds Rx functionality to Cocoa classes and Rx types which can better cooperate with Cocoa. To start with *RxSwift* and *RxCocoa* just add the libraries ([project on GitHub](https://github.com/ReactiveX/RxSwift)) to your project and import them (`import ...`) in the file where you want to use Rx.

## Observable sequences

Almost everything in *RxSwift* is an observable sequence (`Observable`) and almost every Swift collection can be transformed to `Observable`.

{% highlight swift %}
let sequence = Observable.from([0, 1, 2, 3, 4, 5])
{% endhighlight %}

## *Hot* and *cold* observables

There are two main types of `Observable`s - hot and cold. A hot `Observable` emits events no matter if there is an object that subscribes to the `Observable`, an example of a hot `Observable` is `Variable` type. The `Observable` is usually stateful and computation resources are usually shared between all of the subscribed observers.  
A cold `Observable` doesn't emit events until an observer subscribes to it, is used i.e. for HTTP connections. The `Observable` is usually stateless and computation resources are usually per observer.

## Subscribing to Observables

It is natural that if something produces events, the other thing wants to listen to them. We can listen to events from `Observable`s using `subscribe(on: (Event<T>) -> Void)` method.

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

If you no longer use a subscription you need to dispose of it to avoid a memory leak. You can do it manually by calling `dispose()` on the subscription or by adding it to the `DisposeBag`, then, the subscription is disposed of automatically when `DisposeBag` is deinitialised.

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

It is quite similar to the subscription but it does not register an observer, it means, that a cold `Observable` will not emit any events when you use the method until you register an observer.

## Subjects

While an `Observable` has an output only (it can only emit values), a subject is an `Observable` that can add new values to the sequence dynamically. *RxSwift* has four kinds of subjects

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

Traits are wrapped `Observable`s, they introduce more contextual meaning and syntactical sugar for specific use cases, they are fully optional and you can do the same thing using raw `Observable`s (but with more effort). Some Traits have been introduced specifically for *RxCocoa* (i.e. `Driver`). You can always transform the Trait to the `Observable` using `asObservable()` method.

## RxSwift Traits

### Single

* Emits exactly one element, or an error.
* Doesn't share side effects.

{% highlight swift %}
let single = Single<String>.create { single in
    if true {
        single(.success("Value"))
    } else {
        single(.error(SimulatedError.genericError("Some error")))
    }
    
    return Disposables.create {
        print("Dispose trait's resources")
    }
}

single.subscribe { event in
    print(event)
}.disposed(by: disposeBag)
{% endhighlight %}

Output:

```
success("Value")
Dispose trait's resources
```

### Completable

* Emits zero elements.
* Emits a completion event, or an error.
* Doesn't share side effects.

{% highlight swift %}
let completable = Completable.create { completable in
    let didErrorOccur = true
    
    if didErrorOccur {
        completable(.error(SimulatedError.genericError("Some error")))
    } else {
        completable(.completed)
    }
    
    return Disposables.create {
        print("Dispose trait's resources")
    }
}

completable.subscribe { event in
    print(event)
}.disposed(by: disposeBag)
{% endhighlight %}

Output:

```
error(TestProject.SimulatedError.genericError("Some error"))
Dispose trait's resources
```

### Maybe

* Emits either a completed event, a single element or an error.
* Doesn't share side effects.

{% highlight swift %}
let maybe = Maybe<String>.create { maybe in
    let number = Int.random(in: 0...10)
    let isEven = number % 2 == 0
    
    if number == 0 {
        maybe(.error(SimulatedError.genericError("Number is 0")))
    } else if isEven {
        maybe(.success("Number is even"))
    } else {
        maybe(.completed)
    }
    
    return Disposables.create {
        print("Dispose trait's resources")
    }
}

maybe.subscribe { event in
    print(event)
}.disposed(by: disposeBag)
{% endhighlight %}

Output:

```
completed
Dispose trait's resources
```

## RxCocoa Traits

### Driver

`Driver` is mainly for driving the UI layer.

* Can't error out.
* Observe occurs on main scheduler.
* Shares side effects (`share(replay: 1, scope: .whileConnected)`).

{% highlight swift %}
let results = searchBar.rx.text.asDriver().debounce(0.3).flatMapLatest { queryString -> SharedSequence<DriverSharingStrategy, [String]> in
    let query = queryString ?? ""
    return self.fetch(query: query).asDriver(onErrorJustReturn: [])
}

results.map { "\($0.count)" }.drive(countLabel.rx.text).disposed(by: disposeBag)

results.map { results -> String in
    var resultsString = ""
    
    for item in results {
        resultsString += "\(item)\n"
    }
    
    return resultsString
}.drive(resultsTextView.rx.text).disposed(by: disposeBag)
{% endhighlight %}

Any observable sequence can be converted to the Driver trait, as long as it satisfies 3 properties:

* Can't error out.
* Observe on main scheduler.
* Sharing side effects (`share(replay: 1, scope: .whileConnected)`).

`asDriver(onErrorJustReturn: T)` does something like this internally

{% highlight swift %}
let safeSequence = sequence.observeOn(MainScheduler.instance).catchErrorJustReturn(onErrorJustReturn).share(replay: 1, scope: .whileConnected)
return Driver(raw: safeSequence)
{% endhighlight %}

### Signal

`Signal` is quite similar to the `Driver` but it does not replay the last event when an observer subscribed.

* Can't error out.
* Delivers events on Main Scheduler.
* Shares computational resources (share(scope: .whileConnected)).
* Does **not** replay elements on subscription.

### ControlProperty

`ControlProperty`, as the name suggests, represents a property of a UI element but in the Rx manner. The sequence emits events when an observer subscribes (the initial value) or a **user** changes the value of the property. It doesn't emit events when the property is changed programmatically.

* It never fails
* `share(replay: 1) behavior`
  * It's stateful, upon subscription (calling subscribe) last element is immediately replayed if it was produced
* It will `Complete` sequence on control being deallocated
* It never errors out
* It delivers events on `MainScheduler.instance`

Those properties are automatically delivered by *RxCocoa*, i.e. `textField.rx.text`.

### ControlEvent

This Trait is a sequence that represents an event on a UI element.

* It never fails
* It won't send any initial value on subscription
* It will `Complete` sequence on control being deallocated
* It never errors out
* It delivers events on `MainScheduler.instance`

For example, it can be `tap` event of `UIButton` - `button.rx.tap`.

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