---
layout: post
tagline: "Where your data goes"
---
{% include JB/setup %}

One day at work I was presented with a seemingly pretty simple problem. We use AWS's (Amazon Web Services) EC2 (Elastic Cloud Compute, because at Amazon you're not allowed to use the same initial 2+ times in a row for any of your cloud products) for a lot of things. Well, I was told that the "EBS volume" needed to be "mounted" on the correct directory.

I had a vauge idea of what this means. I spend a bit more mental energy trying not to mix up "memory" (meaning RAM, which does not keep things after you shut down the computer) and "storage" (meaning something like your hard disc or SSD, which permanently keeps things) in my usage than I would like to, but one of the directories needs more space to put things in a permanent way. Cool.

Diagram - registers, memory, cache, and disc

I'll let you struggle through using the AWS console yourself. Let it be known, however, that in an 

Point 1: What is an EBS Volume anyway?

Point 2: If I had my own Linux machine, what exactly would this correspond to?

Point 3: Why do I need to create a file system?

Point 4: What are all the kinds of filesystems and why?

Point 5: WTF is the deal with the way that EBS volumes are named?

