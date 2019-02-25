---
title: "Defer in Swift"
category: swift
---
Continuing an explanation of topics which were touched during my meeting with an iOS developer (the story is contained in [the preface of the post about `isHidden` and `alpha`]({{ site.baseurl }}{% post_url 2019-02-13-ishidden-vs-alpha %})), I would like to explain a couple of details regarding `defer` in Swift.  
`defer` has been introduced in Swift 2.0, thus in 2015, and it is a quite comfortable solution which eliminates the possibility of an error occurrence. In the post I use a `guard` construction, you can read about what it is in my [previous post]({{ site.baseurl }}{% post_url 2019-02-20-guard %}).

### How does `defer` work?

The statement allows to execute instructions inside `defer` when our program leaves a scope where `defer` is. In the simple words, instructions from `defer` will be executed when a program goes out of a block (a zone between `{``}`). `defer` is used very often in companion with `guard`. Let's go to the example.  
For the purposes of the post, I made a simple class which is supposed to simulate a communication with a remote server.

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

In this example we have a couple of ugly things, first of all, we wrote an instruction that disconnects twice, secondly, we open the connection at the begining of the method and we close it at the end, it means that we can forget about the diconnecting instruction.    
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
        try connection.produceError() // Error! We `try` isn't inside `do`
    }

    (...)
} catch {
    print(error.localizedDescription)
}
{% endhighlight %}


### Multiple `defer` statements

Can we have multiple `defer` statements? Yes, we can, but we have to know how a more than one statement works exactly. If we wanted to defer several instructions, we would write them in the same block belonging to one `defer`, so what is the purpose of multiple `defer`s? The answer is: if there is a more than one `defer`, they are executed **in the reverse order** of an occurrence.

> If multiple defer statements appear in the same scope, the order they appear is the **reverse** of the order they are executed.

source: [https://docs.swift.org/swift-book/ReferenceManual/Statements.html#grammar_defer-statement](https://docs.swift.org/swift-book/ReferenceManual/Statements.html#grammar_defer-statement)

If you thought about it, you would admit that it is a logical mechanism.
<!-- example, nested defer -->

<!-- linearlity -->