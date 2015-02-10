---
layout: post
category : python
tagline: "Eggs, distributions, etc"
tags : [python, packaging]
---
{% include JB/setup %}

I'm going to talk about something really boring. That thing is packaging and distributing Python code. 

For most of my Python-writing career, I've written either code for myself, or as part of a larger proprietary project that used an archaic directory and packaging conventions that predated the most modern iteration of Python packaging tools. Trying to share an integrate those codebases with another team has been... interesting, and I've finally had to get a grasp on a bunch of packagingy stuff I'd taken for granted.

My workflow went something like this: 
1. Install virtualenv and virtualenvwrapper
2. Make a virtualenv for each project, using the correct python interpreter
3. Pip install stuff on that virtualenv 
4. Profit. Prophet? 

<sup>Aside: getting virtualenvwrapper set up is always a bit of a pain. `venv` is now included in the standard Python 3 library, so that will subsume at this point, but a lot of inertia is keeping me on `virtualenv`. [Here](http://docs.python-guide.org/en/latest/dev/virtualenvs/) is a tutorial.</sup>

For the hobbiest, this is remarkably easy and convenient. But GREAT HORRORS AWAIT under the hood. For some rants on why Python packaging is suboptimal, check out these:

But, we can only deal with what we have. First question first... what is the difference between a *module* *package* *distribution* *egg* and *wheel*?

The PyPI (Python Package Index, otherwise known as the Cheese Shop {except no one actually calls it that}), has a [glossary](https://packaging.python.org/en/latest/glossary.html).

`module` - In Python, a single .py file is a module. This means that Python handles encapsulation without classes quite nicely. 
`package` - A collection of modules. The tricky thing is that it can either be an import package, which is module that contains other modules that you can `import`, or a `distribution package`. It colloquial usage it seems to mean "a collection of Python code published as a unit".
`distribution` - Technically. This term tends to be avoided to thwart confusion with Linux distros or larger software projects.
`egg` - A packaging format of Python files with some metadata, introduced by the `setuptools` library, but gradually being phased out.
`wheel` - The new version of the egg. It has many purported benefits (link), and about half of the most popular Python pacages are now wheels on PyPI rather than eggs.

The history of setuptools vs distutils is mildly interesting (that article from stack overflow).
Basically, distutils came first, but it lacked some really integral features, for example uninstallation. Setuptools was written to be a superset of distutils, but inherited some of the same problems. According to this enlightening but slightly soporific article from the Architecture of Open Source, one key issue is that the same code was written to take care of publishing and installing Python packages.

Meanwhile, there is/was a also a fight between `easy_install` and `pip`. `Easy_install`'s main virtues are that it always works as well as it can work, including on Windows. `Pip` has a much richer feature set, like keeping track of requirements heirarchies and uninstallation.









Easy install => eggs
Just put them on the Python Path. 

from pkg_resources import require
require("FooBar>=1.2")

PyPI -> Python package index. Easy enough. Pip.
"Cheese shop" => Secret code name for PyPI

What is is the pip/easy install hierarchy? 
Pip came later and has these features: http://stackoverflow.com/questions/3220404/why-use-pip-over-easy-install

    All packages are downloaded before installation. Partially-completed installation doesn’t occur as a result.
    Care is taken to present useful output on the console.
    The reasons for actions are kept track of. For instance, if a package is being installed, pip keeps track of why that package was required.
    Error messages should be useful.
    The code is relatively concise and cohesive, making it easier to use programmatically.
    Packages don’t have to be installed as egg archives, they can be installed flat (while keeping the egg metadata).
    Native support for other version control systems (Git, Mercurial and Bazaar)
    Uninstallation of packages.
    Simple to define fixed sets of requirements and reliably reproduce a set of packages.

What is the difference between distutils and setuptools?
distutils is in the standard library, and setuptools has more features.


What is installation? When you get a system set up to be able to run new software
What is bootstrapping? 

What an egg looks like: https://pythonhosted.org/setuptools/formats.html

Installation philosophies: http://www.aosabook.org/en/packaging.html
CMS - Content management system
What are the important things about setuptools?

THis looks useful: http://www.simplistix.co.uk/presentations/python_package_management_08/python_package_management_08.pdf

Architecture of open source article: http://www.aosabook.org/en/packaging.html

Why `__init__.py`?
