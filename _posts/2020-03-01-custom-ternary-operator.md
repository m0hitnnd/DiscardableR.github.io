---
layout: post
cover: 'assets/images/cover7.jpg'
navigation: True
title: Custom Ternary Operator in Swift
date: 2020-03-08 09:30:00
tags: Tech Swift
subclass: 'post tag-swift tag-tech'
logo: 'assets/images/ghost.png'
author: mohitnnd
categories: mohitnnd
---

<br />
In our everyday Life, we are constantly taking decisions somewhere or the other at different cross-paths. Our mind consciously evaluates
<blockquote> 
<p>If first path, then what? If second path, then what?</p>
</blockquote> 
and our code is no different. 

Whenever we have some logic around a variable that can be nil, we do a nil check by directly checking if the variable is nil or not, or we unwrap it using if let.

Like in the below case where we are setting a string(<code>info</code>) on the basis of an optional var(<code>newTask</code>).
{% highlight swift %}
let newTask: String? = "Complete Blog"
let info: String
if newTask == .some {
    info = "1 New Task"
} else {
    info = "No new task"
}
{% endhighlight %}


To increase readability of the above code, we can make use of the <b>only ternary operator swift provides</b> - <code>?:</code>

{% highlight swift %}
let info = newTask == .some ? "1 New Task" : "No new task"
{% endhighlight %}
<i>Now, what if we want to use the unwrapped value of <code>newTask</code> for our <code>info</code> variable?</i>
<br /> 
<br />
We can unwrap it using <code>if let</code> and use it for our info string.
{% highlight swift %}
let info: String
if let task = newTask {
    info = "1 New Task - \(task)"
} else {
    info = "No new task"
}
{% endhighlight %}

<i>But, wouldn't that be great if we can convert the above code to a one liner?</i>
<br />

Something like this:
{% highlight swift %}
let info = newTask ? "1 New Task - \($0)" : "No new task"
{% endhighlight %}
Where, $0 denotes the unwrapped value of optional var. <br /><br />
We cannot write the above code with our existing ternary operator <code>?:</code>, as it doesn't unwrap the value of optional variable(<code>newTask</code> in the above case).<br /><br />
<b>So, we will have to make a custom ternary operator which will be used in a similar way as our existing operator(<code>?:</code>) -  <code>a ? b : c</code>, where if a is true then b is returned else c is returned, but it would also provide the unwrapped value of a to b, and b may or may not use that unwrapped value, but it would have access to it.</b>


Sounds Great, right?<br /> Lets see how we can achieve this by defining our custom ternary operator.

<ol>
<li> 
We need to pick the characters first.<br /> Since, we are trying to make a ternary operator we need 2 characters. For our use case, lets go ahead with <code><b>!|</b></code>
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
<b>Infact, in Swift there's no defined way to write a custom ternary operator. We will have to take two separate infix operators(<code>!</code> & <code>|</code>) and will make them work together.</b>
<br />
<br />
{% highlight swift %}
infix operator !
infix operator |
{% endhighlight %}
</li>
<li>
Lets try to understand how we will achieve this.<br /><br />
Given the statement <code>a ! b | c</code>, we need to create a function which has access to all the three operands, so that it can apply the logic on a and return either b or c.<br />
We will tell the compiler to execute <code>b | c</code> first (<i>How?...will talk about this later</i>). 
Will store b & c in closures and return those closures in a tuple to operator <code>!</code> as a second operand. Lets's call that tuple <code>d</code>.<br />
Now <code>a ! d</code> will be executed, where we can apply our logic as we have acess to all the operands. 
<br />
<br />
<ol>
<li>Function for <code>|</code> operator:
{% highlight swift %}
func |<T1, T2>(
    lhs: @escaping (T1) -> T2,
    rhs: @escaping () -> T2
) -> (second: (T1) -> T2, third: () -> T2) {
    return (lhs, rhs)
}

{% endhighlight %}

