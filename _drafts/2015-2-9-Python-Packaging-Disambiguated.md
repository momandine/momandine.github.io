---
layout: post
category : python
tagline: "Eggs, distributions, etc"
tags : [python, packaging]
---
{% include JB/setup %}

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

What are the important things about setuptools?

