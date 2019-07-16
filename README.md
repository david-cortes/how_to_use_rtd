# How to document Python packages

This is a small guide on how to create documentation for scientific/mathematical/statistical Python packages and host it in [ReadTheDocs](https://readthedocs.org), paying attention to the small details that differentiate these types of packages from regular Python packages.

# Overview

The vast majority of scientific/mathematical/statistical Python packages all use some common conventions: NumPy-style docstrings, SciKit-Learn-Like API, docs generated through `sphinx`, LaTeX notation for mathematical formulas, compiled code wrapped through Cython/NumPy instead of regular Python way, sometimes using hard-to-build tools like TensorFlow, among others.

Sticking to these conventions makes code easier to understand by other data/math-oriented Python users, but the regular tools available in Python are not friendly towards these use cases, and getting an auto-generated documentation for a package, such as the one from SciPy ([example](https://scipy.github.io/devdocs/generated/scipy.special.gamma.html#scipy.special.gamma])), is far more complicated and much less intutitive than in, say, R. Even after generating this documentation, hosting it only in [ReadTheDocs](https://readthedocs.org), where most small packages upload such documentation, is yet another set of troubles every time one tries to make it build in their servers.


This is guide was written after many frustrations and countless hours trying to document my own packages. All examples here are from small statistical packages that I've had the misfortune of not getting to build in RTD. It is by no means comprehensive, and it's missing instructions on how to obtain lots of commonly-desired funcionality, but I hope it will still be useful for people who create packages.


# Steps

## 1. Document functions/classess/methods using NumPy-style docstrings

In order for documentation to be picked automatically and rendered properly in an IDE like Spyder, it has to follow NumPy style (there are many other styles such as Google's, which do not render in the same way and are not typically used in math/data-oriented packages), and needs to be created as so-called docstrings - that is, a string that is put right beneath a function/class/method definition - e.g.
```python
def a_function(a, b):
	"""
	Lorem Ipsum

	Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.

	Note
	----
	You can put many notes section

	Parameters
	----------
	a : array(n_rows, )
		Some random 1-d array
	b : array(n_rows, )
		Some random 1-d array

	Returns
	-------
	sum : array(n_rows, )
		Sum of the inputs
	"""
	return a + b
```

* **Highlights:** String starts with a title that describes the function shortly, then a paragraph explaining in more detail, then some **pre-defined** sections: `Note` (optional), `Parameters`, `Returns` (for classes initializers, replace `Returns` with `Attributes`; can also add `Examples`). These sections should be followed in the next line by dashes of the same length. Python doesn't have typed variables like C/C++, so the expected type (and size when they are arrays) must be specified (as `variable_name : type(size_dim1, size_dim2, ...)`), and the next line, which describes the variable in words, must have an extra indentation.

* **Pitfalls:** Not all keywords are valid - e.g. if you try `Note2` in the same way as `Note`, it will not end up showing. When documenting a class, the docstrings might be put right under the class definition, or under the `__init__` method, but sometimes some software will only accept one of those - as of 16/07/2019, ReadTheDocs will end up showing what you put under the class definition. Backticks don't turn into unformatted code.

* **Tips:** The docs will also be available as an attribute `.__doc__` in the object that has it (e.g. `myObject.__doc__`, `myObject.some_method.__doc__`). You can use LaTeX for formulas. It's not mandatory to use triple quotes, and line breaks can be put anywhere and will be treated as regular spaces when the docs are generated.

* **Example:** See Scikit-Learn's [Logistic Regression docs](https://github.com/scikit-learn/scikit-learn/blob/master/sklearn/linear_model/logistic.py).


## 2. Auto-document with Sphinx

Requires Python package `sphinx`, which can be installed from `pip`:
```
pip install Sphinx
```

Once you have structured the package files into some folder, e.g.
```
your_package/
	__init__.py
	other_file.py
setup.py
```

Create a new folder called `docs` at the root (should be at the same level where folder `your_package` is), go to that folder, open a command prompt there, and type:
```
sphinx-quickstart
```

It will start asking some questions and offer some default answers. **DO NOT GO FOR THE DEFAULT CHOICES!!!**. For some reason, the default is not to pick those docstrings from the previous steps to insert them into the docs. Start putting all the default choices (some don't have a default, such as the project name or the author), but **be careful to choose the following:**
```
> autodoc: automatically insert docstrings from modules (y/n) [n]: ---> answer 'y' (without the quotes)
```

You might also want to choose these other non-defaults:
```
> Separate source and build directories (y/n) [n]: y
> mathjax: include math, rendered in the browser by MathJax (y/n) [n]: y
> viewcode: include links to the source code of documented Python objects (y/n) [n]: y
```

## 3. Make Sphinx show the documented modules

Despite what the previous step looked like it was doing, that didn't generate rendable documentation yet. Now you need to add the package's modules to the list of docs that are shown. First make it pick the autodocs - inside the same folder `/docs`, execute in the command prompt:
```
sphinx-apidoc -o source/ ../your_package
```
(Replace `your_package` with the actual name of the folder, and if there's no `source` folder, replace `source/` with `.`)

It generated some files inside the `/docs` folder - now look for the file `index.rst` (will be inside `/source` if you chose separate source/build directories), and edit it (uses `rst` format, which is similar to markdown, but not the same and not compatible). At the end of the file, you should find something like this:
```
Welcome to <package name> documentation!
========================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

```

Replace it with this:
```
Your fancy title
================

.. toctree::
	:maxdepth: 2

.. automodule:: your_package
    :members:
    :undoc-members:
    :show-inheritance:
    :inherited-members:

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
```
(Replace `your_package` and `Your fancy title`).
(But don't delete the auto-generated header)

Optionally, you might also want to put a better introductory message and add a link to the package's official webpage - replace what's before the text above with something like this:
```
Your fancy title
================
Your description goes here

`<https://www.your.project/link>`_
```

If the package has multiple modules, you'll need to add them separately - example here from package [contextualbandits](https://raw.githubusercontent.com/david-cortes/contextualbandits/master/docs/index.rst). 

## 4. Configure Sphinx to render your NumPy-style docs

Now in the same folder that had the `index.rst`, you'll find also a `conf.py` file. There you need to make some replacements to make your documentation render properly, by using the Sphinx extension called Napoleon. Make the following replacements:

From
```python
extensions = ['sphinx.ext.autodoc']
```

To:
```python
extensions = ['sphinx.ext.autodoc', 'sphinx.ext.napoleon']
napoleon_google_docstring = False
napoleon_use_param = False
napoleon_use_ivar = True
```

(The following assumes you are building the docs for ReadTheDocs, for local docs the steps will be different as you need to manually add the theme which RTD adds automatically)

Optionally, if you selected in step 2 to have links to the source code, and you want docs to render with a better theme than the ugly default white, add:
```python
extensions = [
    'sphinx.ext.autodoc',
    'sphinx.ext.viewcode',
    'sphinx.ext.napoleon',
    'sphinx_rtd_theme'
]
napoleon_google_docstring = False
napoleon_use_param = False
napoleon_use_ivar = True
```

In order to avoid the ugly default theme which makes the docs harder to read, comment out the line that sets the style to Alabaster.

From:
```python
html_theme = 'alabaster'
```

To:
```python
# html_theme = 'alabaster'
```

Example here from package [outliertree](https://github.com/david-cortes/outliertree/blob/master/docs/source/conf.py).

## 5. Add dependencies/requirements

Your package will very likely have dependencies. When distributing packages, it's usually enough to specify them in the `setup.py` file in argument `install_requires`, but for RTD you'll need to add a file `requirements.txt` in the package root folder:
```
your_package/
	__init__.py
	other_file.py
setup.py
requirements.txt << create it now
```

This `requirements.txt` should have one line per package, e.g.
```
numpy
pandas
```

If you need a version above something (e.g. `pandas` introduced methods `.to_numpy()` in 0.24.0), you can add:
```
numpy
pandas>=0.24.0
```

#### Compiled code or problematic dependencies

Some dependencies are more problematic - for example, if your package depends on another package that depends on `cython`, you might need to add what's called a build-time dependency, which is specified through yet another file with its own mechanism (otherwise you might end up with empty docs on RTD). If that's the case (and it's anyway a good practice even if it's no the case), create in the package folder another file `pyproject.toml` and `MANIFEST.in` if you don't already have it (it controls the files that are included in the final package).

```
your_package/
	__init__.py
	other_file.py
setup.py
requirements.txt
pyproject.toml <<  create now
MANIFEST.in    <<  create if it doesn't exist
```

* pyproject.toml:
```
[build-system]
requires = [""numpy", "pandas>=0.24.0", "cython", "setuptools", "wheel"]
```

* MANIFEST.in:
```
include pyproject.toml
```

**Important:** if you add the `pyproject.toml`, it will need to specify every single dependency, that's why you need to add `setuptools` and `wheel`, without which your package will not be installable.

**Important:** you must use `setuptools` instead of distutils - e.g. your `setup.py` should look like this:
```python
from setuptools import setup
from setuptools.extension import Extension
```

and **NOT** like this:
```python
from distutils.core import setup
from distutils.extension import Extension
```

#### RTD Mock: when dependencies are a basket-case

Some dependencies are just overall problematic. `tensorflow` is an example of a bad dependency that refuses to build or to follow normal rules. If the dependencies do not need to execute anything in order for the package to build (and they shouldn't unless it's `numpy` or `cython`), RTD offers the possibility of creating a 'mock' package, which will basically define a module, import it, and let it occupy the otherwise problematic name. Go back to `conf.py` in `/docs` or `/docs/source`, this time edit the beginning as follows:

Before:
```python
# import os
# import sys
# sys.path.insert(0, os.path.abspath('.'))
```

After:
```python
import os
import sys
sys.path.insert(0, os.path.abspath('../'))

import mock 
MOCK_MODULES = ['tensorflow', 'pandas']
sys.modules.update((mod_name, mock.MagicMock()) for mod_name in MOCK_MODULES)
```

(depending on whether you have a `source/` folder, it might notbe necessary to insert the syspath above)

For more problematic cases, RTD will define an environment variable `READTHEDOCS` (`os.environ.get('READTHEDOCS')`) which you can use to tweak or skip things conditionally. For example, if you want to offer the possibility of using two equivalent packages as backend, but don't want to force any as dependency, you can adjust your `setup.py` and/or `conf.py` to have a mock of it, or you can have a special `setup` command in your `setup.py` that will be executed only for RTD.

See example in package [findblas](https://github.com/david-cortes/findblas/blob/master/findblas/distutils.py) which wants to have MKL or OpenBLAS without having to list one of them as a requirement.

## 6. Building docs at RTD

Finally, the package is ready for upload at ReadTheDocs. You need to register in there and follow the steps. They have integration with platforms such as GitHub, but otherwise you can add your project URL there to build.

But this is not the end of the story, this is now where the problems begin. At the first try, it's very likely that the docs it will build will just be empty.

* If you have a Cython extension, you'll need to enable it to install itself locally. Go to the project entry in your RTD account, click the `Admin` button, go to `Settings`, go to `Advanced Settings`, and tick the box that says `Install Project`.
* If you use non-Python packages such as `ODBC` drivers, you also need to tick `Use system packages`.
* If you have some other error, you can check the build detail, click each of the boxes, and look for some `NotFound` error. It will likely be covered through some of the cases in here.
* Sometimes it can help to just change the `sys.path` in `conf.py`, like in the instructions from the step above, but without the `mock` part.
* Googling and StackOverflow typically offer outdated answers that will not work. You can also try the [issue tracker for RTD](https://github.com/readthedocs/readthedocs.org/issues).


## END

That's it. Contributions and tips welcome for addition to this guide.
