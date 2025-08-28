---
# You can also start simply with 'default'
theme: default
lineNumbers: true
title: Pycon GR 2025
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
---

<style lang="css" global>
:root {
 --shiki-token-comment: #fff;
 --shiki-ansi-bright-black-dim: #fff;
 --slidev-code-background: #f6f8fa;
}

.slidev-page ul > li::marker {
  content: "👉 ";
  font-size: 1.1em;
}

.slidev-page:not(:first-child)::after {
  content: attr(data-slidev-no);
  position: absolute;
  bottom: 0.8rem;
  right: 1rem;
  font-size: 0.8rem;
}

.slidev-code-dishonored:not(.shiki-magic-move-line-number) {
 opacity: 1 !important;
}

.slidev-code-highlighted:not(.shiki-magic-move-line-number) {
 background-color: #d2d4fe !important;
}

.shiki-magic-move-line-number {
 opacity: 0.3 !important;
}

.shiki .highlghted {
 background-color: red !important;
}

h1 {
 color: teal;
}

.mark {
 background: yellow;
}

.twocol {
 display: grid;
 grid-template-columns: 1fr 1fr;
 gap: 20px;
}

</style>


# To mock or not to mock?

<!--
Hello everyone, I’m Giorgos. I work as a software engineer, and whenever I sit down to write new tests, I run into the same questions:

Should I mock here or not? And if I do, which kind of test double is the right fit?

To answer that, let’s start at the beginning: what exactly is a test double?
-->

---

# What is a test double?
A test double is to code what a stunt double is to an actor.


<img border="rounded" src="/test_double_bears.png" alt="" class="w-full h-[90%] object-contain"/>

<!--
Just like actors don’t perform every dangerous scene themselves and stunt doubles step in for them.

Okay maybe not Tom Cruise, he does his own stunts.

In the same way, we use test doubles in place of real objects. They take the risks for us — making our tests faster, safer, and more predictable.

Without them, our tests get tied to real services and environments, and that’s when things become slow and flaky.
-->

---

# Types of test doubles

<br/>

<div class="grid grid-cols-5 auto-rows-min gap-0 max-w-4xl mx-auto mt-10">
  <div v-click class="col-span-2 row-start-1 border-2 rounded-2xl p-6 shadow-lg text-center text-xl font-semibold -mt-4 bg-white">
    Mocks 🎭
  </div>
  <div v-click class="col-span-2 col-start-2 row-start-2 border-2 rounded-2xl p-6 shadow-lg text-center text-xl font-semibold -mt-4 bg-white">
    Stubs 🪤
  </div>
  <div v-click class="col-span-2 col-start-3 row-start-3 border-2 rounded-2xl p-6 shadow-lg text-center text-xl font-semibold -mt-4 bg-white">
    Fakes 🏗️
  </div>
  <div v-click class="col-span-2 col-start-4 row-start-4 border-2 rounded-2xl p-6 shadow-lg text-center text-xl font-semibold -mt-4 bg-white">
    Spies 🕵️
  </div>
</div>

<!--
Having said that, let's take a look at the different types of test doubles.

We have...

All of them can replace objects in our code, but each one serves a different purpose.

And in fact they can be used together as well.
-->

---

# What is a mock?

A type of test double that we can use to replace real objects in our code.

Replace real object ➡️ Control behavior ➡️ Track interactions (call arguments) 

In `unittest` framework ➡️ create mocks with either `Mock` or `MagicMock`.

```python [example_mock.py] {none|5,8-9}
import unittest.mock as mock

mock_object = mock.Mock()

mock_object.some_method(x=1, y=2) # returns a new mock object
mock_object.some_method.assert_called_once()  # after the call, track interactions with the mock

mock_object.some_method(3, y=4)  # call again
mock_object.some_method.assert_has_calls([mock.call(x=1, y=2), mock.call(3, y=4)]) # assert many calls

```

<!--
A mock is a type of `test_double` that stands in place for real objects in our code.

It replaces the original object with one that we can control the behavior of, and allows us to track the interactions with the mocked object.

In Python's built-in test framework there are two foundational classes that create mocks, 
`Mock` & `MagicMock`.

