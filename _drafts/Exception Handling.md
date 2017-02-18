---
layout: post
title: "Paradigms in Exception Handling"
description: ""
category: 
tags: ["Python", "best practices"]
---
{% include JB/setup %}

In software, exceptions date from the late 1960s, when they were introduced into Lisp-based languages. By definition, they are intended to indicate something unusual is happening. Hardware often allows execution to continue in it's original flow directly after an exception, but over time "resumable" exceptions have gradually been entirely phased out of software. They are essentially a way to jump to an entirely different logical path. 

Intuitively there are good reasons for exceptions to exist. It is difficult to anticipate the full scope of possible inputs to your program, and outputs of your dependencies. Even if you were able to somehow able to guarantee that your program will only ever be called with the same parameters, cosmic rays happen, disks fail, and network connectivity flakes.

I primarily program in Python, which is a very flexible programming language. Given that it has "duck typing", values are happily passed around as long as possilbe until it is proven that computation cannot succeed. I had noticed a tendency in my own programming and other code I read to use exception handling as program flow control. Here is a really small example that I saw (paraphrased) in code earlier this week:

```
import json

def get_foo(filename):
    with open(filename, 'rb') as fobj:
        data = json.load(fobj)
    try:
        return data['foo']
    except KeyError:
        return None

```

But, there are a lot of other possible exceptions in this code! What if the `data` doesn't turn out to be a dict? As far as I know, this would raise a `TypeError`. What if `data` isn't valid JSON? With the standard `json` library, it would be a `ValueError`. What if there isn't even a file called `filename`? `IOError`. All of these are indicative of malformed data, rather than just a value that hasn't been set. Say we code all of these explicitly:


```
import json

class MalformedDataError(Exception):
    pass

def get_foo(filename):
    try:
        with open(filename, 'rb') as fobj:
            data = json.load(fobj)
    except IOError, ValueError:
        raise MalformedDataError
        
    try:
        return data['foo']
    except TypeError:
        raise MalformedDataError
    except KeyError:
        return None

```

The exception handlers are being used for two fundamentally different things: an expected, but maybe unusual case, and violation of the underlying assumptions of the function. Trying to think of every single possible assumption violation is arduous. It makes me nervous now, even in this toy example, trying to think of all the other Exceptions that could be raised from these four lines of code. What if the filename isn't a `str`? eThere are already a lot of except clauses, and they're starting to get in the way of the readabilily of the code. 

Ok, new idea. Clearly the `IndexError` should be replaced with a simple `get`. Probably that would be more performant anyway.Then, I could throw in a general try/except.

```
import json

class MalformedDataError(Exception):
    pass

def get_foo(filename):
    try:
        with open(filename, 'rb') as fobj:
            data = json.loads(fobj)
            print data
        return data.get('foo')
    except Exception:
        raise MalformedDataError()
```

Suddenly I'm getting a MalformedDataError on every run of this function... because I accidentally used the wrong json load function (`load` and `loads` are sneakily similar). This is clearly a third kind of issue - a bug within my code. 

Aside: This is a somehwat nerve-racking thing about using the fundamental Python Exception types anyway. Say my typo had caused a TypeError in line 14 of the 2nd example, maybe I did `return data[['foo']]` by accident. The exception handler wouldn't know the difference! Clearly my library should consider this when deciding between the MalformedDataError above and, say, a general `TypeError` to indicate the data of the worng type. 

I could rabbit-hole further, and maybe try to catch only the errors I definitely know aren't my fault, and then stick a pass and retry in there in case there's some transient error, like the computer is out of memory or `filename` turns out to be on a network drive and connectivity is flaky. It's time to get axiomatic.

It's worth noting that all of the above examples pass a standard set of pep8 linters. They are all patterns that I've seen somewhere in thoroughly tested and used production code, and they can be made to work. But the question I want to answer is, what are the best practices to make code as readable and maintainable as possilbe, so that editing the code has the lowest possible risk of new bugs.

# Classifying exceptions
So far we've enumerated three ways exception handlers are used:
- Unusual, expected cases
- Things aren't what they're supposed to be
- A bug in this section of code





# Handling exceptions
## A real life example
As projects grow, there is another lurking deamon. One really effective way to use exceptions is to dump a lot of information and report that alongside the exception as they occur, to aid troubleshooting. At work, we used to have a general-purpose expepiton handler that would report this kind of information centrally, but behaved somewhat differently in "debug" and "stable" contexts. In debug situations, it would bubble up the excpetion, causing a crash for the developer to notice and fix, since they were right there editing code anyway. In stable, it would continue as is. The intention was noble - we want to minimize crashes for end users. But this made it really difficult for devleopers to reason about what would happen when their code was deployed to the world, and the application could continue running in a untested, unknown state. 

TODO Look up the exact bug that caused this.
http://se.ethz.ch/~meyer/publications/computer/contract.pdf

