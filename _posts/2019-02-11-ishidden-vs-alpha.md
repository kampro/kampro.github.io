---
title: "UIView's isHidden vs alpha property"
category: swift
---
At the begining a short story. Some time ago a friend of mine asked me to took part in a recruitment in a company which he has worked in. So I sent a CV and after a couple of days, I set a meeting with a recruiter and an iOS team. The first part of the meeting was a talk with the recruiter, it was like usual talks of this type. &#x1F642; After this short first part, there was the second part - a talk with the iOS team. After several minutes of the conversation, I knew that I didn't want to work in this company, a developer with "lead" title (with a short experience) was incompetent and arrogant, after that observation, my answers mainly sounded "mmm... I have heard something, but I have no idea what it is", there was no sense to debate. Generally, the developers were trying to mark their position and discredit me, however, they were wrong in many situations.  
One of the first questions was the question about the difference between `isHidden` and `alpha` property of `UIView`, it was quite trivial so I felt a little bit confused. I started talking and I said that `alpha` is "animatable" so we can animate it and maybe when `isHidden` is set to `true` a rendering engine does optimizations but the outcome is the same as `alpha` equal to `0`. I had no idea what I could say more. And then I heard

> no...

from the iOS developer, he kept going

> when you set `isHidden` to `true` a view's frame disappears and other views fill its space, this is the difference &#x274c;

Of course, he was wrong, I was arguing later on, but I thought "this is a checkmate, game over, finish... it also means that you have no experience". There were much more "glitches", at least I have topics for the next posts.  
That circumstance was my inspiration to start this blog, so let's go to the topic.

### To be continued...

<!-- {% highlight swift %}
if let superview = collectionView.superview {
    superview.frame = CGRect(x: 0, y: 0, width: 320, height: 568)
}
{% endhighlight %}

```
if let superview = collectionView.superview {
    superview.frame = CGRect(x: 0, y: 0, width: 320, height: 568)
}
``` -->
