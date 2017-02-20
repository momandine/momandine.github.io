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

If I run this, all that will ever happen is either `foo` will be returned, or None will be, right? (no.) There are a lot of other possible exceptions in this code. What if the `data` doesn't turn out to be a dict? As far as I know, this would raise a `TypeError`. What if `data` isn't valid JSON? With the standard `json` library, it would be a `ValueError` or `JSONDecodeError`. What if there isn't even a file called `filename`? `IOError`. All of these are indicative of malformed data, rather than just a value that hasn't been set. Say we enumerate all of these explicitly:


```
import json

class MalformedDataError(Exception):
    pass

def get_foo(filename):
    try:
        with open(filename, 'rb') as fobj:
            data = json.load(fobj)
    except IOError, ValueError, JSONDecodeError:
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
        return data.get('foo')
    except Exception:
        raise MalformedDataError()
```

Suddenly I'm getting a MalformedDataError on every run of this function... because I accidentally used the wrong json load function (`load` and `loads` are sneakily similar). This is clearly a third kind of issue - a bug within my code. 

Aside: This is a somehwat nerve-eracking thing about using the fundamental Python Exception types. Say my typo had caused a `TypeError` in line 14 of the 2nd example, by, for example, trying to index with a mutable type, like `return data[['foo']]`. The exception handler wouldn't know the difference. Conversely, this is another thing to think about in my own library. Should I raise `TypeError`, which everyone knows about and  may be more self-explanatory or a custom exception `MalformedDataError`?

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

Howevever, there are of places where the distinction between the possibilities doesn't seem particularly obvious. Did the callee raise an error because it failed, or because I violated its assumptions? Or is it because my caller violated an assumption? In the examples above, `IOError` would be raised by `json.loads(filename)`, regardless of whether the filename didn't exist (probably the caller's problem) or the disk was broken (the callee's problem if you can consider hardware a callee a couple of layers down the stack). 

# Design by Contract
The concepts might sound familiar if you've heard much about [Design by Contract](https://en.wikipedia.org/wiki/Design_by_contract), a concept defined by Bertrand Meyer in the late 1990s. Thinking about our functions explicitly as contracts, and then enforcing them like contracts, is actually potentially the solution to both of the problems above: attributing error and handling it. 

The basic idea behind Design by Contract is that interfaces should have specific, enforcable expectations, guarantees, and invariants. If a "client" meets the expectations, the "supplier" must complete the guarantee. If the "supplier" cannot complete the guarantee, at a minimum it maintains the invariant, and indicates failure. If the "supplier" cannot maintain the invariants, crash the program. An example of an invarient is someting like, list size must not be negative. [TODO check this. Maybe re-work example to include invariants]

## In applicaiton

The language Eiffel was designed to include these concepts at the syntax level, but we can backport a lot of the same benefit to Python. A first step is documenting what the contract acutally is. Here is the list of required elements, from the Wiki on DbC directly:

- Acceptable and unacceptable input values or types, and their meanings
- Return values or types, and their meanings
- Error and exception condition values or types that can occur, and their meanings
- Side effects
- Preconditions
- Postconditions
- Invariants

Most Python comment styles have ways of specifying the first three directly. Taking the last version of our example from above, and applying the Google comment style + type comments (which has the advantage of being compact, and the option of Mypy static analysis to enforce), we might end up with something like this:


```
import json
from typing import Optional

class MalformedDataError(Exception):
    pass

def get_foo(filename: str) -> Optional[int]:
    """Return the integer value of `foo` in JSON-encoded dictionary, if found.

    Args:
        filename: Full or relative path to JSON-encoded dictionary.

    Returns:
        The integer at key `foo`, or `None` if not found.

    Raises:
        MalformedDataError: if the JSON-encoded dictionary cannot be found and parsed to find `foo`.
    """

    try:
        with open(filename, 'rb') as fobj:
            data = json.loads(fobj)
            print data
        if 'foo' in data:
            return int(data['foo'])
        else:
            return None

    except Exception:
        raise MalformedDataError()
```
We even got a new piece of information! I hadn't been forced to think about the return type previously, but with this docstring, I realized `foo` is expected to be an intand added a bit of code to enforce that.

## Side effects and invariants
Generally speaking there aren't good ways of enumerating or statically checking the execution of side effects, which is a reason to avoid them when possible. A common side effect would be updating an instance variable on a class, in which case the side effect would be included in the docstring and hopefully obvious from the method name, and they should obey any invariants. 

This tiny code snippet doesn't have an obvious invariant, in large part becuase it IS side-effect free. Conceivably, another function in the module would mutate the dictionary and write it, and our invariant would be that `filename` would preserve a JSON encoded dictionary, maybe even of a certain schema. The other classic example is a bank account: a transaction can succeed or fail, due to caller error (inputting an invalid currency) or callee error (account database isn't available), but the accounts should never go negative.

## Preconditions and postconditions
Preconditions are things that must be true before a function can execute. They are they responsibility of the "client", in the business contract analogy, and if they are not met, the "supplier" does not have to meet. Postconditions are the criteria by which the function is graded. If they are not met, the bug can be attributed to the function. This maps really cleanly to the caller and callee errors above.

