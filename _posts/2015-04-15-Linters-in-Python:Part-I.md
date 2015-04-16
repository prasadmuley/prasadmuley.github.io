---
layout: post
title: Linters in Python - Part I
excerpt: "Static code analyzers and PEP8 for python programming"
tags: [python, linters, pep8, programming, beginner, static, code, analyzer]
comments: true
language: python
image:
    feature: #asdf
    credit: #asdf
    credtlink: #asdf
---

Most of python programmers are familiar with [**pep8**](https://www.python.org/dev/peps/pep-0008/).
The **Pep8** isn't a linter. I'll explain **what is the difference between pep8 and linters**.
Let's see about **what is linter** first and pep8 later.

### Concept of Linter:

*  [**Lint**](http://en.wikipedia.org/wiki/Lint_%28software%29) refers to a general class of programming tools which
analyze the source code and raise warning and errors about it.
*  It simply means *linter is a [**Static**](http://en.wikipedia.org/wiki/Static_program_analysis) code analyzer.*
*  **Examples:**
   *  [**Syntax errors**](https://docs.python.org/2/tutorial/errors.html)
   *  It also points out **unused/undeclared variables**.
   *  Reports calls to [**deprecated**](http://en.wikipedia.org/wiki/Deprecation) functions.

*  **Linters for Python Programming:**
   *  [PyChecker](http://pychecker.sourceforge.net/)
   *  [PyFlake](https://pypi.python.org/pypi/pyflakes)
   *  [Flake8](https://pypi.python.org/pypi/flake8)
   *  [PyLama](https://github.com/klen/pylama)
   *  [PyLint](http://www.pylint.org/)

*  **Advantages:**
   *  Detect and Prevent a large class of **programming errors.**
   *  Code review is done easily by addressing many mechanical and formatting problems automatically.

### [**PEP8:**](https://www.python.org/dev/peps/pep-0008/)

*  It is style guidelines for python. We can call it as **simple python style checker**.
*  It checks your **python** code against style guidelines which are defined in pep8.
*  **Examples:**
   *  **4 space** indents.
   *  **line too long** i.e greater than *79* characters.
   *  **expected 1 blank line, found 0**

*  **Advantages:**
   * Code style consistency.
   * Improve code readability.

### Difference between Linters and PEP8:

Let's see sample program.

{% highlight python %}

import string
import os



foo_bar = 'foo'

def foo():
    return "foo"
  
def bar():
    asdf = 'xyz'
    return foo() + "bar"
     

if __name__ == '__main__':
    print bar()
{% endhighlight %}

> **PEP8's** output:

{% highlight bash %}
$ pep8 sample_prg.py                                   
sample_prg.py:6:1: E302 expected 2 blank lines, found 1
sample_prg.py:9:1: W293 blank line contains whitespace
sample_prg.py:10:1: E302 expected 2 blank lines, found 1
sample_prg.py:14:1: W293 blank line contains whitespace
sample_prg.py:15:1: W293 blank line contains whitespace
sample_prg.py:16:1: W293 blank line contains whitespace
sample_prg.py:17:1: E303 too many blank lines (3)
FAIL: 1
{% endhighlight %}

**PEP8 reports only style guide related errors.
It doesn't report errors like unused variable, unused import module etc.**

> **Linter's** output:

{% highlight bash %}
$ pylint sample_prg.py                            
************* Module sample_prg
C:  9, 0: Trailing whitespace (trailing-whitespace)
C: 14, 0: Trailing whitespace (trailing-whitespace)
C: 15, 0: Trailing whitespace (trailing-whitespace)
C: 16, 0: Trailing whitespace (trailing-whitespace)
C:  1, 0: Missing module docstring (missing-docstring)
C:  4, 0: Invalid constant name "foo_bar" (invalid-name)
C:  6, 0: Black listed name "foo" (blacklisted-name)
C: 10, 0: Black listed name "bar" (blacklisted-name)
W: 12, 4: Unused variable 'asdf' (unused-variable)
W:  1, 0: Unused import string (unused-import)
W:  2, 0: Unused import os (unused-import)
FAIL: 32
{% endhighlight %}

Here,


   **Pylint reports various errors compared to pep8.
This is the main difference between linter and pep8**.


Pylint also checks code against **pep8** style guide and reports **pep8** errors.
I'll explain available linters for Python in next part.