Now let's have a look at some features of the `Mock` class.

We can call any method with any arguments, the method exists automatically, the return value will be a new mock.

If we call the same method again, we can assert all of them at once. Be consistent  -- in mixing argument with keyword arguments in call assertions. You have to assert them the way they were called.
-->

---

# Difference between `MagicMock` and `Mock`

`MagicMock` supports "magic methods", while `Mock` does not.

<div v-click>

Prefer `MagicMock` when using **special methods**, like `Sequence` (`list`, `tuple`) or a context manager that defines `__enter__` & `__exit__`.

<div class="twocol">

```python [example_magic_mock_pass.py ✅] {6-7}
import unittest.mock as mock

mock_object = mock.MagicMock()

# MagicMock has __enter__ & __exit__ magic methods 👍
with mock_object as m:
    pass

# any other magic methods are also available 👍
len(mock_object)

# track interactions with the mock object
mock_object.__enter__.assert_called_once()
mock_object.__len__.assert_called_once()
```

```python [example_mock_fail.py ❌] {6-7}
import unittest.mock as mock

mock_object = mock.Mock()

# Mock does not have __enter__ & __exit__
with mock_object as m:
    pass

# TypeError💥 'Mock' object does not support 
# the context manager protocol
```
</div>

</div>

<!--
What is the difference between the two though?
`MagicMock` supports magic methods .

--

Let's see an example, suppose we want to use the object in a `with` block. 

In order to do that our object needs to define the context manager protocol, this means it needs to define `__enter__` & `__exit__` ,  which are magic methods.

However, `Mock` objects do not do this automatically! As we can see in the right column our snippet fails with a `TypeError`

We have to use a `MagicMock` if we are going call magic methods in the code we are replacing. 

And this is not limited to `with` blocks, it might as well be other special methods like `length`, if the object we are replacing behaves like a  `Sequence`.
-->

---

# How to replace an object with a test double?
Does the function we are testing **own** the dependency we want to replace?
<br/> ➡️ Then use `mock.patch` **where the dependency is used** not where it is defined.

<div class="twocol -mt-2">

```python [test_example_with_dep.py] {5|4} {at: 1}
import example_with_dep
import unittest.mock as mock

# @mock.patch("io.FileIO")  ❌ does not work
@mock.patch("example_with_dep.FileIO") # ✅ works
def test_read_file(mock_io: mock.MagicMock):
  example_with_dep.read_file("some-file.txt")

  # assert that we entered the with block
  mock_io.return_value.__enter__.assert_called_once()
```

```python [example_with_dep.py] {5} {at: 1}
from io import FileIO

# function directly uses FileIO
def read_file(filename: str) -> bytes:
  with FileIO(filename) as f:
    return f.read()
```
</div>

<div v-click class="mt-2 p-2 border rounded-xl bg-yellow-50 bg-opacity-75">
Why global patching does not work?

1. `from io import FileIO` binds local reference to `FileIO` in the `example_with_dep` module.
2. Our patch targets `io.FileIO`, but `read_file` uses the local reference.
3. We would have to patch before importing or `reload(example_with_dep)` - both not a good practice.

</div>

<!--
But how to actually replace the real object with a test double?

In the case of a function that owns the dependency, we need to use `mock.patch` and patch the imported name where the dependency is being used. 

By default `mock.patch`, replaces the object with a `MagicMock`.

But you can provide your own mock object with the `new` argument.

--

Note that patching globally at the place `FileIO` is defined does not work. Why is that though? [bullets]

--
-->

---

# Better way: use Dependency Injection
If the function we are testing expects the dependency as a parameter.
<br/> ➡️ pass the `MagicMock` instead, no need to use `mock.patch`

<div class="twocol -mt-2">

```python [test_example_with_di.py] {none|5,14-15}
import example_with_di
import unittest.mock as mock

def test_read_file():
  mock_io = mock.MagicMock(spec=FileIO)

  mock_io.open()
  example_with_di.read_file(mock_io)
  mock_io.close()

  # assert that we entered the with block
  mock_io.read.assert_called_once()

  mock_io.arbitrary_method() # raises AttributeError 💥 
  # Mock object has no attribute 'arbitrary_method'
```

