---
layout: post
title: "Python Exception Handling"
description: ""
category: 
tags: ["Python", "best practices"]
---
{% include JB/setup %}

### Definitions and Background

In software, exceptions date from the late 1960s, when they were introduced into Lisp-based languages. By definition, they are intended to indicate something unusual is happening. Hardware often allows execution to continue in its original flow directly after an exception, but over time these kinds of "resumable" exceptions have been entirely phased out of software. You can read more about this in the [Wikipedia article](https://en.wikipedia.org/wiki/Exception_handling), but the important point is that these days, software exceptions are essentially a way to jump to an entirely different logical path. 

Intuitively there are good reasons for exceptions to exist. It is difficult to anticipate the full scope of possible inputs to your program, or the potential outputs of your dependencies. Even if you were able to somehow able to guarantee that your program will only ever be called with the same parameters, cosmic rays happen, disks fail, and network connectivity flakes, and these can manifest in a variety ways that all result in your program needing to metaphorically vomit.

### In Python

I primarily program in Python, which is a very flexible programming language. Given that it has ["duck typing"](https://en.wikipedia.org/wiki/Duck_typing), values are happily passed around as long as possible until it is proven that computation cannot succeed. I had noticed a tendency in my own programming and in other code I read to use exception handling as program flow control. Here is a really small example that I saw (paraphrased) in code earlier this week:

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

If I run this, all that will ever happen is either `foo` will be returned, or `None` will be returned, right?

(no.) 

There are a lot of other possible exceptions in this code. What if the `data` doesn't turn out to be a dict? As far as I know, this would raise a `TypeError`. What if `data` isn't valid JSON? With the standard `json` library, it would be a `ValueError` or `JSONDecodeError`. What if there isn't even a file called `filename`? `IOError`. All of these are indicative of malformed data, rather than just a value that hasn't been set. 

It's really tempting to enumerate all of these explicitly:

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

When I look at this however, it feels a little weird, like it's not the best practice. The exception handlers are being used for two fundamentally different things: an expected, but maybe unusual case, and violation of the underlying assumptions of the function. 

Plus, trying to think of every single possible source of exceptions is arduous. It makes me nervous now, even in this toy example of four lines of code non-try/except code. What if the filename isn't a `str`? Should I add another handler? There are already a lot of except clauses, and they're starting to get in the way of the readabilily of the code. 

Ok, new idea. Clearly the `IndexError` should be replaced with a simple `get`. Probably that would be more performant anyway (maybe some other time I'll delve into the implementation of exception handlers and their perfomance). Around everything else, I could throw in a general try/except.

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

Suddenly I'm getting a MalformedDataError on every run of this function... Am I missing the file? Is it formatted correctly? Eventually, in a hypothetical debugging session, I'd read closely and add some print statements and figure out that I accidentally used the wrong json load function: `load` and `loads` are sneakily similar. 

The general try/except is disguising what is clearly a third kind of issue - a bug within *my* code, locally in this function. Try as I might, the first draft of my code regularly has this kind of "dumb" mistake. Things like misspellings or wrong parameter order are really hard for humans to catch in code review, so we want to fail local bugs early and loudly. 

***
#### Aside: Exception hierarchies
The possibility of conflating two problems that raise the same exception, but need to be handled differently, is a nerve-wracking part of Python. Say my typo had caused a `TypeError` in line 16 of the 2nd code snippet, by, for example, trying to index with a mutable type, `return data[['foo']]`. Even if I tried to switch to a `except TypeError` around just that line, in case the JSON object was not a dict, the local-code bug would not be uniquely identified.

Conversely, when I write my own libraries, it can be overwhelming to decide whether two situations actually merit the same exception. If I found a list instead of a dict, I could raise `TypeError`, which is builtin and seems self-explanatory, but might get mixed up with thousands of other `TypeError`s from other parts of the stack. Or I could a custom exception `WrongJSONObjectError`, but then I have to import it into other modules to catch it, and if I make too many my library can become bloated.

***

I could rabbit-hole further on this code, exploring more potential configurations. Maybe I should check the type of `foo` before returning. Maybe I could try to catch only the errors I definitely know aren't my fault, and then stick a pass and retry in there in case there's some transient error. Hey, `filename` me be on a network drive and connectivity is flaky. The proliferation is huge, so it's time to get axiomatic.

It's worth noting that all of the above full examples pass a standard pep8 linter. They are all patterns that I've seen somewhere in thoroughly tested and used production code, and they can be made to work. But the question I want to answer is, what are the best practices to make code as correct, readable, and maintainable as possible?

### Classifying exceptions
So far we've enumerated three ways exception handlers are used:

#### 1. Unusual, expected cases. 

Probably we should just not use exceptions, and try an if statement instead of relying on the underlying code to fail. The only exception (hehhh) to this rule that I can think of is if there is no way to catch a expected-unusual case early, but I can't think of a good example for this. The Python language has made the design decision to make `StopIteration` and handful of other program flow oriented concepts an `Exception` subclass, but it's worth noting they are not a subclass of `StandardError`. Probably our own custom subclasses should respect this nomenclature, using  `Error` for something gone wrong and `Exception` for something unusual but non-problematic. 

Another subtle facet: do we fold these mildly exceptional cases into the expected return type, by maybe returning an empty string if `foo` expected to be a string, or return something special like `None`? That is beyond the scope of this specific article, but I will give it more thought. I recommend looking into [mypy](http://mypy-lang.org/), a static type analyzer for Python, and its "strict optional" type checking to help keep `if foo is None` checks under control. 

#### 2. A logical error in this section of code. 

Logical errors happen, since writing code means writing bugs. Python does it best to find and dispatch obviously line-level buggy code with `SyntaxError`. 

```
   >>> deg get_foo(filename):
  File "<stdin>", line 1
    deg get_foo(filename):
              ^
SyntaxError: invalid syntax 
```

In fact, SyntaxErrors AFAIK can't be caught, which is a pretty good indicator of what should happen when executing locally buggy code - fail fast and with as much explicit information as possible on location and context. Otherwise, local bugs that are not raised immediately will show up as a violated expectation in the next layer up or down of the code. 

That brings us to the next type:

#### 3. Other things are effed.

Some other code broke the rules. Ugh. It feels like it shouldn't be my job to clean up if something else messed up. It's really attractive, when starting out on a project, to just let all exceptions bubble up, give other code the benefit of the doubt. If something consistently comes up in testing or production, maybe then it's worth adding a handler.

But, there is some some more nuance here. In our examples above, the "culprit" of errors could be many different things. They roughly break down into the following:

#### Problems with dependent systems
The callee of this code did not meet my expectations. Perhaps I called a function, and it raised an error. Could be an error due to a bug in their code, or it could be something further down the stack, like the disk is full. Maybe it returned the wrong type. Maybe the computer exploded. 

#### Problems with depending systems
The caller of the code did not meet my expectations. They passed in a parameter that isn't valid. They didn't do the setup I expected. The program was executed in Python 3 and this is Python 2.

However, there are plenty of places where the distinction between these possibilities doesn't seem particularly obvious. Did the callee raise an error because it failed, or because I violated its assumptions? Or did I pass through a violated assumption from my caller? In the examples above, `IOError` would be raised by `json.loads(filename)`, regardless of whether the filename didn't exist (probably the caller's problem) or the disk was broken (the callee's problem, since the ultimate callee of any software is hardware).

### Design by Contract
The concepts might sound familiar if you've heard much about [Design by Contract](https://en.wikipedia.org/wiki/Design_by_contract), a paradigm originated by Bertrand Meyer in the late 1990s. Thinking about our functions explicitly as contracts, and then enforcing them like contracts, is actually potentially the solution to both of the top-level problems we've come across: attributing error correctly and handling it effectively. 

The basic idea behind Design by Contract is that interfaces should have specific, enforceable expectations, guarantees, and invariants. If a "client" meets the expectations, the "supplier" must complete the guarantee. If the "supplier" cannot complete the guarantee, at a minimum it maintains the invariant, and indicates failure. If the "supplier" cannot maintain the invariants, crash the program, because the world is broken and nothing makes sense anymore. An example of an invariant is something like, list size must not be negative, or there must be 0 or 1 winners of a game.

### In application

The language Eiffel was designed by Meyer to include these concepts at the syntax level, but we can port a lot of the same benefit to Python. A first step is documenting what the contract actually is. Here is the list of required elements for a contract, quoted from the Wikipedia page:

> - Acceptable and unacceptable input values or types, and their meanings
> - Return values or types, and their meanings
> - Error and exception condition values or types that can occur, and their meanings
> - Side effects
> - Preconditions
> - Postconditions
> - Invariants

Most Python comment styles have standard ways of specifying the first three item. Static analyzers, like mypy or [Pycharm](https://www.jetbrains.com/pycharm/)'s code inspection, can even identify bugs by parsing comments before you ever run the code. Taking the last version of our example from above, and applying the [Google comment style](http://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html) + [type hints](https://www.python.org/dev/peps/pep-0484/), we might end up with something like this:


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
We even got a new piece of information! I hadn't been forced to think about the return type previously, but with this docstring, I realized `foo` is expected to be an int, and added a bit of code to enforce that.

### Side effects and invariants
Generally speaking there aren't good ways of enumerating or statically checking the execution of side effects, which is a reason to avoid them when possible. A common example would be updating an instance variable on a class, in which case the side effect would be hopefully obvious from the method name and specified in the dostring. At a minimum, side effects should obey any invariants. 

This tiny code snippet doesn't have an obvious invariant, and I theorize this is because it *is* side-effect free. Conceivably, another function in the module would mutate the dictionary and write it, and our invariant would be that `filename` would preserve a JSON encoded dictionary, maybe even of a certain schema. The other classic example is a bank account: a transaction can succeed or fail, due to caller error (inputting an invalid currency) or callee error (account database isn't available), but the accounts should never go negative.

### Preconditions and postconditions
Preconditions are things that must be true before a function can execute. They are they responsibility of the "client" in the business contract analogy, and if they are not met, the "supplier" does not have to meet the guarantee. Postconditions are the criteria by which the function is graded. If they are not met, and no error is indicated, the bug can be attributed to that function. This maps really cleanly to the caller and callee problems we delineated above.

Left up to the programmer is where to draw the line between the two in the software design. In our toy example above, I've implicitly decided to make the preconditions pretty strict: `filename` must exist and be a JSON encoded dictionary. I could have chosen to, for example, accept a file with a single integer ascii-encoded integer, or return None if the file doesn't exist. However, I might have been more strict: I do allow dictionaries with a missing `foo` value, and simply return None, and I'm willing to cast the value at `foo` to an integer even if it's a string or float.

Having a seen a lot of code that tries to lump similar things together with zillions of if statements, usually in the name of deduplication or convenience, and at the cost of unreadability, I would argue that stronger preconditions are better. The deduplication is better done via shared helpers with strong preconditions themselves. Casting it int is probably even a bad idea... but I'll leave it as is given that the requirements of this program are pretty arbitrary. 

My postconditions are pretty simple in this case: return the int or None that corresponds to the value of foo at that path. 

### Identifying violations.
A lot of the ideas in this post are based on [this chapter](http://se.ethz.ch/~meyer/publications/computer/contract.pdf,) of Interactive Software Engineering by Meyer, which has a bit of a vendetta against "Defensive Programming". In the defensive paradigm, we would anticipate all possible problems and end up with a check at every stage. I added a client to the `get_foo` function to demonstrate this:

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

Indeed, this is really verbose. But there isn't a hard and fast rule that the article offers for who is supposed to check. The main thing it argues explicitly against is the exception handler usage I first identified as potentially dangerous above: special but expected cases. My interpretation of Meyer's work, in the context of Python, is that assertions can be used to make it really clear what is at fault when something goes wrong. In fact, that should be the primary goal of all exception handling: ease of debugging.

I think this the final version I would go with:

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
    """Print 'hello' the number of times that foo specifies"""

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
 - `JSONDecodeError` is also pretty easily to interpret. Plus, there isn't really a way to assert for the validity of the file other than to use the `json` library's own contracts.
 - I generally find failing to index a presumed dictionary to be confusing error to debug, especially since mypy and and `json` itself can't help us out to catch type issues from validly deserialized data. Therefore I make an explicit check for the type of `data`. 
 - `foo_times_five` wants to ensure its own guarantees, and since the `*` operator works in strings and lists and so on as well as ints, I added an assertion.

### Handling violations
You'll notice that my two functions have drastically different ways of handling exceptional cases once they occur. `foo_times_five` returns an int almost at all costs, while `get_foo` is more raises and asserts more. Once again, this is a really toy example, but the goal is to fail as close to the source of the error as possible. If the highest wrapping application wants to try and make everything all right, pending satisfaction of invariants, that should be the a decision made at a layer that has the sufficient context, with the smallest amount of intermediary code executed. The [Midori language](http://joeduffyblog.com/2016/02/07/the-error-model/) is really opinionated about this (and that blog post is a long-but-good read). Any error that can not be caught statically results in the immedate teardown and recreation of an entire process, which is called "abandonment, ensuring no state is littered around in memory afterwards.

I would add two more debug-facilitating rules: leave some trace every time an exceptions is strategically swallowed, and be as consistent as possible (perhaps, via documented invariants) about the criteria for exception recoverability across environments and parts of the codebase. 

#### A war story
At work, we used to have a general-purpose exception handler that would report stacktraces to a central server, but behaved somewhat differently in "debug" and "stable" contexts. In debug situations, it would bubble up the exception, causing a crash for the developer to notice and fix, since they were right there editing code anyway. In stable, it would continue as is. The intention was noble - we want to minimize crashes for end users. But this made it really difficult for developers to reason about what would happen when their code was deployed to the world, and in a worst case scenario the application could continue running after an error in a untested, unknown state. This was thrust into the light when a unittest began failing only when the stable environment flag was on. We got rid of the handler, and also now run all tests with a mock-stable configuration on every commit to guard against other configuration-dependent regressions.

