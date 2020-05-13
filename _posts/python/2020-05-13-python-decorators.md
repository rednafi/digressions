---
title: Untangling Python Decorators
toc: true
comments: true
layout: post
description: Dissecting the anatomy of decorators in Python
categories: [Python]
---


When I first learned about Python decorators, using them felt like doing voodoo magic. Decorators can give you the ability to add new functionalities to any callable without actually touching or changing the code inside it. This can typically yield better encapsulation and help you write cleaner and more understandable code. However, *decorator* is considered as a fairly advanced topic in Python since understanding and writing it requires you to have command over multiple additional concepts like first class objects, higher order functions, closures etc. First, I'll try to introduce these concepts as necessary and then unravel the core concept of decorator layer by layer. So let's dive in.

## First Class Objects

In Python, basically everything is an object and functions are regarded as first-class objects. It means that functions can be passed around and used as arguments, just like any other object (string, int, float, list, and so on). You can assign functions to variables and treat them like any other objects. Consider this example:

```python
def func_a():
    return "I was angry with my friend."


def func_b():
    return "I told my wrath, my wrath did end"


def func_c(*funcs):
    for func in funcs:
        print(func())


main_func = func_c
main_func(func_a, func_b)
```

```
>>> I was angry with my friend.
>>> I told my wrath, my wrath did end
```

The above example demonstrates how Python treats functions as first class citizens. First, I defined two functions, `func_a` and `func_b` and then `func_c` takes them as parameters. `func_c` runs the functions taken as parameters and prints the results. Then we assign `func_c` to variable `main_func`. Finally, we run `main_func` and it behaves just like `func_c`.

## Higher Order Functions

Python also allows you to use functions as return values. You can take in another function and return that function or you can define a function within another function and return the inner function.

```python
def higher(func):
    """This is a higher order function.
    It returns another function.
    """

    return func


def lower():
    return "I'm hunting high and low"


higher(lower)
```

```
>>> <function __main__.lower()>
```


Now you can assign the result of `higher` to another variable and execute the output function.

```python
h = higher(lower)
h()
```

```
>>> "I'm hunting high and low"
```


Let's look into another example where you can define a nested function within a function and return the nested function instead of its result.


```python
def outer():
    """Define and return a nested function from another function."""

    def inner():
        return "Hello from the inner func"

    return inner


inn = outer()
inn()
```

```
>>> 'Hello from the inner func'
```

Notice how the nested function `inner` was defined inside the `outer` function and then the return statement of the `outer` function returned the nested function. After definition, to get to the nested function, first we called the `outer` function and received the result as another function. Then executing the result of the `outer` function prints out the message from the `inner` function.

## Closures

You saw examples of inner functions at work in the previous section. Nested functions can access variables of the enclosing scope. In Python, these non-local variables are read only by default and we must declare them explicitly as non-local (using `nonlocal` keyword) in order to modify them. Following is an example of a nested function accessing a non-local variable.

```python
def burger(name):
    def ingredients():
        if name == "deli":
            return ("steak", "pastrami", "emmental")
        elif name == "smashed":
            return ("chicken", "nacho cheese", "jalapeno")
        else:
            return None

    return ingredients
```

Now run the function,

```python
ingr = burger("deli")
ingr()
```

```
>>> ('steak', 'pastrami', 'emmental')
```

Well, that's unusual.

The `burger` function was called with the string `deli` and the returned function was bound to the name `ingr`. On calling `ingr()`, the message was still remembered and used to derive the outcome although the outer function `burger` had already finished its execution.

This technique by which some data ("deli") gets attached to the code is called closure in Python. The value in the enclosing scope is remembered even when the variable goes out of scope or the function itself is removed from the current namespace. Decorators uses the idea of non-local variables multiple times and soon you'll see how.

## Creating Decorators

With these prerequisites out of the way, let's go ahead and create your first simple decorator.

```python
def deco(func):
    def wrapper():
        print("This will get printed before the function is called.")
        func()
        print("This will get printed after the function is called.")

    return wrapper
```

Before using the decorator, let's define a simple function without any parameters.

```python
def ans():
    print(42)
```

Treating the functions as first-class objects, you can use your decorator like this:


```python
ans = deco(ans)
ans()
```

```
>>> This will get printed before the function is called.
    42
    This will get printed after the function is called.
```