```python [example_with_di.py]
from io import FileIO

# function directly uses FileIO
def read_file(opened_file: FileIO) -> bytes:
  # just operate on the file object
  # caller manages opening/closing
  return opened_file.read()
```

</div>

<!--
In what other ways can we replace a dependency?

In this example, we call read_file. The implementation simply returns the contents of the already opened file.

Notice that read_file does not own the FileIO object, it just operates on it.

When a function accepts its dependency as a parameter, there’s no need to patch. Instead, pass a mock object in place of the dependency.

-- In the test, we create a MagicMock with a spec argument. The spec makes the mock strictly follow the protocol of the FileIO class.

That’s why calling an arbitrary method on it raises an AttributeError the mock only allows real FileIO methods.
-->

---

# Extra: Other ways to patch
Besides the decorator, we can also use `mock.patch` as a context manager.

You can also use `mock.patch.object` to patch attributes on an **already imported** object.

<div class="twocol -mt-2">

```python [test_example_with_dep_context.py] {none|5-7}
import example_with_dep
import unittest.mock as mock

def test_read_file():
  with mock.patch(
    "example_with_dep.FileIO", autospec=True
  ) as mock_io:
    example_with_dep.read_file("some-file.txt")

    # assert that we entered the with block
    mock_io.return_value.__enter__.assert_called_once()
```

```python [test_example_with_dep_object.py] {none|5-7}
import example_with_dep
import unittest.mock as mock

def test_read_file():
  with mock.patch.object(
    example_with_dep, "FileIO", autospec=True
  ) as mock_io:
    example_with_dep.read_file("some-file.txt")

    # assert that we entered the with block
    mock_io.return_value.__enter__.assert_called_once()
```

</div>

Use `autospec` to automatically follow `FileIO`'s protocol.

<!--
If we can’t inject, we patch as we saw earlier. 

-- Besides decorators, we can use `patch`  as context manager. 

-- `patch.object` is also handy when the module is already imported, and we patch its attributes directly.

Just like the decorator, the patch only lives for the with block.

We can use the `autospec` flag to have the mock automatically deduce which protocol to follow.
-->

---

# What is a stub? How does it differ from a mock?
It provides predefined responses to function calls, but does not track interactions.

We can use a `Mock` or `MagicMock` object for stubbing by fixing the `return_value` of a method.

These objects can double as both a mock and a stub.

```python [example_stub_fixed.py] {none|5-6|8-10|13-14}
import unittest.mock as mock

mock_object = mock.Mock()

mock_object.some_method.return_value = "some value"
assert mock_object.some_method() == "some value"

mock_object.some_method.side_effect = [1, 2]
assert mock_object.some_method() == 1
assert mock_object.some_method() == 2

# we can also raise exceptions
mock_object.raise_method.side_effect = ValueError("some value")
mock_object.raise_method()  # raises ValueError 💥 
```

<!--
As of now we have covered mocks but what is a stub, how does it differ?
`Stubs` provide canned responses, helping us test outputs or control intermediate results.

`Mock` & `MagicMock`, the usual suspects can be used to provide these results for functions that we expect to be called by the code we test. In our example this function is called `some_method`

-- One way to set fixed response it to set the `return_value`,  this way every call will always return `some_value`

-- In the case of repeated calls we can provide an iterable to the `side_effect` attribute

-- Finally, to test against errors we can use `side_effect` to raise Exceptions like a `ValueError`
-->

---

# An example: generating HTTP headers for a request
You are given an in-house authentication library to get a token, with the following signature:

```python [authlib.py]
def authenticate(account_id: str, resource_id: Optional[str] = None) -> str:
    """
    Calls a remote authentication service to get a token for a specific resource.
    
    Args
    ----
        account_id: str, example "some-project-dev"
        resource_id: Optional[str], a target service id that 
            we want to authneticate for, e.g. "STORAGE-SERVICE-XXXXXX"
    
    Returns
    -------
        str, a token
    """
    ...
```

<!--
We have an in-house authentication library with a function called authenticate. 

It goes off to some remote service and gives us back a token.

