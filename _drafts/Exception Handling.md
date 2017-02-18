---
layout: post
title: "Paradigms in Exception Handling"
description: ""
category: 
tags: ["Python", "best practices"]
---
{% include JB/setup %}

# Definitions and Background

In software, exceptions date from the late 1960s, when they were introduced into Lisp-based languages. By definition, they are intended to indicate something unusual is happening. Hardware often allows execution to continue in it's original flow directly after an exception, but over time "resumable" exceptions have gradually been entirely phased out of software. They are essentially a way to jump to an entirely different logical path. 

Intuitively there are good reasons for exceptions to exist. It is difficult to anticipate the full scope of possible inputs to your program, and outputs of your dependencies. Even if you were able to somehow able to guarantee that your program will only ever be called with the same parameters, cosmic rays happen, disks fail, and network connectivity flakes.

# In Python

I primarily program in Python, which is a very flexible programming language. Given that it has "duck typing", values are happily passed around as long as possible until it is proven that computation cannot succeed. I had noticed a tendency in my own programming and in other code I read to use exception handling as program flow control. Here is a really small example that I saw (paraphrased) in code earlier this week:

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

If I run this, all that will ever happen is either `foo` will be returned, or None will be, right? (no.) There are a lot of other possible exceptions in this code. What if the `data` doesn't turn out to be a dict? As far as I know, this would raise a `TypeError`. What if `data` isn't valid JSON? With the standard `json` library, it would be a `ValueError`. What if there isn't even a file called `filename`? `IOError`. All of these are indicative of malformed data, rather than just a value that hasn't been set. Say we enumerate all of these explicitly:


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
        raise MalformedDataError()
    except KeyError:
        return None

```

This feels a little weird. The exception handlers are being used for two fundamentally different things: an expected, but maybe unusual case, and violation of the underlying assumptions of the function. Plus, trying to think of every single possible assumption violation is arduous. It makes me nervous now, even in this toy example of four lines of code non-try/except code. What if the filename isn't a `str`? Should I add another handler? There are already a lot of except clauses, and they're starting to get in the way of the readabilily of the code. 

Ok, new idea. Clearly the `IndexError` should be replaced with a simple `get`. Probably that would be more performant anyway. Then, I could throw in a general try/except.

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

Aside: This is a somehwat nerve-racking thing about using the fundamental Python Exception types. Say my typo had caused a TypeError in line 14 of the 2nd example, by, for example, trying to index with a mutable type, like `return data[['foo']]`. The exception handler wouldn't know the difference. Conversely, this is another thing to think about in my own library. Should I raise `TypeError`, which everyone knows about and  may be more self-explanatory or a custom exception `MalformedDataError`?

I could rabbit-hole further, and check the type of `foo` before returning. Maybe I try to catch only the errors I definitely know aren't my fault, and then stick a pass and retry in there in case there's some transient error, like `filename` turns out to be on a network drive and connectivity is flaky. It's time to get axiomatic.

It's worth noting that all of the above examples pass a standard set of pep8 linters. They are all patterns that I've seen somewhere in thoroughly tested and used production code, and they can be made to work. But the question I want to answer is, what are the best practices to make code as readable and maintainable as possilbe, so that editing the code has the lowest possible risk of new bugs.

# Classifying exceptions
So far we've enumerated three ways exception handlers are used:
1. Unusual, expected cases. 

Probably we should just not use exceptions, and try an if statement instead of causing some underlying to fail. The exception (hehhh) is probably if there is no way to catch these situations early, but I can't think of anything off the top of my head that fits into this category.

One tricky facet is whether to fold these mildly exceptional cases into the expected return type, by maybe returning an empty string if `foo` expected to be a string, or return something special (like `None`), but that is beyond the scope of this article. I recommend checking out MyPy and strict optional type checking to help keep `if foo is None` checks under control.

2. A logical error in this section of code. 

They happen, since writing code means writing bugs Python does it best to find and dispatch buggy code with `SyntaxError`. 
```
   >>> deg get_foo(filename):
  File "<stdin>", line 1
    deg get_foo(filename):
              ^
SyntaxError: invalid syntax 
```    
In fact, SyntaxErrors practically speaking can't be caught, which is a pretty good indicator of what should happen when executing buggy code - fail fast and with as explicit as possible information on location and context. Bugs that are not raised immediately, will show up as a violated expectation in the next layer up or down of the code. That brings us to the next type:

3. Other things are effed.

Someone broke the rules - ugh. This feels like it shouldn't be my job if something else messed up. It's really tempting to just let all the exceptions bubble to start and deal with them add hoc, as it seems worthwhile to catch them. 

But, this actually merits a bit more thought. In our examples above, the culprit could be many different things. They roughly break down into the following:

## Problems with dependent systems
The callee of this code did not meet my expectations. Perhaps I called a function, and it bubbled up an error. Could be an error due to a bug in their code, or it could be something further down the stack, like the disk is full. Maybe it returned the wrong type. Maybe the computer exploded. 

## Problems with depending systems
The caller of the code did not meet my expectations. They passed in a parameter that isn't valid. They didn't do the setup I expected. The program was executed in Python 3 and this is Python 2.

There are a couple of places where things might get tangled and confused, though. Did the callee raise an error because it failed, or because I violated its assumptions? Or is it because my caller violated an assumption? In the examples above, `IOError` would be raised by `json.loads(filename)`, regardless of whether the filename didn't exist (probably the caller's problem) or the disk was broken (the callee's problem if you can consider hardware a callee a couple of layers down the stack). 

This might sound familiar if you've heard much about Design by Contract. [put in link] 

# Handling exceptions
So, we've figured out what the differnt types of errors that could befall our little program are. What do we actually do with this information?

## Detecting failed assumptions
`AssertionError`. It's a thing. We can use it.




## Halt the world or keep executing?


### A real life example
As projects grow, there is another lurking deamon. One really effective way to use exceptions is to dump a lot of information and report that alongside the exception as they occur, to aid troubleshooting. At work, we used to have a general-purpose expepiton handler that would report this kind of information centrally, but behaved somewhat differently in "debug" and "stable" contexts. In debug situations, it would bubble up the excpetion, causing a crash for the developer to notice and fix, since they were right there editing code anyway. In stable, it would continue as is. The intention was noble - we want to minimize crashes for end users. But this made it really difficult for devleopers to reason about what would happen when their code was deployed to the world, and the application could continue running in a untested, unknown state. 

TODO Look up the exact bug that caused this.
http://se.ethz.ch/~meyer/publications/computer/contract.pdf