In the above two lines, you can see a very simple decorator in action. Our `deco` function takes in a target function, manipulates the target function inside a `wrapper` function and then returns the `wrapper` function. Running the function returned by the decorator, you will get your modified result. To put it simply, decorators wraps a function and modifies its behavior.

> The decorator function runs at the time the decorated function is imported/defined, not when it is called.

Before moving onto the next section, let's see how we can get the return value of target function instead of just printing it.


```python
def deco(func):
    """This modified decorator also returns the result of func."""

    def wrapper():
        print("This will get printed before the function is called.")
        val = func()
        print("This will get printed after the function is called.")
        return val

    return wrapper


def ans():
    return 42
```

In the above example, the wrapper function returns the result of the target function and the wrapper itself. This makes it possible to get the result of the modified function.

```python
ans = deco(ans)
print(ans())
```

```
>>> This will get printed before the function is called.
    This will get printed after the function is called.
    42
```

Can you guess why the return value of the decorated function appeared in the last line instead of in the middle like before?

## The @ Syntactic Sugar

The way you've used decorator in the last section might feel a little clunky. First, you have to type the name `ans` three times to call and use the decorator. Also, it becomes harder to tell apart where the decorator is actually working. So Python allows you to use decorator with the special syntax `@`. You can apply your decorators while defining your functions, like this:


```python
@deco
def func():
    ...


# Now call your decorated function just like a normal one
func()
```


Sometimes the above syntax is called the **pie syntax** and it's just a syntactic sugar for `func = deco(func)`.

## Decorating Functions with Arguments

The naive decorator that we've implemented above will only work for functions that take no arguments. It'll fail and raise `TypeError` if your try to decorate a function having arguments with `deco`. Now let's create another decorator called `yell` which will take in a function that returns a string value and transform that string value to uppercase.

```python
def yell(func):
    def wrapper(*args, **kwargs):
        val = func(*args, **kwargs)
        val = val.upper() + "!"
        return val

    return wrapper
```

Create the target function that returns string value.

```python
@yell
def hello(name):
    return f"Hello {name}"
```

```python
hello("redowan")
```

```
>>> 'HELLO REDOWAN!'
```

Function `hello` takes a `name:string` as parameter and returns a message as string. Look how the `yell` decorator is modifying the original return string, transforming that to uppercase and adding an extra `!` sign without directly changing any code in the `hello` function.

## Lost Identity

In Python, you can introspect any object and its properties via the interactive shell. A function knows its identity, docstring etc. For instance, you can inspect the built in `print` function in the following ways:

```python
print
```

```
>>> <function print>
```

```python
print.__name__
```

```
>>> 'print'
```

```python
print.__doc__
```

```
>>> "print(value, ..., sep=' ', end='\\n', file=sys.stdout, flush=False)\n\nPrints the values to a stream, or to sys.stdout by default.\nOptional keyword arguments:\nfile:  a file-like object (stream); defaults to the current sys.stdout.\nsep:   string inserted between values, default a space.\nend:   string appended after the last value, default a newline.\nflush: whether to forcibly flush the stream."
```

```python
help(print)
```

```
>>> Help on built-in function print in module builtins:

    print(...)
    print(value, ..., sep=' ', end='\n', file=sys.stdout, flush=False)

    Prints the values to a stream, or to sys.stdout by default.
    Optional keyword arguments:
    file:  a file-like object (stream); defaults to the current sys.stdout.
    sep:   string inserted between values, default a space.
    end:   string appended after the last value, default a newline.
    flush: whether to forcibly flush the stream.
```

This introspection works similarly for functions that you defined yourself. I'll be using the previously defined `hello` function.


```python
hello.__name__
```

```
>>> 'wrapper'
```

```python
help(hello)
```

```
>>> Help on function wrapper in module __main__:
    wrapper(*args, **kwargs)
```


Now what's going on there. The decorator `yell` has made the function `hello` confused about its identity. Instead of reporting its own name, it takes the identity of the inner function `wrapper`. This can be confusing while doing debugging. You can fix this using builtin `functools.wraps` decorator. This will make sure that the original identity of the decorated function stays preserved.

```python
import functools


def yell(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        val = func(*args, **kwargs)
        val = val.upper() + "!"
        return val

    return wrapper


@yell
def hello(name):
    "Hello from the other side."
    return f"Hello {name}"
```

