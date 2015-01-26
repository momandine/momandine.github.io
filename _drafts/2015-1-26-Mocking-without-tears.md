---
layout: post
category : code
tagline: "Things that are kind of hard"
tags : [python, mocking]
---
{% include JB/setup %}

This time last year, I was a n00b programmer. I hope I think that about myself every year, in fact. As my friend Alex Clemmer (@hausdorff_space) once said to me even more than a year ago when I was even more of a n00b programmer, the nice thing about needing to build basic skills is that you know you'll usee those skills in the future. They're the basis of further specialized knowledge, which for all you know could just be mental masturbation masquerading as learning. (There's absolutely nothing wrong with, in fact many excelellent things about, self-love, but it's well known that it will not cause procreation). 

Enough philosophical musing, this post is about mocking. I'd never really written tests for projects previous to becoming a professional software engineer - they tended to be provided to me by a professor as part of the project, or since I wrote a lot of Python, I'd mess around in the REPL or run the code (not run the jewels) until I was satisfied my code was correct. I wish I'd written this post back then, so I have very fresh in my mind exactly what was confusing about different types of unittesting, but this will be a good exercise in empathy for your past self.

I loooooooooove tests. 


Here's the thing about open source: the things that are in the standard library are not necessarily the things best suited to your usecase, so there's a tension between sticking with what's simple and reducing the scope of your dependency graph and not using the cutting edge thing.


Mocking 
-------
Everything I write will be in the context of Python, but I hope there will be some universal themes.

vmock: https://github.com/vburenin/vmock

how to read legacy codebases

pytest vs. unittest. 
