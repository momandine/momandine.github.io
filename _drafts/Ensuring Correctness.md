---
layout: post
title: "Ensuring Correctness, or What I've Learned The Hard Way About Getting Things to Actually Work"
description: ""
category: 
tags: []
---
{% include JB/setup %}

I've been programming for about 6 years, in some capacity or another. I think a lot of my maturation as a software engineer has come from gaining an understanding (or at least an opinion) of when, how, and how much to test, where "testing" includes any activity I can undertake to try and verify that my code works as I intended it to. (Whether the strategy I pursed was effectively solved the underlying need that caused me to write software is another story.)

I don't think I'm done with this process, but it seems worthwhile to write these things down now, while I marginally remember being a beginner. So, here is a record of my evolution in ensuring correctness to date, with many turning points written as misadventures. 

### First off, run the code.
Even when I was a first time programmer, writing MatLab code in a Mathemetical Methods for Physics class in college, I don't think I tried to turn in assignments without running it. However, I did (and do to this day), write code, look at it, think it's right, and then run it only to watch it crash and burn, output entirely the wrong thing, or completely fail to compile. 

I don't want to undervalue taking only this first step. Sometimes it's all that needed. I wrote a little script to move files from one place to another the other day. If it moved them to the wrong place, it was recoverable, so I just wrote it, executed it on one file to make sure it did the right thing and smoke out syntax errors, and then added a for loop.

### Print statements and code review
The first two things I learned to troubleshoot obvious incorrectness after unning code are probably first two most people learn:

1. Evaluate small chunks, ideally in context, and see if they are what I expected. MatLab and Mathematica, the first two tools I learned, both have interactive ways to evaluate small pieces without the whole. MatLab also has `disp` to insert into larger programs to do print-statement debugging.

2. Get someone else to look at it. A second set of eyes can catch that typo when yours have glazed over. Frequently, this was a classmate or lab parnter. 

