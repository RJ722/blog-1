---
layout: post
title: "Why use coverage to find which parts of a python code were executed?"
subtitle: "Case Study - How we decided that coverage was the best option for detecting false positives in vulture."
date: 2018-05-13 04:09:40
author: "Rahul Jha"
header-img: "img/post-bg-unix-linux.jpg"
tags:
- vulture
- GSoC
---

In this post, I'll walk you through the decision making process the team behind Vulture underwent to come up with a way to deal with false positives in it's results. Let's first start with a brief introduction of [Vulture](https://github.com/jendrikseipp/vulture):

### Vulture

As the name suggests, vulture helps find [dead code](https://en.wikipedia.org/wiki/Dead_code) for Python programs. There are many reasons for dead code ending up in a project. The most common is refactoring, but another is misspellings, which are only detected at runtime for dynamic languages. Finding and removing dead code allows to keep the code base clean and reduces bugs. 

Vulture can detect unused imports, variables, attributes, functions, methods, properties and classes. Other than these, code after return statements and checking for a Boolean False (eg. `if False:` or `while 0:`) can also be detected.

#### Using vulture

Vulture is a standard Python package, that is installed with `pip`:

```
(venv)$ pip install vulture
```

Let us say that you have the following program (say `program.py`) on which you want to perform analysis:

```py
import os


def hello(msg):
	print("Hello", msg)

def hello_world():
	message = "Hello World"
	print("Hello World")


def main():
	hello_world()


if __name__ == '__main__':
	main()
```

Analysing the program with vulture is as simple as running the following command:

```
$ vulture program.py
```

which would produce the following output (on `vulture 0.26`):

```
program.py:1: unused import 'os' (90% confidence)
program.py:4: unused function 'hello' (60% confidence)
program.py:8: unused variable 'message' (60% confidence)
```

As you can see, vulture also reports a confidence value which is a measure of how sure vulture is about that part of code being unused. This output can be made even more meaningful with the help of flags like `--min-confidence` and `--sort-by-size` which are self explanatory.

Owing to Python's dynamic nature, Vulture is likely to miss some dead code. Also, code which is only implicitly used is reported unused, such as overloading a parent class method, overriding methods of C/C++ extensions, etc.

Some other examples where vulture may report "useful" code as unused:
- **API endpoints** - They are meant for users and are not employed to any use directly in the source code, therefore confusing vulture.
- **ORM Schema** - Again, they aren't used by program's source code directly.

#### Handling false positives

One of the ways is to use something that this is not the one I am talking about I can type

Some other ways are to:
- Mark unused variables by starting them with an "`_`".  (e.g., `_x, y = get_pos()`)
- Use different files for API endpoints, ORM, etc. and exclude them with the help of `--exclude` flag.

You can find more information about [vulture in it's documentation](https://github.com/jendrikseipp/vulture/tree/master/README.rst).


## What more does vulture want?

Currently, vulture gives results with dead code and sometimes false positive. We want to be able to develop such a system which should be able to tell that the result given by vulture is a false positive.
