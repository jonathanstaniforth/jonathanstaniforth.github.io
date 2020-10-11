---
layout: post
title: "Mocking in Python"
date: 2020-06-10 22:00:00 +0800
tags: [python,testing]
---
When testing, the behaviour of production code should be checked in all possible states.
However, creating these states can be difficult and each state may require large scripts to establish the right environment.
Therefore, a method of imitating the behaviour of production code, called **mocking**, is used to alleviate this problem.

Mocking involves creating [**mock objects**](https://en.wikipedia.org/wiki/Mock_object) that replace real objects established by production code and simulate their behaviour.
Python has a library that contains several tools to create mock objects easily, however, there are a few concepts that are not so apparent and can cause problems if not understood.
This post reviews the main tools found in the library and goes into detail for the areas may present issues.

## The Mock Library
To perform mocking, python has a standard library called [**mock**](https://docs.python.org/3/library/unittest.mock.html) that contains several classes and decorators that can be used to mock different types of objects.
Using these tools, the interaction of production code with a mock object can be examined and, therefore, is useful in understanding its behaviour.
Additionally, mock objects can help to isolate production code, reducing their reliance on external factors, such as [communication with servers](https://realpython.com/python-mock-library/).

> For more information on reviewing the behaviour of production code, view the [Using Test-Driven Development to Design & Enforce Behaviour]({% post_url 2020-05-23-using-test-driven-development-to-design-and-enforce-behaviour %}) post.

### Mock class
The mock library contains the [**Mock class**](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock) which can be used to create mock objects.

```python
from unittest.mock import Mock

mock_object = Mock()
# <Mock id='...'>
```

To simulate the behaviour of any object, mock objects create all attributes and methods as they are accessed.
These attributes and methods accept any arguments and return new singleton instances of `Mock`, i.e. an attribute or method called multiple times will return the same `Mock` instance.

```python
# A new mock object
mock_object.get()
# <Mock name='mock.get()' id='1'>

# A new mock object that is different to above
mock_object.set()
# <Mock name='mock.set()' id='2'>

# The same mock object as the first above
mock_object.get()
# <Mock name='mock.get()' id='1'>
```

Additionally, it contains several methods that can be used to check how production code interacts with the mock object.

```python
mock_object.get.assert_called_once()

AssertionError: Expected 'get' to have been called once. Called 2 times.
Calls: [call(), call()].
```

> These assert methods will raise an `AssertionError` exception if the condition is not met, otherwise they return `None`.

#### Return value
A mock object can be set to return a specific value when called, using the `return_value` attribute, as shown below:

```python
# Set in the constructor
mock_object = Mock(return_value='Hello')
mock_object()
# Hello

# Set in the attribute
mock_object = Mock()
mock_object.return_value = 'World'
mock_object()
# World
```

#### Side effects
If the behaviour of a mock object when called is more complex than returning a value, the `side_effect` attribute can be used.
For example, the mock object may need to instantiate new objects that can be used in an assignment operation.
Similarly to `return_value`, the side effect can be set in either the constructor or in the attribute.

```python
# Function to be set as the side effect
def side_effect(*args, **kwargs):
    return True

# Set in the constructor
mock_object = Mock(side_effect=side_effect)
mock_object()
# True

# Set in the attribute
mock_object = Mock()
mock_object.side_effect = side_effect
mock_object()
# True
```

### MagicMock class
When mocking classes, sometimes the protocol methods are needed and are not available by default with `Mock`, which requires some work to set them up.

```python
# Setting the __len__ method on a mock object
mock_object = Mock()
mock_object.__len__ = Mock(return_value=3)
len(mock_object)
# 3
```

> If an `AttributeError` is thrown, the cause may be from the mock object not implementing the required [protocol methods](https://docs.python.org/3/library/unittest.mock.html#id3).

To use the protocol methods without having to set each one manually, the [**MagicMock class**](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.MagicMock) can be used which contains default implementations for each of the methods.
This class is a subclass of `Mock` and, therefore, the features discussed above are also available in `MagicMock`.

### Patch
Also included in the library is a versatile function called [**patch**](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.patch), which can be used as a decorator or context manager.
It examines a module to find a specific object and replaces it with the mock object, known as **patching** or **monkey patching**.

> Monkey patch is the replacement of one object with another at runtime [Wikipedia](https://en.wikipedia.org/wiki/Monkey_patch).

When patching, the mock object is placed only within a defined scope and handles the unpatching of the object when it is outside of the scope.
An example of its use is shown below:

```python
# Decorator
@patch('module.Class')
def test_function(mock_object):
    pass

# Context manager
def test_function():
    with patch('module.Class') as mock_object:
        pass
```

Here, patch replaces `Class` with a `MagicMock` instance, i.e. a mock object, inside of **module**.
The mock object is also referenced in the variable `mock_object`, which can be used to configure its behaviour.

To patch correctly, ["specify the location of the resource to be mocked, relevant to where it's imported"](https://www.integralist.co.uk/posts/mocking-in-python/).
For example, if a module, let say ```foo```, is being tested and you want to replace a class, ```bar```, that is used inside of ```foo``` with a mock object, the reference passed to patch should refer to where ```bar``` is imported into ```foo```.
This is due to how patching operates, as it changes the object which a name points to.

> More information on how to patch correctly is [available here](https://docs.python.org/3/library/unittest.mock.html#where-to-patch).

The patch library also has several other tools to patch [objects](https://docs.python.org/3/library/unittest.mock.html#patch-object) and [dictionaries](https://docs.python.org/3/library/unittest.mock.html#patch-dict).

### AsyncMock class
So far this post has looked at mocking synchronous code.
However, as the **asyncio** library grows and its usage increases, the need for testing asynchronous code arises.

To facilitate this, the mock library has the [**AsyncMock class**](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.AsyncMock).
The behaviour of this class is slightly different from `Mock` and `MagicMock`, which can cause some difficulties.

When calling an instance of `AsyncMock`, an awaitable is returned, i.e. an async function.

```python
mock_object = AsyncMock()
mock_object
# <AsyncMock id='4558004032'>

mock_object()
# <coroutine object AsyncMockMixin._execute_mock_call at 0x10fab2bc0>
```

The `return_value` and `side_effect` attributes can be set on an `AsyncMock` instance to define the value returned by the coroutine.
If they are not set, the coroutine returns a new instance of `AsyncMock`.

> To receive the return value from the coroutine, it must be awaited.
> More information is available at the [asyncio docs](https://docs.python.org/3/library/asyncio.html).

## Use cases
Now that the basics of mocking in python are understood, some use cases can be reviewed.

### Mocking functions
The first use case is to mock a function.
As shown previously, this can be achieved using `Mock`.

```python
mock_function = Mock(return_value=True)
mock_function()
# True
mock_function.assert_called_once()
```

### Mocking class methods
The second use case may be to mock methods of classes.
Doing so requires setting a class method to a Mock instance, as shown below:

```python
real_instance = RealClass()
real_instance.method = Mock()

real_instance.method()
# <Mock name='mock.method()' id='...'>

real_instance.method.assert_called_once()
```

### Mocking Classes
The third use case is to mock an entire class.
When a class is mocked, the class is replaced with `Mock`.
This means that calling the original class results in new instances of `Mock`.
However, to access the new instances, the `return_value` attribute must be accessed first.
This is due to `return_value` storing a reference to the new instances.

```python
mock_class = Mock()
# Access return_value to access the new
# Mock instance created later on.
mock_class.return_value.get.return_value = True

mock_class_instance = mock_class()
mock_class_instance.get()
# True

mock_class.return_value.get.assert_called_once()
# or
mock_class_instance.get.assert_called_once()
```

### Mocking chained calls
[Mocking chained calls](https://docs.python.org/3/library/unittest.mock-examples.html#mocking-chained-calls) is a more complex use case, however, one which the mock library is capable of handling through the use of the `return_value` attribute.
As previously mentioned in **Mocking classes**, every time a call is made a new Mock instance is created and it is accessible via the `return_value` attribute.
Therefore, to step through a chain of calls, where each call is a new `Mock` instance, access the `return_value` attribute of each `Mock` instance.
An example is shown below:

```python
mock_object = Mock()
# <Mock id='...'>

mock_object()
# <Mock name='mock()' id='...'>
mock_object.assert_called()

mock_object.get()
# <Mock name='mock.get()' id='...'>
mock_object.get.assert_called()

mock_object.get().set()
# <Mock name='mock.get().set()' id='...'>
mock_object.get.return_value.set.assert_called()

mock_object.get().set().get()
# <Mock name='mock.get().set().get()' id='...'>
mock_object.get.return_value.set.return_value.get.assert_called()
```

Being able to access the new instance in a chained call means the behaviour of a mock object can be defined:

```python
mock_object = Mock()
config = {'get.return_value.set.return_value.get.return_value': True}
mock_object.configure_mock(**config)

mock_object.get().set().get()
# True
```

As you may have noticed, the chain of calls can become long, and asserting that a set of particular calls have been made can become difficult.
Therefore, the `call.call_list()` [function](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.call.call_list) can be used to create the chain of calls.

```python
chained = call.get().set().get()
call_list = chained.call_list()
assert mock_object.mock_calls == call_list
```

To reduce the call chain further, a reference to a particular mock object can be stored and used later:

```python
mock_object = Mock()
mock_object_foo = mock_object.foo
mock_object_bar = mock_object.bar

mock_object_foo.something()
# <Mock name='mock.foo.something()' id='...'>
mock_object_bar.other.thing()
# <Mock name='mock.bar.other.thing()' id='...'>

mock_object.mock_calls
[call.foo.something(), call.bar.other.thing()]

expected_calls = [call.foo.something(), call.bar.other.thing()]
manager.mock_calls == expected_calls
# True
```

### Asynchronous Context Managers
The final use case may see the mocking of context managers, which can be tricky to configure.
This is especially true for asynchronous context managers as the use of the ```async``` keyword can result in an `AsyncMock` instance returning a coroutine, which does not have the necessary protocol methods: `___aenter__()`; and, `__aexit__()`.
Therefore, to prevent an `AttributeError` being thrown for either of the protocol methods, an instance of `MagicMock` should be explicitly stated.

```python
Mock = MagicMock()
config = {
    'return_value.__aenter__.return_value.get': MagicMock(),
    'return_value.__aenter__.return_value.get.return_value.__aenter__.return_value.text.return_value': 100
}
Mock.configure_mock(**config)

url = 'https://localhost'

async with Mock() as session:
    async with session.get(url) as response:
        result = await response.text()

assert result == 100
```

Stepping through the example above, first, an instance of `MagicMock` is created and a reference to it is stored in the variable `Mock`.
Next, the mock object is configured to return a new instance of `MagicMock` for a method called get, and the value 100 for a method called text.
With this first section of code, we have explicitly set `MagicMock` to be used for the two asynchronous context managers that are present in the next section of code.

> The [python documentation](https://docs.python.org/3/library/unittest.mock-examples.html#mocking-asynchronous-context-manager) provides an example of mocking asynchronous context managers by creating a class and passing it as a spec to a MagicMock instance, however, I have not been able to successfully apply this approach.

## Conclusion
This post has looked at the mock library to see how objects can be mocked in python.
The core tools available in the library were discussed, including the Mock, MagicMock, and AsyncMock classes.
These classes allow for the creation of mock objects, the configuration of specific behaviour, and assertion of interaction.
Additionally, the patch library was reviewed and its ability to replace objects with mock objects.
Finally, a few use cases were discussed and the use of the `return_value` attribute to not only set a value but, also to access returned values by methods and functions for configuration and assertion purposes.
