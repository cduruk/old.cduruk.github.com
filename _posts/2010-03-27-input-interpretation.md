---
layout: post
tumblr_id: 477478721
date: 2010-03-27 19:24:00 UTC
title: Input Interpretation
---

It really drives me nuts when command line interfaces interpret what I am trying to do but only goes a certain distance.

Look at this screenshot from YouTube.

![YouTube](/static/images/youtube-entry.png)

So YouTube is smart enough to realize that I am looking for HD videos but it doesn't seem to be doing that search for me itself. Instead it gives me a link with some ambigous text. Do not get me wrong; this is far and away better than what most search engines in this specific case would do --and YouTube should be lauded for that-- but it's still a bit hacky for my liking.

Speaking of, I am pretty sure Python takes the cake with it's weird user input interpretation.

{% highlight bash %}
    $ python
    >>> exit
    Use exit() or Ctrl-D (i.e. EOF) to exit #what?
    >>> quit
    Use quit() or Ctrl-D (i.e. EOF) to exit #really?
    >>> quit()
    $ #prompt
{% endhighlight %}

That being said, it's at least a bit smarter than the Ruby command.

{% highlight bash %}
    $ ruby
    # Yep, nothing. Should have used irb, for some ungodly reason
    ^Cruby: Interrupt
    $ #prompt
{% endhighlight %}

But perhaps, it's better to have your input interpretation not so smart. I mean, it can get outright creepy at times.

![Google](/static/images/google-penguins.png)