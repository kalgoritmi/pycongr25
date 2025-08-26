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

Okay maybe not Tom Cruise.

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

It replaces the orginal object with that we can control the behavior of, and allows us to track the interactions with the mocked object. 

When using the `unittest` framework we can use either `Mock` or `MagicMock` classes to create a mock object.

```python [example_mock.py]
import unittest.mock as mock

mock_object = mock.Mock()

mock_object.some_method()  # calling any method returns another mock object

mock_object.some_method.assert_called_once() # we can track interactions with the mock object

```

<!--
A mock is a type of test_double that stand in place for real objects in our code.

It replaces the original object with one that we can control the behavior of, and allows us to track the interactions with the mocked object.

In Python's built-in test framework there are two foundational classes that create mocks, 
Mocks & MagicMocks.
-->

---

# Difference between `MagicMock` and `Mock`?

The main difference is that `MagicMock` has "magic methods" ready to use, while `Mock` does not.

Prefer `MagicMock` whenever an object uses special methods, like `Sequence` (`list`, `tuple`) or a context manager object that defines `__enter__` & `__exit__`.

<div class="twocol">

```python [example_magic_mock_pass.py ✅]
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

```python [example_mock_fail.py ❌]
import unittest.mock as mock

mock_object = mock.Mock()

# Mock does not have __enter__ & __exit__
with mock_object as m:
    pass

# TypeError💥 'Mock' object does not support 
# the context manager protocol
```
</div>

<!--
This means that if you need to mock an object that uses magic methods, you should use `MagicMock`. 

These can be a `Sequence` like a `List`, or a context manager that defines `__enter__` & `__exit__`.
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

But if you recall `authenticate`'s signature, in fact we didn't actually propagate the `resource_id` we got in the configuration object.

Our unit test would still be successful, but we would totally miss the link between `resource_id` and the generated token.

--

To fix that, we can use mocks to also assert the arguments of the call to `authenticate`.

Now if we forget to pass `resource_id`, the test will fail, alerting us to the bug.

-->

---

# Stubs

---