In the above function, we are storing the operands <i>lhs</i> & <i>rhs</i>,(b & c in our case) in a closure and returning them as is, in a tuple.<br /> <i>Since, we are storing the closures and not calling them inside the function, these are marked as escaping closures.</i> 
</li>
<li> Function for <code>!</code> operator:
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
Now, if we try to use the custom operator, the compiler will throw an error:
<p><img src="https://raw.githubusercontent.com/m0hitnnd/iOSStack/master/CustomTernaryOperator/DefaultPrecedenceError.png" alt="Default Precedence error" /></p>
This means that we are using two operators adjacent to each other(! & |), but the compiler doesn't know which one to execute first,
{% highlight swift %} nextTask ! { "1 new task - \($0)" } {% endhighlight %}
<p style="text-align:center;">or</p>
{% highlight swift %} { "1 new task - \($0)" } | { "No new task" } {% endhighlight %}
This happens when operators adjacent to each other have same priority or precedence. <i>Note, higher priority operators are executed first.</i> Since, we didn't define any priority to our custom operators in step 2, both of them defaults to same priority group i.e. <b>Default Precedence.</b>
<br />
<br />
Because we want <code>{ "1 new task - \($0)" } | { "No new task" }</code> this piece of code to be executed first, we need compiler to execute <code>|</code> first and then the operator <code>!</code>.
<br />
We can solve this by either defining <code>|</code> operator as a part of higher priority group than <code>!</code>, or we let both the operators have the same priority but also tell the compiler to execute operations from right first. And, this is what <b>associativity</b> is. When the operators with same priority are used adjacent to each other, associativity tells the compiler to either start from left(<b>Left Associativity</b>) or from right(<b>Right Associativity</b>). Check out <a title="this link" href="https://docs.swift.org/swift-book/LanguageGuide/AdvancedOperators.html">this link</a>  to read more on associativity and precedence.
<br />
<br />
For our use case, we can use the existing precedence group - <b>TernaryPrecedence</b> for both the operators. TernaryPrecedence is define for the predefined ternary operator - <code>?:</code>, and it also has right associativity. This means that both the operators will have same priority but when placed adjacent to each other, the evaluation will start from right. To know more on predefined precedence groups, check Apple's documention on <a title="operator declarations" href="https://developer.apple.com/documentation/swift/swift_standard_library/operator_declarations">Operator Declarations</a>.
{% highlight swift %}
infix operator !: TernaryPrecedence
infix operator |: TernaryPrecedence
{% endhighlight %}
</li>
<li> And, we are done! No compiler errors :)
</li>
</ol>
{% highlight swift %}
let newTask: String? = "Complete Blog"
let info = newTask ! { "1 new task - \($0)" } | { "No new task" }
{% endhighlight %}


The above code looks great, but we can still make it cleaner.<br />
The statement <code>{ "No new task" }</code> is a self resulting statement which doesn't need any params to return the result.
We can use <b>@autoclosure</b> here so that we can write <code>"No new task"</code> without the curly braces.
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
let newTask: String? = "Complete Blog"
let info = newTask ! { "1 new task - \($0)" } | "No new task"
{% endhighlight %}

Looks damn good. Cheers!!

Check out the entire code in the <a title="custom ternary operator playground" href="https://github.com/m0hitnnd/iOSStack/tree/master/CustomTernaryOperator/CustomTernaryOperator.playground">playground.</a>
<br />

<h6 id="heading6">References:</h6>
<br />
Apple Documentation on <a title="operator declarations" href="https://developer.apple.com/documentation/swift/swift_standard_library/operator_declarations">Operator Declarations</a>
<br />
<br />
Swift Documention on <a title="Basic Operators" href="https://docs.swift.org/swift-book/LanguageGuide/BasicOperators.html">Basic Operators</a>
<br />
<br />
Swift Documentation on <a title="Advanced Operators" href="https://docs.swift.org/swift-book/LanguageGuide/AdvancedOperators.html">Advanced Operators</a>
<br />
<br />
Swift Documentation on <a title="Precedence & Associativity" href="https://docs.swift.org/swift-book/LanguageGuide/AdvancedOperators.html#precedence_and_associativity">Precedence & Associativity</a>
<br />
<br />
Similar article on <a title="ternary-unwrapping" href="https://dev.to/danielinoa/ternary-unwrapping-in-swift-903">Ternary Unwrapping in Swift</a>
<br />
<br />