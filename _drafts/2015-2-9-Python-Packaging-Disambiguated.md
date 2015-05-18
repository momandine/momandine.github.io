---
layout: post
category : python
tagline: "Eggs, distributions, etc"
tags : [python, packaging]
---
{% include JB/setup %}

NOTE: What do I want to do exactly with this blog post?
- What exactly is happening when you pip install?
    + Do a literature search of previous blogs.
    + Read the pip source?
    + Read documentation?
- What about easy_install?
- What is a .pth file?
- Site packages, what's the deal with that?
- What is zope?
    + See if I can actually watch this presentation? http://www.simplistix.co.uk/presentations/python_package_management_08/python_package_management_08.pdf
- How do entry points work? What if the code needs configuration files to run? Does the `etc` directory screw over Juniper?


#Motivation
I'm going to talk about something really boring. That thing is packaging and distributing Python code. 

For most of my Python-writing career, I've written either code for myself, or as part of a larger proprietary project where someone else took care of packaging (if at all). A couple of months ago, trying to share an integrate those codebases with another team was... interesting, and I finally had to get a grasp on a bunch of packagingy stuff I'd taken for granted.

Previously, I'd heavily used virtualenv and virutalenvwrapper. For the hobbiest, this is remarkably easy and convenient. It turns out that actually packaging Python is generally considered to be abysmal, and my beloved virtualenv has its issues. I'll let some other people rant about that if you're interested: [example](https://pythonrants.wordpress.com/2013/12/06/why-i-hate-virtualenv-and-pip/)

But, it's useful to know what we have. First question first... what is the difference between a *module* *package* *distribution* *egg* and *wheel*?

#Important terms
The PyPI (Python Package Index, preivously known as the Cheese Shop, has a [glossary](https://packaging.python.org/en/latest/glossary.html). Here are the ones I was previously confused about:

`module` - In Python, a single .py file is a module. This means that Python handles encapsulation without classes quite nicely. You can put some methods and constants into a file and whoop, there it is, a logically isolated bit of code.
`package` - A collection of modules. The tricky thing is that it can either be an `import package`, which is module that contains other modules that you can `import`, or a `distribution package`, described by `distribution` below. In colloquial usage it seems to mean "a collection of Python code published as a unit".
`distribution` - An archive with a version and all kinds of files necessary to make the code run. This term tends to be avoided to thwart confusion with Linux distros or larger software projects.
`egg` - A packaging format of Python files with some metadata, introduced by the `setuptools` library, but gradually being phased out.
`wheel` - The new version of the egg. It has many purported benefits, like being more implementaton-agnostic. [More info](http://wheel.readthedocs.org/en/latest/story.html). About half of the most popular Python packages are now wheels on PyPI rather than eggs. [Citation](http://pythonwheels.com/).

#A breif history of tools and what the implications of each are in terms of workflow
The history of setuptools vs distutils is mildly interesting. Basically, distutils came first, but it lacked some really integral features. For example: uninstallation. Setuptools was written to be a superset of distutils, but inherited some of the same problems. According to [this](http://www.aosabook.org/en/packaging.html) enlightening but slightly soporific article from the Architecture of Open Source, one key issue is that the same code was written to take care of both publishing and installing Python packages.

Meanwhile, there is/was a also a fight between `easy_install` and `pip`. `easy_install`'s main virtues are that it always works as well as it can work, including on Windows. `pip` has a much richer feature set, like keeping track of requirements heirarchies and uninstallation, but is more finicky.

#The golden standard now - the wheel


#How does pip work?


#Comparison to other package managers (ruby gems?) and the general philosophy of package managment.



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

"Virutalenv and pip basics" -> An article I've apparently read in the past and enjoyed. 



TODO:
Why `__init__.py`?
What is pacman?

Thought: it's true that eventually everyone needs isolation beyond that of Python and pip. Looking back at my projects from Juniper, the most important things I tended to install via apt (i.e. python-psychopg2.https://pythonrants.wordpress.com/2013/12/06/why-i-hate-virtualenv-and-pip/

Some weaknesses of pip:
- Everything installed from source.

    TODO read this
    http://lucumr.pocoo.org/2014/1/27/python-on-wheels/ 

    TODO read the wheels source code.


