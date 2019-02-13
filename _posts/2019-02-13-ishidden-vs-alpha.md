---
title: "UIView's isHidden vs alpha property"
category: swift
---
At the beginning a short story. Some time ago a friend of mine asked me to took part in a recruitment in a company which he has worked in. So I sent a CV and after a couple of days, I set a meeting with a recruiter and an iOS team. The first part of the meeting was a talk with the recruiter, it was like usual talks of this type &#x1F642;. After this short first part, there was the second part - a talk with the iOS team. After several minutes of the conversation, I knew that I didn't want to work in this company, a developer with "lead" title (with a short experience) was incompetent and arrogant, after that observation, my answers mainly sounded "mmm... I have heard something, but I have no idea what it is", there was no sense to debate. Generally, the developers were trying to mark their position and discredit me, however, they were wrong in many situations.  
One of the first questions was the question about the difference between `isHidden` and `alpha` property of `UIView`, it was quite trivial so I felt a little bit confused. I started talking and I said that `alpha` is "animatable" so we can animate it and maybe when `isHidden` is set to `true` a rendering engine does optimizations but the outcome is the same as `alpha` equal to `0`. I had no idea what I could say more. And then I heard

> no...

from the iOS developer, he kept going

> when you set `isHidden` to `true` a view's frame disappears and other views fill its space, this is the difference &#x274c;

Of course, he was wrong, I was arguing later on, but I thought "this is a checkmate, game over, finish... it also means that you have no experience". There were much more "glitches", at least I have topics for the next posts.  
That circumstance was my inspiration to start this blog, so let's go to the topic.

### The project

I created a very simple project which consists of one view controller that has 3 views inside the main view: two rectangles and one button.  
This is how `viewDidLoad` looks like

{% highlight swift %}
override func viewDidLoad() {
    super.viewDidLoad()
    
    setUpViews()
}
{% endhighlight %}

`setUpViews` doesn't do nothing special, it just sets colours, adds views as subviews and pins constraints:

{% highlight swift %}
private func setUpViews() {
    view = UIView()
    view.backgroundColor = .white
    
    blueRectangle.backgroundColor = .blue
    redRectangle.backgroundColor = .red
    hideViewButton.setTitle("Hide the red view", for: .normal)
    hideViewButton.addTarget(self, action: #selector(didTapHideViewButton), for: .touchUpInside)
    
    view.addSubview(blueRectangle)
    view.addSubview(redRectangle)
    view.addSubview(hideViewButton)
    
    if let superview = blueRectangle.superview {
        blueRectangle.translatesAutoresizingMaskIntoConstraints = false
        blueRectangle.heightAnchor.constraint(equalToConstant: 100).isActive = true
        blueRectangle.topAnchor.constraint(equalTo: superview.safeAreaLayoutGuide.topAnchor, constant: 16).isActive = true
        blueRectangle.rightAnchor.constraint(equalTo: superview.safeAreaLayoutGuide.rightAnchor, constant: -16).isActive = true
        blueRectangle.leftAnchor.constraint(equalTo: superview.safeAreaLayoutGuide.leftAnchor, constant: 16).isActive = true
    }
    
    if let superview = redRectangle.superview {
        redRectangle.translatesAutoresizingMaskIntoConstraints = false
        redRectangle.heightAnchor.constraint(equalToConstant: 100).isActive = true
        redRectangle.topAnchor.constraint(equalTo: blueRectangle.bottomAnchor, constant: 16).isActive = true
        redRectangle.rightAnchor.constraint(equalTo: superview.safeAreaLayoutGuide.rightAnchor, constant: -16).isActive = true
        redRectangle.leftAnchor.constraint(equalTo: superview.safeAreaLayoutGuide.leftAnchor, constant: 16).isActive = true
    }
    
    if let superview = hideViewButton.superview {
        hideViewButton.translatesAutoresizingMaskIntoConstraints = false
        hideViewButton.topAnchor.constraint(equalTo: redRectangle.bottomAnchor, constant: 16).isActive = true
        hideViewButton.centerXAnchor.constraint(equalTo: superview.centerXAnchor).isActive = true
    }
}
{% endhighlight %}

---

**Note**

I wrote a very similar piece of code for the recruitment's purpose, the iOS developer said

> This code is wrong, what if we forget to write `view.addSubview(...)`? We won't get an error.

No, it isn't wrong, many programmers use a similar approach. The iOS developer suggested to use `self` instead of my `superview` &#x1F914;, we can't do this, `self` is `UIViewController`, not `UIView`).  
Of course, we could add `else` and throw an error if a view doesn't have its superview, for example

{% highlight swift %}
if let superview = hideViewButton.superview {
    ...
} else { throw UserInterfaceError.noSuperview(hideViewButton) }
{% endhighlight %}

but firstly, this isn't a complicated project, secondly, in real life, we should do UI tests. We could always say "what if you forget write an instruction ..." - *just test your code*... What if you forget to write a code, to build an app and to send it to the store? Another example, let's suppose that the code looks like this

