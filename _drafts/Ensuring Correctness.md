---
layout: post
title: "Ensuring Correctness, or What I've Learned The Hard Way About Getting Things to Actually Work"
description: ""
category: 
tags: []
---
{% include JB/setup %}

I've been programming for about 6 years, in some capacity or another. I think a lot of my maturation as a software engineer has come from gaining an understanding (or at least an opinion) of when, how, and how much to test, where "testing" includes any activity I can undertake to try and verify that my code works as I intended to. 

I don't think I'm done with this process, but it can be hard to remember what it was like to learn something after enough time passes --- with the exception for specific mistakes where I learned a concrete lesson. So here is a record of my evolution in ensuring correctness so far, written as misadventures. (Not covered: trying to verify that I'm writing the right kind of product to solve the problem at hand).

### First off, run the code.
Even when I was a baby first time programmer, writing MatLab code in a Mathemetical Methods for Physics class in college, I don't think I tried to turn in assignments without running it. However, I did (and do to this day), write code, look at it, think it's right, and then run it only to watch it crash and burn, or completely fail to compile. 

### Print statements and code review
The first two things I learned to troubleshoot this are probably first two most people learn:

1. Evaluate small chunks, ideally in context, and see if they are what I expected. MatLab and Mathematica, the first two tools I learned, both had interactive ways to evaluate small pieces without the whole. MatLab also had `disp` to insert into larger programs. 

2. Get someone else to look at it. A second set of eyes can catch that typo when yours have glazed over.

### Debugging tools
I took a basic Data Structures and Algorithms course in college that was in C++. My labmate and I were absolutely stuck on some segault, until the person he was dating at the time, who was a more experienced programmer and is in fact (currently an esteemed member of C++ community)[://meetingcpp.com/index.php/sv16/items/33.html], came by and showed us Valgrind. 

What a godsend. So much to easier figure out what's going on when you have some kind of independent window on your code! This is the step where I learned that other people had process tips and tools that would probably help, which I tended to use without trying to understand them. 

### Static analyzers
You know what's great? A good integrated development environment. Previously, I'd used environments given to me by my instructor to learn a language - Dr. Racket, Eclipse, MatLab, etc. They had lots of features, some of which i was required to learn via structured courseowrk. 

I learned Python at Recurse Center (then called Hacker School), in the summer of 2013. This was the first time I was given free reign over my workflow, and so I cast about for Vim plugins for syntax highlighting, autocomplete and things like that to make my life easier. Then I found PyCharm.

You know what's great about PyCharm? It has a bunch of built in static analyzers, things that parse the code and look for known pitfalls, in *addition* to basic syntax highlighting to make the code more readable. It can find type errors or mismatched input arguments before I even run the code. Some compiled languages have these things built in to the compilation step.

## Unittests
When I started my first job, I was suddenly writing things other people had to build on top of and rely on. Culturally, I was encouraged to write unittests. Practically, they were a great way to document how something was supposed to work.

Writing unittests is hard! There is a cost to any kind of testing, especially ramping up on a new set of tools. At the time there was a big barrier to entry for me to learn how to effectively mock out components. Mocking means replacing real, production-accurate bits of code with fake functions or objects that can dumbly parrot whatever you tell them to. My tech lead had written his own mocking library for Python, called [vmock](https://github.com/vburenin/vmock) and was evangelizing it to the entire team. Some other parts of code used the standard python mock library. They had different philosophies, different APIs, and different documentation. 

I also needed to learn how to write code that is easier to test in the first place. Often times, when first Functions without side effects, clean interfaces, etc are all nice outcomes of thinking about writing test code, but the real point of all that effort is this:

- Reducing the chance that small units of code are broken. 

If 10% of your code has bugs, then the chance that both your test and production code have complimentary bugs that cause false positives, if there is no confounding factors, should be less than 1%. When your engineering team is writing or editiing hundreds of small units of code per day, basically making hundreds or thousands of mole-holes to keep covered, the risk savings adds up.

- Documenting what that expected behavior is for posterity.

Assuming the tests are run regularly and kept green (an important function of my current role), the tests are forced to stay up to date with the code, unlike docstrings or external documentation. My new coworkers would be annoyed if I broke something they made, and wanted to make sure that there was an automated bar for quality.

At this point, it has become such a habit that I often write unittests for untested code in order to understand it. I also write them for anything new that I will share with others. This is in part a product of laziness and fear; it shows I'm putting in effort to make sure code I might rely on works, without having to run it over and over.

## Integration tests.
Early at Dropbox, I was trying to fix a regression in the screenshots feature on a new OS version that wasn't yet in wide release. I wrote some new unittests, ran the code on the target system, and felt pretty confident. In the process, due to subtle differences in the operating system APIs (which I had mocked out in the unittests) I broke it on every version of the platform except the one I was repairing.

This shows two of the flaws of unittests:

1. Mocks encode your assumptions, and can lead to false positives. 

Unittests are intended to pass or fail deterministically, and therefore cannot rely on outside depenencies. Even if you're very careful about mocking as little as possible, you will have to feed your code inputs and check the outputs according to what you assume the outside systems expect. If your code calls a remote API with the wrong HTTPS method, because you just thought it was the wrong thing, it's not going to help.

2. Unittests are notoriosuly difficult for UI code. 

Any person executing the code would have noticed that a dialog didn't appear when it was supposed to. But, it is *really* annoying and repetitive to run that code on every single platform every time I make a small change. 

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

