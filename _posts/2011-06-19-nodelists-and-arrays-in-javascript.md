---
layout: post
date: 2011-06-19 20:00:00 UTC
title: NodeLists and Arrays in JavaScript
---

JavaScript is an admittedly quirky language, and its almost array-like objects
is one of the most glaring issues. And of all those array-like objects,
NodeLists are one of the most commonly used ones array-like objects that
beginners trip on when they use methods like
`document.getElementsByTagName()`.

In this post, I'll try to explain what NodeLists are, how they relate to
arrays in JavaScript and how you can work with them in a comfortable way.

So what is exactly an NodeList? The W3C defines the NodeList [as follows](http://www.w3.org/TR/DOM-Level-2-Core/core.html#ID-536297177):

> The NodeList interface provides the abstraction of an ordered collection of
> nodes, without defining or constraining how this collection is implemented.

Essentially, a NodeList is what you get when you call any method such as
`document.getElemetsByTagName()`, `document.querySelectorAll()` and such.

We should note here that NodeLists aren't exactly part of the JavaScript but
they are instead part of the DOM APIs the browsers provide through JavaScript.
We will come back to discussing the relationship between NodeLists and arrays
later in the article again.

Careful readers of the W3C definition will note that arrays in most languages,
like JavaScript, are almost what this definition implies NodeLists are:
"ordered collection of (things)". In fact, NodeLists do in fact resemble
arrays in a lot of ways. Let's test things out by firing up our JavaScript
console on the Digg homepage.

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
the way our console inspects `myList` starts and ends with a square bracket.
Let's try some basic array actions with it:

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
JavaScript start to surface. Let's try to see if NodeList actually an array (which is actually a [tricky
problem](http://javascript.crockford.com/remedial.html) in JavaScript itself).

{% highlight javascript %}
    > myList.constructor.toString();
      "[object NodeListConstructor]"
{% endhighlight %}

Let's see what an array, that we definitely know to be an array looks like
using the same technique (where we look at its' constructor).

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

<aside>
  <p>
Note that my examples here are contrived examples and using the `toString()`
method on the constructor property of an Array is not how you would want to
check if something is an arrays. A better approach would be using `instanceof`
or some other more robust method.
  </p>
</aside>

In fact, it turns out that NodeLists support accessing elements by their index
and they do have `length` property but that's essentially where the
similarities end. If you want to call any of the array methods on a NodeList,
you will just get an error.

Let's think about why this is the case. If you think about what NodeLists are
and the re-read the definition, this (kind of) makes sense. While arrays are
essentially a collection of elements held in memory and are part of the
JavaScript, NodeLists are _live_ references to actual DOM elements, except for
the `document.querySelectorAll()` method, which returns not live but static
NodeLists.

<aside>
  <p>
Mozilla engineer [Frank Yan](http://twitter.com/frankyan/) explains this
better than I could in the comments. The main reason NodeLists exist as a
different data structure is that the DOM API is supposed to be language
agnostic. If it were the case that NodeLists inherited from arrays, the DOM
API would be tied to the constructs of the JavaScript language.
  </p>
</aside>

Luckily, you can relatively easily convert NodeLists into arrays so that you
can easily call all your favorite array methods like `push()`, `slice()` on
them.

Let's see a quick way to convert a NodeList into an array:

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
end with a real array, as opposed to a NodeList. More explanation on `call()`
works can be found on Robert Sosinski's excellent post on [Binding Scope in
JavaScript](http://www.robertsosinski.com/2009/04/28/binding-scope-in-javascript/).

There you have it. Except, if you use Internet Explorer. It turns out that
Internet Explorer versions before Internet Explorer 9 cannot handle calling
`slice()` on NodeLists --and in general behave badly on host object
interactions.

One way to fix that problem is simple create a new array, iterate over your
existing NodeList and push things into your new array.

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

This is a good start but however there's a slight issue with that code that
[James-David Dalton](http://twitter.com/jdalton/) pointed out. It's possible
for a NodeList to have an element in it that has the id `length`. In that
case, the expression `myList.length` would no longer be an integer but the
element with the id `length` itself. Nevertheless, this is easy to fix.

{% highlight javascript %}
    > var node, i = -1, myIEArray = [];
      undefined
    > var myList = document.querySelectorAll('.story-item');
      undefined
    > while (node = myList[++i]) { myIEArray[i] = node; }
      <div class=​"story-item">​…​</div>​
{% endhighlight %}

This code involves a bit more trickery. In essence, we are again looping over
the elements in the `myList` NodeList but making sure that a given index is
not `undefined` which is false in JavaScript. The reason we initialize our
index variable `i` to -1 is that we increment the index before we actually
try to access the `i`th index. So if we were to set the `i` to 0 instead of
-1, we would both miss the first element in the MyList NodeList and also start
setting the elements in the `myIEArray` from its 1st index, leaving
`myIEArray[0]` undefined.

There you have it, for real this time, a real array from your NodeList.

One thing is worth mentioning though; when you convert your NodeList into an
array, you are no longer dealing with a _live_ NodeList (again,
`document.querySelectorAll()` actually returns not live but static NodeLists)
but instead an array of DOM nodes.

Converting a NodeList into an array has some interesting consequences. Going
back to our example where we ran the `myArray.pop()`, you'd not the top Digg
story disappearing as your array is just a collection of DOM nodes, not
representation of your DOM anymore.

However, the DOM objects themselves are _live_. So if you were to do something
like `myArray[0].innerHTML('foo bar')`, you would instead change the DOM.

NodeLists are a powerful tool and with more and more browsers getting decent
DOM APIs, you might find yourself dealing with more NodeLists instead of
jQuery objects and such.

In fact, if you look at the source code for Thomas Fuchs' Zepto.JS
micro-library, you'll find that [it implements](https://github.com/madrobby/zepto/blob/c03bef955913afa858116538e59c6e7a6ac04207/src/zepto.js#L68) arguably the most important
aspect of jQuery, the CSS DOM selector using the (relatively) new
`document.querySelectorAll()` method and converting the NodeList to an array
for easy further manipulation.

I hope this brief introduction to NodeLists (and arrays) has been useful.
Please let me know in the comments if you have any questions or corrections.