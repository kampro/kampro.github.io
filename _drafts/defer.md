---
title: "Defer in Swift"
category: swift
---
Continuing an explanation of topics which were touched during my meeting with an iOS developer (the story is contained in [the preface of the post about `isHidden` and `alpha`]({{ site.baseurl }}{% post_url 2019-02-13-ishidden-vs-alpha %})), I would like to explain a couple of details regarding `defer` in Swift.  
`defer` has been introduced in Swift 2.0, thus in 2015, and it is a quite comfortable solution which eliminates the possibility of an error occurrence. In the post I use a `guard` construction, you can read about what it is in my [previous post]({{ site.baseurl }}{% post_url 2019-02-20-guard %}).

### How does `defer` work?

The statement allows executing instructions inside `defer` when our program leaves a scope where `defer` is. In the simple words, instructions from `defer` will be executed when a program goes out of a block (a zone between `{``}`). `defer` is used very often in companion with `guard`. Let's go to the example.  
For the purposes of the post, I made a simple class which is supposed to simulate communication with a remote server.

{% highlight swift %}
class Connection {
    
    private(set) var isConnected = false
    private(set) var hasReadPermission = false
    
    func connect() {
        print("connect")
        
        isConnected = true
        hasReadPermission = Bool.random()
    }
    
    func disconnect() {
        print("disconnect")
        
        isConnected = false
        hasReadPermission = false
    }
    
    func countFiles() -> Int {
        return Int.random(in: 0...10)
    }
    
}
{% endhighlight %}

If we would like to use the class, without a usage of `defer`, we could write something like this

{% highlight swift %}
class ViewController: UIViewController {

    (...)

    private func countRemoteFiles() {
        let connection = Connection()
        connection.connect()
        
        guard connection.hasReadPermission else {
            connection.disconnect()
            return
        }
        
        print(connection.countFiles())
        
        connection.disconnect()
    }

}
{% endhighlight %}

In this example we have a couple of ugly things, first of all, we wrote an instruction that disconnects twice, secondly, we open the connection at the beginning of the method and we close it at the end, it means that we can forget about the disconnecting instruction.    
Using `defer` we can keep this code more readable and better organised, `countRemoteFiles` might look like this

{% highlight swift %}
private func countRemoteFiles() {
    let connection = Connection()
    connection.connect()

    defer {
        connection.disconnect()
    }
    
    guard connection.hasReadPermission else {
        return
    }
    
    print(connection.countFiles())
}
{% endhighlight %}

Now, we keep `connection.disconnect()` just after `connection.connect()` and the instruction is written only once. This is a very simple example but let's imagine that we have much more checks and more ways where the method can return.

### Errors inside a `defer`'s block

You have to bear in mind, that all errors (produced by instructions that `throws` or by `throw` statement) inside `defer` will exist inside and won't be passed outside of the `defer` block. If we added a simple method to `Connection` class

{% highlight swift %}
func produceError() throws {
    throw SimulatedError.genericError("Error inside of defer")
}
{% endhighlight %}

and called it in `defer`

{% highlight swift %}
defer {
    try connection.produceError()
}
{% endhighlight %}

we would have to handle the error inside of `defer`

{% highlight swift %}
defer {
    do {
        try connection.produceError()
    } catch {
        print(error.localizedDescription)
    }
}

// OR

defer {
    try? connection.produceError()
}
{% endhighlight %}

we are not able to pass it over, for example

{% highlight swift %}
do {
    (...)

    defer {
        try connection.produceError() // Error! `try` isn't inside `do`
    }

    (...)
} catch {
    print(error.localizedDescription)
}
{% endhighlight %}


### Multiple `defer` statements

Can we have multiple `defer` statements? Yes, we can, but we have to know how more than one statement works exactly. If we wanted to defer several instructions, we would write them in the same block belonging to one `defer`, so what is the purpose of multiple `defer`s? The answer is: if there is a more than one `defer`, they are executed **in the reverse order** of an occurrence.

> If multiple defer statements appear in the same scope, the order they appear is the **reverse** of the order they are executed.

source: [https://docs.swift.org/swift-book/ReferenceManual/Statements.html#grammar_defer-statement](https://docs.swift.org/swift-book/ReferenceManual/Statements.html#grammar_defer-statement)

If you thought about it, you would admit that it is a logical mechanism. Let's suppose we have the following situation

{% highlight swift %}
do {
    let connection = Connection()
    connection.connect()
    
    defer {
        connection.disconnect()
        print("Disconnect") // 2
    }
    
    let file = try connection.openFile(with: url)
    
    defer {
        file.closeFile()
        print("Close file") // 1
    }
    
    try file.write(content: "This is a file content")
} catch {
    print(error.localizedDescription) // 3
}
{% endhighlight %}

as you can see I close the connection and the file just after opening but these closings are inside of `defer`, it keeps the code less error-prone (there is always closing just after opening) and thanks to `defer`, closings will be called in the proper moment. This is logical that if we opened the connection and then opened the file, we would close the file firstly and then close the connection after we do all the operations on the file, this is how `defer` would work. If we run the code, we would get the following output

```
Close file
Disconnect
Couldn't write to the file
```

Firstly, our code throws an exception, then the code leaves its scope so `defer`s are fired up, the file and connection are closed respectively and then we catch the exception lastly.

### Nested `defer` statements

If you place `defer` inside of other `defer` the same rules apply, it means that the inner `defer` is limited by the outer one, let's go to the example.

{% highlight swift %}
do {
    let connection = Connection()
    connection.connect()
    
    defer { // A
        connection.disconnect()
        print("Disconnect")
    }
    
    let file = try connection.openFile(with: url)
    
    defer { // B
        defer { // C
            defer { // D
                print("Nested defer")
            }
        }
    }
    
    defer { // E
        file.closeFile()
        print("Close file")
    }
    
    try file.write(content: "This is a file content")
} catch {
    print(error.localizedDescription)
}
{% endhighlight %}

If we run the code we should get

```
Close file
Nested defer
Disconnect
Couldn't write to the file
```

Going from the last `defer` to the first one, the code executes `D`, `B`, *and then, everything that is inside of `B`. In `B` there is another `defer` (`C`) which will be executed when the execution leaves the scope of `B`, this happens immediately, because `C` is the only instruction in `B`. The same situation is with `D` inside of `C`.*  
In the summary, the `defer`s are executed in the following order `E - B-C-D - A`. We can put some `print`s to see that we are right.

{% highlight swift %}
(...)

defer { // B
    defer { // C
        defer { // D
            print("Nested defer")

            print("D")
        }

        print("C")
    }

    print("B")
}

(...)
{% endhighlight %}

The output is

```
Close file
B
C
Nested defer
D
Disconnect
Couldn't write to the file
```

<!-- linearlity -->