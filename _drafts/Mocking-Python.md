Writing unittests is hard! It's also important. Coworkers tend to get mad at you if you don't do it, and if you're 80% sure you got the logic in your code right, you can be 96% after you write a test for it.

Writing unittests is a hell of a lot harder if you're not really sure which part of your code you're testing (or breaking), or if your code has external dependencies, like network requests, databases, filesystems, etc. This is mocking is useful.

### The basics.

Here are three words: mock. patch. stub. They all have English concepts that you're probably familiar with, but let's take a trip to unittest-land.

Mocks are fake versions of things. In the case of Python, they most often take the place of any callable (thing with a `__call__` method that gets invoked using parentheses) like functions and classes in additon to instantiated objects. Since "everything in Python is an object" you really could use them for whatever the hell you want, but practically speaking ints and strings and so ons can be left laone.

Mocks allow you to check whether and how something was used, and with default configuration, they will let you call any method or access any attribute. 

With Python's `mock`, the base mock class is, unsurprisingly, `Mock`. Most commonly, I end up using the subclass `MagicMock` which automatically creates Python's magic methods, like `__new__` and `__getattr__`. For more on magic methods, I highly recommend (this post by Rafe Kettler)[http://www.rafekettler.com/magicmethods.html].

Here is a MagicMock. Let's see what it has to say. 

>>> import mock
>>> m = mock.MagicMock()
>>> m
<MagicMock id='4428915856'>
>>> m()
<MagicMock name='mock()' id='4429573072'>
>>> m('oh hey')
<MagicMock name='mock()' id='4429573072'>
>>> m.a
<MagicMock name='mock.a' id='4429456464'>
>>> m.sup()
<MagicMock name='mock.sup()' id='4429547856'>.

It's mocks all the way down. How do I know what was done with my mock? It's preloaded with a lot of sweet helper methods:

>>> dir(m)
['a', 'assert_any_call', 'assert_called_once_with', 'assert_called_with', 'assert_has_calls', 'attach_mock', 'call_args', 'call_args_list', 'call_count', 'called', 'called_once', 'configure_mock', 'method_calls', 'mock_add_spec', 'mock_calls', 'reset_mock', 'return_value', 'side_effect', 'sup']

Wait hey. There's also `a` and `sup` in there. That's because by using them we attached them to the mock. This can actually create dangerous false positives - which I might talk about later. 

Practically, I tend to use these the most in asserts in my tests, but YMMV. 
>>> m.called
True
>>> m.assert_called_with('!!!CALL THIS')  # Gonna fail. Which is good! Tests that don't fail ever are not useful.
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/Cellar/python/2.7.9/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages/mock-1.0.1-py2.7.egg/mock.py", line 835, in assert_called_with
    raise AssertionError(msg)
AssertionError: Expected call: mock('!!!CALL THIS')
Actual call: mock('oh hey')
>>> m.call_args[0][0] == 'oh hey'  # This can be brittle and hard to read, but if you want to know the exact order and args of calls, there's no better way to do it with this mock library. 
True

### An example 

When are they useful? Let's say I'm making an application to manage my haberdashery. In my database layer I have a function that checks the number of hats in the inventory, and it takes in a cursor object so that I can pool and reuse cursors. I don't want to touch a real database for my unittest. Let's say this is the first rev of my code:

hat_db.py:

    def check_inventory(cursor)
        """Returns list of tuple (`hat_id`, `hat_name`, `quantity` for all hats in the inventory"""

        cursor.exceute("SELECT hat_id, hat_name, quantity from hat_invetory")
        cursor.fetchall()

And this is the corresponding test file:

test_hat_db.py:

    import pytest

    from mock import MagicMock

    from hat_db import check_inventory
    
    def test_check_inventory():
        """Tests the check_inventory function"""
        cursor_mock = MagicMock()
        expected_hats = [(1, 'fedora', 10), (2, 'Homburg', 8), (3, 'Panama', 22)]
        cursor_mock.return_value = expected_hats
        
        assert check_inventory(cursor) == expected_hats

This test will fail (good job, unittest!) Why? Because I forgot to return anything first of all. But when I fix that, the unittest will pass, while the production code will break.

Hazard a guess?

Here's the tricky bit: `cursor.exceute` is spelled wrong, and the mock doesn't give a crap. It'll call anything you want. It might also slip by in code review, because we humans are really good at ignoring transpositions when reading. 

This is why specs are really useful. If I get around to writing about patches, you can do something called auto-speccing, which assign the same API to the mock as to the patched objects. In this case, we need an explicit spec. 
     
    import pytest
    
    from mock import MagicMock
    from sqlite3 import Cursor

    from hat_db improt check_inventory

    def test_check_inventory():
    """Tests the check_inventory function"""
        cursor_mock = MagicMock(spec=Cursor)
        expected_hats = [(1, 'fedora', 10), (2, 'Homburg', 8), (3, 'Panama', 22)]
        cursor_mock.return_value = expected_hats    
        
        assert check_inventory(cursor) == expected_hats

### The `call` object

When would call order be important? I have a former colleague who made another mocking library called [vmock](https://pypi.python.org/pypi/vmock/0.1). It went by a "record and replay" style of mocking, and I have been meaning to read more about mocking philosophy. This made it really easy to say what you expected to happen and when - though sometimes harder if, for example, you knew a function was going to be passed in a string but only had a vague idea which and didn't really care.

The standard `mock` libarary of Python 3.x + is not as well suited to extreme call order and argument specificity. There are methods on `Mock` that let you look at the last thing called, or if it was called only once, etc. I recently had a situation where I was instantiating a class twice, with some different kwargs, and it was critical that they happen in a speicific order. I ended up spending way too long trying to figure out how to 

There are lots of other things that can be said about mocks. The python documentation is pretty thorough, but this is an example of what I find most useful about them. 






A patch replaces a thing with something else, usually a mock. 

A stub 

    kk  kk
### Mocking an external package.
### Mocking a custom module.
### Mocking the standard library?




