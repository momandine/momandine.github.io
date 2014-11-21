---
layout: post
category : unix
tagline: "Not sure what a tagline is for yet"
tags : [unix, command line]
---
{% include JB/setup %}

Batch Editing a File 3 Ways
===========================

I knew the day would come when I would want to edit a group of almost identical 
files all at once. In the past I've been told that AWK and sed and Vim macros were all
useful to this end, but while I maybe have looked at a few tutorials, without having a problem 
to apply it to, nothing stuck.

Well, my day has come.

The context: I needed to add a few lines to a bunch of very similar config files in the same directory.
Tedius to do by hand, convenient to do by computer.

1. Vim Macros
-------------
I'm an avid Vim user but not what you'd call an *advanced* one. I mostly navigate around the 
file, delete and replace some chunks of text, and occassionally hit keys that trigger things 
that I do not yet understand and then press escape (by which I mean CAPSLOCK, obvi) as many times as 
possible until things reset.

Many a time have I ended up with this little word in the bottom left:

![recording]({{ site.url }}/assets/images/recording.png)

Turns out it's recording a vim macro! Cool! For those of you who want to cut to the quick, the basic commands
are as follows:

    q
    @@

I knew that there were things called "registers" in Vim,
where you can copy things. Tursn out


{% highlight vim %}

{% end highlight %}

 

