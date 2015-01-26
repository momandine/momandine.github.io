---
layout: post
category : code
tagline: "Things that are kind of hard"
tags : [python, mocking]
---
{% include JB/setup %}

This time last year, I was a n00b programmer. I hope I think that about myself every year, in fact. As my friend Alex Clemmer (@hausdorff_space) once said to me even more than a year ago when I was even more of a n00b programmer, the nice thing about needing to build basic skills is that you know you'll usee those skills in the future. They're the basis of further specialized knowledge, which for all you know could just be mental masturbation masquerating as progress. 

Enough philosophical musing, this post is about mocking. I'd never really written tests for projects previous to becoming a professional software engineer - they tended to be provided to me by a professor as part of the project, or since I wrote a lot of Python, I'd mess around in the REPL or run the code (not run the jewels) until I was satisfied my code was correct.

I wish I'd written this post back then, so I have very fresh in my mind exactly what was confusing about different types of unittesting.


Mocking 
-------
Everything I write will be in the context of Python, but I hope there will be some universal themes.