It takes an account_id — that’s required — and an optional resource_id. 

The idea is: sometimes we want a generic token for an account

but other times we need a token specific to a resource, like a storage service.
-->

---

# An example: generating HTTP headers for a request

We want to include the token in the `Authorization` header of our HTTP requests, along with some static headers. These headers are part of a client library we are writing.

```python [headers.py]
class Configuration:
 user_id: str
 resource_id: Optional[str] = None

def get_headers(config: Configuration) -> dict:
 token = authlib.authenticate(
   config.user_id
 )

 return {
   "Content-Type": "application/json",
   "Authorization": f"Bearer {token}"
 }
```

<!--
Our utility function get_headers takes this configuration, calls authlib.authenticate, and uses the returned token to build a dictionary of HTTP headers.

We include the token in the Authorization header of our request, along some static headers like content-type.

When testing get_headers, we need to make sure to replace authenticate with a test double

but what type of test double to use in this case?
-->

---

# Mocks + stubs: verify behaviour and results
We stub `authenticate` to control the token with expected results<span v-click=1>, then assert the arguments of the call.</span>

<div class="twocol -mt-4">

````md magic-move [test_headers.py] {lines: true}
```python {12}
@mock.patch("headers.authlib.authenticate")
def test_get_headers(self, mock_auth):
   mock_auth.return_value = "token"
  
   expected = {
     "Content-Type": "application/json",
     "Authorization": "Bearer token"
   }
  
   actual = get_headers(mock.Mock())
   self.assertEqual(actual, expected)  # validate result
```

```python {3-6,18-21}
@mock.patch("headers.authlib.authenticate")
def test_get_headers(self, mock_auth):
   config = {
     "user_id": "a_machine",
     "resource_id": "STORAGE-SERVICE-XXXXXX",
   }
   mock_auth.return_value = "token"
  
   expected = {
     "Content-Type": "application/json",
     "Authorization": "Bearer token"
   }
  
   actual = get_headers(mock.Mock(**config))
   self.assertEqual(actual, expected)  # validate result

   # validate interaction with 3rd party
   mock_auth.assert_called_once_with(
     config["user_id"],
     config["resource_id"]
   )
```
````

````md magic-move [headers.py] {at:1, lines:true}
```python {18}  <!-- add plus one -->
import authlib

@dataclass
class Configuration:
 user_id: str
 resource_id: Optional[str] = None

def get_headers(config: Configuration) -> dict:
 token = authlib.authenticate(
    config.user_id 
    # a bug 🐞 here
 )

 return {
   "Content-Type": "application/json",
   "Authorization": f"Bearer {token}"
 }
```

```python {11}
import authlib

@dataclass
class Configuration:
 user_id: str
 resource_id: Optional[str] = None

def get_headers(config: Configuration) -> dict:
 token = authlib.authenticate(
   config.user_id,
   config.resource_id  # was missing
 )

 return {
   "Content-Type": "application/json",
   "Authorization": f"Bearer {token}"
 }
```
````
</div>

<!--
We maybe tempted to just stub authenticate by fixing its return value to a dummy `token` string.
This test has full coverage, and it passes. 

But if you recall `authenticate`'s signature, in fact we didn't actually propagate the `resource_id` that we got in the configuration object.

Our unit test would still be successful, but we would totally miss the link between `resource_id` and the generated token.

--

To fix that, we can use mocks to also assert the arguments of the call to `authenticate`.

Now if we forget to pass `resource_id`, the test will fail, alerting us to the bug.
-->

---

# What is a fake?
A fake is a working implementation, but it is kept lightweight for testing purposes.

Avoids complex dependencies, I/O operations, or external services ➕ no need maintaining stub states.

<div class="twocol">

```python [storage.py]
# real dependency (talks to a service)
class StorageClient:
  def __enter__(self):
    self.connect()
    return self

  def __exit__(self, exc_type, exc_value, traceback):
    self.disconnect()

  def __getitem__(self, key: str):
    return self.read_from_service(key)

  def __setitem__(self, key: str, value):
    self.write_to_service(key, value)
```

