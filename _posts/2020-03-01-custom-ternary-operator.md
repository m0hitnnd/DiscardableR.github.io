---
layout: post
cover: 'assets/images/cover7.jpg'
navigation: True
title: Custom Ternary Operator in Swift/ Curried Function
date: 2020-03-01 09:30:00
tags: Tech iOS
subclass: 'post tag-ios tag-tech'
logo: 'assets/images/ghost.png'
author: mohitnnd
categories: mohitnnd
---



Lets say we have to return a string on the basis of an optional var.
{% highlight swift %}
let optionalVar: String? = nil//"blah"
let result3: String
if optionalVar != nil {
    result3 = "Got it"
} else {
    result3 = "Damn"
}
{% endhighlight %}

To improve the readability of the above code or to make it more clean, we can use ternary operator ?:
{% highlight swift %}
let result4 = optionalVar != nil ? "Got it" : "Damn"
{% endhighlight %}


Now, what if, we want to use value of optionalVar when it's not nil. We can write something like this:
{% highlight swift %}
let result2: String
if let unwrappedVar = optionalVar {
    result2 = "Got it " + unwrappedVar
} else {
    result2 = "Damn"
}
{% endhighlight %}

Can we make the above code cleaner? Can we have something similar to what we used in the previous example(result4)? Something like:
{% highlight swift %}
let result5 = optionalVar ! "Got it \($0)" | "Damn"
{% endhighlight %}
Here $0 is the unwrapped value of optional var
Basically it's similar to existing ternary operator a ? b : c, where if a is not nil then b is returned else c is returned. In the above case - a ! b | c, if a is not nil then unwrapped the value, pass it to b and return b(b may or may not use unwrapped value, but it does have access to it) else just return c.


So, lets define our custom operator - <b> !| </b>
{% highlight swift %}
operator !|
{% endhighlight %}

