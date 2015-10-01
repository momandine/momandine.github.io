---
layout: post
category : python
tagline: "Eggs, distributions, etc"
tags : [python, packaging]
---
{% include JB/setup %}

I'm going to talk about something really boring. That thing is packaging and distributing Python code.

###Motivation 
How do you make a Python package?

I dunno. Well, I didn't know. Do I look like I work on open source or something? (Don't answer that.)

But actually, if you want continuous deployment, having your code in easily installable bundles is pretty useful.

It turns out some people think that Python packacking is sucky. I wasn't aware, because I happily used pip and virtualenv and virtualenvwrapper and kept my environments pretty tidy. The problem is that the ONLY thing that Python tries to encapsulate is Python code - not dependencies, not environment variables, etc. Here is some more ranting about that from other people if you're interested: [example](https://pythonrants.wordpress.com/2013/12/06/why-i-hate-virtualenv-and-pip/).

But, to be descriptivist before we're prescriptivist, let's define what's out there. 

###Important terms
The PyPI (Python Package Index, preivously known as the Cheese Shop, has a [glossary](https://packaging.python.org/en/latest/glossary.html). Here are the ones I was previously confused about:

`module` - In Python, a single .py file is a module. This means that Python handles encapsulation without classes quite nicely. You can put some methods and constants into a file and whoop, there it is, a logically isolated bit of code.
`package` - A collection of modules. The tricky thing is that it can either be an `import package`, which is module that contains other modules that you can `import`, or a `distribution package`, described by `distribution` below. In colloquial usage it seems to mean "a collection of Python code published as a unit".
`distribution` - An archive with a version and all kinds of files necessary to make the code run. This term tends to be avoided in the Python community to thwart confusion with Linux distros or larger software projects. 
    Many distributions are `binary` or `built`, shortened to `bdist`, and are binary, platform-specific, and python-version specific. `sdist` or `source` distributions are made up only of source files and various other data files you might need. As you might expect, they must be built before they are run.
`egg` - A built packaging format of Python files with some metadata, introduced by the `setuptools` library, but gradually being phased out. It can be put direcly on the PYTHONPATH without unpacking, which is kind of nice.
`wheel` - The new version of the egg. It has many purported benefits, like being more implementaton-agnostic. [More info](http://wheel.readthedocs.org/en/latest/story.html). About half of the most popular Python packages are now wheels on PyPI rather than eggs. [Citation](http://pythonwheels.com/).

###A brief history
The history of setuptools vs distutils is mildly interesting. Basically, distutils came first, but it lacked some really integral features. For example: uninstallation. Setuptools was written to be a superset of distutils, but inherited some of the same problems. According to [this](http://www.aosabook.org/en/packaging.html) enlightening but slightly soporific article from the Architecture of Open Source, one key issue is that the same code was written to take care of both publishing and installing Python packages. There's a great diagram in that article with all of the flows by which setup.py is used in the packaging and installation process. That definitely violates the single responsibility principle.

Meanwhile, there is/was a also a fight between easy\_install and pip. Easy_install's main virtues are that it always works as well as it can work, including on Windows. It comes automatically with Python. Pip has a much richer feature set, like keeping track of requirements heirarchies and uninstallation, but is more finicky.

###The golden standard now - the wheel
Out of curiosity, I read the PEP for the wheel, because most overviews of both wheel and egg kept referring to a file with some metadata. The form of the metadata would obviously be different, but I couldn't catch the significance for each.
https://www.python.org/dev/peps/pep-0427/
###What happens when you pip install?

###Comparison to other package managers (ruby gems?) and the general philosophy of package managment.


