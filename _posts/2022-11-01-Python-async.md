# Understanding how python callstacks work in async

## Generator points

* Generators are functions that `yield`, or generator expressions.
* Their execution works just like a normal function, except that they can `yield` mid-call.
	* In this case, the stack (ie. current state of any local vars & exception handling) is preserved
	* When the generator is resumed, that stack is restored and execution resumes after the `yield`.
	* Resumption takes one of three forms:
		* The caller calls `__next__` on the generator: resumes after the yield; the yield evaluates to `None`.
		* The caller calls `send(val)` on the generator: resumes after the yield; the yield evaluates to `val`
		* The caller calls `throw(E)` on the generator: acts as if the yield threw the provided exception. If the generator function doesn't catch the exception, or if it throws a different one, then that exception will be propagated to the caller.

### `Send` turns generators into consumers

Because `send` causes 'yield' to return a value, you can write generators to act as consumers in a pipeline, for example. You write the generator as an infinite loop where it does a `yield` at the start of that loop to receive a value. It then uses that value (processes it in some way) before passing it on to another generator that was configured as the destination.

In the calling function, you create the generator and advance it to its first yield by calling `next` once. Then, you generate values that need to be sent down the pipeline and call `send` on the generator to pass them along. In [PEP-0342](https://peps.python.org/pep-0342/) there is a decorator that supports this pattern:

```python
def consumer(func):
    def wrapper(*args,**kw):
        gen = func(*args, **kw)
        gen.next()
        return gen
    wrapper.__name__ = func.__name__
    wrapper.__dict__ = func.__dict__
    wrapper.__doc__  = func.__doc__
    return wrapper
```

## Basics

* Declare a function as a coroutine with async def.
* Start the main async event loop by executing a coroutine with 

```python
asyncio.run(myfunc(a, b))
```

* From within a coroutine, you can call other coroutines.
* Awaitables are things you can block the current coroutine on (ie. schedule a new task and cede the thread back to the event loop). 
* You can wait for awaitables to finish with `await`, which cedes the thread from the current coroutine back to the event loop. 
* Types of awaitable:
  * Call coroutines directly: await my_func(a,b)
  * asyncio.create_task(my_func(a, b)) returns a Task object wich is awaitable, but also cancellable. (`mytask.cancel()`: cancels the next time the event loop gets control)
  * Futures, which aren't used in app code generally.
* Awaitables can be awaited (sequential), or asyncio.gather(list_of_awaitables]) (simultaneous)
* asyncio.wait_for(awaitable, timeout) will cancel an awaitable after timeout time

## Async generator function

* Any function declared as `async def`, and has `yield` statements in it.
* Used to produce values for an `async for` loop.
	* `async for` does an `await` on every next (or `__anext__` on the async iterator)
	* The upshot is that the generator function's calculation of the next value will be asynchronous, but the calling coroutine will cede until it's ready

## Async with

* Used with an async context manager
* Just like a normal context manager, except that the __enter__ and __exit__ are coroutines, and the `async with` will `await` them.
* So, the activities in the enter and exit can be done asynchronously, with the calling function ceding until they're done. 