---
layout: post
date: 2011-06-19 20:00:00 UTC
title: NodeLists and Arrays in JavaScript
---

JavaScript is an admittedly quirky language, and its almost array-like objects
is one of the most glaring issues. And of all those array-like objects,
NodeLists are one of the most commonly used ones.

The first thing to note is that NodeLists aren't exactly part of the
JavaScript but they are instead part of the DOM API the browsers provide
through JavaScript. The relationship between DOM and JavaScript is beyond the
scope of this article but knowing that NodeLists aren't exactly part of
JavaScript language itself might be useful in understanding the relationship
between JavaScript arrays and NodeLists.

So what is exactly an NodeList? The W3C defines the NodeList [as follows](http://www.w3.org/TR/DOM-Level-2-Core/core.html#ID-536297177):

> The NodeList interface provides the abstraction of an ordered collection of
> nodes, without defining or constraining how this collection is implemented.

Essentially, a NodeList is what you get when you call any method such as
`document.getElemetsByTagName()`, `document.querySelectorAll()` and such. In
fact, it was my experience with `document.querySelectorAll()` that motivated
me to write this post.

Careful readers of the W3C definition will note that arrays in most languages,
like JavaScript, are almost what this definition implies NodeLists are:
"ordered collection of (things)". In fact, NodeLists do in fact resemble
arrays in a lot of ways. Let's see some examples, by testing things out on the
Digg homepage.

{% highlight javascript %}
    > var myList = document.querySelectorAll('.story-item');
      undefined
    > myList
      myList
      [
      <div class=​"story-item">​…​</div>​
      ,
      <div class=​"story-item">​…​</div>​
      ,
      [...]
      ,
      <div class=​"story-item">​…​</div>​
      ,
      ]
{% endhighlight %}

Definitely looks like an array from here, if nothing else other than fact that
the way our console inspects `myList` starts with a square bracket. Let's try
some basic array actions with it:

{% highlight javascript %}
    > myList.length;
      17
    > myList[3];
      <div class=​"story-item" >…</div>​
{% endhighlight %}

So far, `myList` has been talking and walking like an array so we can probably
assume that it's an array of some sorts. However, it all goes to hell when you
try to call any of the basic array methods. Suppose we want to slice the array
from it's 3rd element.

{% highlight javascript %}
    > myList.slice(2) // indexed from 0
      TypeError: Result of expression 'myList.slice' [undefined] is not a
      function.
{% endhighlight %}

Wait, what happened? Well, this is where the between NodeLists and arrays in
JavaScript start to come to surface. Let's try to see if NodeList actually
an array (which is actually a [tricky
problem](http://javascript.crockford.com/remedial.html) in JavaScript itself).

{% highlight javascript %}
    > myList.constructor.toString();
      "[object NodeListConstructor]"
{% endhighlight %}

Let's see what an array that we sure know is an array looks like
in JavaScript using the same method.

{% highlight javascript %}
    > var surelyArray = ['foo', 'bar'];
      undefined
    > surelyArray.constructor.toString();
      "function Array() {
          [native code]
      }"
{% endhighlight %}

So those two elements, `myList` and `surelyArray` are definitely constructed
by different constructors so it's no wonder that they don't share the same
methods.

In fact, it turns out that NodeLists support accessing elements by their index
and they do have `length` property but that's essentially where the
similarities end. If you want to call any of the array methods on a NodeList,
you will just get an error.

If you think about what NodeLists are and the re-read the definition, this
(kind of) makes sense. While arrays are essentially a collection of elements
held in memory and are part of the JavaScript, NodeLists are _live_ references
to actual DOM elements. I am not sure if that explanation is more a
justification or an after-the-fact rationalization but it's one way to look
at things.

Luckily, you can relatively easily convert NodeLists into arrays so that you
can easily call all your favorite array methods like `push()`, `slice()` on
them.

Let's see a quick one-liner to do so:

{% highlight javascript %}
    > var myArray = Array.prototype.slice.call(myList, 0);
      undefined
    > myArray.constructor.toString();
      "function Array() {
          [native code]
      }"
{% endhighlight %}

Definitely looks like an array! Before we look at what happened with that
one-liner, let's really make sure that this `myArray` is an actual array, not
some other crazy creation.

{% highlight javascript %}
    > myArray.pop();
      <div class=​"story-item">​…​</div>​
{% endhighlight %}

Not bad. We are definitely sure that we are dealing with an array
here.

The one-liner we used before is actually pretty simple. What we are
essentially doing here is borrowing the `slice()` method from the Array's
`prototype` and applying it to on NodeList, slicing it from it's beginning, so
getting the entire list itself. And as `slice()` returns an actual array, we
end with a real array, as opposed to a NodeList. More explanation on `slice()`
works can be found on Robert Sosinski's excellent post on [Binding Scope in
JavaScript](http://www.robertsosinski.com/2009/04/28/binding-scope-in-javascr
pt/).

There you have it. Except, if you use Internet Explorer, your clever one-liner
may not work as expected so you actually have to resort to iterating over your
NodeList and putting them into an array. Pretty simple:

{% highlight javascript %}
    > var myIEArray = [];
      undefined
    > for (var i = 0; i < myList.length; ++i) { myIEArray.push(myList[i]); }
      17
    >  myIEArray.constructor.toString();
      "function Array() {
          [native code]
      }"
{% endhighlight %}

There you have it, for real this time. Of course, if you really wanted to make
things robust, you'd probably

NodeLists are a powerful tool and with more and more browsers getting decent
DOM APIs, you might find yourself dealing with more NodeLists instead of
jQuery objects and such.

In fact, if you look at the source code for Thomas Fuchs' Zepto.JS
micro-library, you'll find that [it implements](https://github.com/madrobby/zepto/blob/c03bef955913afa858116538e59c6e7a6ac04207/src/zepto.js#L68) arguably the most important
aspect of jQuery, the CSS DOM selector using the (relatively) new
`document.querySelectorAll()` method and converting the NodeList to an array
for easy further manipulation.