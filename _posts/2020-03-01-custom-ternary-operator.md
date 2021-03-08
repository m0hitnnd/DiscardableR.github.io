---
layout: post
cover: 'assets/images/cover7.jpg'
navigation: True
title: Custom Ternary Operator in Swift
date: 2020-03-01 09:30:00
tags: Tech Swift
subclass: 'post tag-swift tag-tech'
logo: 'assets/images/ghost.png'
author: mohitnnd
categories: mohitnnd
---

<br />
In Swift, whenever we have some logic around an optional variable, we do a nil check by directly checking if the variable is nil or not, or by unwrapping it using if let.

Like in the below case where we are setting a string on the basis of an optional var.
{% highlight swift %}
let optionalVar: String? = "some value"
let result: String
if optionalVar != nil {
    result = "Got it"
} else {
    result = "blah"
}
{% endhighlight %}


To increase readability of the above code, we can make use of the only ternary operator swift provides - <code>?:</code>

{% highlight swift %}
let result = optionalVar != nil ? "Got it" : "blah"
{% endhighlight %}


<i>Now, what if we want to use the unwrapped value for our result?</i>
<br /> 
<br />
We can unwrap the optional variable using if let and use it in our result.
{% highlight swift %}
let result: String
if let unwrappedVar = optionalVar {
    result = "We got " + unwrappedVar
} else {
    result = "blah"
}
{% endhighlight %}

<i>But, can we make the above code cleaner?</i>
<br />
<i>Can we have something similar to what we used in our first example?</i>
<br />
Something like:
{% highlight swift %}
let result = optionalVar ? "We got \($0)" : "blah"
{% endhighlight %}
Here, $0 is the unwrapped value of optional var. <br /><br />
<b>Basically, it's similar to existing ternary operator a ? b : c, where if a is true then b is returned else c is returned, but it also provides the unwrapped value of a to b, and b may or may not use that unwrapped value, but it does have access to it.</b>


Sounds Great, right?<br /> Lets see how we can achieve this by defining our own custom ternary operator.

<ol>
<li> 
We need to pick the characters. Since, we are trying to make a ternary operator we need 2 characters. For our use case, lets go ahead with <b>!|</b>
{% highlight swift %}
operator !|
{% endhighlight %}
Check out Operators section of <a title="this link" href="https://docs.swift.org/swift-book/ReferenceManual/LexicalStructure.html">this link</a> , which describes all the characters that can be used to define custom operators.
</li>
<li> 
We also need to tell the compiler that whether our operator meant to be used between 2 operands(<b>infix</b>), before an operand(<b>prefix</b>) or after an operand(<b>postfix</b>).<br /><br />
And, if we see, our custom operator requires 3 operands(a, b & c), which does not satisfy either infix, prefix or postfix.
<br />
<br />
<b>Infact, in Swift you can't define a custom ternary operator.</b>
<br />
<br />
But, there's another way to do this. We will create 2 separate infix operators(! & |).
{% highlight swift %}
infix operator !
infix operator |
{% endhighlight %}
</li>
<li>
Lets try to understand how we will achieve this.<br /><br />
Given the statement <code>a ! b | c</code>, we need to create a function which has access to all the three operands, so that it can apply the logic on a and return either b or c.<br />
We will tell compiler to execute <code>b | c</code> first (<i>How?...will talk about this later</i>). 
Will store b & c in closures and return those closures in a tuple to operator <code>!</code> as a second operand. Lets's call that tuple <code>d</code>.<br />
Now <code>a ! d</code> will be executed, where we can apply our logic as we have acess to all the operands. 
<br />
<ol>
<li>Function for <code>|</code> operator.
{% highlight swift %}
func |<T1, T2>(
    lhs: @escaping (T1) -> T2,
    rhs: @escaping () -> T2
) -> (second: (T1) -> T2, third: () -> T2) {
    return (lhs, rhs)
}

{% endhighlight %}

In the above code, we are storing the operands <i>lhs</i> & <i>rhs</i>,(b & c in our case) in a closure and returning them as is, in a tuple.<br /> <i>Note, we are not calling the closures inside the function, hence these are escaping closures.</i> 
</li>
<li> Now, lets create a function for <code>!</code> operator.
{% highlight swift %}
func !<T1, T2>(
    lhs: Optional<T1>,
    rhs: (second: (T1) -> T2, third: () -> T2)
) -> T2 {
    guard let unwrappedLhs = lhs else {
        return rhs.third()
    }
    return rhs.second(unwrappedLhs)
}
{% endhighlight %}

