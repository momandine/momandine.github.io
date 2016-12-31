---
layout: post
title: "new post"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Ensuring Correctness

I work on Release Engineering at Dropbox, which means running the service that does continuous integration and deployment. Before that I worked on applicaiton software for the Desktop Client, and before that a pre-launch internal web service for an enterprise security product at Juniper Networks. A lot of the maturation I've done as a software engineer has to do with understanding when, how, and how much to test.

I expect this sense will only develop more, but here is a rough outline ensuring correctness depending on the scope of project. My other caveat is that I primarily work in python). 

## Run the code.
I wrote a script last week that needed to only be run once, wouldn't touch anything visible to other software engineers or customers, and wasn't checked into source control. Here's how I tested it:

1. Write core function.
2. Run it.
3. Fix any excpetions or syntax errors (this is Python, some some classes of errors may have been caught at the complication 
4. Run again.
5. Make sure the output looks approximately right.
6. Put in a loop, rpeat steps 2 -5, add some flags for configurability, repeat steps 2 - 5.

This is similar to the way I coded for projects in college - though occasionally I'd need additional debugging tools like valgrid, debuggers etc. It works pretty well if you don't mind playing whack-a-mole with bugs by fixing one and creating another in another place. IT

## Unittests.
When I started my first job, I was suddenly writing things other people relied on for the first time. My tech lead had written their own mocking library, and was evangelizing it to the entire team. 

At the time there was a big barrier to entry for me to learn how to effectively mock out components (a topic that deserves it's own psot), and also how to write code that is easier to test in the first place. Functions without side effects, clean interfaces, etc are all nice outcomes of thinking about writing test code, but the real point of all that effort is this:

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

## Running replacement systems side by side


## "Belt and suspsender" approach


- how do you test tools
- creating canary environments for things that are difficult to test, rolling out, fallback functionality ,and looking at the results. 

