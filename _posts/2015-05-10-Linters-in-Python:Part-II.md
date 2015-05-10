---
layout: post
title: Linters in Python - Part II
excerpt: "Static code analyzers for python programming"
tags: [python, linters, pep8, programming, beginner, static, code, analyzer]
comments: true
language: python
image:
    feature: #asdf
    credit: #asdf
    credtlink: #asdf
---

We understood the difference between [pep8](https://www.python.org/dev/peps/pep-0008/) and [linters](http://en.wikipedia.org/wiki/Lint_%28software%29) in [previous](http://rootpy.com/Linters-in-Python:Part-I/) post.
We'll see various linters available in python with sample example.

* Available linters for Python Programming:
   *  [PyChecker](http://pychecker.sourceforge.net/)
   *  [PyFlakes](https://pypi.python.org/pypi/pyflakes)
   *  [Flake8](https://pypi.python.org/pypi/flake8)
   *  [PyLint](http://www.pylint.org/)
   *  [PyLama](https://github.com/klen/pylama)

### Sample program:
  I've written a error-prone program. It contains some unused imported modules, variable etc and pep8 errors.
  It also overriddes one function. Let's see output of each linter.

{% highlight python %}
#!/usr/bin/env python
# encoding: utf-8

#unused modules
import os
import sys
import argparse


#unused global constant variable
user_name = 'rootpy'



def setUserName(self, name):
    fname = 'Prasad'
    u_name = name
    return u_name



class User(object):

    def __init__(self, str, int):
        self.name = None
        self.password = None
        return

    def getUserPassword(self, password):
        self.password = password
        return

    def changeUserName(self, email):
        return
        print "How can I reach here?"

    def getUserPassword(self, password):
        return self.password == password

{% endhighlight %}


###[PyChecker](http://pychecker.sourceforge.net/)
* It imports each module and compiles the each function, class and method for possible errors.
* It determines the imported modules, classes and functions. It creates a tree based on it.
* It can't process the module if there is an import error.
* Let's pass the sample_program.py and analyze the output.
{% highlight bash %}
prasad@Rootpy: master ⚡
$ pychecker sample_program.py
Processing module test_program_rootpy (test_program_rootpy.py)...

Warnings...

sample_program.py:5: Imported module (os) not used
sample_program.py:6: Imported module (sys) not used
sample_program.py:7: Imported module (argparse) not used
sample_program.py:15: self is argument in function
sample_program.py:16: Local variable (fname) not used
sample_program.py:23: Parameter (int) not used
sample_program.py:23: Parameter (str) not used
sample_program.py:32: Redefining attribute (getUserPassword) original line (28)
FAIL: 1


prasad@Rootpy: master ⚡
$
{% endhighlight %}

Here, 

  * It detects following errors:
    * unused imported module, local variable, arguments error.
    * self is argument in function 
  * But it doesn't  detect following errors: 
    * unused global variable user_name.
    * pep8 related errors (some linters detect it).
  * It compiles and executes the imported module so your code get executed.
  * For example: If you've SQL queries in python program then it executes every time which is bad.
  * It becomes slower than other linters because it imports all the modules and execute it one by one.
  * You can create configure it by creating configuration file (.pycheckerr)
  * Check [here](http://legacy.python.org/workshops/2002-02/papers/02/index.htm) for more information

###[PyFlake](https://pypi.python.org/pypi/pyflakes)
* It is a static code analyzer as It identifies errors without executing python code.
* It is faster than [PyChecker](http://pychecker.sourceforge.net/) because It doesn't import any module and execute it.
* Let's see output for sample_program.py
{% highlight bash %}
prasad@Rootpy: master ⚡
$ pyflakes sample_program.py
sample_program.py:5: 'os' imported but unused
sample_program.py:6: 'sys' imported but unused
sample_program.py:7: 'argparse' imported but unused
sample_program.py:16: local variable 'fname' is assigned to but never used
sample_program.py:32: redefinition of unused 'getUserPassword' from line 28

FAIL: 1

prasad@Rootpy: master ⚡
$
{% endhighlight %}


Here, 

  * It detects unused imported module, local variable, arguments errors.
  * It doesn't detect following errors:
    * Unused global variable user_name.
    * pep8 style guide related error.
    * Self is used as argument in function
    * Redefining parameters like int, str which are built-in data type in python
  * It examines the syntax tree of each file individually So it is faster.
  * Check on [github](https://github.com/pyflakes/pyflakes/) or [docs](https://divmod.readthedocs.org/en/latest/products/pyflakes.html) for more information.

###[Flake8](https://pypi.python.org/pypi/flake8)
  * It is a static code analyzer. Actually it is combination of [pep8](https://www.python.org/dev/peps/pep-0008/) and [PyFlake](https://pypi.python.org/pypi/pyflakes).
  * It identifies programming errors as well as pep8 style-guide errors.
  * Let's check what it gives on sample_program.py

{% highlight bash %}
prasad@Rootpy: master ⚡
$ flake8 sample_program.py

sample_program.py:5:1: F401 'os' imported but unused
sample_program.py:6:1: F401 'sys' imported but unused
sample_program.py:7:1: F401 'argparse' imported but unused
sample_program.py:15:1: E303 too many blank lines (3)
sample_program.py:16:5: F841 local variable 'fname' is assigned to but never used
sample_program.py:32:5: F811 redefinition of unused 'getUserPassword' from line 28
sample_program.py:39:1: W391 blank line at end of file
FAIL: 1

prasad@Rootpy: master ⚡
$
{% endhighlight %}

Here,

  * It detects following errors:
    * unused imported module, local variable, arguments error.
    * pep8 related errors like blank line, too many blank line etc.
  * It doesn't detect following errors:
    * unused global variable user_name.
	* Self is used as argument in function
    * Parameters like int, str which are built-in data type in python
 * Check flake8's [docs](https://flake8.readthedocs.org/en/2.3.0/) for more information.

###[PyLint](http://www.pylint.org/)
  * It is a static code analyzer i.e It checks errors in python code.
  * It looks for bad codes and tries to enforce a coding standard.
  * It displays statistics about the number of warnings and errors found in different files.
  * It also gives overall mark, based on numbers of warnings and errors.
  * Refactoring:
     * It also detects duplicates codes.
  * Fully Customizable:
     * You can create your own .pylintrc and modify which errors are important for you.
  * UML Diagram: 
     * It creates UML diagrams for python code using [Pyreverse](https://pypi.python.org/pypi/pyreverse/0.5.1)
  *  Editor and IDE integration:
     * Editor Integration are available for various editors like [emacs](http://www.emacswiki.org/emacs/PythonProgrammingInEmacs#toc8), [vim](http://www.vim.org/scripts/script.php?script_id=891) and [eclipse](http://pydev.org/)
     * IDE integration are also available for various IDEs like [spyder](https://pythonhosted.org/spyder/pylint.html), [editra](https://code.google.com/p/editra-plugins/wiki/PyAnalysis) and [TextMate](http://blogs.ethz.ch/halfdome_by_night/2008/04/26/pylint-command-in-textmate/)
     * Check [list of supported editors and IDEs](http://docs.pylint.org/ide-integration)
  * error messages type:
     * [**R**]efactor for a “good practice” metric violation
     * [**C**]onvention for coding standard violation
	 * [**W**]arning for stylistic problems, or minor programming issues
	 * [**E**]rror for important programming issues (i.e. most probably bug)
	 * [**F**]atal for errors which prevented further processing.
   * It gives very descriptive output. The output is divided into two categories message and reports.
     * message contains warnings, errors, coding standard convention, Refactoring and Fatal errors.
     * Reports contains statistics information for by type, raw metrics, duplication, message by category etc.
   * Let's run pylint on sample_program.py and check the output.

{% highlight bash%}

prasad@Rootpy: master ⚡
$ pylint sample_program.py                                          

************* Module sample_program
C:  1, 0: Missing module docstring (missing-docstring)
C: 11, 0: Invalid constant name "user_name" (invalid-name)
C: 15, 0: Invalid function name "setUserName" (invalid-name)
C: 15, 0: Missing function docstring (missing-docstring)
W: 15,16: Unused argument 'self' (unused-argument)
W: 16, 4: Unused variable 'fname' (unused-variable)
C: 21, 0: Missing class docstring (missing-docstring)
W: 23,28: Redefining built-in 'int' (redefined-builtin)
W: 23,23: Redefining built-in 'str' (redefined-builtin)
W: 23,28: Unused argument 'int' (unused-argument)
W: 23,23: Unused argument 'str' (unused-argument)
C: 28, 4: Invalid method name "getUserPassword" (invalid-name)
C: 28, 4: Missing method docstring (missing-docstring)
E: 32, 4: method already defined line 28 (function-redefined)
C: 32, 4: Invalid method name "getUserPassword" (invalid-name)
C: 32, 4: Missing method docstring (missing-docstring)
C: 35, 4: Invalid method name "changeUserName" (invalid-name)
C: 35, 4: Missing method docstring (missing-docstring)
W: 37, 8: Unreachable code (unreachable)
W: 35,29: Unused argument 'email' (unused-argument)
R: 35, 4: Method could be a function (no-self-use)
W:  5, 0: Unused import os (unused-import)
W:  6, 0: Unused import sys (unused-import)
W:  7, 0: Unused import argparse (unused-import)


Report
======
22 statements analysed.

Statistics by type
------------------

+---------+-------+-----------+-----------+------------+---------+
|type     |number |old number |difference |%documented |%badname |
+=========+=======+===========+===========+============+=========+
|module   |1      |NC         |NC         |0.00        |0.00     |
+---------+-------+-----------+-----------+------------+---------+
|class    |1      |NC         |NC         |0.00        |0.00     |
+---------+-------+-----------+-----------+------------+---------+
|method   |4      |NC         |NC         |25.00       |75.00    |
+---------+-------+-----------+-----------+------------+---------+
|function |1      |NC         |NC         |0.00        |100.00   |
+---------+-------+-----------+-----------+------------+---------+



Raw metrics
-----------

+----------+-------+------+---------+-----------+
|type      |number |%     |previous |difference |
+==========+=======+======+=========+===========+
|code      |19     |55.88 |NC       |NC         |
+----------+-------+------+---------+-----------+
|docstring |1      |2.94  |NC       |NC         |
+----------+-------+------+---------+-----------+
|comment   |4      |11.76 |NC       |NC         |
+----------+-------+------+---------+-----------+
|empty     |10     |29.41 |NC       |NC         |
+----------+-------+------+---------+-----------+



Duplication
-----------

+-------------------------+------+---------+-----------+
|                         |now   |previous |difference |
+=========================+======+=========+===========+
|nb duplicated lines      |0     |NC       |NC         |
+-------------------------+------+---------+-----------+
|percent duplicated lines |0.000 |NC       |NC         |
+-------------------------+------+---------+-----------+



Messages by category
--------------------

+-----------+-------+---------+-----------+
|type       |number |previous |difference |
+===========+=======+=========+===========+
|convention |11     |NC       |NC         |
+-----------+-------+---------+-----------+
|refactor   |1      |NC       |NC         |
+-----------+-------+---------+-----------+
|warning    |11     |NC       |NC         |
+-----------+-------+---------+-----------+
|error      |1      |NC       |NC         |
+-----------+-------+---------+-----------+



Messages
--------

+-------------------+------------+
|message id         |occurrences |
+===================+============+
|missing-docstring  |6           |
+-------------------+------------+
|invalid-name       |5           |
+-------------------+------------+
|unused-argument    |4           |
+-------------------+------------+
|unused-import      |3           |
+-------------------+------------+
|redefined-builtin  |2           |
+-------------------+------------+
|unused-variable    |1           |
+-------------------+------------+
|unreachable        |1           |
+-------------------+------------+
|no-self-use        |1           |
+-------------------+------------+
|function-redefined |1           |
+-------------------+------------+



Global evaluation
-----------------
Your code has been rated at -2.73/10

FAIL: 30

prasad@Rootpy: master ⚡
$  

{% endhighlight %}

Here,

   * It detects highest errors compared to other linters.
     Following messages are reported:
     * unused variables, imported modules, arguments etc.
     * Invalid variable name, method name etc.
   * Above errors mostly detected by all linters But it detects some special messages which other linters don't detect:
     * Unreachable code
     * Redefining built in int and str
     * unused global variable user_name which other linters are failed to detect.
     * missing doc strings.
   * This makes a difference between pylint and other linters.
   * Report contains various statistics information which is useful for programmer.
   * It's awesome code rating. It has given -2.7 rating to sample program. (OMG)
   * Pylint has great [documentation](http://docs.pylint.org/). Every thing is well explained.
   * You can configure pylint by creating configuration file i.e .pylintrc file.
      * For example: I've created a [pylint configuration file](https://github.com/prasadmuley/pylint_web2py) for [Web2Py](http://www.web2py.com/). 

###[PyLama](https://github.com/klen/pylama)
  * It is a static code analyzer which combines power of other linters.
  * It wrapes following tools:
    * [PEP8](https://github.com/jcrocholl/pep8)
    * [PEP257](https://github.com/GreenSteam/pep257)
    * [PyFlakes](https://pypi.python.org/pypi/pyflakes)
    * [Mccabe](http://nedbatchelder.com/blog/200803/python_code_complexity_microtool.html)
    * [PyLint](http://pylint.org/)
  * Let's check it's output
{% highlight bash%}
prasad@Rootpy: master ⚡
$ pylama sample_program.py
sample_program.py:4:1: E265 block comment should start with '# ' [pep8]
sample_program.py:5:1: W0611 'os' imported but unused [pyflakes]
sample_program.py:6:1: W0611 'sys' imported but unused [pyflakes]
sample_program.py:7:1: W0611 'argparse' imported but unused [pyflakes]
sample_program.py:10:1: E265 block comment should start with '# ' [pep8]
sample_program.py:15:1: E303 too many blank lines (3) [pep8]
sample_program.py:16:1: W0612 local variable 'fname' is assigned to but never used [pyflakes]
sample_program.py:32:1: W0404 redefinition of unused 'getUserPassword' from line 28 [pyflakes]
sample_program.py:39:1: W391 blank line at end of file [pep8]
FAIL: 1

prasad@Rootpy: master ⚡
{% endhighlight%}

Here,

  * It detects following errors:
     * unused variable, imported module and argument errors.
     * redefinition of function.
     * pep8 errors
  * It doesn't detect following errors:
     * unreachable code
     * missing doc string
     * Self is used as argument in function
     * redefining built-in datatype like int, str etc.
     * invalid function name like getUserPassword etc.

### Conclusion:
   * Most of these tools detect some of the errors in sample_program.py. We saw all tools output for sample_program.py
   * [PyLint](http://www.pylint.org/) gives us very descriptive output as well as detects various errors.
   * So we can deduce that Pylint's is a very powerful linters for python programming.