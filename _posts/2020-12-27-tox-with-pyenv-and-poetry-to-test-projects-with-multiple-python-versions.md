---
layout: post
title: "Tox with Pyenv & Poetry to Test Projects with Multiple Python Versions"
date: 2020-12-27 19:10:00 +0000
tags: [tests,python]
---
Recently, I was working on [Limber Framework](https://github.com/limber-project/limberframework) and I wanted to increase the availability of the framework by supporting multiple versions of Python.
However, in order to say that the framework can work with a version of Python, I need to check that there's no issues with that particular version, i.e. run the automated tests.
Manually running the tests for each supported version of Python will require a lot of work, which will only grow as the project develops.
Therefore, an automated approach is needed and after doing some research I came across [tox](https://github.com/tox-dev/tox).

tox describes itself as a "command line driven CI frontend and development task automation tool", and one of the automated tasks it can do is run tests with different Python versions.
However, I found that getting tox to work was not so straight forward and required some research.
This post will cover what this research found and how to get tox to use [pyenv](https://github.com/pyenv/pyenv) and [poetry](https://github.com/python-poetry/poetry) to test a project against multiple versions of Python.

## Installing tox
While tox will create a new environment for each version of Python that it tests, tox itself can be installed into the project's virtual environment, as follows:

```console
$ poetry shell
$ poetry add tox
```

To tell tox which versions of Python to test a configuration file is required at the root of the project called **tox.ini**:

```ini
[tox]
envlist = py38,py39
```

Inside **tox.ini** is a section called **tox** with a property called **envlist** which lists the versions of Python to test, in this case 3.8 and 3.9.

## Using poetry for the virtual environment
To get tox to use poetry to build the virtual environment the **tox.ini** file needs updating to the following:

```ini
[tox]
isolated_build = True
envlist = py38,py39

[testenv]
whitelist_externals = poetry
commands =
  poetry install -v
  poetry run pytest tests/
```

First, [isolated_build](https://tox.readthedocs.io/en/latest/config.html?highlight=isolated#conf-isolated_build) is set to **True** which tells tox to use a virtual environment when building the source distribution.

Second, a new section called **testenv** has been added with the properties **whitelist_externals** and **commands**.
[whitelist_externals](https://tox.readthedocs.io/en/latest/config.html?highlight=isolated#conf-allowlist_externals) prevents tox from raising a warning and, in this case, allows the use of poetry.
[commands](https://tox.readthedocs.io/en/latest/config.html?highlight=isolated#conf-commands) lists the steps needed to run the tests.
Here poetry installs the dependencies into the virtual environment and then runs the tests using [pytest](https://docs.pytest.org/en/latest/).

## Configuring pyenv with multiple versions of Python
The final piece of the puzzle involves configuring the project with multiple versions of Python using pyenv so that tox can find the versions that it's testing.
In this case, Python 3.8 and 3.9 is being tested, therefore, they must be installed:

```console
$ pyenv install 3.8.6
$ pyenv install 3.9.0
```

Next, the project should be set with these versions:

```console
$ pyenv local 3.9.0 3.8.6
```

A new file called **.python-version** is created with the following content:

```
3.9.0
3.8.6
```

## Running tox
With the instructions to run the tests and the different versions of Python set, tox can now be run:

```console
$ tox
```

tox will then execute the tests for Python 3.8 and 3.9:

```console
___________________________________ summary ____________________________________
  py38: commands succeeded
  py39: commands succeeded
  congratulations :)
```
