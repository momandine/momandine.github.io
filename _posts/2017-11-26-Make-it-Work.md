---
layout: post
title: "Make it Work"
description: "A saga of mistakes and learnings in getting code running the way I want it to"
category: 
tags: ["testing"]
---
{% include JB/setup %}


I've been programming for about 6.5 years, in some capacity or another. I don't know if you ever try to remember what it was like to be a past version of yourself, but I do sometimes, and it's *hard*. Remembering not knowing something is really especially difficult - this is a big challenge in teaching.

I’m better at programming than past me. I’m MUCH better at getting things to work reliably. My effort is more efficiently expended, while my standards for "working" have changed - the code I write might be part of a collaborative project, roll up into a larger system or service with emergent behavior, or have a lot of potential edge cases. 

This is what I remember of some key turning points in my “wake it work” strategies. Like most learning, a lot of my knowledge came from ~~failure~~ educational misadventure. 

#### Run the code

Like many people I first learned to program in a class. I studied physics, so our goals were usually to get a simulation working to demonstrate a concept, but i could in theory turn in my code. without running it at all. If I did 9 times of of 10 there would be some glaring bug.

So the first line of the defense in getting code to work is to run the code. Sometimes this is still enough - when writing little scripts to assist me I often write once and run once. Sometimes, when the environment is tricky or the computation takes a long time, it is actually quite hard to do.

#### Print statements

Running code only tells you if the inputs and outputs are correct, and maybe any side effects that you can obviously observe. Intermediate checkpoints are incredibly useful. One of the first tools I learned in MatLab for physics computation was `disp`. 

#### Code review

In my college programming classes, we were encouraged to work with a lab parter. Getting someone else to look at your code can smoke out obvious mistakes - just the other day a coworker pointed out that a bug I thought was complicated was just a misspelling of the word “response”.

#### Out-of-the-box debugging tools

