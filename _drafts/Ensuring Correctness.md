---
layout: post
title: "Make it Work"
description: "Growth and misadvetnure in ensuring code correctness."
category: 
tags: []
---
{% include JB/setup %}

I've been programming for about 6.5 years, in some capacity or another. I don't know if you ever try to remember what it was like to be a past version of yourself, but I do sometimes, and it's *hard*. I think this phenomenon makes teaching difficult for those with domain expertise but not a lot of teaching experience. A potential teacher may have a much broader and deeper understanding than their students, and could give holistic context our point out strategic redirection. But likely, they've forgotten what it's like to learn concepts and tools they breathe day to day, and have a really hard time explaining their ideas in a way approachable to their students.

Pedagogical musings aside, I'm better at programming than past me. I think I'm especially MUCH better at getting things to work reliably, with effort more efficiently expended, than past me. My standards for "working" have changed too - the code I write might be part of a project others collaborate on, or a part larger system or service with emergent behavior, or have a lot of potential edgecases. This is what I remember of some key turning points in my strategies - and most of them were bugs, feature breakages, or outages that seared that learning into my brain 

## Run the code
Like many people I first learned to program in a class. I studied physics, so our goals were usually to get a simulation working to demonstrate a concept, but in my first Computer Science class no one ever made me actually run the code. I could turn in some Java written from conjecture (I had some exams like this, and whiteboard interviews), and 9 times of of 10 it wouldn't actually compile the first time. I write Python most often now, and I still get SyntaxErrors and ImportErrors and so on the first time I try to run something - which is 100% normal.

So the first line of the defense is to run code. Sometimes this is still enough. I wrote a little script to move some stuff around on my computer, with backups and no real consequences, and just running it and fixing apparant bugs worked pretty well for my purposes.

## Print statements and code review
Getting intermediate from code that is not working as expected is incredibly useful. At this point I don't remember my first few lines of MatLab very well, but one of the first bits we learned was how to print things to the console. Since then I've taught JavaScript to high schoolers, and I absolutely loved watching check they're understanding of what was going on throughout the program. From there you can bisect, and construct bigger chunks from smaller ones you're pretty sure about.

The other crucial thing is getting a second pair of eyes, or yours after some time away. My college courses generally had lab partners, and I worked in groups for physics assignments involing a lot of calculation for this reason. Another person can ask questions and check your assumptions.

