---
layout: post
title: "Using Fixtures in Pytest to Handle Test Dependencies"
date: 2020-11-29 17:50:00 +0000
tags: [tests,python]
---
[Fixtures](https://docs.pytest.org/en/stable/fixture.html) in pytest offer a great way to establish 'things' required to perform a test.
These 'things' can include class instances, closures, values, or setup and teardown instructions.
When utilised, fixtures can clear code from tests, leaving only the essential elements.
This post discusses some of the key parts of fixtures, showing how they can be used to greatly improve testing.

## Dependency injection

[Dependency injection](https://docs.pytest.org/en/stable/fixture.html#fixtures-a-prime-example-of-dependency-injection) is the idea of passing objects required by another object so that they are immediately available for use.
This can be achieved using fixtures, as follows:

```python
from pytest import fixture

class App:
    def __init__(self, identifier=1):
        self.identifier = identifier

@fixture
def app():
    return App()

def test_create_app(app):
    assert app.identifier == 1
```

In this example, the `test_create_app` test function requires an instance of `App`, i.e. is dependent on `App`, to check that it can be created successfully.
This dependency can be passed to, i.e. injected into, `test_create_app` by stating the fixture `app` as an argument.
The `app` fixture will create an instance of the class `App` and passed it to the test function.
Once it has been injected, the instance can be used as if it was created within `test_create_app`.

## Passing values to fixtures

In some situations a fixture may require values to be able to create and inject a dependency.
[Introspection](https://docs.pytest.org/en/stable/fixture.html#fixtures-a-prime-example-of-dependency-injection) enables a fixture to inspect the test that is currently running and allows values to be passed to a fixture through examining either a test's parameters or marked values.
A test's parameters and marked values are exposed by the [`request`](https://docs.pytest.org/en/stable/reference.html#std-fixture-request) object, which can be injected into a test function as discussed above.

### Examining parameters

The example below shows how a test's parameters can be examined by injecting the `request` object into the `app` fixture and accessing its `param` attribute:

```python
from pytest import fixture, mark

class App:
    def __init__(self, identifier):
        self.identifier = identifier

@fixture
def app(request):
    return App(request.param)

@mark.parametrize("identifier", [1, 2])
def test_create_app(identifier, app):
    assert app.identifier == identifier
```

This example can be extended by matching an argument name of the `app` fixture with an argument name of the `test_create_app` test function.
In doing so, pytest assumes that the value of the test function argument should be passed to the fixture, as shown below:

```python
from pytest import mark

class App:
    def __init__(self, identifier):
        self.identifier = identifier

@fixture
def app(identifier):
    return App(identifier)

@mark.parametrize("identifier", [1, 2])
def test_create_app(identifier, app):
    assert app.identifier == identifier
```

### Marked values

Values can also be passed to a fixture by adding a [marker](https://docs.pytest.org/en/stable/fixture.html#using-markers-to-pass-data-to-fixtures) to a test function and accessing its value using the `request` object.

```python
from pytest import fixture, mark

class App:
    def __init__(self, identifier):
        self.identifier = identifier

@fixture
def app(request):
    return App(request.node.get_closest_marker("identifier").args[0])

@mark.identifier(1)
def test_create_app(app):
    assert app.identifier == 1
```

## Creating multiple instances

Occasionally a test may require multiple instances of a dependency, however, it is not possible to inject a fixture twice into a test function due to pytest's fixture caching mechanism.
To solve this, a fixture can inject a closure into the test function, which is capable of producing multiple instances of a dependency, as known as a [factory function](https://docs.pytest.org/en/stable/fixture.html#factories-as-fixtures). An example of this is shown below:

```python
from pytest import fixture

class App:
    def __init__(self, identifier):
        self.identifier = identifier

@fixture
def app():
    def closure():
        return App()

    return closure

def test_create_app(app):
    first_app = app()
    second_app = app()

    assert first_app != second_app
```

## Teardowns in fixtures

Fixtures can also be used to perform [teardowns](https://docs.pytest.org/en/stable/fixture.html#fixture-finalization-executing-teardown-code) once a test has finished by yielding a dependency, as shown below:

```python
class App:
    def __init__(self, identifier=1):
        self.identifier = identifier

    def close(self):
        print('Closing app...')

@fixture
def app(request):
    app = App()

    yield app

    app.close()

def test_create_app(app):
    assert app.identifier == 1
```

In this example, the `app` fixture first creates a new instance of the `App` class and passes this instance to the `test_create_app` test function.
Once the test has assert that the identifier equals 1, it returns to the `app` fixture and calls the `close()` method on the `App` instance.

## Summary

This post has discussed how pytest fixtures can be used to inject dependencies into a test function, removing setup and teardown code from inside test functions.
In some scenarios, to inject a dependency some values are required by the fixture, and introspection of the test function can be used to access marked values or parameters.
Also discussed was how fixtures can provide a closure for situations where a test requires multiple instances of a dependency.
Finally, an example was shown of how fixtures can yield a dependency to a test function so that teardown code can be executed after the test has finished.