My second Computer Science course was Data Structures and Algorithms in C++ -- which meant memory management. My labmate and I were absolutely stuck on some segault, until a friend a couple of courses ahead at the time (and later a contributing [member](https://vortex.pp.dropbox.com/alert/alert/227221) of the C++ community) came by and showed us Valgrind, in exchange for oreos. I didn't totally get what was going on, but having an independent program with a view onto my totally malfunctioning, un-print-statementable code was key.

#### Static analyzers

You know what's great? IDEs. During college, I'd used environments given to me by my instructor to learn a language - Dr. Racket, Eclipse, MatLab, etc. I learned Python at Recurse Center (then called Hacker School), in the summer of 2013, and for the first time I had total choice over my workflow. After experimenting with vim plugins, I found PyCharm, and a lot of the rough edges of Python were (and continue) to be smoothed.

You know what's great about PyCharm? It has a bunch of built in static analyzers, things that parse the code and look for known pitfalls, on top of standard syntax highlighting. PyCharm now supports type hints, which I cannot bang the drum of enough. 

You might be thinking “my compiled language doesn’t have these problems”, but languages do not (and maybe should not) implement every feature you want by default. I was delighted to learn recently that there is a syntax and checker for null exceptions in Java, and seems to be similar to “strict optional” for Python.

#### Unittests

My first time writing code that other people truly relied on, was at my first software engineering job at Juniper Networks. I worked on a team of about twelve people. I was encouraged to write unittests, and people would block code reviews if I didn't.

The thing is, writing unit tests is hard! There is a cost to any kind of testing, and an up-front cost of ramping up on a new set of tools. The big barrier to entry for me was learning how to effectively mock out components. Mocking means replacing real, production-accurate bits of code with fake functions or objects that can parrot whatever you tell them. My tech lead had written his own mocking library for Python, called [vmock](https://github.com/vburenin/vmock) and evangelized it to the entire team. Some other parts of code used the standard python mock library. They had different philosophies, different APIs, and different documentation.

I still run into barriers to entry in learning a new testing paradigm today. I wanted to add a tiny bit of logging to an unfamiliar codebase recently, and reading and understanding the test codebase felt like a huge amount of overhead. Thankfully the existing unit tests on this code saved my butt - that tiny bit of logging had about 3 SyntaxErrors in it.

This brings up why unit testing is useful:

- It reduces the chance that small units of code are broken. If 10% of your code has bugs, then the chance that both your test and production code have complimentary bugs that cause false positives should be less than 1%.
- It documents what the expected behavior is for posterity. Assuming the tests are run regularly and kept green, the tests are forced to stay up to date with the code, unlike docstrings or external documentation. It's a great way to remember what exactly the inputs and outputs to something should look like, or what side effects should hapen.

At this point, unit tests have become such a habit that I often write them for untested code in order to understand it. I also write them for anything new that I will share with others. Unit tests can be  the happy outcome of laziness and fear. Writing a unit test when you understand code well is less work than running it over and over by hand, and you're less likely to embarrass yourself by presenting someone else with broken code.

#### Integration tests

Early at Dropbox, I was trying to fix a regression in the screenshots feature on a new operating system version that wasn't yet in wide release. I wrote some new unit tests, ran the code on the target system, and felt pretty confident. In the process, due to subtle differences in the operating system APIs (which I had mocked out) I broke it on every version of the platform except the one I was repairing. It rolled out to some users, who caught it. I could probably point you to the forums post with the reports :/. 

Then I learned something about the limitations of unit tests:

1. Mocks encode your assumptions, and can lead to false positives.

Unit tests are intended to pass or fail deterministically, and therefore cannot rely on outside dependencies. It's very easy to inaccurately mock a web API, for example.  Even if you're very careful about mocking as little as possible, you will feed the tests a constrained sets of inputs and outputs that may or may not reflect real usage.

2. Unit tests don't test UI well.

Any person executing the code would have noticed that a dialog didn't pop up saying their screenshot was saved. But it is *really* annoying and repetitive to run that code on every single platform every time I made a small change. 

Enter integration tests. Integration tests often affect multiple system, often from the outside, and usually test features end-to-end. This means they take more thought and time to write. They are are also necessarily more expensive and are more likely to fail inconsistently. Frankly, they are not always worth the expense to write and maintain, but when the tradeoff is between an explosion of manual test cases or integration tests, integration tests are way to go. It is an interesting exercise to figure out how to insert test hooks, and may force you (like unit tests) to improve the modularity of the code. 

## Slow rollouts, flagging, and logging

My first project at Dropbox touched our installer, a pretty important bit of code for the success of the application. I had written unit tests and done a battery of manual tests. When the new feature was starting to roll out, we got error reports from a handful of users, so we halted rollout, turned off the feature, and furiously investigated.

The root cause was something I never would have thought of - they had changed the name of their home directory at some point while Dropbox was installed, which broke some assumptions in the application surfaced by my code.

My three lessons here: even integration tests or thorough manual tests only catch your known unknowns. You’re still vulnerable to the things you cannot anticipate. Also, having ways to quickly turn off code, from the server if possible, is fantastic for limiting the exposure of risky code. Third, testing in a small subset of users only works if you know the important high level of what’s going on while the code is testing, either from event logging “step complete” or error reporting.

## Acceptance testing

This is the rare case where I think I learned from best practices recommended by others. If you are going to cut over from one system to a supposedly parallel one, it is much better the run the new one "dark" and confirm they produce the same output rather than do the swap and wonder why things seem to be different, or why it's falling over under realistic. A recent surprise application: hardware misconfigurations. I help run a continuous integration cluster recently, and trying to add new hardware to expand capacity actually bit us when some machines were misconfigured.

## "Belt and suspsenders"

A year or so ago I worked a migration for of our built application binaries storage. Build scripts are really hard to test end-to-end without just making a build, since they mostly call other scripts and move files around, and I didn’t want to make too many extra builds. I already had a neat little acceptance testing plan with a double write. But it turned out I had runtime errors in the new posting code, which caused the script to crash.

The takeaway: when there's something critical and hard to run or write a test for, it's better to build a moat around your new bit (in this case, a try/except and logging would have done), and fall back to the old code if it errors for any reason. The tricky thing here is that the moat-code itself can be buggy (who amongst us has not mistyped the name of a feature flag on the first try, not I), so that moat-code needs to be well tested using the previous other tools. 

## Configuration is code

A lot of code is at least a little dependent on its running environment. It can be as simple as the flags passed into the main function, or as complicated as the other applications running on the machine at the same time. As previously mentioned, I work on a continuous integration service right now, and I have learned to respect the maxim “configuration is code” over and over. For one example, and we have a few “pet” machines that have been configured over time by many people. It is neigh on impossible to duplicate these machines, or look to a reference of how they were configured. This is why containers, and tools like chef and puppet are useful. For another, our scripts run via a coordination service that has a UI with hand-editable JSON configuration. While it may be nice to be able to change the flags passed into scripts with little friction, there is no record of changes and it’s difficult to deploy the changes atomically. 

I hope you enjoyed these accounts, and I may add on as I misadventure more and learn more :)