{% highlight swift %}
myView.addSubview(mySubview)
mySubview.topAnchor.constraint(equalTo: myView.topAnchor, constant: 16).isActive = true
mySubview.rightAnchor.constraint(equalTo: myView.rightAnchor, constant: -16)
mySubview.leftAnchor.constraint(equalTo: myView.leftAnchor, constant: 16).isActive = true
mySubview.bottomAnchor.constraint(equalTo: myView.bottonAnchor, constant: -16).isActive = true
{% endhighlight %}

Now we should be sure, that our code works perfectly! As the iOS developer suggested, we even don't have to test it, because if we forget about something, we will get an error. But wait... We forgot about `.isActive = true` in the third line &#x1F642;. Everything compiles, we don't get any errors but our app doesn't work as we expect, because we forgot the tiny piece of code...

---

If we build the app, we should see something like this
![ScreenShot](/assets/ishidden-vs-alpha-01.png)

OK, this is the time to examine what happens with our views' frames. I added two helper methods

{% highlight swift %}
private func printViewsFrames() {
    printFrame(of: blueRectangle, viewName: "blue rectangle")
    printFrame(of: redRectangle, viewName: "red rectangle")
    printFrame(of: hideViewButton, viewName: "button")
}

private func printFrame(of view: UIView, viewName: String = "") {
    print("\(viewName):\n\torigin: \(view.frame.origin)\n\tsize: \(view.frame.size)")
}
{% endhighlight %}

and `printViewsFrames` is being called in `viewDidAppear`

{% highlight swift %}
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    
    printViewsFrames()
}
{% endhighlight %}

In the log we can see that our views have been laid out

```
blue rectangle:
	origin: (16.0, 36.0)
	size: (288.0, 100.0)
red rectangle:
	origin: (16.0, 152.0)
	size: (288.0, 100.0)
button:
	origin: (100.0, 268.0)
	size: (120.0, 30.0)
```

So let's check if `isHidden` really changes the view's frame, I added `redRectangle.isHidden = true` at the begining of `setUpViews`

{% highlight swift %}
private func setUpViews() {
    redRectangle.isHidden = true
    
    view = UIView()
    ...
{% endhighlight %}

and the red rectagle's frame didn't change itself! &#x1F631; The app looks like in the picture below
![ScreenShot](/assets/ishidden-vs-alpha-02.png)

and the log says (no changes)

```
blue rectangle:
	origin: (16.0, 36.0)
	size: (288.0, 100.0)
red rectangle:
	origin: (16.0, 152.0)
	size: (288.0, 100.0)
button:
	origin: (100.0, 268.0)
	size: (120.0, 30.0)
```

Maybe in the runtime it acts differently, I removed `redRectangle.isHidden = true` from `setUpViews` and implemented `didTapHideViewButton` selector of the button as follows

{% highlight swift %}
@objc func didTapHideViewButton(_ sender: Any) {
    redRectangle.isHidden = true
    view.layoutIfNeeded() // this is NOT needed but I added this to eliminate any understatements
    printViewsFrames()
}
{% endhighlight %}

![ScreenShot](/assets/ishidden-vs-alpha-03.gif)

### The documentation

There is nothing unexpected, the official documentation says

> A hidden view disappears from its window and does not receive input events. It remains in its superview’s list of subviews, **however, and participates in autoresizing as usual**.

source: [https://developer.apple.com/documentation/uikit/uiview/1622585-ishidden](https://developer.apple.com/documentation/uikit/uiview/1622585-ishidden)

### What about `alpha`?

As I have already mentioned at the beginning, the differences are:

- `alpha` can be animated
- `alpha` has intermediate states, for example, we can set `alpha` to `0.7` (70%), contrary, `isHidden` is binary, it can be either `true` (hidden) or `false` (visible)
- considering the outcome, there is no difference between `alpha = 0` and `isHidden = true` (maybe OS does some optimization)

So, if you know CSS, `isHidden` acts exactly the same way as `visibility` in CSS.

### One more thing...

In 2015, along with Apple Watch, Apple introduced a concept of container view which can arrange its subviews in stacks automatically, that was the only approach to building UI in watchOS, later on, iOS 9 got this container known as `UIStackView`. The documentation says

>Stack views let you leverage the power of Auto Layout, creating user interfaces that can dynamically adapt to the device’s orientation, screen size, and any changes in the available space. The stack view manages the layout of all the views in its arrangedSubviews property. These views are arranged along the stack view’s axis, based on their order in the arrangedSubviews array.

source: [https://developer.apple.com/documentation/uikit/uistackview](https://developer.apple.com/documentation/uikit/uistackview)

Nowadays, this is the only view that can arrange its subviews automatically. Let's see how it works, I put my three views inside `UIStackView`, instead of the view controller's main view directly, the rest of the code is the same (only `setUpViews` has changed slightly).

![ScreenShot](/assets/ishidden-vs-alpha-04.gif)

Even then, `UIStackView` doesn't remove our subview or change its size, we can see that subview's origin has changed only - from `(0.0, 116.0)` to `(0.0, 108.0)`. `UIStackView` has hidden the subview and changed constraints of the other views to rearrange them, in order to mimic behavior as the subview doesn't exist in the stack.

`UIStackView` is quite powerful, I remember times when I had to track all constraints, animate and change them manually, now, if we don't have a very complicated layout, we can just "throw views into the bag" and OS does the job.
