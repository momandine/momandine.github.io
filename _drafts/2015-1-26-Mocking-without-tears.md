---
layout: post
category : code
tagline: "Things that are kind of hard"
tags : [python, mocking, testing]
---
{% include JB/setup %}

Last winter, I was a n00b programmer. I hope I think that about myself every year, in fact. 

In any case, I'd never really written unittests prior to becoming a professional software engineer, and a couple of months into my first job, I had to unittest all the things I had been working on. These days I looooooooooooooooove tests, but writing pure unittests has a surprisingly sharp learning curve.

Everything I write will be in the context of Python, but I hope there will be some universal themes.

What makes a good unittest?
===========================
At the root of all "best practices" is a set of target values. When people disagree over best practices, they are probably arguing over values and/or assumptions. 

For me, unittests are useful when I can run them habitually after every code change and confirm that I'm not totally off my rocker about the code's correctness,^ Or, if I'm observing unexpected behavior or exceptions in the system as a whole, unittests are hopefully a way to quickly narrow done the scope of possible problems.

They should therefore:
1. run quickly
2. and deterministically
3. on pretty much any system, and
4. test reasonably small areas of code, and
5. be clear in what the expected behavior of the test is. 

^ caveat: Sometimes your code and your tests are both broken in a way such that the test gives a false positive. For example, I frequently get a happy green check, celebrate at my passing tests, and then realize that I forgot to include a critical assert statement and am really testing nothing. While neither source code and test code are perfect, I like to think about it like extra layers of oversight on a critical system. Your average airline pilot may be perfectly safe to fly alone 96% of the time (actually a much higher success rate than untested code), but adding automated anti-doing-obviously-stupid-things systems (tests) will catch their mistakes 80% of the time and finally a co-pilot (code reviewer) is another great backstop in more flexible problems, and as this probabilities stack up, altogether you can feel pretty good about putting your mortal vessel into a highly velocitized metal can flying miles up in the sky.

From these we, can gather some more practical principles. Unittests should:
1. *Not use external resources*, like the internet or other applications. Otherwise, you can't run them on the airplane, or if that thing isn't installed, and then you'll stop running them.
2. *Leave state the same as they found it*. Otherwise, you will be disincentivized to use them if your hard-drive becomes bloated with leftover files and the like.
3. *Obviously correspond to units of code, with as few dependencies as possible*. Java people talk about writing a test that runs on a VacuumSealedClass. I think you should isolate your code as much as produces a good return on your investment. If you can get the module alone without too much work, great. If it's hard to unentangle from some other dependency, then at least test the shit out of that dependency too, so when something breaks you can tell the difference between both A and B test breaking and just B breaking.
4. *Have comments! And useful function names!* And all those usual code quality standards! Because banging your head against the wall trying to understand what a test does is less fun than banging your head against a wall trying to figure out what production code does. I would also let to make a shoutout to good test organization, because figuring out what tests might relate to the module you're modifying makes life easier.
5. *Never be broken* on your master branch/whatever you use to keep track of the most highly polished code. Tests are a source of truth. Tests that fail on master are as detrimental to your suite as gaffes to a politician - they take away trust from the test suite/politician and then people don't vote for them anymore. Flakey tests, which pass sometimes and not others, are also a pain, because then people don't know if they can trust a pass and get anxiety even releasing a green build. 

What does this all have to do with mocking?
===========================================

I cannot think of a way to implement any of these principles without mocking. Unfortunately, mocking is also kind of a pain. 

A mock database will allow to you submit queries, and get back exactly the expected response without having to run database software. A mock network call will allow you to make sure your code can handle a 404 error. 

What does this all have to do with mocking?
===========================================

I cannot think of a way to implement any of these principles without mocking. Unfortunately, mocking is also kind of a pain. 

Definiton of "to mock": _To replace a complicated production with a simple, predictable one._

JUnit 


http://www.hackerchick.com/2008/06/my-unit-tests-are-purer-than-yours.html
Here's the thing about open source: the things that are in the standard library are not necessarily the things best suited to your usecase, so there's a tension between sticking with what's simple and reducing the scope of your dependency graph and not using the cutting edge thing.




Mocking 
-------

vmock: https://github.com/vburenin/vmock

how to read legacy codebases

pytest vs. unittest. 

Lita's blog post about learning Mock. http://www.blaggregator.us/post/nJ98va/view