Left up to the programmer is where to draw the line between the two. In our toy example above, I've implicitly decided to make the preconditions pretty strict: `filename` must exist and be a JSON encoded dictionary. I could have chosen to, for example, accept a file with a single integer ascii-encoded integer, or return None if the file doesn't exist. However, I might have been more strict: I do allow dictionaries with a missing `foo` value, and simply return None, and I'm willing to cast the value at `foo` to an integer even if it's a string or float.

Having a seen a lot of code that tries to lump similar things together with zillions of if statements in the name of deduplication, at the cost of unreadability, I would argue that stronger preconditions are better. The deduplication is better done in the form of shared helpers with strong preconditions themselves. Casting it int is probably even a bad idea... but I'll leave it as is given that the requirements of this program are pretty abitrary. 

My postconditions are pretty simple in this case, return the int or None that corresponds to the value of foo at that path. 

## Identifying violations.
A lot of the ideas in this post are based on http://se.ethz.ch/~meyer/publications/computer/contract.pdf, which has a bit of a vendetta against Defensive programming. In the defensive paradigm, we would anticipate all possible porblems and end up with a check at every stage:

```
import json
import os
import sys

from typing import Optional

FOO_FILE = 'somefile.json'

class MalformedDataError(Exception):
    pass

def get_foo(filename: str) -> Optional[int]:
    """..."""
    if not (isintance(filename, str) and os.path.exists and os.path.isfile(filename)):
        raise MalformedDataError()

    with open(filename, 'rb') as fobj:
         data = json.loads(fobj)

    if not isinstance(data, dict):
        raise MalformedDataError()
    
    foo = dict.get('foo')

    if foo is not None
        try:
            foo = int(foo)
        except TypeError:
            raise MalformedDataError()

        assert isinstance(foo, int)
        return foo

    return None

def foo_times_five(): -> int
    """Print 'hello' the number of times that foo specifies."""

    if not os.path.isfile(filename):
        return 0
    
    try:
        foo = get_foo(FOO_FILE)
    except MalformedDataError():
        return 0

    assert isintance(foo, (int, None)), "Foo should be an integer value or None"

    if foo is None:
        return 0
    else:
        return foo * 5
```

This is really verbose! But there isn't a hard and fast rule that the article offers for who is supposed to check. The main thing it argues against is the exception handler usage I first identified as potentially dangerous above: special but expected cases. My interpretation, in the context of Python, is that assertions can be used to make it really clear what is at fault when something goes wrong. 

I think the final version I would go with.
```
import json

FOO_FILE = 'somefile.json'

def get_foo(filename: str) -> Optional[int]:
    """Return the integer value of `foo` in JSON-encoded dictionary, if found.

    Args:
        filename: Full or relative path at which to look for the value of 'foo'. File must exist and be a valid JSON-encoded dictionary.

    Returns:
        The integer-casted value at key `foo`, or `None` if not found.
    """

    with open(filename, 'rb') as fobj:
         data = json.loads(fobj)

    assert isinstance(data, dict), 'Must be a JSON-encoded dictionary'   
       
    if 'foo' in data:
        try:
            foo = int(dict.get('foo'))
        except TypeError:
            return None
    else:
        return None

def foo_times_five(): -> int
    """Print 'hello' the number of times that foo specifies
    """

    if not os.path.isfile(FOO_FILE)):
        return 0
    
    foo = get_foo(FOO_FILE)

    assert isintance(foo, (int, None)), "Foo should be an integer value or None"

    if foo is None:
        return 0
    else:
        return foo * 5
```

My rationale is the following:
 - A static type checker would take care of the filename not being a string.
 - A failure of opening the file would be easily attributed to the caller, given the docstring. So, my caller takes care of that check, since it's using a module-level variable it may not entirely trust (it could get mutated at runtime). 
 - JSONDecodeError are also pretty easily to interpret. Plus, there isn't really a way to assert for the validity of the file other than to use the `json` library's own contracts.
 - I generally find failing to index a presumed dictionary to be confusing error to debug, especially since Mypy and and `json` itself can't help us out to catch type issues from validaly deserialized data, so I make an explicit check for the type of `data`. 
 - `foo_times_five` wants to ensure its own guarantees, and since the `*` operator works in strings and lists and so on as well as ints, I added an assertion.

# Handling violations
You'll notice that my two functions have drastically different ways of handling errors once they occur. `foo_times_five` returns a 


### A real life example
As projects grow, there is another lurking deamon. One really effective way to use exceptions is to dump a lot of information and report that alongside the exception as they occur, to aid troubleshooting. At work, we used to have a general-purpose expepiton handler that would report this kind of information centrally, but behaved somewhat differently in "debug" and "stable" contexts. In debug situations, it would bubble up the excpetion, causing a crash for the developer to notice and fix, since they were right there editing code anyway. In stable, it would continue as is. The intention was noble - we want to minimize crashes for end users. But this made it really difficult for devleopers to reason about what would happen when their code was deployed to the world, and the application could continue running in a untested, unknown state. 

TODO Look up the exact bug that caused this.
http://se.ethz.ch/~meyer/publications/computer/contract.pdf