Here, we are unwrapping <i>lhs</i>(a, in our case), and if it's not nil, then we call closure of second operand(b) with the unwrapped value(<code>rhs.second(unwrappedLhs)</code>), else we call closure of the third operand(<code>rhs.third()</code>).
</li>
</ol>
</li>
<li>
One last thing we need to do is to tell the compiler to execute <code>|</code> first and then the operator <code>!</code>.<br /> We can achieve this by setting <b>associativity</b>.<br /><br /> Associativity comes into picture when two operators have the same precedence or priority. Higher priority operators are executed before lower priority ones. But if the priority is same, then associativity defines how operators will grouped, left or right. Check out <a title="this link" href="https://docs.swift.org/swift-book/ReferenceManual/LexicalStructure.html">this link</a>  to read more on associativity and precedence.
<br />
<br />
In our use case, we can either define <code>|</code> operator as a part of higher priority group than <code>!</code> , or we can make both the operators in the same priority group but with right associativity. Both the options will execute <code>|</code> operation first.
<br />
<br />
Infact, we can use the existing precedence group - <b>TernaryPrecedence</b> for both the operators. TernaryPrecedence is define for our predefined ternary operator - <code>?:</code>, and it also has right associativity.
{% highlight swift %}
infix operator !: TernaryPrecedence
infix operator |: TernaryPrecedence
{% endhighlight %}
</li>
<li> And, we are done! Lets see how we can use our custom ternary operator.
</li>
</ol>
{% highlight swift %}
let optionalVar: String? = "some value"
let result: String = optionalVar ! { "We got \($0)" } | { "blah" } 
{% endhighlight %}


The above code looks great, but we can still make it cleaner by using autoclosure.<br />
With <b>@autoclosure</b>, we can write <code>"blah"</code> without the braces.
{% highlight swift %}
func |<T1, T2>(
    lhs: @escaping (T1) -> T2,
    rhs: @autoclosure @escaping () -> T2
) -> (second: (T1) -> T2, third: () -> T2) {
    return (lhs, rhs)
}
{% endhighlight %}
<i>Note, we can't have autoclosure for lhs since autoclosure attribute only works with closures which doesn't take any parameter.</i>

Let's see the final usage.
{% highlight swift %}
let optionalVar: String? = "some value"
let result: String = optionalVar ! { "We got \($0)" } | "blah"
{% endhighlight %}

{% highlight swift %}
let anotherOptionalVar: Int? = nil
let result: String = anotherOptionalVar ! { "\($0)" } | "-1"
{% endhighlight %}

Looks damn good. Cheers!!

Check out the entire code in the <a title="custom ternary operator playground" href="https://github.com/m0hitnnd/iOSStack/tree/master/CustomTernaryOperator/CustomTernaryOperator.playground">playground.</a>
<br />

<h6 id="heading6">References:</h6>
<br />
<a title="operator declarations" href="https://developer.apple.com/documentation/swift/swift_standard_library/operator_declarations">https://developer.apple.com/documentation/swift/swift_standard_library/operator_declarations</a>
<br />
<br />
<a title="Basic Operators" href="https://docs.swift.org/swift-book/LanguageGuide/BasicOperators.html">https://docs.swift.org/swift-book/LanguageGuide/BasicOperators.html</a>
<br />
<br />
<a title="Advanced Operators" href="https://docs.swift.org/swift-book/LanguageGuide/AdvancedOperators.html">https://docs.swift.org/swift-book/LanguageGuide/AdvancedOperators.html</a>
<br />
<br />
<a title="function-currying" href="https://thoughtbot.com/blog/introduction-to-function-currying-in-swift">https://thoughtbot.com/blog/introduction-to-function-currying-in-swift</a>
<br />
<br />
<a title="ternary-unwrapping" href="https://dev.to/danielinoa/ternary-unwrapping-in-swift-903">https://dev.to/danielinoa/ternary-unwrapping-in-swift-903</a>
<br />
<br />
