---
layout:     post
title:      Python comprehensions
date:       2016-10-25 05:23:30
summary:    "{ x2 | x ∈ ℕ }"
categories: python, programming languages
---

Initially introduced by Barry Warsaw on <a href="https://www.python.org/dev/peps/pep-0202/" target="_blank">PEP-202</a>, comprehensions became part of the Python language in version 2.0 almost twenty years ago. This language feature enables a beautiful and convienient way to work with iterables. 
By using comprehensions the code is written in a more declarative and concise way. 
I think of it as an elegant way of Python's for implementing a well-known Mathematics notation: <b>{ x2 | x ∈ ℕ }</b>.

Let's look at a code example. The lines of code bellow add the square numbers from 0 to 9 to a list and print them. 
{% highlight python lineanchors %}
squares = []
for x in range(10):
    squares.append(x ** 2)
print(squares) 
{% endhighlight %}

### List comprehensions 
The same functionality can be re-written using list comprehensions with the following syntax: <b>[ expression for element in iterable ]</b>
{% highlight python lineanchors %}
squares = [x ** 2 for x in range(10)]
print(squares)
{% endhighlight %}
 
#### Using conditionals statements with list comprehensions
The example is simplified to explain the syntax. In this case applying conditional statements, like described bellow.
{% highlight python lineanchors %}
squares = []
for x in range(10):
    if (x % 2 == 0):
        squares.append(x ** 2)
print(squares)
{% endhighlight %}
 
The code above can be re-written with the following syntax: <b>[ expression for element in iterable if condition ]</b>
{% highlight python lineanchors %}
squares = [x ** 2 for x in range(10) if (x % 2 == 0)]
print(squares)
{% endhighlight %}

### Dictionary comprehensions
In <a href="https://www.python.org/dev/peps/pep-0274/" target="_blank">PEP-274</a> a propose for a similar syntatical extension for dictionary objects instead of lists is introduced. This is one is called "Dictionary Comprehension" and you can see an example bellow using a conditional statement. 

{% highlight python lineanchors %}
squares = {x: x**2 for x in range(10) if (x % 2 == 0)}
print(squares)
{% endhighlight %}

### Set comprehensions
A set is an unordered and mutable collection of items. All it's elements are unique (no duplicates) and immutable. Set comprehensions are created in much the same way as dictionary comprehensions.

{% highlight python lineanchors %}
numbers = [0, 1, 2, 2, 3, 4, 5, 5, 6, 7, 8, 8, 9, 9, 9]
squares = {x**2 for x in numbers}
print(squares)
{% endhighlight %}

### Generator comprehensions
They are similar to the examples above, the main difference is that they do not allocate memory for the whole expected type but generate one item at a time. This results in a better memory usage efficiency. See also <a href="https://www.python.org/dev/peps/pep-0289/" target="_blank">PEP-289</a> 

{% highlight python lineanchors %}
squares = (x**2 for x in range(10) if x % 2 == 0)
print(squares)
for s in squares:
    print(s)
{% endhighlight %}

### TLDR
Comprehensions are in my humble opinion an awesome feature of Python and they are supported in both versions 2.x and 3.x. As you can see in the examples described they provide a clean and concise way to avoid nested loops, lambda functions or even <a href="https://docs.python.org/3.7/library/functions.html#filter" target="_blank">filter()</a> or/and <a href="https://docs.python.org/3.7/library/functions.html#map" target="_blank">map()</a> functions. 