### Debugging tools
I took a intro-level Data Structures and Algorithms course in college that was in C++. My labmate and I were absolutely stuck on some segault, until one of our friends, who was a couple of years ahead of in coursework and is in fact (currently an esteemed member of C++ community)[://meetingcpp.com/index.php/sv16/items/33.html], came by and showed us Valgrind. 

What a revelation! It was so much to easier figure out what's going on when I had some kind of independent window on your code, that came with an instruction manual and that I didn't have to write. This is the step where I learned that other people had process tips and tools that would probably help, which I didn't always seek to thoroughly understand before using.

### Static analyzers
You know what's great? A good integrated development environment. Previously, I'd used environments given to me by my instructor to learn a language - Dr. Racket, Eclipse, MatLab, etc. They had lots of features, some of which i was required to learn via structured courseowrk. 

I learned Python at Recurse Center (then called Hacker School), in the summer of 2013. This was the first time I was given free reign over my workflow. I was used to Vim from school Linux terminals, and it was fashionable, and so I cast about for Vim plugins for syntax highlighting, autocomplete and things like that to make my life easier. Then I found PyCharm.

You know what's great about PyCharm? It has a bunch of built in static analyzers, things that parse the code and look for known pitfalls, in *addition* to basic syntax highlighting to make the code more readable. It can find type errors or mismatched input arguments before I even run the code. Some compiled languages have these things built in to the compilation step. Regardless, there exist quick tools that simply parse and analyze your code and avoid a lot of small errors. Recently, over the course of 3 days, [mypy](http://mypy-lang.org/) and static type checking for Python caught three bugs in code before I had even run the code. 

## Unittests
When I started my first job, I was suddenly writing things other people had to build on top of and rely on. Culturally, I was encouraged to write unittests. I think a lot of people me included, are told that they are a best practice, and that they "should" be doing it before it is clear how much they actually help. This was especially true for me given that all my previous programming endeavers were write-once, throw-away school or side projects.

Writing unittests is hard! There is a cost to any kind of testing, especially ramping up on a new set of tools. When I first began there was a big barrier to entry for me to learn how to effectively use mocking libraries. Mocking means replacing real, production-accurate bits of code with faked-out versions of functions or objects that can dumbly parrot whatever you tell them to. My tech lead had written his own mocking library for Python, called [vmock](https://github.com/vburenin/vmock) and was evangelizing it to the entire team. Some other parts of code used the standard python mock library. They had different philosophies, different APIs, and different levels of documentation. 

I had also never thought about writing code that is easy to test before. Because I had taken a class in Racket, I frequently thought about functions as abstracted bits of work that could be composed together. That's certainly neat, but writing unittests 

I'm not by default an authority skeptic, so I dutifully progressed up the unittesting curve to satisfy my code-reviewers, and I don't remember a singular moment when I became a devotee. So today, why do I write unittests?

- To reduce the chance that small units of code are broken. 

If 10% of my code has bugs on the first try, then the chance that both my test and the code I'm testing have complimentary bugs that cause false positives (assuming no confounding factors), should be less than 1%. The more I write code, especially when that code executes in enviornments that are hard to spin up (i.e. has a lot of dependencies on other services), or is a bad idea to execute live (i.e. does operations that affect other people), the more I just want to write the test once, the more I just want to write a little proof that each new bit of code works and move on.

- To document what that expected behavior is for posterity.

Assuming the tests are run regularly and kept green passing, the tests are forced to stay up to date with the code, unlike docstrings or external documentation. If I break someone else's unittest, I will either update the test to reflect the new behavior, or figure out what I misunderstood. I assume future people working on my code will do the same. Unfortunately, it's easy to value a passing test over actually making sure the test tests someothing useful. But it's a start. 

At this point, it has become such a habit that I often write unittests for untested code in order to understand it, and sometimes refactor it to make it more unittestable if that's really hard to do. 

## Integration tests.
Early at Dropbox, I was trying to fix a regression in the screenshots feature on a new operating system version that wasn't yet in wide release. I wrote some new unittests, ran the code on the target system, and felt pretty confident. In the process, due to subtle differences in the operating system APIs (which I had mocked out in the unittests) I broke it on every version of the platform except the one I was repairing.

This shows two places that unittests do not ensure correctness:

1. Mocks encode your assumptions, and can lead to false positives. 

Unittests are intended to pass or fail deterministically, and therefore cannot rely on outside depenencies. Even if I'm very careful about mocking as little as possible, I will have to feed your code inputs and check the outputs according to what I assume the outside systems expect. If my code calls a remote API with the wrong HTTPS method, because I just say, misread the documentation, unittests are not going to help.

2. Unittests are notoriously difficult for UI code. 

Any person executing the code would have noticed that a screenshot dialog didn't appear when it was supposed to. But, it is *really* annoying and repetitive to run that code on every single platform every time I made a small change. 

This is where integration tests come in. They, for example, use UI automation libraries to click on buttons and make sure the correct windows appear, or ensure the server has the correct information when a new user signs up. 

Integration tests are written *without* the assumption of an isolated system, and test features end-to-end. This means they take more thought and time to write, usually including platform cost to make reusable test components. They are are also necessarily more expensive, flaky and difficult to debug. Therefore, especially in a client-side environment when each release can be expensive, they become necessary when you have integral, production-ready features that you would be testing manually to gate releasing new code anyway.

## Montoring, slow rollouts, and feature flagging
My first project at Dropbox touched our installer, which is a fairly critical code path. I had written unittests and done manual tests for all of the configurations that I could think of, and considered writing integration tests, but our platform wasn't ready yet and instead enlisted the help of manual QA engineers. But when the new feature was starting to be served to users, we got error reports from a handful of users, so we halted rollout and turned off the feature. It was mysterious! On later investigation these users turned out to have changed their operating system usernames at some point. 

Automated tests only test the platform that you run them on. For server side software, like I had worked on previously, where you control the code execution environemnt, this is unlikely to cause you issues. Client side software often supports a few to dozens of browsers or operating systems - and then there's everything else the end user might have unique about their system.

## Exception reporting and aggregating.

- Doing a big old migration. 

## Running replacement systems side by side

Maybe something about deploys how to lay the groundwork and don't really use something. Not sure I've really learned that.


## "Belt and suspsender" approach

- Swapping out backend with redudancy (i.e. darkwing). 


- creating canary environments for things that are difficult to test, rolling out, fallback functionality ,and looking at the results. 
- Making tests that are actuallly a suite of containers. Deploy scripts!

- how do you test tools


TODO read and understand anything from that chapter of SRE


Not to knock the utility of "just run it." I wrote a script last week that needed to only be run once, wouldn't touch anything visible to other software engineers or customers, and wasn't checked into source control. Here's how I got it to work:

1. Write core function.
2. Run it.
3. Fix any exceptions or syntax errors (a compiled language may substitue in "get it toc compile"
4. Run again.
5. Make sure the output looks approximately right.
6. Put in a loop to call the function several, repeat steps 2 -5.
7. Add some flags for configurability, repeat steps 2 - 5 ... etc.

No one ever looked at it. I wrote no tests. It worked. But it's not the whole story for most code.