```python
hello("Galaxy")
```

```
>>> 'HELLO GALAXY!'
```


Introspecting the `hello` function decorated with modified decorator will give you the desired result.

```python
hello.__name__
```

```
>>> 'hello'
```

```python
help(hello)
```

```
>>> Help on function hello in module __main__:

    hello(name)
        Hello from the other side.
```

## Real World Decorators

Before moving on to the next section let's see a few real world examples of decorators. To define all the decorators, we'll be using the following template that we've perfected so far.


```python
import functools


def decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # Do something before
        val = func(*args, **kwargs)
        # Do something after
        return val

    return wrapper
```

### Timer

Timer decorator will help you time your callables in a non-intrusive way. It can help you while debugging and profiling your functions.


```python
import time
import functools


def timer(func):
    """This decorator prints out the execution time of a callable."""

    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        val = func(*args, **kwargs)
        end_time = time.time()
        run_time = end_time - start_time
        print(f"Finished running {func.__name__} in {run_time:.4f} seconds.")
        return val

    return wrapper


@timer
def dothings(n_times):
    for _ in range(n_times):
        return sum((i ** 3 for i in range(100_000)))
```

In the above way, we can introspect the time it requires for function `dothings` to complete its execution.

```python
dothings(100_000)
```

```
>>> Finished running dothings in 0.0231 seconds.
    24999500002500000000
```


### Exception Logger

Just like the `timer` decorator, we can define a logger decorator that will log the state of a callable. For this demonstration, I'll be defining a exception logger that will show additional information like timestamp, argument names when an exception occurs inside of the decorated callable.


```python
import functools
from datetime import datetime


def logexc(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):

        # Stringify the arguments
        args_rep = [repr(arg) for arg in args]
        kwargs_rep = [f"{k}={v!r}" for k, v in kwargs.items()]
        sig = ", ".join(args_rep + kwargs_rep)

        # Try running the function
        try:
            return func(*args, **kwargs)
        except Exception as e:
            print("Time: ", datetime.now().strftime("%Y-%m-%d [%H:%M:%S]"))
            print("Arguments: ", sig)
            print("Error:\n")
            raise

    return wrapper


@logexc
def divint(a, b):
    return a / b
```

Let's invoke ZeroDivisionError to see the logger in action.


```python
divint(1, 0)
```

```
>>> Time:  2020-05-12 [12:03:31]
    Arguments:  1, 0
    Error:

        ------------------------------------------------------------

        ZeroDivisionError         Traceback (most recent call last)
        ....
```

The decorator first prints a few info regarding the function and then raises the original error.

### Validation & Runtime Checks

Python’s type system is strongly typed, but very dynamic. For all its benefits, this means some bugs can try to creep in, which more statically typed languages (like Java) would catch at compile time. Looking beyond even that, you may want to enforce more sophisticated, custom checks on data going in or out. Decorators can let you easily handle all of this, and apply it to many functions at once.

Imagine this: you have a set of functions, each returning a dictionary, which (among other fields) includes a field called “summary.” The value of this summary must not be more than 30 characters long; if violated, that’s an error. Here is a decorator that raises a ValueError if that happens:


```python
import functools


def validate_summary(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        data = func(*args, **kwargs)
        if len(data["summary"]) > 30:
            raise ValueError("Summary exceeds 30 character limit.")
        return data

    return wrapper


@validate_summary
def short_summary():
    return {"summary": "This is a short summary"}


@validate_summary
def long_summary():
    return {"summary": "This is a long summary that exceeds character limit."}


print(short_summary())
print(long_summary())
```

```
>>> {'summary': 'This is a short summary'}

    -------------------------------------------------------------------

    ValueError                       Traceback (most recent call last)

    <ipython-input-178-7375d8e2a623> in <module>
            19
            20 print(short_summary())
    ---> 21 print(long_summary())
    ...
```

### Retry

Imagine a situation where your defined callable fails due to some I/O related issues and you'd like to retry that again. Decorator can help you to achieve that in a reusable manner. Let's define a `retry` decorator that will rerun the decorated function multiple times if an http error occurs.