```python [example_fake.py]
# fake dependency (in-memory implementation)
class FakeStorageClient:
  def __init__(self):
    self.__store = {}

  def __enter__(self):
    return self

  def __exit__(self, exc_type, exc_value, traceback):
    pass

  def __getitem__(self, key: str):
    return self.__store.get(key)

  def __setitem__(self, key: str, value):
    self.__store[key] = value
```

</div>

<!--
Do we have to always use stubs? What is a fake?
A fake is a working implementation, but simplified for testing.

Unlike stubs, you don’t have to predefine return values. The fake behaves like the real thing, just without the heavy parts.

That means no network calls, no file I/O, and no external services — but still realistic behavior.

On the left, the real StorageClient - let's suppose it follows the Mapping protocol -  talks to real a service.

On the right, the FakeStorageClient just uses an in-memory dictionary.

This avoids maintaining stub states and makes tests faster and more reliable, while being close enough to production behavior.
-->

---

# Concrete example: using a fake for testing
We have a service that uses a `StorageClient` to read and write data.

Let's say we have a slightly more sophisticated service that uses a `StorageClient` to read and write data, serilize and deserialize JSON, and chunk them before saving if they are too large.

I will show that this way I can test serialization, deserialization, and chunking logic without needing to stub every interaction with the storage client. I can check the number of chunks for a write operation, and I can check that the data I read is the same as what I wrote.

---

# What is a spy?
A spy is a test double that wraps a real object, allowing us to monitor its interactions while still using its actual implementation.

We can use a `MagicMock` / `Mock` with the `wraps` argument to create a spy.

<div class="flex flex-col -mt-4">

```python [example_spy.py]
import unittest.mock as mock

class DollarConverter:
  rates = {
    "USD": 1,
    "EUR": 0.9,
    "GBP": 0.8,
  } # static data

  def convert(self, amount: float, currency: str) -> float:
    return rate.get(currency, 0) * amount

spy_object = mock.Mock(wraps=DollarConverter(), autospec=True)
euros = spy_object.convert(10, "EUR")

assert euros == 9  # uses the real method
spy_object.convert.assert_called_once_with(10, "EUR")  # tracks the call
```

</div>

---

# Using a spy to inspect serialization/deserialization for StorageClient
We can use a spy to wrap the real `StorageClient` and monitor its interactions while still using its actual implementation.

show code

---

# Takeaway

<div class="grid grid-cols-5 auto-rows-min gap-0 max-w-4xl mx-auto mt-10">
  <div v-click class="col-span-2 row-start-1 border-2 rounded-2xl p-6 shadow-lg text-center -mt-4 bg-white">
    Mocks 🎭 <br/> <span class="text-xl font-semibold italic"> verifies behavior </span>
  </div>
  <div v-click class="col-span-2 col-start-2 row-start-2 border-2 rounded-2xl p-6 shadow-lg text-center -mt-4 bg-white">
    Stubs 🪤 <br/> <span class="text-xl font-semibold italic"> controls outputs </span>
  </div>
  <div v-click class="col-span-2 col-start-3 row-start-3 border-2 rounded-2xl p-6 shadow-lg text-center  -mt-4 bg-white">
    Fakes 🏗️ <br/> <span class="text-xl font-semibold italic">working minimal replacements</span>
  </div>
  <div v-click class="col-span-2 col-start-4 row-start-4 border-2 rounded-2xl p-6 shadow-lg text-center -mt-4 bg-white">
    Spies 🕵️ <br/> <span class="text-xl font-semibold italic">wraps real code for inspection</span>
  </div>
</div>

<!--
This brings us to the takeaways.

We covered...
-->

---

# Takeaway


<v-clicks>


* Prefer `MagicMock` over `Mock` unless you are sure you don't need magic methods

* Use `autospec` / `spec` to make the mock strictly follow the real object's protocol

* Prefer Dependency Injection over patching when possible, use patching when you interact with 3rd party code, that is not modular

* If you need to patch, **patch where the dependency is used**

* In `unittest`, `Mock` & `MagicMock` can be used in a way that combines all types of test doubles

</v-clicks>

<!--
A few more things to watch for...
-->

---
layout: center
---

# Thank you! 😊
