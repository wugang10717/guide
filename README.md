# Developer-Guide
A crash course on developing for SSI's STMS python projects with dotfiles, style guides, wiki, and more.

## How-To Use
This guide will walk new STMS developers through the following specifications and processes:

* Setting up your development environment
* Python project file hierarchy/structure (with distribution)
* Python style guide and linting
* Test-driven development and coverage
* Github/Git usage and pull request flow

## Environment Setup

First, ensure that you have a text editing application that you are comfortable with. Vim, emacs, and notepad++ are fine, but we would recommend using [Sublime Text](https://www.sublimetext.com/) or [Atom](https://atom.io/) as lightweight editors with lots of plugin support, or [PyCharm](https://www.jetbrains.com/pycharm/) for a full IDE experience.

Next, we will want ensure that you have `git` installed on your computer. You can run `git --version` to see if you already have it installed (chances are, you do). Otherwise, go ahead and install it from [git's download website](https://git-scm.com/downloads).

Now, you will need the latest version of python 2.x! You can download the latest 2.x release [here](https://www.python.org/downloads/). Once python is installed, you will also need its package manager, `pip`. To install `pip`, download [get-pip.py](https://bootstrap.pypa.io/get-pip.py) and run it with: `python get-pip.py`.

All STMS python development within virtualenvs. Virtualenv allows you separate your projects into different workspaces that can have different versions of python pkg dependencies installed. Go ahead and install virtualenv with: `pip install`. virtualenv

To make working with virtualenvs even easier, install virtualenvwrapper: `pip install virtualenvwrapper`.

Finish setting up virtualenvwrapper by adding the following to your .shellrc file:

		export WORKON_HOME=$HOME/.virtualenvs
		export PROJECT_HOME=$HOME/Devel
		source /usr/local/bin/virtualenvwrapper.sh

and `sourcing` the profile. You can now swap between your virtualenvs with a simple `workon` command. To see how to use virtualenvwrapper in more detail, refer to the [docs](https://virtualenvwrapper.readthedocs.org/en/latest/command_ref.html).

This is really all you need to start hacking on STMS projects. Simply ensure you have a virtualenv associated with a particular git repo before you start working. For example, if you were to begin developing in STMS-Eos, you would do the following:

		git clone git@github.corp.ebay.com:STMS/Eos.git
		mkvirtualenv -a /path/to/Eos eos
		cd Eos
		pip install -r requirements/dev-requirements.txt

and now, at anytime, running `workon eos` will activate your eos virtualenv with all of eos' dependencies installed and will cd into the eos project folder, where you are ready to work!

## File Hierarchy

When writing an STMS python project, your file hierarchy will typically fall under three distinct package types:

1. Django Web App (with or without celery)
2. Generalized python module (for use in other projects)
3. Python module to distribute as command-line tool

### Django Web App

All STMS django applications use a file hierarchy that looks like the following:

	stmsproject/
        manage.py
        common/
        	__init__.py
        	util/
            	__init__.py
                randomutils.py
            overridesforsomemodule/
            	__init__.py
                overrides.py
            config.py
            constants.py
        stmsproject/
            __init__.py
            urls.py
            wsgi.py
            settings.py
        config/
        	gunicorn.py
            nginx.conf
        exampleapp/
            __init__.py
            models.py
            views.py
            urls.py
            migrations/
            	__init__.py
            	0001_initial.py
            templates/
                exampleapp/
                    base.html
                    list.html
                    detail.html
            static/
                ...
            tests/
                __init__.py
                test_models.py
                test_managers.py
                test_views.py
         tasks/
         	__init__.py
         	tasks.py
         static/
             css/
                 ...
             js/
                 ...
         templates/
             base.html
             index.html
         requirements/
             base.txt
             dev.txt

Most of this hierarchy should be self-explanatory, but do note the following:

* `common/` will contain a variety of file types from generic utility functions/classes to overriden functionality in 3rd-party modules (like django rest framework) to defining constants and configs
* `config/` will contain configuration files specific to our prod deployment environment. You can read about deploying STMS web applications and project configuration details in the [STMS-Tyche repo](https://github.corp.ebay.com/STMS/Tyche).
* `tasks/` is a central place for defining anything related to [celery](http://www.celeryproject.org/)

If you don't have an understanding of Django or what most of these directories mean, run through the [Django tutorial](https://docs.djangoproject.com/en/1.9/intro/tutorial01/).

For an example of how an STMS web application is structured, see the [STMS-Atlas repo](https://github.corp.ebay.com/STMS/Atlas).

### Python Module

All STMS python modules use a file hierarchy that looks like the following:

	stmsmodule/
        stmsmodule/
            __init__.py
            stms.py
            example.py
		tests/
        	__init__.py
            test_stms.py
        requirements/
        	requirements.txt
            develop-requirements.txt
        setup.py

For an example of how an STMS module is structured, see the [pyCMS repo](https://github.corp.ebay.com/mmedal/pyCMS).

### Python Module with CLI Component

All python modules with a CLI component should use the file hierarchy shown in the previous section but with the addition of a `bin/` directory in the root of the repo. Here, place executable scripts that import from another module and call their respective CLI component. For example:

	stmsmodule/
    	bin/
        	stms-cli
        stmsmodule/
            __init__.py
            stms.py
            example.py
		tests/
        	__init__.py
            test_stms.py
        requirements/
        	requirements.txt
            develop-requirements.txt
        setup.py

where `stms-cli` contains the following:

    #!/usr/bin/env python

    from stmsmodule.stms import main


    main()

These executable scripts can then be automatically symlinked into `/usr/local/bin` during a `python setup.py install` by adding the following to your `setup.py`:

	setup(
    	...
        scripts=[
        	'bin/stms-cli'
        ],
        ...
    )

We will touch on the `setup.py` file some more in the next section.

### What Dotfiles Should Be in the Root of My Projects?

All STMS projects should contain the following files in the root directory:

#### .coveragerc

The `.coveragerc` file contains configuration details relating to our testing requirements. Ensure that it specifies all source files to include in coverage-mapping under the `include=` heading and omits all non-source files (like tests, config files, etc) under the `omit=` heading. An STMS `.coveragerc` file should look like the following:

    [run]
    branch = True
    include =
               stmsmodule/*
    omit =
            *__init__*
            *test*

    [coverage]
    always-on = True

Note: Only the `include=` and `omit=` headers should be edited. Uses of this file will be covered later in the **Testing** section.

#### .flake8

The `.flake8` file simply overrides PEP-8's requirement that lines of code be no longer that 79 characters. We will cover code-style and linting in a later section. Simply ensure that your project contains a `.flake8` with the following:

	[flake8]
    max-line-length = 119

#### .gitignore

While many will have a global `.gitignore` that they use, we like to keep a `.gitignore` in our project repos. Ensure that your project's `.gitignore` at least contains the following:

    # Byte-compiled / optimized / DLL files
    __pycache__/
    *.py[cod]
    *$py.class

    # C extensions
    *.so

    # Distribution / packaging
    .Python
    env/
    build/
    develop-eggs/
    dist/
    downloads/
    eggs/
    .eggs/
    lib/
    lib64/
    parts/
    sdist/
    var/
    *.egg-info/
    .installed.cfg
    *.egg

    # STMS
    dump.rdb
    config/allowed_hosts.txt

    # OSX
    .DS_Store

    # PyInstaller
    #  Usually these files are written by a python script from a template
    #  before PyInstaller builds the exe, so as to inject date/other infos into it.
    *.manifest
    *.spec

    # Installer logs
    pip-log.txt
    pip-delete-this-directory.txt

    # Unit test / coverage reports
    htmlcov/
    .tox/
    .coverage
    .coverage.*
    .cache
    nosetests.xml
    coverage.xml
    *,cover
    .hypothesis/
    .idea/

    # Translations
    *.mo
    *.pot

    # Django
    *.log
    local_settings.py
    settings.py
    settings.extra.py

    # Celery
    celerybeat-schedule.db

    # IDE
    .idea/

    # Sphinx documentation
    docs/_build/

    # PyBuilder
    target/

    # Ipython Notebook
    .ipynb_checkpoints

    # pyenv
    .python-version

#### .editorconfig

This file is another config to help enforce STMS coding style. Please ensure the following contents are contained within your project's `.editorconfig`.


    root = true

    [*]
    end_of_line = lf
    insert_final_newline = true
    trim_trailing_whitespace = true

    [*.{html,js,json,py}]
    indent_style = space
    indent_size = 4

#### README.rst/.md

All STMS projects should contain a README that specifies how the project is used and/or deployed. Use your best judgement here in determining its verbosity. You should either use [markdown](https://daringfireball.net/projects/markdown/syntax) or [reStructuredText](http://docutils.sourceforge.net/rst.html).

**All STMS django projects should additionally contain:**

#### .bowerrc

STMS web applications use [bower](http://bower.io/) to specify static dependencies. Ensure that your `.bowerrc` contains to following to ensure that our standard file hierarchy for django projects is maintained:

    {
      "directory": "static/lib"
    }

#### bower.json

Used to specify static dependencies for the project. An example can be found in Atlas' [bower.json](https://github.corp.ebay.com/STMS/Atlas/blob/master/bower.json). Documentation for `bower.json` can be found in [bower's docs](http://bower.io/docs/creating-packages/#bowerjson).

#### Dockerfile

STMS web applications are deployed using Docker. Read about our deployment process in the [STMS-Tyche Repo](https://github.corp.ebay.com/STMS/Tyche) and about how to construct a `Dockerfile` in the [Docker docs](https://docs.docker.com/engine/reference/builder/). An example STMS `Dockerfile` can be found in the [Atlas repo](https://github.corp.ebay.com/STMS/Atlas/blob/master/Dockerfile).

**All STMS modules (with or without CLI components) should additionally contain:**

#### setup.py

We want our modules to be easily installable from a `git clone`, github directly, or even a private pkg repo we maintain. As such, we want to ensure that our STMS modules contain a `setup.py` file that allows the module to be installable via a `python setup.py install`. Please read the [python docs](https://docs.python.org/2/distutils/setupscript.html) on how to write a `setup.py` file and see [Eos' setup.py](https://github.corp.ebay.com/STMS/Eos/blob/master/setup.py) for an example.

**Finally, every STMS project should contain a `requirements/` directory containing at least two files:**

* requirements.txt
* develop-requirements.txt

We use [pip](https://pip.pypa.io/en/stable/) for installing all python dependencies. As such, we use these two files to specify dependencies for our projects. Use `requirements.txt` to specify all dependencies needed for the code to be used and `development-requirements.txt` to specify all dependencies needed for the code to be worked on (i.e. testing dependencies). You can read about `requirements.txt` files in the [pip docs](https://pip.pypa.io/en/stable/user_guide/#requirements-files).

## STMS Code Style Guide

STMS python coding style uses a slightly modified version of PEP-8 as a basis for styling its code. Please read the [pep-8.md](https://github.corp.ebay.com/STMS/Developer-Guide/blob/master/pep-8.md) included in this repo for a full explanation of code style.

As explained above, ensure that you have the `.flake8` file in the root of your repo and that flake8 is listed as a dependency in your developer-requirements.txt. Before making an official pull requests, ensure that running flake8 does not return any errors on changes you have made by simply running the `flake8` cmd in the root of your project repo.

Other than explicit syntax, there is also the matter of _good taste_ when writing python. We want our STMS code to be uniform, neat, and easy to read. Please follow the following guidelines concerning _taste_:

### General concepts

#### Explicit code

While any kind of black magic is possible with Python, the
most explicit and straightforward manner is preferred.

**Bad**

    def make_complex(*args):
        x, y = args
        return dict(**locals())

**Good**

    def make_complex(x, y):
        return {'x': x, 'y': y}

In the good code above, x and y are explicitly received from
the caller, and an explicit dictionary is returned. The developer
using this function knows exactly what to do by reading the
first and last lines, which is not the case with the bad example.

#### One statement per line

While some compound statements such as list comprehensions are
allowed and appreciated for their brevity and their expressiveness,
it is bad practice to have two disjoint statements on the same line of code.

**Bad**

    print 'one'; print 'two'

    if x == 1: print 'one'

    if <complex comparison> and <other complex comparison>:
        # do something

**Good**

    print 'one'
    print 'two'

    if x == 1:
        print 'one'

    cond1 = <complex comparison>
    cond2 = <other complex comparison>
    if cond1 and cond2:
        # do something

#### Function arguments

Arguments can be passed to functions in four different ways.

1. **Positional arguments** are mandatory and have no default values. They are
   the simplest form of arguments and they can be used for the few function
   arguments that are fully part of the function's meaning and their order is
   natural. For instance, in `send(message, recipient)` or `point(x, y)`
   the user of the function has no difficulty remembering that those two
   functions require two arguments, and in which order.

In those two cases, it is possible to use argument names when calling the
functions and, doing so, it is possible to switch the order of arguments,
calling for instance `send(recipient='World', message='Hello')` and
`point(y=2, x=1)` but this reduces readability and is unnecessarily verbose,
compared to the more straightforward calls to `send('Hello', 'World')` and
`point(1, 2)`.

2. **Keyword arguments** are not mandatory and have default values. They are
   often used for optional parameters sent to the function. When a function has
   more than two or three positional parameters, its signature is more difficult
   to remember and using keyword arguments with default values is helpful. For
   instance, a more complete `send` function could be defined as
   `send(message, to, cc=None, bcc=None)`. Here `cc` and `bcc` are
   optional, and evaluate to `None` when they are not passed another value.

Calling a function with keyword arguments can be done in multiple ways in
Python, for example it is possible to follow the order of arguments in the
definition without explicitly naming the arguments, like in
`send('Hello', 'World', 'Cthulhu', 'God')`, sending a blind carbon copy to
God. It would also be possible to name arguments in another order, like in
`send('Hello again', 'World', bcc='God', cc='Cthulhu')`. Those two
possibilities are better avoided without any strong reason to not follow the
syntax that is the closest to the function definition:
`send('Hello', 'World', cc='Cthulhu', bcc='God')`.

As a side note, following `YAGNI <http://en.wikipedia.org/wiki/You_ain't_gonna_need_it>`_
principle, it is often harder to remove an optional argument (and its logic
inside the function) that was added "just in case" and is seemingly never used,
than to add a new optional argument and its logic when needed.

3. The **arbitrary argument list** is the third way to pass arguments to a
function.  If the function intention is better expressed by a signature with an
extensible number of positional arguments, it can be defined with the `*args`
constructs.  In the function body, `args` will be a tuple of all the
remaining positional arguments. For example, `send(message, *args)` can be
called with each recipient as an argument: `send('Hello', 'God', 'Mom',
'Cthulhu')`, and in the function body `args` will be equal to `('God',
'Mom', 'Cthulhu')`.

However, this construct has some drawbacks and should be used with caution. If a
function receives a list of arguments of the same nature, it is often more
clear to define it as a function of one argument, that argument being a list or
any sequence. Here, if `send` has multiple recipients, it is better to define
it explicitly: `send(message, recipients)` and call it with `send('Hello',
['God', 'Mom', 'Cthulhu'])`. This way, the user of the function can manipulate
the recipient list as a list beforehand, and it opens the possibility to pass
any sequence, including iterators, that cannot be unpacked as other sequences.

4. The **arbitrary keyword argument dictionary** is the last way to pass
   arguments to functions. If the function requires an undetermined series of
   named arguments, it is possible to use the `**kwargs` construct. In the
   function body, `kwargs` will be a dictionary of all the passed named
   arguments that have not been caught by other keyword arguments in the
   function signature.

The same caution as in the case of *arbitrary argument list* is necessary, for
similar reasons: these powerful techniques are to be used when there is a
proven necessity to use them, and they should not be used if the simpler and
clearer construct is sufficient to express the function's intention.

It is up to the programmer writing the function to determine which arguments
are positional arguments and which are optional keyword arguments, and to
decide whether to use the advanced techniques of arbitrary argument passing. If
the advice above is followed wisely, it is possible and enjoyable to write
Python functions that are:

* easy to read (the name and arguments need no explanations)

* easy to change (adding a new keyword argument does not break other parts of
  the code)

#### Avoid the magical wand

A powerful tool for hackers, Python comes with a very rich set of hooks and
tools allowing you to do almost any kind of tricky tricks. For instance, it is
possible to do each of the following:

* change how objects are created and instantiated

* change how the Python interpreter imports modules

* it is even possible (and recommended if needed) to embed C routines in Python.

However, all these options have many drawbacks and it is always better to use
the most straightforward way to achieve your goal. The main drawback is that
readability suffers greatly when using these constructs. Many code analysis
tools, such as pylint or pyflakes, will be unable to parse this "magic" code.

We consider that a Python developer should know about these nearly infinite
possibilities, because it instills confidence that no impassable problem will
be on the way. However, knowing how and particularly when **not** to use
them is very important.

Like a kung fu master, a Pythonista knows how to kill with a single finger, and
never to actually do it.

#### We are all responsible users

As seen above, Python allows many tricks, and some of them are potentially
dangerous. A good example is that any client code can override an object's
properties and methods: there is no "private" keyword in Python. This
philosophy, very different from highly defensive languages like Java, which
give a lot of mechanisms to prevent any misuse, is expressed by the saying: "We
are all responsible users".

This doesn't mean that, for example, no properties are considered private, and
that no proper encapsulation is possible in Python. Rather, instead of relying
on concrete walls erected by the developers between their code and other's, the
Python community prefers to rely on a set of conventions indicating that these
elements should not be accessed directly.

The main convention for private properties and implementation details is to
prefix all "internals" with an underscore. If the client code breaks this rule
and accesses these marked elements, any misbehavior or problems encountered if
the code is modified is the responsibility of the client code.

Using this convention generously is encouraged: any method or property that is
not intended to be used by client code should be prefixed with an underscore.
This will guarantee a better separation of duties and easier modification of
existing code; it will always be possible to publicize a private property,
but making a public property private might be a much harder operation.

#### Returning values

When a function grows in complexity it is not uncommon to use multiple return
statements inside the function's body. However, in order to keep a clear intent
and a sustainable readability level, it is preferable to avoid returning
meaningful values from many output points in the body.

There are two main cases for returning values in a function: the result of the
function return when it has been processed normally, and the error cases that
indicate a wrong input parameter or any other reason for the function to not be
able to complete its computation or task.

If you do not wish to raise exceptions for the second case, then returning a
value, such as None or False, indicating that the function could not perform
correctly might be needed. In this case, it is better to return as early as the
incorrect context has been detected. It will help to flatten the structure of
the function: all the code after the return-because-of-error statement can
assume the condition is met to further compute the function's main result.
Having multiple such return statements is often necessary.

However, when a function has multiple main exit points for its normal course,
it becomes difficult to debug the returned result, so it may be preferable to
keep a single exit point. This will also help factoring out some code paths,
and the multiple exit points are a probable indication that such a refactoring
is needed.

    def complex_function(a, b, c):
    	if not a:
    		return None  # Raising an exception might be better
    	if not b:
    		return None  # Raising an exception might be better
    	# Some complex code trying to compute x from a, b and c
    	# Resist temptation to return x if succeeded
    	if not x:
    		# Some Plan-B computation of x
    		return x  # One single exit point for the returned value x will help
    				  # when maintaining the code.

### Idioms

A programming idiom, put simply, is a *way* to write code. The notion of
programming idioms is discussed amply at [c2](http://c2.com/cgi/wiki?ProgrammingIdiom)
and at [Stack Overflow](http://stackoverflow.com/questions/302459/what-is-a-programming-idiom).

Idiomatic Python code is often referred to as being *Pythonic*.

Although there usually is one --- and preferably only one --- obvious way to do
it; *the* way to write idiomatic Python code can be non-obvious to Python
beginners. So, good idioms must be consciously acquired.

Some common Python idioms follow:

#### Unpacking

If you know the length of a list or tuple, you can assign names to its
elements with unpacking. For example, since `enumerate()` will provide
a tuple of two elements for each item in list:

    for index, item in enumerate(some_list):
        # do something with index and item

You can use this to swap variables as well:

	a, b = b, a

Nested unpacking works too:

	a, (b, c) = 1, (2, 3)

#### Create an ignored variable

If you need to assign something (for instance, in `unpacking-ref`) but
will not need that variable, use `__`:

    filename = 'foobar.txt'
    basename, __, ext = filename.rpartition('.')

__Note__:

    Many Python style guides recommend the use of a single underscore "`_`"
    for throwaway variables rather than the double underscore "`__`"
    recommended here. The issue is that "`_`" is commonly used as an alias
    for the `~gettext.gettext` function, and is also used at the
    interactive prompt to hold the value of the last operation. Using a
    double underscore instead is just as clear and almost as convenient,
    and eliminates the risk of accidentally interfering with either of
    these other use cases.

#### Create a length-N list of the same thing

Use the Python list `*` operator:

    four_nones = [None] * 4

#### Create a length-N list of lists

Because lists are mutable, the `*` operator (as above) will create a list
of N references to the `same` list, which is not likely what you want.
Instead, use a list comprehension:

    four_lists = [[] for __ in xrange(4)]

Note: Use range() instead of xrange() in Python 3

#### Create a string from a list

A common idiom for creating strings is to use `str.join` on an empty
string.

    letters = ['s', 'p', 'a', 'm']
    word = ''.join(letters)

This will set the value of the variable *word* to 'spam'. This idiom can be
applied to lists and tuples.

#### Searching for an item in a collection

Sometimes we need to search through a collection of things. Let's look at two
options: lists and sets.

Take the following code for example:

    s = set(['s', 'p', 'a', 'm'])
    l = ['s', 'p', 'a', 'm']

    def lookup_set(s):
        return 's' in s

    def lookup_list(l):
        return 's' in l

Even though both functions look identical, because *lookup_set* is utilizing
the fact that sets in Python are hashtables, the lookup performance
between the two is very different. To determine whether an item is in a list,
Python will have to go through each item until it finds a matching item.
This is time consuming, especially for long lists. In a set, on the other
hand, the hash of the item will tell Python where in the set to look for
a matching item. As a result, the search can be done quickly, even if the
set is large. Searching in dictionaries works the same way. For
more information see this
[StackOverflow page](http://stackoverflow.com/questions/513882/python-list-vs-dict-for-look-up-table).
For detailed information on the amount of time various common operations
take on each of these data structures, see
[this page](https://wiki.python.org/moin/TimeComplexity?).

Because of these differences in performance, it is often a good idea to use
sets or dictionaries instead of lists in cases where:

* The collection will contain a large number of items

* You will be repeatedly searching for items in the collection

* You do not have duplicate items.

For small collections, or collections which you will not frequently be
searching through, the additional time and memory required to set up the
hashtable will often be greater than the time saved by the improved search
speed.

### Zen of Python

Also known as :pep:`20`, the guiding principles for Python's design.

    >>> import this
    The Zen of Python, by Tim Peters

    Beautiful is better than ugly.
    Explicit is better than implicit.
    Simple is better than complex.
    Complex is better than complicated.
    Flat is better than nested.
    Sparse is better than dense.
    Readability counts.
    Special cases aren't special enough to break the rules.
    Although practicality beats purity.
    Errors should never pass silently.
    Unless explicitly silenced.
    In the face of ambiguity, refuse the temptation to guess.
    There should be one-- and preferably only one --obvious way to do it.
    Although that way may not be obvious at first unless you're Dutch.
    Now is better than never.
    Although never is often better than *right* now.
    If the implementation is hard to explain, it's a bad idea.
    If the implementation is easy to explain, it may be a good idea.
    Namespaces are one honking great idea -- let's do more of those!

For some examples of good Python style, see [these slides from a Python user
group](http://artifex.org/~hblanks/talks/2011/pep20_by_example.pdf).

### Conventions

Here are some conventions you should follow to make your code easier to read.

#### Check if variable equals a constant

You don't need to explicitly compare a value to True, or None, or 0 - you can
just add it to the if statement. See [Truth Value Testing]
(http://docs.python.org/library/stdtypes.html#truth-value-testing) for a
list of what is considered false.

**Bad**:

    if attr == True:
        print 'True!'

    if attr == None:
        print 'attr is None!'

**Good**:

    # Just check the value
    if attr:
        print 'attr is truthy!'

    # or check for the opposite
    if not attr:
        print 'attr is falsey!'

    # or, since None is considered false, explicitly check for it
    if attr is None:
        print 'attr is None!'

#### Access a Dictionary Element

Don't use the `dict.has_key` method. Instead, use `x in d` syntax,
or pass a default argument to `dict.get`.

**Bad**:

    d = {'hello': 'world'}
    if d.has_key('hello'):
        print d['hello']    # prints 'world'
    else:
        print 'default_value'

**Good**:

    d = {'hello': 'world'}

    print d.get('hello', 'default_value') # prints 'world'
    print d.get('thingy', 'default_value') # prints 'default_value'

    # Or:
    if 'hello' in d:
        print d['hello']

#### Short Ways to Manipulate Lists

[List comprehensions]
(http://docs.python.org/tutorial/datastructures.html#list-comprehensions)
provide a powerful, concise way to work with lists. Also, the `map` and `filter` functions can perform operations on lists using a different, more concise syntax.

**Bad**:

    # Filter elements greater than 4
    a = [3, 4, 5]
    b = []
    for i in a:
        if i > 4:
            b.append(i)

**Good**:

    a = [3, 4, 5]
    b = [i for i in a if i > 4]
    # Or:
    b = filter(lambda x: x > 4, a)

**Bad**:

    # Add three to all list members.
    a = [3, 4, 5]
    for i in range(len(a)):
        a[i] += 3

**Good**:

    a = [3, 4, 5]
    a = [i + 3 for i in a]
    # Or:
    a = map(lambda i: i + 3, a)

Use `enumerate` keep a count of your place in the list.

    a = [3, 4, 5]
    for i, item in enumerate(a):
        print i, item
    # prints
    # 0 3
    # 1 4
    # 2 5

The `enumerate` function has better readability than handling a
counter manually. Moreover, it is better optimized for iterators.

#### Read From a File

Use the `with open` syntax to read from files. This will automatically close
files for you.

**Bad**:

    f = open('file.txt')
    a = f.read()
    print a
    f.close()

**Good**:

    with open('file.txt') as f:
        for line in f:
            print line

The `with` statement is better because it will ensure you always close the
file, even if an exception is raised inside the `with` block.

#### Line Continuations

When a logical line of code is longer than the accepted limit, you need to
split it over multiple physical lines. The Python interpreter will join
consecutive lines if the last character of the line is a backslash. This is
helpful in some cases, but should usually be avoided because of its fragility:
a white space added to the end of the line, after the backslash, will break the
code and may have unexpected results.

A better solution is to use parentheses around your elements. Left with an
unclosed parenthesis on an end-of-line the Python interpreter will join the
next line until the parentheses are closed. The same behavior holds for curly
and square braces.

**Bad**:

    my_very_big_string = """For a long time I used to go to bed early. Sometimes, \
        when I had put out my candle, my eyes would close so quickly that I had not even \
        time to say “I’m going to sleep.”"""

    from some.deep.module.inside.a.module import a_nice_function, another_nice_function, \
        yet_another_nice_function

**Good**:

    my_very_big_string = (
        "For a long time I used to go to bed early. Sometimes, "
        "when I had put out my candle, my eyes would close so quickly "
        "that I had not even time to say “I’m going to sleep.”"
    )

    from some.deep.module.inside.a.module import (
        a_nice_function, another_nice_function, yet_another_nice_function)

However, more often than not, having to split a long logical line is a sign that
you are trying to do too many things at the same time, which may hinder
readability.

## Testing

Testing is incredibly important in the STMS environment. Not only are our systems incredibly complex, but introducing a bug into the production storage environment can be extremely detrimental.

While you are more than welcome to write your tests using the `unittest` module in the standard lib, we recommend using `nose2`, which offers some niceties and abstractions above `unittest`. STMS projects feature complex data/api interactions and testing can prove difficult when relying on external services. To make this easier and faster, use the `mock` library to mock data returns from external dependencies. Read about how to use `nose2` [here](http://nose2.readthedocs.org/en/latest/getting_started.html) and the docs for `mock` [here](https://docs.python.org/dev/library/unittest.mock.html). Note that `mock` is included in the python 3.x standard library, but its api docs will be relevant for the python 2.x pkg.

As stated above, ensure that a `.coveragerc` is contained in the root of your project repo. We will use this file for enabling a code coverage report when running tests. Ensure that both `nose2` and `cov-core` are installed and listed in your `developer-requirements.txt`. When writing tests and submitting a pull request for a new feature or bug fix, ensure that your test coverage is at least 90% (but strive for 100%). You can check your test coverage by running: `nose2 --config .coveragerc`.

For an example in STMS of how to write some tests (that include mocks), look at the `test/` directory in [STMS-Eos](https://github.corp.ebay.com/STMS/Eos/tree/master/test).

Here are some basic rules for writing tests:

* A testing unit should focus on one tiny bit of functionality and prove it correct.
* Each test unit must be fully independent. Each of them must be able to run alone, and also within the test suite, regardless of the order they are called. The implication of this rule is that each test must be loaded with a fresh dataset and may have to do some cleanup afterwards. This is usually handled by `setUp()` and `tearDown()` methods.
* Try hard to make tests that run fast. If one single test needs more than a few milliseconds to run, development will be slowed down or the tests will not be run as often as is desirable. In some cases, tests can’t be fast because they need a complex data structure to work on, and this data structure must be loaded every time the test runs. Keep these heavier tests in a separate test suite that is run by some scheduled task, and run all other tests as often as needed.
* Learn your tools and learn how to run a single test or a test case. Then, when developing a function inside a module, run this function’s tests very often, ideally automatically when you save the code.
* Always run the full test suite before a coding session, and run it again after. This will give you more confidence that you did not break anything in the rest of the code.
* If you are in the middle of a development session and have to interrupt your work, it is a good idea to write a broken unit test about what you want to develop next. When coming back to work, you will have a pointer to where you were and get back on track faster.
* The first step when you are debugging your code is to write a new test pinpointing the bug. While it is not always possible to do, those bug catching tests are among the most valuable pieces of code in your project.
* Use long and descriptive names for testing functions. The style guide here is slightly different than that of running code, where short names are often preferred. The reason is testing functions are never called explicitly. `square()` or even `sqr()` is ok in running code, but in testing code you would have names such as `test_square_of_number_2()` or `test_square_negative_number()`. These function names are displayed when a test fails, and should be as descriptive as possible.
* When something goes wrong or has to be changed, and if your code has a good set of tests, you or other maintainers will rely largely on the testing suite to fix the problem or modify a given behavior. Therefore the testing code will be read as much as or even more than the running code. A unit test whose purpose is unclear is not very helpful in this case.

## Git & Github

Hopefully, you already know how to use git! If not, go ahead and read up on its usage [here](https://www.atlassian.com/git/tutorials/) and run through this [15 minute interactive tutorial](https://try.github.io/levels/1/challenges/1).

Okay, you're back and know your stuff? Let's review how we use Github for our revision management in STMS. For every STMS project, we maintain two branches: _master_ and _develop_. The _master_ branch should always be fully production-ready. This means all its tests are passing and it holds the latest stable release of the project that is actively deployed in production. No developer's code should be directly pushed or pulled into _master_. The _develop_ branch is where, you guessed it, active development happens. However, this does not mean that _develop_ should contain any broken code. All tests in the _develop_ branch should also be passing!

Create a new development branch off of `origin develop` and name it after yourself. For example, my development branch would be named `mmedal-develop`. Do all of your work on this branch and when your feature/fix is complete, your tests are passing, and you are ready to merge your code into the main branch, create a pull request on github between _develop_ and _yourname-develop_. Another member of the STMS development team will review your changes and merge it in. When we are ready to release a new version into production, we will merge _develop_ into _master_ and deploy.

It's very important to keep your commits organized by feature or fix and their corresponding messages relevant. Each commit should encapsulate **one** change that may be entirely reverted by simply moving back that commit. The commit message should be informative and accurately describe the main changes of that commit. This means avoiding commit messages like: `"whoops"` or `"fixing stuff"` and writing commit messages more like: `"fixed bug in /arrays api that caused count to return the wrong value"` or `"added ui button that allows users to refresh user credentials on demand"`. Sometimes, despite our best efforts, we end up commiting code many more times than we need to, with entirely unhelpful messages. A useful technique to cleaning up our git histories before pushing code is "squashing commits". Read [this tutorial](https://ariejan.net/2011/07/05/git-squash-your-latests-commits-into-one/) for an example of commit squashing.

## Wrap Up

Hopefully, this guide will serve as a helpful introduction to development in STMS. If you have any questions or need help with anything, drop a line on the `#stms-dev` channel in _ebay-ssi.slack_ and @mmedal or @esalisbury will be more than happy to help.

**Happy Hacking!**
