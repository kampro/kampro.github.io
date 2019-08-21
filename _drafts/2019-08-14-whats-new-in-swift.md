---
title: "What's new in Swift 5 and 5.1"
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

SourceKit, a service responsible for autocompletion, colors and similar code processing, will be using LSP, it means that external editors would be able to use that native service to support Swift syntax.

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

## Synthesized default values for the memberwise initialiser

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

## SIMD Vectors API

Swift 5 brings new vector types and improves vector programming (it can be connected with the fact that Apple pushes towards AI lately).  
New matrix types:

{% highlight swift %}
SIMD2<T>, SIMD3<T>, SIMD4<T>, SIMD8<T>, SIMD16<T>, SIMD32<T>, SIMD64<T>
{% endhighlight %}

New operators:

{% highlight swift %}
// Initialize from array literals
let x: SIMD4<Int> = [1,2,3,4]
let y: SIMD4<Int> = [3,2,1,0]

// Pointwise equality, inequality, and ordered comparisons
let gr = x .> y
// gr : SIMDMask<SIMD4<Int>>(false, false, true, true)

// Boolean operations on SIMDMasks
let lteq = .!gr
// lteq : SIMDMask<SIMD4<Int>>(true, true, false, false)
{% endhighlight %}

## New design for string interpolation

Format strings are inscure and uncomfortable to use, so we got a new way of string interpolation. Since Swift 5 we have had the ability to customize string interpolation in a modern way, we can extend `DefaultStringInterpolation` or implement `ExpressibleByStringInterpolation`.

Using `DefaultStringInterpolation`

{% highlight swift %}
struct User {
    let name: String
}

extension DefaultStringInterpolation {
    fileprivate mutating func appendInterpolation(_ value: User, uppercased: Bool = false) {
        let username = uppercased ? value.name.uppercased() : value.name
        
        appendInterpolation(username)
    }
}

let user = User(name: "Steve")

print("Username is \(user, uppercased: true)")
// Username is STEVE
{% endhighlight %}

The `ExpressibleByStringInterpolation` approach gives us more control and should he used when your case is more complex.

{% highlight swift %}
struct Article {
    let text: String
}

extension Article: ExpressibleByStringInterpolation {
    init(stringLiteral value: String) {
        self.init(text: value)
    }

    init(stringInterpolation: StringInterpolation) {
        self.init(text: stringInterpolation.result)
    }

    struct StringInterpolation: StringInterpolationProtocol {
        var result: String = ""

        init(literalCapacity: Int, interpolationCount: Int) {
            result.reserveCapacity(literalCapacity)
        }

        mutating func appendLiteral(_ literal: String) {
            result.append(literal)
        }

        mutating func appendInterpolation<T>(_ value: T) where T: CustomStringConvertible {
            result.append("\u{1F4AC}\u{201C}\(value)\u{201D}")
        }
    }
}

let quote = "These are very smart words"
let article: Article = "This is a very famous article and I would like to quote some words:\n\(quote)"
print(article.text)
// This is a very famous article and I would like to quote some words:
// 💬“These are very smart words”
{% endhighlight %}

This is how SwiftUI's `Text` works, thanks to that we can easily localise our texts which take arguments.

## Opaque result types

Let's see the problem, if we had this code

{% highlight swift %}
protocol Shape { ... }
struct Square: Shape { ... }

struct Union<A: Shape, B: Shape>: Shape { ... }
struct Transformed<S: Shape >: Shape { ... }
{% endhighlight %}

we shouldn't create an another type like this one

{% highlight swift %}
struct EightPointedStar {
    var shape: Shape {
        return Union(Square(), Transformed(Square(), by: .fortyFiveDegrees))
    }
}
{% endhighlight %}

Why? The answer is: because returning a protocol type in such case has a few limitations. It looses the type identity, it doesn't work well with the generic system: we can't use `==` operator because the compiler doesn't know concrete types, returned type can't have any associated types and requirements that involve `Self`. It disables optimisations also.  
OK, so we could do this

{% highlight swift %}
struct EightPointedStar {
    var shape: Union<Square, Transformed<Square>> {
        return Union(Square(), Transformed(Square(), by: .fortyFiveDegrees))
    }
}
{% endhighlight %}

But... now we are exposing implementation details. This is the moment when an opaque result type helps.

{% highlight swift %}
struct EightPointedStar {
    var shape: some Shape {
        return Union(Square(), Transformed(Square(), by: .fortyFiveDegrees))
    }
}
{% endhighlight %}

It returns the same, specific type but hides implementation details. We can think about this as a reverse of generics. The generic system chooses a proper return type based on the passed values and the values can be of any type that conforms to the constraints, whereas the opaque result types system chooses a proper return type based on the implementation without exposing the concrete type to the caller. It also implies that a function which returns `some` type can't return different types according to the logic. For example, it won't work &#x274c;

{% highlight swift %}
struct EightPointedStar {
    var shape: some Shape {
        if someBool {
            return Union(Square(), Transformed(Square(), by: .fortyFiveDegrees))
        } else {
            return Transformed(Square(), by: .twentyDegrees)
        }
    }
}
{% endhighlight %}

## Property wrapper types

## `<` compiler condition

Before Swift 5 we could use `>=` only, when we wanted to mark a code for a specific language or compiler version

{% highlight swift %}
#if !swift(>=5)
    // code
#else
    // code
#endif

#if !compiler(>=5)
    // code
#else
    // code
#endif
{% endhighlight %}

Now we can use `<` operator

{% highlight swift %}
#if swift(<5)
    // code
#else
    // code
#endif

#if compiler(<5)
    // code
#else
    // code
#endif
{% endhighlight %}

## Identity key paths

Every value in Swift gets `.self` property, which refers to the value, for example

{% highlight swift %}
class User {
    let name: String
    let password: String

    init(name: String, password: String) {
        self.name = name
        self.password = password
    }
}

var user = User(name: "John", password: "secret")
user.self = User(name: "Steve", password: "top secret")
{% endhighlight %}

now, in Swift 5, we can access the value using a key path

{% highlight swift %}
// ...
var user = User(name: "John", password: "secret")
user[keyPath: \.self] = User(name: "Steve", password: "top secret")
{% endhighlight %}

## Removing customization points from collections

Before Swift 5 we could overwrite the default behaviour of collections, for example writing an extension for `Array` which gives a logic of `first` property, now this is impossible.

## Flatten nested optionals from `try?`

Using `try?` with a method, which can throw an error, of an optional object produced a double wrapped result

{% highlight swift %}
class MyClass {
    func processEvenNumber(_ number: Int) throws -> Int {
        if number % 2 == 0 {
            return number
        } else {
            throw //...
        }
    }
}

let obj: MyClass? = MyClass()
let result = try? obj?.processEvenNumber(2)
// result: Int??
{% endhighlight %}

in Swift 5 the result will be flattened to `Int?`

## References

[https://developer.apple.com/videos/play/wwdc2019/402/](https://developer.apple.com/videos/play/wwdc2019/402/)  
[https://swift.org/blog/utf8-string](https://swift.org/blog/utf8-string)  
[https://developer.apple.com/documentation/swift/defaultstringinterpolation](https://developer.apple.com/documentation/swift/defaultstringinterpolation)
[https://docs.swift.org/swift-book/LanguageGuide/OpaqueTypes.html](https://docs.swift.org/swift-book/LanguageGuide/OpaqueTypes.html)  
[https://swift.org/blog/swift-5-released/](https://swift.org/blog/swift-5-released/)