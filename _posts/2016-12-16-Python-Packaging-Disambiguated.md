---
layout: post
category : python
tagline: "Eggs, distributions, etc"
tags : [python, packaging]
---
{% include JB/setup %}

I'm going to talk about something really boring. That thing is packaging and distributing Python code.

###Motivation 
I have been programming primarily in Python for about 3.5 years now, happily creating virtualenvs and pip installing things - and sometimes yak shaving non-Python dependencies of those libraries - without any idea how python code is packaged and distributed. And then, one day I had to share a library my team had written with another team, and lo and behold, I wrote my first requirements.txt file. I was inspired to peek under the hood.

It turns out some people think that Python packaging is sucky. For most of my purposes - working on small pet projects or in fully isolated development environments maintained by another engineer, this works fine. The problem is that the ONLY thing that Python tries to encapsulate is Python code - not dependencies, not environment variables, etc. There is some more exposition about that from other people if you're interested: [example](https://pythonrants.wordpress.com/2013/12/06/why-i-hate-virtualenv-and-pip/).

The biggest hurdle to getting started was understanding the lingo.

###Important terms
The PyPI (Python Package Index), previously known as the Cheese Shop, has a [glossary](https://packaging.python.org/en/latest/glossary.html) with everything that you might want to know. Here are the ones I was previously confused about:

`module` - In Python, a single .py file is a module. This means that Python can be organized without classes quite nicely. You can put some methods and constants into a file and tada, a logically isolated bit of code.
`package` - A collection of modules. The tricky thing is that it can either be an `import package`, which is module that contains other modules that you can `import`, or a `distribution package`, described by `distribution` below. In colloquial usage it seems to mean "a collection of Python code published as a unit".
`distribution` - An archive with a version and all kinds of files necessary to make the code run. This term tends to be avoided in the Python community to prevent confusion with Linux distros or larger software projects. 
    Many distributions are `binary` or `built`, shortened to `bdist`, and are binary, platform-specific, and python-version specific. `sdist` or `source` distributions are made up only of source files and various other data files you would need. As you might expect, they must be built before they are run.
`egg` - A built packaging format of Python files with some metadata, introduced by the `setuptools` library, but gradually being phased out. It can be put direcly on the PYTHONPATH without unpacking, which is kind of nice.
`wheel` - The new version of the egg. It has many purported benefits, like being more implementaton-agnosticmm and faster than building from source. [More info](http://wheel.readthedocs.org/en/latest/story.html). About two thirds of the most popular Python packages are now wheels on PyPI rather than eggs. [Citation](http://pythonwheels.com/).

I also found it mildly interesting to figure out how this somewhat fractured environment came to me.

###A brief history
There is a [chapter](http://www.aosabook.org/en/packaging.html) of The Architecture of Open Source dedicated to nitty gritty of python packaging. The CliffNotes version is that there was some turbulence over Python's packinging libraries, setuptools and distutils. distutils came first, but it lacked some really integral features... like uninstallation. Setuptools was written to be a superset of distutils, but inherited some of the same problems. One key issue is that the same code was written to take care of both publishing and installing Python packages, which meant bad separation of responsibility.

Meanwhile, there was a also some friction between easy\_install and pip. Easy\_install comes automatically with Python, can install from eggs, and has perfect feature parity on Windows, but pip has a much richer feature set, like keeping track of requirements heirarchies and (most of the time) uninstallation, but is more finicky.

I may follow up with more on how virtualenv and pip actually work, stay tuned!
