---
title: "Swift's guard construction"
category: swift
---
Today I would like to describe a simple construction introduced in Swift 2.0 - `guard`. I know that this can be valuable for very fresh Swift developers only, but I have to be sure that everything what will be used in the future posts has been explained.

### What is the purpose of `guard`?

Let's suppose we have the following example

{% highlight swift %}
class SomeClass {
    
    var directory: URL?
    var filePath: String?
    
    func createFile() {
        if let directory = directory {
            if let filePath = filePath {
                let fileURL = directory.appendingPathComponent(filePath)
                
                do {
                    if try fileURL.checkResourceIsReachable() {
                        print("File exists")
                    } else {
                        print("File doesn't exist")
                        
                        try Data().write(to: fileURL)
                    }
                } catch {
                    print(error.localizedDescription)
                }
            } else {
                print("File path is not defined")
            }
        } else {
            print("Directory is not defined")
        }
    }
    
}
{% endhighlight %}

As we can see, we have a stub of the indentation hell, this is the place where `guard` helps. Using `guard` we can write it like this

{% highlight swift %}
class SomeClass {
    
    var directory: URL?
    var filePath: String?
    
    func createFile() {
        guard let directory = directory else {
            print("Directory is not defined")
            return
        }
        
        guard let filePath = filePath else {
            print("File path is not defined")
            return
        }
        
        let fileURL = directory.appendingPathComponent(filePath)
        
        do {
            guard try fileURL.checkResourceIsReachable() else {
                print("File exists")
                return
            }
            
            print("File doesn't exist")

            try Data().write(to: fileURL)
        } catch {
            print(error.localizedDescription)
        }
    }
    
}
{% endhighlight %}

A lot more readable. `guard` guarantes that every check which is applied in `guard` is complied from this place to the end of the current scope, otherwise the execution has to leave the scope.

### The syntax

{% highlight swift %}
guard <condition> else {
    <statements>
}
{% endhighlight %}

`else` keyword is mandatory and inside of `else`'s block there must occur `return`, `break`, `continue`, `throw` or a call of a function which returns `Never` (something what would break the program's current flow).