## Out-of-the-box debugging tools.
My second Computer Science course was Data Structures and Algorithms in C++ - which meant memory management. My labmate and I were absolutely stuck on some segault, until a friend a couple of courses ahead at the time and current an esteemed [member](https://vortex.pp.dropbox.com/alert/alert/227221) of the C++ community came by and showed us Valgrind. (I think it was in exchange for oreos.) I didn't totally get what was going on, but having an independent program with a view onto my totally malfunctioning, unprintstatementable code was very helpful. 

## Static analyzers
You know what's great? IDEs. During college, I'd used environments given to me by my instructor to learn a language - Dr. Racket, Eclipse, MatLab, etc. I learned Python at Recurse Center (then called Hacker School), in the summer of 2013, and his was the first time I was given total choice over my workflow. There are many barebones editor devotees, and I tried some plugins for ViM, but then I found PyCharm, and a lot of the rough edges of Python were (and continue) to be smoothed.

You know what's great about PyCharm? It has a bunch of built in static analyzers, things that parse the code and look for known pitfalls, in addition to syntax highlighting. It now supports type hints, which I use prolifically in combination with MyPy at Dropbox. I was delighted to learn recently that strict null annotations and checking exist for compiled languages to - the addition of strict optional to Python has made me a better programmer and caught dozens of bugs for me.

## Unittests.
My first time writing code that other people truly relied on, both to build on top of and to run, was at my first software engineering job at Juniper Networks. I worked on a team of about 12 people. I was encouraged to write unittests, and people would block code reviews if I didn't.

The thing is, writing unittests is hard! There is a cost to any kind of testing, especially ramping up on a new set of tools. At the time there was a big barrier to entry for me to learn how to effectively mock out components. Mocking means replacing real, production-accurate bits of code with fake functions or objects that can parrot whatever you tell them. My tech lead had written his own mocking library for Python, called [vmock](https://github.com/vburenin/vmock) and was evangelizing it to the entire team. Some other parts of code used the standard python mock library. They had different philosophies, different APIs, and different documentation. 

I still run into barriers to entry in learning a testing paradigm today. I wanted to add a tiny bit of logging to an unfamiliar codebase recenntly, and reading the test code and figuring out the patterns would potentially double the amount I had to learn. But, the existing unittests on this code saved my butt - that tiny bit of logging had about 3 SyntaxErrors in it.

This brings up why unittesting is useful:

- Reducing the chance that small units of code are broken. If 10% of your code has bugs, then the chance that both your test and production code have complimentary bugs that cause false positives, should be less than 1% (if there are no confounding factors).

- Documenting what that expected behavior is for posterity. Assuming the tests are run regularly and kept green, the tests are forced to stay up to date with the code, unlike docstrings or external documentation. It's a great way to remember what exactly the inputs and outputs to something should look like, or what side effects should hapen.

At this point, it has become such a habit that I often write unittests for untested code in order to understand it. I also write them for anything new that I will share with others. It is the happy outcome of laziness and fear - writitng a unittest when you understand code well is less work than running it over and over by hand, and you're less likely to present someone else with clealry broken code.

## Integration tests.
Early at Dropbox, I was trying to fix a regression in the screenshots feature on a new operating system version that wasn't yet in wide release. I wrote some new unittests, ran the code on the target system, and felt pretty confident. In the process, due to subtle differences in the operating system APIs (which I had mocked out in the unittests) I broke it on every version of the platform except the one I was repairing. It rolled out to some users, who caught it. It was pretty embarassing. 

But I learned something about the limitations of unittests:

1. Mocks encode your assumptions, and can lead to false positives. 

Unittests are intended to pass or fail deterministically, and therefore cannot rely on outside depenencies. It's very easy to inaccurately mock a web API for example. Even if you're very careful about mocking as little as possible, you will feed the tests a constrained sets of inputs and outputs that may or may not reflect real usage. 

2. Unittests don't check that the UI looks right.

Any person executing the code would have noticed that a dialog didn't pop up saying their screenshot was saved. But it is *really* annoying and repetitive to run that code on every single platform every time I made a small change. This is where integration tests come in.

Integration tests are written *without* the assumption of an isolated system, and test features end-to-end. This means they take more thought and time to write. They are are also necessarily more expensive and are more likely to fail inconsistently. So they are not always worth it, but when the tradeoff is between an explosion of manual test cases or integration tests, integration tests are way to go. 

## Logging and slow rollouts.
My first project at Dropbox touched our installer, a pretty important bit of code for the success of the application. I had written unittests and done manual tests for all of the configurations that I could of with the help of QA. When the new feature was starting to rolled out, we got error reports from a handful of users, so we halted rollout, made a new version with the feature turned off, and furiously investigated. 

The root cause was something I never would have thought of - they had changed the name of their home directory at some point while Dropbox was installed, which broke some assumptions in the application surfaced by my code.

In this specific case we were using manual QA rather than integration tests because the framwork for testing was still in beta phase, but either way, end to end tests only test the platform that you run them on. I recently caused a similar bug because I didn't aniticpate that the application would lack permissions to write to a configuration file for a small percentage of users. At Juniper I worked on server side software, where we control the code execution environemnt, but client side software often supports a few to dozens of browsers or operating systems, so the best way to make sure it's actually working is s robust exception logging, montoring that reports behavior in the wild, and rolling out slowly and checking the results rather than all at once. 

## Acceptance testing
This is the rare case where I think I learned from best practices recommended by others. If you are going to cut over from one system to a supposedly parallel one, it is much better the run the new one "dark" and confirm they produce the same output rather than do the cutover and wonder why things seem to be different, or why it's falling over under realistic load. That beinga said this is an issue that still comes up regularly. I now help manage our CI cluster, and misconfigurations of new machines added to our cluster have caused issues that affect our reliability - when adding new capacity is supposed to improve it!

## "Belt and suspsenders"
A year or so ago I worked on moving where our build scripts put binaries. Build scripts are really hard to test without just making a build, since they mostly call other scripts and move files around, and you can't make infinite builds. I already had a neat little acceptance testing plan with a double write for fetching the binaries after. But it turned out I had runtime errors in the new posting code, which caused the script to crash.

The lesson learned here is that when there's something critical and hard to test, but that has a backup, it's better to build a moat around your new bit (in this case, a try/except and logging would have done), and fall back to the old code if it errors for any reason. The tricky thing here is that the moat-code itself can be buggy (who among us has not mistyped the name of a feature flag on the first try), so that moat-code needs to have all our other tools thrown at it. 

## Configuration is code
In the year or so I've gone back to primarily working on service code rather than application, and especially our continous integration service. My team has the power to choose choose to how to configure and deploy. 


- creating canary environments for things that are difficult to test, rolling out, fallback functionality ,and looking at the results. 
- Making tests that are actuallly a suite of containers. Deploy scripts!

- how do you test tools


TODO read and understand anything from that chapter of SRE


## Understanding your capcity
