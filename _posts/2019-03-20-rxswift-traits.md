---
title: "RxSwift and RxCocoa Traits"
categories: [swift, ios]
---
Being in the flow of `Rx` topic, there is time to the next term from this world.

## Traits

Traits are wrapped `Observable`s, they introduce more contextual meaning and syntactical sugar for specific use cases, they are fully optional and you can do the same thing using raw `Observable`s (but with more effort). Some Traits have been introduced specifically for *RxCocoa* (i.e. `Driver`). You can always transform the Trait to the `Observable` using `asObservable()` method.  
If you are looking for fundamentals of *RxSwift*, you can find them in my [previous post]({{ site.baseurl }}{% post_url 2019-03-13-rxswift-basics %}).

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

## References

[https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Traits.md](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Traits.md)  