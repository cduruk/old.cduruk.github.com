---
layout: post
categories: [javascript, css, ui, dev, tablet, touchscreen]
tumblr_id: 473067888
date: 2010-03-25 20:00:00 UTC
title: Some CSS and JS Tricks for Tablet Web Apps
---

I am currently working in a team creating a web application that is supposed to be used on a touch screen, not a touch screen phone but an actual tablet PC that we we were using a touch screen. That rather unconventional setup initially created some problems all of which I'll document how to solve.

One word of caution; these are not the kinds of problems you'd face if you were developing a web application for a touch screen device running a touch screen optimized device (yes, I am talking about the iPhone and the iPad). I just wanted to document our solutions for my own benefit and in the name of helping others.

### Moving cursor on a Tablet
First of all, although the tablet PC we had sported a capacitive touch which meant that we could use our fingers, there were problem with differentiating the difference between a touch or a move of the cursor.

This is a hard problem to describe but essentially the problem stems from the fact that capacitive touch screens require a lot of pressure (compared to resistive touch screens) to register a click. This, combined with the fact that our target demographic had not so great fine motor skills, created a unique challenge; a lot of the time the users would touch the screen but the touch would register a movement.

The solution we came up with that problem was treating any movement of the cursor that was abrupt enough as a click. We set up a time threshold and then compared where the mouse was in intervals of that threshold and if the mouse moved in that threshold, we'd just make the mouse click. It is an ugly JavaScript hack but the effect it had on our usability was truly stunning.

### Removing the Cursor Entirely
Another problem that came up was getting rid of unnecessary and distractive UI elements such as the cursor and the scrollbars.

Initially, we tried to get rid of the cursor directly in the browser, using this small CSS trick.

{% highlight css %}
    * {
      cursor: none;
    }
{% endhighlight css %}

This code seemed to work in our browsers (Google Chrome running on Mac) but didn't work in the production environment (Google Chrome running on Vista). Finally, we came up a rather hacky solution; we just changed the system-wide cursor from an arrow to a single 1 pixel dot. This essentially made sure that there would be no cursor what so ever. Although this has made the computer pretty much impossible to use with the touchpad but we got around that by using the stylus (or just our fingers).

### Removing the Scrollbars
One other small problem was getting rid of the scrollbars. Since we had total control over our production environment (and thus the browser), I just used a new CSS3 trick I learned.

{% highlight css %}
    body {
      overflow-x:hidden;
      overflow-y:hidden;
    }
{% endhighlight css %}

### Hiding the Selection Highlighting
The last problem we had to fix was caused by how people used the touch screens and admittedly was the most interesting one. The problem was caused by how our users had never used a touch screen before. I could tell you all about it but a picture would probably give you a better idea.

<img src="http://media.tumblr.com/tumblr_kzusfqufPn1qazj16.jpg" />

Here is the crux of the problem: instead of simply tapping (Hi Apple HIG documents) the touch-screen, our users would touch and almost invariably move their fingers ever so slightly on the screen so that the browser to select essentially every single element. Not only it looked ugly, it confused our users and we had to fix it.

I initially tried a very complex solution; put every single element on the page behind another invisible element so that the elements that were behind that invisible curtain would not get selected. That solution however meant that we had to change our JavaScript that did the movement calculation and was in general an inelegant solution.

Thankfully, CSS3 again came to my rescue.

{% highlight css %}
    *::selection {
      background: transparent
    }
{% endhighlight css %}

What this does is makes the background of selection pseudo-element on every single tag transparent. Since the selection pseudo-element essentially consists of just color and background properties, setting the background transparent was enough to do the trick.

### More
I have created a gist on GitHub for all those who need the code in a central location. But what's even better is that the entire web app is actually open source (just because it is easier for us to deploy the app to the production environment with a single git command).

*  [Gist](http://gist.github.com/340689)
*  [Touchkiosk](http://github.com/cduruk/touchkiosk)