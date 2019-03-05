---
layout: post
title: "Versioning and Release Process"
description: "An unexpected passion"
category: 
tags: ["testing"]
---
{% include JB/setup %}

I just wrapped up a couple of years working on things in the general space “release engineering” “continuous integration” and “delivery/auto-update”, with a bit of “build scripting”.

I now irrevocably think about software with two things in mind:

- versioning
- release process

These two bullet points are very closely related but quite the same thing. Just like software architecture, they are easy to over-engineer if you add a lot of structure too early, or to under-invest in if you wait too long. Ad-hoc designs that were totally logical and understandable at the time often become unwieldy and impair productivity.

### First, some definitions. 
#### Versioning

Let’s take **versioning** to mean any way to point to a snapshot of code. The formats can vary widely. Every commit in git is a distinct version described by a hash and some number of "refs" (branches and tags). End user software is usually published with a version - at the time of this example I’m running Chrome 64.0.3282.167 and PyCharm 2017.2.4 Build # PY-127.4343.27, on a MacBook with MacOS Sierra Version 10.12.6 G1651212.

In addition, versions are often used to embed a pointer into another piece of software, specifying that it should be incorporated as a dependency. There are programming language and operating system specific package managers designed to help us keep track of our versions and do upgrades (npm, apt, pip, brew), and configuration management tools to deploy changes of those versions (chef, puppet, salt, ansible). Dependency upgrades often cause cascading work to maintain compatibility, even just for consumer software. I personally have spent an unreasonable amount of time trying to install the right version of Java or Silverlight (and maybe had to upgrade my browser) to view a webpage or video. 

The version formats themselves encode information about release ordering and hierarchy, with any number of digits/letter, a date stamp, a unique unordered id or hash, or a catchy name. Numerals may have hidden information. At Dropbox we used to use even/odd middle digits to keep track of whether a Desktop Client build was experimental, and Ubuntu’s major number is the year of release, the minor is the quarter, and even major numbers imply “Long Term Support”. 


#### Release Process

This is a convenient segue into **release process**. Ubuntu does not want to have to maintain backwards compatibility forever, so it explicitly publishes a contract on the life cycle of the code. We are not all on the same version of Chrome (if you use Chrome), depending on whether you booted your laptop recently, how successful the auto-updater is, and maybe even whether you did some manual intervention like accepting new permissions or performing a force upgrade. Most larger projects have some way to feed higher risk-tolerance users earlier updates, with internal versions and/or alpha/beta testing programs. All of these versions have some cadence, some threshold of testing or quality, and some way they handle “ship-stopper” bugs. Rollouts can go even beyond the binary version and have A/B testing via dynamic flags to change code pathways depending on the user. 

There is a proliferation of dependency management tools because when one piece of software relies on another, they have to pay attention to new releases, evaluate whether it’s a breaking change, maybe patch the code, potentially introduce bugs (from dependency change, from the code change, or from unveiled assumption mismatch between the two<sup>1</sup>),  and then release… maybe causing similar effort for someone upstream. Not doing this work often causes a panic when end of support happens on a critical library. 

What's more, the same dependency might be used in more than one place in the same codebase or company. Can they be upgraded at the same time? Did someone assume they’d always be pinned together?<sup>2</sup> Do they have dependencies on *each other*, further expanding the decision/testing matrix?<sup>3</sup>


### The catch

I’m more or less convinced that when people say they like the low overhead of small companies, they’re not just talking about polynomial communication cost or political hierarchy. When you don’t have very much code, you don’t have very many dependencies. 

So, why can't we all share a standard three digit version number with strict hierarchy, and take out some of the complexity? Sometimes versioning feels like continuously reinventing the wheel - I have had to work on several almost-the-same little libraries that parse and do common operations on version numbers.

The problem is that version numbers are a manifestation of the developers' assumptions of the life cycle of the code. Developers may want to record the date of the release, or have an ordering hierarchy, or specify build flags, or mark a version as backwards incompatible. In some cases the ecosystem enforces some really strict rollout process. Python’s packages are all versioned in the way I described above, because pip provides a contract for the author and end user.<sup>4</sup> You get a single snapshot in time, rolling monotonically forward on a single logical branch. End users can cherry pick and patch changes if they want to, but it's up to them to manage that complexity, eat the consequences of using a new version of a package that is insufficiently battle tested, and upgrade if e.g. a security vulnerability is found. 

This doesn't work well when the consumer is not another software project. Users generally hate having to manage their own updates, and it's your bad if they end up with a buggy or insecure product. For-profit software orgs also need to figure out what to build, trying experiments early and often. At Dropbox a couple of years ago, we realized that even/odd thing wasn't going to cut it. We needed to move faster, and to be able to segment metrics by the alpha/beta type even with overlapping rollouts. So we redesigned the system to be a simple three numbers, a major version to indicate a distinct “release train” (ever-hardening logical branch) that would go out to all users, middle/minor version 1-4 to indicate build type, a final point version always incrementing as builds are created on each release train. We made the necessary refactors, thought carefully about the ordering consequences because Dropbox doesn't support non-manual downgrades, and wrote up a [post](https://www.dropboxforum.com/t5/Client-builds-archive/New-Versioning-Scheme/td-p/182393) for our forums users. But we didn't leave enough space in between the minor versions! When we wanted to indicate new types of thing, like to break out versioning of some subcomponents or introduce a different type of experimental build, we were stuck, and had to do the same headache all over again. 


### Lessons learned

The end lesson, as hinted above, is to approach versioning and release like architecture. Implement what is useful to you now, but think about assumptions and whether they could change. Design it in a way so that adding new things is possible, and ideally not too painful. Evolving is good, but thrashing on design just causes confusion and overhead for you and your collaborators. 

Release processes, like testing, should serve some quantifiable good for your effort. Do you actually care if this thing breaks? Then have more and longer verification periods. Just throwing spaghetti on the wall? Not so much. Don't put code in stage/alpha/etc for a shorter amount of time than you can actually find the bugs you care about, or you're just doing pretend quality assurance.  And if that time is too long, improve you telemetry so you can find bugs. 

Regardless, have a really really good plan for last minute changes, including bugfixes for emergent issues. The biggest take away I got from writing [this series](https://blogs.dropbox.com/tech/2017/03/accelerating-iteration-velocity-on-dropboxs-desktop-client-part-1/) of blog posts for Dropbox is that your product is only as good as the test pass/canary period since the last code change. You can and should design code and processes to default to turning things back to a known good state in an emergency. Hot fix after hot fix only works if you can push quickly and are tolerant of poor quality. It doesn’t work well if you need strong consistency, durability, or availability, or care about miffing users with a buggy interface. Quick feedback loops are key - as long as you consciously think about the price you willing to pay for that feedback, whenever it be in hardware costs, user loss, or developer time. 

1. Fun bug I've heard about recently: a new hardware specification uncovered a bug in the Go compiler. Go figure ;)
2. I have seen a system like this. It was good for forcing everyone to be on the same old version forever :).
3. AKA "Migrapocalypse".
4. Turns out this is a bit more flexible than I'd realized, including allowing a variety of pre and post fixes, thanks @Sumana Harihareswara! Check out [PEP440](https://www.python.org/dev/peps/pep-0440/) for more detail. Pip is still pretty prescriptive.