```python
import functools
import requests


def retry(func):
    """This will rerun the decorated callable 3 times if
    the callable encounters http 500/404 error."""

    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        n_tries = 3
        tries = 0
        while True:
            resp = func(*args, **kwargs)
            if resp.status_code == 500 or resp.status_code == 404 and tries < n_tries:
                print(f"retrying... ({tries})")
                tries += 1
                continue
            break
        return resp

    return wrapper


@retry
def getdata(url):
    resp = requests.get(url)
    return resp


resp = getdata("https://httpbin.org/get/1")
resp.text
```

```
>>> retrying... (0)
    retrying... (1)
    retrying... (2)

    '<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">\n<title>404 Not Found</title>\n<h1>Not Found</h1>\n<p>The requested URL was not found on the server.  If you entered the URL manually please check your spelling and try again.</p>\n'
```


## Stacking Decorators

You can apply multiple decorators to a function by stacking them on top of each other. Let's define two simple decorators and use them both on a function.

```python
import functools


def greet(func):
    """Greet in English."""

    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        val = func(*args, **kwargs)
        return "Hello " + val + "!"

    return wrapper


def flare(func):
    """Add flares to the string."""

    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        val = func(*args, **kwargs)
        return "🎉 " + val + " 🎉"

    return wrapper


@flare
@greet
def getname(name):
    return name


getname("Nafi")
```

```
>>> '🎉 Hello Nafi! 🎉'
```


The decorators are called in a bottom up order. First, the decorator `greet` gets applied on the result of `getname` function and then the result of `greet` gets passed to the `flare` decorator. The decorator stack above can be written as `flare(greet(getname(name)))`. Change the order of the decorators and see what happens!

## Decorators with Arguments

While defining the `retry` decorator in the previous section, you may have noticed that I've hard coded the number of times I'd like the function to retry if an error occurs. It'd be handy if you could inject the number of tries as a parameter into the decorator and make it work accordingly. This is not a trivial task and you'll need three levels of nested functions to achieve that.

Before doing that let's cook up a trivial example of how you can define decorators with parameters.


```python
import functools


def repeat(n_times=4):
    """This repeats the output of a callable."""

    def outer_wrapper(func):
        @functools.wraps(func)
        def inner_wrapper(*args, **kwargs):
            for _ in range(n_times):
                val = func(*args, **kwargs)
            return val

        return inner_wrapper

    return outer_wrapper


@repeat(n_times=2)
def hello(name):
    print(f"Hello {name}!")


@repeat(n_times=5)
def greet(name):
    print(f"Greetings {name}!")


@repeat()
def goodbye(name):
    print(f"Goodbye {name}!")


hello("Nafi")
greet("Redowan")
goodbye("Delowar")
```

```
>>> Hello Nafi!
    Hello Nafi!
    Greetings Redowan!
    Greetings Redowan!
    Greetings Redowan!
    Greetings Redowan!
    Greetings Redowan!
    Goodbye Delowar!
    Goodbye Delowar!
    Goodbye Delowar!
    Goodbye Delowar!
```

The decorator `repeat` takes a parameter `n_times` and just repeats the output of the decorated function that many times. The three layer nested definition looks scary but we'll get to that in a moment. Notice how you can use the decorator with different parameters. In the above example, I've defined three different functions to demonstrate the usage of `repeat`. It's important to note that in case of a decorator that takes parameters, you'll always need to pass something to it and even if you don't want to pass any parameter (run with the default), you'll still need to decorate your function with `deco()` instead of `deco`. Try changing the decorator on the `goodbye` function from `repeat()` to `repeat` and see what happens.

Typically, a decorator creates and returns an inner wrapper function but here in the `repeat` decorator, there is an inner function within another inner function. This almost looks like a *dream within a dream* from the movie Inception.

There are a few subtle things happening in the `repeat()` function:

* Defining `outer_wrapper()` as an inner function means that `repeat()` will refer to a function object `outer_wrapper`.

* The `n_times` argument is seemingly not used in `repeat()` itself. But by passing `n_times` a closure is created where the value of `n_times` is stored until it will be used later by `inner_wrapper()`


## Decorators with & without Arguments

You saw earlier that a decorator specifically designed to take parameters can't be used without parameters. But what if we want to design one that can used both with and without arguments. Let's redefine the `retry` decorator so that you can use it with parameters or just like an ordinary parameter-less decorator that we've seen before.

