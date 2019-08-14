---
title: "What's new in Swift 5/5.1"
categories: [swift]
---
Apple showed the new things included in Swift 5 and 5.1 language during the last WWDC, the changes are mainly targeted towards the stability, portability and "maturity" of the language.

## Shared Swift Runtime

Swift Runtime will be included in operating systems and shared for applications, it means that applications' size will be reduced, because there will be no need to include Swift Runtime to the package, also a launch time overhead will be decreased to 0%.

## ABI (application binary interface) and module stability

It means that a way how binaries talk to each other will be normalised, in practice applications, libraries etc. which talk to each other don't need to be compiled with the same compiler, for example: before that change our application compiled with Swift 4 compiler could use libraries compiled with Swift 4 compiler only. Now, applications and libraries compiled with Swift 5 compiler will be able to use (and can be used by) any other library compiled with Swift >=5 compiler.

## Better bridging between Swift and Objective-C

Just performance improvements under the hood.

## SourceKit is exposed through LSP

SourceKit, a service responsible for autocompletion, colors and similar code processing, will be using LSP, it means that external editors would be able to use that native service to support Swift.

## Changed encoding from UTF-16 to UTF-8

Apple decided to ditch UTF-16 in favor of UTF-8, they went to the point where stated that an unification is a much better solution. An interoperability between Swift and Objective-C, design of ABI, text processing etc. is much more efficient with UTF-8.

It works in the similar way like Swift's `Array`'s `map`, it transforms a value emitted by an `Observable` to the new value.

## Implicit return from single expressions

Instead of writing this

{% highlight swift %}
struct Rectangle {
    var width = 0.0, height = 0.0
    var area: Double {
        return width * height
    }
}
{% endhighlight %}

we can write

{% highlight swift %}
struct Rectangle {
    var width = 0.0, height = 0.0
    var area: Double { width * height }
}
{% endhighlight %}

## Synthesized default values for the memberwise initializer

Let's assume that we have this code

{% highlight swift %}
struct Rectangle {
    var width = 1.0
    var height = 1.0
}

let rect = Rectangle()
let rect2 = Rectangle(width: 2.0, height: 4.0)
{% endhighlight %}

in Swift older than 5 we couldn't do this &#x274c;

{% highlight swift %}
let rect3 = Rectangle(with: 2.0)
{% endhighlight %}

but now we can &#x2705;

{% highlight swift %}
let rect3 = Rectangle(with: 2.0)
// or even
let rect4 = Rectangle(height: 2.0)
{% endhighlight %}

## References

[https://developer.apple.com/videos/play/wwdc2019/402/](https://developer.apple.com/videos/play/wwdc2019/402/)  
[https://swift.org/blog/utf8-string](https://swift.org/blog/utf8-string)