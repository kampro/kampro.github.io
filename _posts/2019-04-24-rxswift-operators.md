---
title: "RxSwift operators"
categories: [swift, ios]
---
## Operators

Operators enable making operations on sequences and events, they are groupped in categories according to their functionaly. You can see [the full list of Rx operators](http://reactivex.io/documentation/operators.html). I described the most important ones, in my opinion.

## Transforming

### map

It works in the similar way like Swift's `Array`'s `map`, it transforms a value emitted by an `Observable` to the new value.

![map](/assets/rx-map.png)

{% highlight swift %}
let sequence = Observable.of(1, 2, 3, 4, 5)
sequence.map { $0 * 10 }.subscribe(onNext: { value in
    print(value)
}).disposed(by: disposeBag)
{% endhighlight %}

Output:

```
10
20
30
40
50
```

### flatMap

It transform values emitted by an `Observable` and wraps them into the new `Observable`.

![flatmap](/assets/rx-flatmap.png)

The difference between `map` and `flatMap` is `map`'s transformation returns single value and `flatMap`'s transformation returns a sequence.

{% highlight swift %}
let sequenceA = Observable.of(1, 2, 3)
let sequenceB = Observable.of(10, 100)
sequenceA.flatMap { value in
    return sequenceB.map { $0 * value }    
}.subscribe(onNext: { value in
    print(value)
}).disposed(by: disposeBag)
{% endhighlight %}

Output:

```
10
100
20
200
30
300
```

### scan

It works like Swift's `reduce`, applies operation on the *accumulator* and the current item returned by the `Observable`, then the result is assigned to the *accumulator* for the next item.

![scan](/assets/rx-scan.png)

{% highlight swift %}
let sequence = Observable.of(1, 2, 3, 4, 5)
sequence.scan(1) { $0 * $1 }.subscribe(onNext: { value in
    print(value)
}).disposed(by: disposeBag)
{% endhighlight %}

Output:

```
1
2
6
24
120
```

### buffer

It gathers emitted values into the buffer and then emits the bundle with the values when the buffer is full or the `Observable` terminates. If the `Observable` emits an error, then `buffer` emits it immediately without emission of the incomplete buffer.

![buffer](/assets/rx-buffer.png)

{% highlight swift %}
let sequence = Observable.of(1, 2, 3, 4, 5)
sequence.buffer(timeSpan: 10, count: 2, scheduler: MainScheduler.instance).subscribe(onNext: { value in
    print(value)
}).disposed(by: disposeBag)
{% endhighlight %}

Output:

```
[1, 2]
[3, 4]
[5]
```

## Filtering

### debounce / throttle

The difference between `debounce` and `throttle` is very slight.  
`Debounce` emits an event after a specified time from the moment when an `Observable` stops an emission.  
`Throttle` emits an event at most once a specified time.

![debounce](/assets/rx-debounce.png)

{% highlight swift %}
let sequence = Observable<Int>.timer(1, period: 0.3, scheduler: MainScheduler.instance).take(15)
        
sequence.subscribe(onNext: { value in
    print("•", terminator: " ")
}).disposed(by: disposeBag)

sequence.debounce(1, scheduler: MainScheduler.instance).subscribe(onNext: { value in
    print("debounce", terminator: " ")
}).disposed(by: disposeBag)

sequence.throttle(1, scheduler: MainScheduler.instance).subscribe(onNext: { value in
    print("throttle", terminator: " ")
}).disposed(by: disposeBag)
{% endhighlight %}

Output:

```
• throttle • • • throttle • • • throttle • • • • throttle • • • throttle • debounce throttle
```

### distinctUntilChanged

It emits an event only then, when event's value is different than the value emitted previously.

![distinct](/assets/rx-distinct.png)

{% highlight swift %}
let sequence = Observable.of(1, 2, 2, 2, 3, 3, 1, 1, 1, 4, 2, 5).distinctUntilChanged()
sequence.subscribe(onNext: { value in
    print(value)
}).disposed(by: disposeBag)
{% endhighlight %}

Output:

```
1
2
3
1
4
2
5
```

### filter

It emits only those events that comply a predicate.

![filter](/assets/rx-filter.png)

{% highlight swift %}
let sequence = Observable.of(1, 2, 3, 4, 5)
sequence.filter { $0 > 3 }.subscribe(onNext: { value in
    print(value)
}).disposed(by: disposeBag)
{% endhighlight %}

Output:

```
4
5
```

### take / takeLast

It emits only the first or last *n* items of the sequence.

![take](/assets/rx-take.png)

and

![takelast](/assets/rx-takelast.png)

{% highlight swift %}
let sequence = Observable.of(1, 2, 3, 4, 5)

sequence.take(1).subscribe(onNext: { value in
    print(value, terminator: " ")
}).disposed(by: disposeBag)

print()

sequence.takeLast(2).subscribe(onNext: { value in
    print(value, terminator: " ")
}).disposed(by: disposeBag)
{% endhighlight %}

Output:

```
1
4 5
```

## Combining

### combineLatest

This operator combines the latest results of two observables, applies a function on them and emits a new event every time when at least one observable emits an event.

![take](/assets/rx-combinelatest.png)

{% highlight swift %}
let sequenceA = Observable.of(1, 2, 3, 4, 5)
let sequenceB = Observable.of("A", "B", "C", "D", "E")

let combined = Observable.combineLatest(sequenceA, sequenceB) {
    "\($0)\($1)"
}

combined.subscribe(onNext: {
    print($0)
}).disposed(by: disposeBag)
{% endhighlight %}

Output:

```
1A
2A
2B
3B
3C
4C
4D
5D
5E
```

### merge

`merge` combines multiple observables into one and emits events from all of them.

![zip](/assets/rx-merge.png)

{% highlight swift %}
let sequenceA = Observable.of(1, 2, 3, 4, 5)
let sequenceB = Observable.of(6, 7, 8, 9, 10)

let merged = Observable.merge(sequenceA, sequenceB)

merged.subscribe(onNext: {
    print($0)
}).disposed(by: disposeBag)
{% endhighlight %}

Output:

```
1
6
2
7
3
8
4
9
5
10
```

### zip

It combines respective results of multiple observables, applies a function on them and emits a new event every time when all of the observables emit the next event.

![zip](/assets/rx-zip.png)

{% highlight swift %}
let sequenceA = Observable.of(1, 2, 3, 4, 5)
let sequenceB = Observable.of("A", "B", "C")

let zipped = Observable.zip(sequenceA, sequenceB) {
    "\($0)\($1)"
}

zipped.subscribe(onNext: {
    print($0)
}).disposed(by: disposeBag)
{% endhighlight %}

Output:

```
1A
2B
3C
```

## References

[http://reactivex.io/documentation/operators.html](http://reactivex.io/documentation/operators.html)