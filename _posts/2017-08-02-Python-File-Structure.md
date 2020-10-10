---
title: The Hitchhiker's Python Directory Structure
category: dev
layout: default
---

I've been making a lot of python projects lately, simply due to the fact that they have fast turnaround time, so I feel like I'm getting something done.

I don't want to be called mean names by people on the internet on the way I structure my projects/have bad code smell, so I read the hitchhiker's guide. This is what I found.

# Top Level

At the top level, we should only have the following:

- `README.md` for initial documentation.
- `LICENSE` for your ability to distribute this software
- `setup.py` install this via pip. Everyone uses it for installing their package onto the environment.
- `requirements.txt` for pip to know what dependencies we need.
    - If we don't have a ton of requirements, we can also just merge `requirements.txt` into `setup.py`. 
- `module` this is the core of our product.
- `tests` this is our test suite.
    - But you may ask: "How does the test files access the module?". We'll answer this later.

## Setup.py

`setup.py` describes things like `name`, `version`, `packages`. If you want your project to be indexed on PyPI, then you need a unique `name` field.

An example of metadata in your `setup.py`:

```python
import sys, os
try:
    from setuptools import setup
except ImportError:
    from distutils.core import setup

setup(name='package_name',
      version='version_1',
      description='this is a package',
      license='mit',
      install_requires=['package1','package2'],
      py_modules=['core_module']
    ) 
```

Why do we have `distutils` OR `setuptools`? Here's the history:

> **Distutils** is still the standard tool for packaging in Python. It is included in the standard library (Python 2 and Python 3.0 to 3.3). It is useful for simple Python distributions, but lacks features. It introduces the distutils Python package that can be imported in your setup.py script.

> **Setuptools** was developed to overcome Distutils' limitations, and is not included in the standard library. It introduced a command-line utility called easy_install. It also introduced the setuptools Python package that can be imported in your setup.py script, and the pkg_resources Python package that can be imported in your code to locate data files installed with a distribution. One of its gotchas is that it monkey-patches the distutils Python package. It should work well with pip. The latest version was released in July 2013.




### Makefile(Optional)

Sometimes you'll see a makefile for **python development**, and you might ask:

> Why? Python doesn't require compiling...

And you're right, but makefiles do more than just "make". It's easier to type:

`$make test`

than

`$py.test tests` if you're using the `py.test` package, or `$python tests/test.py`.

Plus, you wrap that abstraction into your makefile so you don't need to know how you're testing.

## Tests

We have 2 options: either use `sys.path.append`, `sys.path.insert(0, ...)`, or `python setup.py develop` and insert the package into `site-packages`, which should be in your `$PYTHONPATH`.

I like to do the `sys` library way. It means that your code does not depend on the environment that you're running on. What if you don't even have a site-packages? 

#### Absolute import
```python
import os
import sys
sys.path.insert(0, os.path.abspath(os.path.join(os.path.dirname(__file__), '..')))
from .module import core
```

or:

#### Relative import
```python
import os
import sys
sys.path.insert(0, '..')))
from .module import core
```

## Module

Python's `import` keyword uses dots to go 1 directory down.

```python
import library.plugin.foo
```

Is equivalent to this file structure:

```
library
    |
    - plugin
        |
        - foo.py
```

One thing that I did before, that you should never do, is this:

```python
from foo import *
```

This obfuscates the namescope; we don't know whether a function `f()` is a part of our standard library, something we implemented in this current python file, or in `foo.py`. That's 3 places to look at if you get a bug!

### What is a package?

Simple: it's a directory with an `__init__.py` file in it. It's used to gather all package definitions within it. 

If you `import library.plugin.foo`, it will look at `__init__.py` in `library`, then execute all of its statements, and then `__init__.py` in `plugin`, and then finally look for a file called `foo.py`, where it will add all of its names to scope.

The issue with adding too much code to `__init__.py` is that if you have a deep directory structure, you need to call `__init__.py` many times. **This will slow down imports.** Thus, the standard solution is to minimize your code in `__init__.py`, as much as possible.

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
