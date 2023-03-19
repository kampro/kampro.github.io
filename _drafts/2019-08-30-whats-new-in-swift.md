---
title: "What's new in Swift 5 and 5.1"
categories: [swift]
---
Apple showed the new things included in Swift 5 and 5.1 language during the last WWDC, the changes are mainly targeted towards the stability, portability and "maturity" of the language. I described a vast majority of changes, the complete list can be found on [https://swift.org/blog/](https://swift.org/blog/)

Table of contents:  
[Shared Swift Runtime](#head-1)  
[ABI (application binary interface) and module stability](#head-2)  
[Better bridging between Swift and Objective-C](#head-3)  
[SourceKit is exposed through LSP](#head-4)  
[Changed encoding from UTF-16 to UTF-8](#head-5)  
[Implicit return from single expressions](#head-6)  
[Synthesized default values for the memberwise initialiser](#head-7)  
[SIMD Vectors API](#head-8)  
[New design for string interpolation](#head-9)  
[Opaque result types](#head-10)  
[Property wrapper types](#head-11)  
["<" compiler condition](#head-12)  
[Identity key paths](#head-13)  
[Removing customization points from collections](#head-14)  
[Flatten nested optionals from "try?"](#head-15)  
[Ranges are codable](#head-16)  
[@dynamicCallable](#head-17)  
["Never" conforms to "Equatable" and "Hashable"](#head-18)  
[Standard Library gets "Result" type](#head-19)  
[@unknown default](#head-20)  
["compactMapValues" for dictionaries](#head-21)  
["Sequence.SubSequence" has been removed](#head-22)  
[Properties for Unicode scalars](#head-23)  
[Properties for characters](#head-24)  
[isMultiple(of:)](#head-25)  
[Enhanced String literals delimiters](#head-26)  

## Shared Swift Runtime {#head-1}

Swift Runtime will be included in operating systems and shared for applications, it means that applications' size will be reduced because there will be no need to include Swift Runtime to the package, also a launch time overhead will be decreased to 0%.

## ABI (application binary interface) and module stability {#head-2}

It means that the way how binaries talk to each other will be normalised, in practice applications, libraries, etc. which talk to each other don't need to be compiled with the same compiler, for example: before that change our application compiled with Swift 4 compiler could use libraries compiled with Swift 4 compiler only. Now, applications and libraries compiled with Swift 5 compiler will be able to use (and can be used by) any other library compiled with Swift >=5 compiler.

## Better bridging between Swift and Objective-C {#head-3}

Just performance improvements under the hood.

## SourceKit is exposed through LSP {#head-4}

SourceKit, a service responsible for autocompletion, colors and similar code processing, will be using LSP, it means that external editors would be able to use that native service to support Swift syntax.

## Changed encoding from UTF-16 to UTF-8 {#head-5}

Apple decided to ditch UTF-16 in favor of UTF-8, they went to the point where stated that unification is a much better solution. Interoperability between Swift and Objective-C, design of ABI, text processing, etc. is much more efficient with UTF-8.

It works in a similar way like Swift's `Array`'s `map`, it transforms a value emitted by an `Observable` to the new value.

## Implicit return from single expressions {#head-6}

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

## Synthesized default values for the memberwise initialiser {#head-7}

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

## SIMD Vectors API {#head-8}

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

## New design for string interpolation {#head-9}

Format strings are insecure and uncomfortable to use, so we got a new way of string interpolation. Since Swift 5 we have had the ability to customize string interpolation in a modern way, we can extend `DefaultStringInterpolation` or implement `ExpressibleByStringInterpolation`.

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

The `ExpressibleByStringInterpolation` approach gives us more control and should be used when your case is more complex.

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

## Opaque result types {#head-10}

Let's see the problem if we had this code

{% highlight swift %}
protocol Shape { ... }
struct Square: Shape { ... }

struct Union<A: Shape, B: Shape>: Shape { ... }
struct Transformed<S: Shape >: Shape { ... }
{% endhighlight %}

we shouldn't create another type like this one

{% highlight swift %}
struct EightPointedStar {
    var shape: Shape {
        return Union(Square(), Transformed(Square(), by: .fortyFiveDegrees))
    }
}
{% endhighlight %}

Why? The answer is: because returning a protocol type in such case has a few limitations. It loses the type identity, it doesn't work well with the generic system: we can't use `==` operator because the compiler doesn't know concrete types, returned type can't have any associated types and requirements that involve `Self`. It disables optimisations also.  
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

## Property wrapper types {#head-11}

Property wrapper types let us reuse code in a better manner. This solution helps to eliminate repeating a code of, for example, custom property accessors. Property wrappers describe a pattern for accessing properties, let's look at an old-fashioned code presented during WWDC

{% highlight swift %}
static var usesTouchID: Bool {
    get {
        return UserDefaults.standard.bool(forKey: "USES_TOUCH_ID")
    }
    set {
        UserDefaults.standard.set(newValue, forKey: "USES_TOUCH_ID")
    }
}

static var isLoggedIn: Bool {
    get {
        return UserDefaults.standard.bool(forKey: "LOGGED_IN")
    }
    set {
        UserDefaults.standard.set(newValue, forKey: "LOGGED_IN")
    }
}
{% endhighlight %}

As we can see there is a duplicated code, this is a bad practice. To solve this problem we can use a new `@propertyWrapper` annotation, later on, we will be able to mark our properties with a just created custom annotation to apply the accessors. For example

{% highlight swift %}
@propertyWrapper
struct UserDefault<T> {
    let key: String
    let defaultValue: T

    init(_ key: String, defaultValue: T) {
        // ...
        UserDefaults.standard.register(defaults: [key: defaultValue])
    }

    var value: T {
        get {
            return UserDefaults.standard.object(forKey: key) as? T ?? defaultValue
        }
        set {
            UserDefaults.standard.set(newValue, forKey: key)
        }
    }
}
{% endhighlight %}

Thanks to `@propertyWrapper` we got `@UserDefault(...)` annotation, now we can change our accessors of UserDefault's values to this code

{% highlight swift %}
@UserDefault("USES_TOUCH_ID", defaultValue: false)
static var usesTouchID: Bool

@UserDefault("LOGGED_IN", defaultValue: false)
static var isLoggedIn: Bool
{% endhighlight %}

## "<" compiler condition {#head-12}

Before Swift 5 we could use `>=` only, when we wanted to mark code for a specific language or compiler version

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

## Identity key paths {#head-13}

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

## Removing customization points from collections {#head-14}

Before Swift 5 we could overwrite the default behavior of collections, for example writing an extension for `Array` which gives a logic of `first` property, now this is impossible.

## Flatten nested optionals from "try?" {#head-15}

Using `try?` with a method, which can throw an error, of an optional object, produced a double wrapped result

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

## Ranges are codable {#head-16}

Rages got conformance to `Codable`, it means that they can be easily used with `JSONEncoder` and `JSONDecoder`.

## @dynamicCallable {#head-17}

Swift 5 adds dynamic callable types to better interoperate with languages like Python. To support that feature we need to mark our type with `@dynamicCallable` annotation and we can implement two methods `func dynamicallyCall(withArguments: <#Arguments#>) -> <#R1#>` and `func dynamicallyCall(withKeywordArguments: <#KeywordArguments#>) -> <#R2#>`

{% highlight swift %}
@dynamicCallable
struct BothCallable {
    func dynamicallyCall(withArguments args: [Int]) -> Int {
        return args.count
    }

    func dynamicallyCall(withKeywordArguments args: KeyValuePairs<String, Int>) -> Int {
        return args.count
    }
}

let dc = BothCallable()
dc() // c3.dynamicallyCall(withArguments: [])
dc(1, 2) // c3.dynamicallyCall(withArguments: [1, 2])
dc(a: 1, 2) // c3.dynamicallyCall(withKeywordArguments: ["a": 1, "": 2])
{% endhighlight %}

## "Never" conforms to "Equatable" and "Hashable" {#head-18}

`Never` in Swift 5 now can be used as a dictionary key.

## Standard Library gets "Result" type {#head-19}

As Swift is improving itself in a server environment, it got `Result` type which is mainly used in asynchronous APIs. Let's see an example from Swift's documentation.

{% highlight swift %}
func dataTask(with url: URL, completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTask
{% endhighlight %}

The code above tells us that data, response or error from the completion handler can be null. Without `Result` we need to check each value.

{% highlight swift %}
URLSession.shared.dataTask(with: url) { (data, response, error) in
    guard error != nil else { self.handleError(error!) }
    
    guard let data = data, let response = response else { return // Impossible? }
    
    handleResponse(response, data: data)
}
{% endhighlight %}

It brought us to the situation where we are checking mutually exclusive cases, if something isn't an error, we shouldn't check its values. `Result` solves this problem and we can keep the code much nicer.

{% highlight swift %}
URLSession.shared.dataTask(with: url) { (result: Result<(response: URLResponse, data: Data), Error>) in // Type added for illustration purposes.
    switch result {
    case let .success(success):
        handleResponse(success.response, data: success.data)
    case let .error(error):
        handleError(error)
    }
}
{% endhighlight %}

## @unknown default {#head-20}

Swift 5 introduces a new annotation for `default` in `switch`, thanks to it the compiler will warn us if we are using `default` in our `switch` and the `switch` isn't exhaustive. For example, we have a code like below

{% highlight swift %}
enum Things {
    case car, bike, shoes
}

func process(_ thing: Things) {
    switch thing {
        case .shoes:
            print("footwear")
        default:
            print("vehicle")
    }
}
{% endhighlight %}

Now, when we added a new value to the enum

{% highlight swift %}
enum Things {
    case car, bike, shoes, computer
}
{% endhighlight %}

the `switch` will work incorrectly, it would print "vehicle" for `.computer`. We can mark `default` with `@unknown` and the code will still work but the compiler will warn us whenever the switch will be unexhaustive.

{% highlight swift %}
func process(_ thing: Things) {
    switch thing {
        case .shoes:
            print("footwear")
        @unknown default:
            print("vehicle")
    }
}
{% endhighlight %}

## "compactMapValues" for dictionaries {#head-21}

Without the new `compactMapValues` we need to use `mapValues`, `filter` and `reduce` to get the same result. For example, if we wanted to filter out key-value pairs which values can be stored as integers we need to write a code like this

{% highlight swift %}
let numbers = ["one": "1", "two": "two", "three": "3"]
let numberValues = numbers.mapValues(Int.init)
    .filter { $0.value != nil }
    .mapValues { $0! }
let reduceNumbers = numbers.reduce(into: [:]) { $0[$1.key] = Int($1.value) }
{% endhighlight %}

Now we can just write

{% highlight swift %}
let values = numbers.compactMapValues(Int.init)
{% endhighlight %}

## "Sequence.SubSequence" has been removed {#head-22}

Before Swift 5, operations which were returning modified versions of sequences, like `dropLast(_:)`, `prefix(_:)`, `sufix(_:)` etc., were returning `SubSequence` type. From Swift 5 the operations return a sequence of concrete type, for example `[Int]`.

## Properties for Unicode scalars {#head-23}

Swift 5 has brought a bunch of properties to Unicode scalars, for example, we can easily check if a scalar is an emoji, whitespace, alphabetic and much more.

{% highlight swift %}
let text = "\u{FB00} \u{0149}: the date is 01/09/2019"
var lettersCount = 0
var title = ""
text.unicodeScalars.forEach {
    lettersCount += $0.properties.isAlphabetic ? 1 : 0
    title.append($0.properties.titlecaseMapping)
}
// lettersCount: 11
// title: Ff ʼN: THE DATE IS 01/09/2019
{% endhighlight %}

## Properties for characters {#head-24}

Similarly to Unicode scalars, characters got a bunch of properties too. Instead of doing gimmicks we can just use a simple property.

{% highlight swift %}
let text = "2 is a number"
let someNumber = "\u{096B}" // ५

if text.first?.isNumber ?? false {
    print("true")
}

if let number = someNumber.first?.wholeNumberValue {
    print(number) // 5
}
{% endhighlight %}

## isMultiple(of:) {#head-25}

Now, `BinaryInteger` has a method which allows to easily check if one number is a multiple of another.

{% highlight swift %}
if 3528495923045.isMultiple(of: 705699184609) {
    print("true")
}
{% endhighlight %}

## Enhanced String literals delimiters {#head-26}

In Swift 4.2 we could use a standard or multi-line String literal, in both cases, we need to use a backslash `\` if we want to escape special characters like a double quote or backslash itself.

{% highlight swift %}
let standard = "You need to use a backslash \\ to escape characters. For example: \"The Great Gatsby\" is a very good book."

let multiline = """
    You need to use a backslash \\ to escape characters.
    For example: \"The Great Gatsby\" is a very good book.
    """
{% endhighlight %}

Swift 5 introduces *raw strings*, if we add a hash `#` symbol at the beginning and end of the string, symbols included in the string will be treated as normal characters and won't be interpreted. If we want to use string interpolation in the raw string, we need to use hash `#` symbol just after a backslash.

{% highlight swift %}
let book = #""The Great Gatsby""#
let text = #"""
    You need to use a backslash \ to escape characters.
    For example: \#(book) is a very good book.
    """#
{% endhighlight %}

What if you want to write `#` inside of a string? You should just embrace the string with a double hash `##`

{% highlight swift %}
let text = ##"Now we can write # symbol without escaping."##
{% endhighlight %}

The place where we can appreciate the greatest power of the delimiters is regular expressions.

## References

[https://developer.apple.com/videos/play/wwdc2019/402/](https://developer.apple.com/videos/play/wwdc2019/402/)  
[https://swift.org/blog/utf8-string](https://swift.org/blog/utf8-string)  
[https://developer.apple.com/documentation/swift/defaultstringinterpolation](https://developer.apple.com/documentation/swift/defaultstringinterpolation)
[https://docs.swift.org/swift-book/LanguageGuide/OpaqueTypes.html](https://docs.swift.org/swift-book/LanguageGuide/OpaqueTypes.html)  
[https://swift.org/blog/swift-5-released/](https://swift.org/blog/swift-5-released/)