```python
def repeat(_func=None, *, n_times=4):
    """This repeats the output of a callable."""

    def outer_wrapper(func):
        @functools.wraps(func)
        def inner_wrapper(*args, **kwargs):
            for _ in range(n_times):
                val = func(*args, **kwargs)
            return val

        return inner_wrapper

    # This part enables your decorator to be used with/ without params
    if _func is None:
        return outer_wrapper
    else:
        return outer_wrapper(_func)
```


```python
@repeat
def hello(name):
    print (f"Hello {name}!"

@repeat(n_times=2)
def greet(name):
    print( f"Greetings {name}!")


hello("Nafi")
greet("Redowan")
```

```
>>> Hello Nafi!
    Hello Nafi!
    Hello Nafi!
    Hello Nafi!
    Greetings Redowan!
    Greetings Redowan!
```

Here, the `_func` argument acts as a marker, noting whether the decorator has been called with arguments or not:

If `repeat` has been called without arguments, the decorated function will be passed in as `_func`. If it has been called with arguments, then `_func` will be None. The * in the argument list means that the remaining arguments can’t be called as positional arguments. This time you can use the `repeat` with or without arguments and function `hello` and `greet` above demonstrate that.

## A Generalized Pattern

Personally, I find it cumbersome how you need three layers of nested functions to define a generalized decorator that can be used with or without arguments. [David Beazly](https://www.dabeaz.com/) in his book [Python Cookbook](https://realpython.com/asins/1449340377/) shows an excellent way to define generalized decorators without writing three levels of nested functions. It uses the built in `functools.partial` function to achieve that. The following is a pattern you can use to define generalized decorators in a more elegant way:

```python
import functools


def decorator(func=None, foo="spam"):
    if func is None:
        return functools.partial(decorator, foo=foo)

    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # Do something with `func` and `foo`, if you're so inclined
        pass

    return wrapper


# Applying decorator without any parameter
@decorator
def f(*args, **kwargs):
    pass


# Applying decorator with extra parameter
@decorator(foo="buzz")
def f(*args, **kwargs):
    pass
```

Let's redefine our `retry` decorator using this pattern.

```python
import functools


def retry(func=None, n_tries=4):
    if func is None:
        return functools.partial(retry, n_tries=n_tries)

    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        tries = 0
        while True:
            resp = func(*args, **kwargs)
            if resp.status_code == 500 or resp.status_code == 404 and tries < n_tries:
                print(f"retrying... ({tries})")
                tries += 1
                continue
            break
        return resp

    return wrapper


@retry
def getdata(url):
    resp = requests.get(url)
    return resp


@retry(n_tries=2)
def getdata_(url):
    resp = requests.get(url)
    return resp


resp1 = getdata("https://httpbin.org/get/1")
print("-----------------------")
resp2 = getdata_("https://httpbin.org/get/1")
```

```
>>> retrying... (0)
    retrying... (1)
    retrying... (2)
    retrying... (3)
    -----------------------
    retrying... (0)
    retrying... (1)
```

In this case, you do not have to write three level nested functions and the `functools.partial` takes care of that. Partials can be used to make new derived functions that have some input parameters pre-assigned.Roughly `partial` does the following:

```python
def partial(func, *part_args):
    def wrapper(*extra_args):
        args = list(part_args)
        args.extend(extra_args)
        return func(*args)

    return wrapper
```

This eliminates the need to write multiple layers of nested factory function get a generalized decorator.

## Defining Decorators with Classes

Let's write a stateful decorator named `Calls` that will remember and print out how many times the decorated function has been called. This time, I'll be using classes to write decorators. Classes are better for writing stateful decorators and you'll see why.

```python
import functools


class Calls:
    def __init__(self, func):
        functools.update_wrapper(self, func)
        self.func = func
        self.num_calls = 0

    def __call__(self, *args, **kwargs):
        self.num_calls += 1
        print(f"Call {self.num_calls} of {self.func.__name__!r}")
        return self.func(*args, **kwargs)


@Calls
def hello(name):
    print(f"Hello {name}!")


hello("Nafi")
hello("Redowan")
```

```
>>> Call 1 of 'hello'
    Hello Nafi!
    Call 2 of 'hello'
    Hello Redowan!
```

The __init__() method stores a reference to the function num_calls and can do other necessary initialization. The __call__() method will be called instead of the decorated function. It does essentially the same thing as the wrapper() function in our earlier examples. Note that you need to use the functools.update_wrapper() function instead of @functools.wraps.

## More Decorator Examples

### Caching Return Values

Decorators can provide an elegant way of memoizing function return values. Imagine you have an expensive API and you'd like call that as few times as possible. The idea is to save and cache values returned by the API for particular arguments, so that if those arguments appear again, you can serve the results from the cache instead of calling the API again. This can dramatically improve your applications' performance. Here I've simulated an expensive API call and provided caching with a decorator.

```python
import functools
import time


def api(a):
    """API takes an integer and returns the square value of it.
    To simulate a time consuming process, I've added some time delay to it."""

    print("The API has been called...")

    # This will delay 3 seconds
    time.sleep(3)

    return a * a


api(3)
```

```
>>> The API has been called...
    9
```

You'll see that running this function takes roughly 3 seconds. To cache the result , we can use Python's built in functools.lru_cache to save the result against an argument in a dictionary and serve that when it encounters the same argument again. The only drawback here is, all the arguments need to be hashable.

```python
import functools


@functools.lru_cache(maxsize=32)
def api(a):
    """API takes an integer and returns the square value of it.
    To simulate a time consuming process, I've added some time delay to it."""

    print("The API has been called...")

    # This will delay 3 seconds
    time.sleep(3)

    return a * a


api(3)
```

```
>>> 9
```

Least Recently Used (LRU) Cache organizes items in order of use, allowing you to quickly identify which item hasn't been used for the longest amount of time. In the above case, the parameter `max_size` refers to the maximum numbers of responses to be saved up before it starts deleting the earliest ones. While you run the decorated function, you'll see first time it'll take roughly 3 seconds to return the result. But if you rerun the function again with the same parameter it'll spit the result from the cache almost instantly.

## Unit Conversion

The following decorator converts length from SI units to multiple other units without polluting your target function with conversion logics.

```python
import functools


def convert(func=None, convert_to=None):
    """This converts value from meter to others."""

    if func is None:
        return functools.partial(convert, convert_to=convert_to)

    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Conversion unit: {convert_to}")

        val = func(*args, **kwargs)

        # Adding conversion rules
        if convert_to is None:
            return val

        elif convert_to == "km":
            return val / 1000

        elif convert_to == "mile":
            return val * 0.000621371

        elif convert_to == "cm":
            return val * 100

        elif convert_to == "mm":
            return val * 1000

        else:
            raise ValueError("Conversion unit is not supported.")

    return wrapper
```

Let's use that on a function that returns the area of a rectangle.

```python
@convert(convert_to="mile")
def area(a, b):
    return a * b


area(1, 2)
```

```
>>> Conversion unit: mile
    0.001242742
```

Using the convert decorator on the area function shows how it prints out the transformation unit before returning the desired result. Experiment with other conversion units and see what happens.

### Function Registration

The following is an example of registering logger function in Flask framework. The decorator `register_logger` doesn't make any change to the decorated `logger` function. Rather it takes the function and registers it in a list called `logger_list` every time it's invoked.

```python
from flask import Flask, request

app = Flask(__name__)
logger_list = []


def register_logger(func):
    logger_list.append(func)
    return func


def run_loggers(request):
    for logger in logger_list:
        logger(request)


@register_logger
def logger(request):
    print(request.method, request.path)


@app.route("/")
def index():
    run_loggers(request)
    return "Hello World!"


if __name__ == "__main__":
    app.run(host="localhost", port="5000")
```

If you run the server and hit the `http://localhost:5000/` url, it'll greet you with a `Hello World!` message. Also you'll able to see the printed `method` and `path` of your http request on the terminal. Moreover, if you inspect the `logger_list`, you'll find the registered logger there. You'll find a lot more real life usage of decorators in the Flask framework.

## Remarks

All the pieces of codes in the blog were written and tested with python 3.8 on a machine running Ubuntu 18.04.

## References

```python
* [Primer on Python Decorator - Real Python](https://realpython.com/primer-on-python-decorators/)
* [Decorators in Python - DataCamp](https://www.datacamp.com/community/tutorials/decorators-python)
* [5 Reasons You Need to Write Python Decorators](https://www.oreilly.com/content/5-reasons-you-need-to-learn-to-write-python-decorators/)
```
