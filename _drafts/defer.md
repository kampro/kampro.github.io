---
title: "Defer in Swift"
category: swift
---
Continuing an explanation of topics which were touched during my meeting with an iOS developer (the story is contained in [the preface of the post about `isHidden` and `alpha`]({{ site.baseurl }}{% post_url 2019-02-13-ishidden-vs-alpha %})), I would like to explain a couple of details regarding `defer` in Swift.  
`defer` has been introduced in Swift 2.0, thus in 2015, and it is a quite comfortable solution which eliminates the possibility of an error occurrence.

### How does `defer` work?

The statement allows to execute instructions inside `defer` when our program leaves a scope where `defer` is. In the simple words, instructions from `defer` will be executed when a program goes out of a block (a zone between `{``}`). Let's go to the example.  
For the purposes of the post, I made a simple class which are supposed to simulate a communication to a remote server.

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
<!-- example without and with defer -->

### Errors inside a `defer`'s block

You have to bear in mind, that all errors (produced by instructions that `throws` or by `throw` statement) inside `defer` will be consumed and won't be passed outside of the block.
<!-- example -->

### Multiple `defer` statements

Can we have multiple `defer` statements? Yes, we can, but we have to know how a more than one statement works exactly. If we wanted to defer several instructions, we would write them in the same block belonging to one `defer`, so what is the purpose of multiple `defer`s? The answer is: if there is a more than one `defer`, they are executed **in the reverse order** of an occurrence.

> If multiple defer statements appear in the same scope, the order they appear is the **reverse** of the order they are executed.

source: [https://docs.swift.org/swift-book/ReferenceManual/Statements.html#grammar_defer-statement](https://docs.swift.org/swift-book/ReferenceManual/Statements.html#grammar_defer-statement)

If you thought about it, you would admit that it is a logical mechanism.
<!-- example, nested defer -->

<!-- linearlity -->