---
title: "Reading CPython Source (5): Coroutines"
date: 2021-08-04T17:38:52+08:00
draft: false
categories: ["python"]
description: "A translated technical note on Reading CPython Source (5): Coroutines, preserving the examples and context of the original article."
---
# Reading CPython Source (5): Coroutines

> Originally published in Chinese on 2021-08-04; this English edition preserves the original scope and technical context.

**Coroutines** are lightweight threads of user space, they can pause or resume at specific positions within a function, and the caller can retrieve the state from or pass the state to a coroutine. A typical application of coroutines in Python is the **generator**, and this article analyzes the implementation of generators in Python.

## 1 Generators

If a function in Python contains the `yield` keyword, it does not run to a `return` statement and return a variable like a regular function does; instead, it immediately returns a generator object. Let's take a Fibonacci sequence generator function as an example:
```python
def FibonacciSequenceGenerator():
    a, b = 0, 1
    while True:
        yield a + b
        a, b = b, a + b

if __name__ == "__main__":
    fsg = FibonacciSequenceGenerator()
    print(fsg)
    print(type(fsg))
```


```shell
$ python3 main.py
<generator object FibonacciSequenceGenerator at 0x7fb4720b1ac0>
<class 'generator'>
```
One can see that the function `FibonacciSequenceGenerator` returns a `generator` object `f`; unlike regular functions, generator objects cannot be called directly; instead, we use `next()` or `fsg.send()` to switch the execution of the generator function, starting or resuming its execution until it reaches a `yield` statement or the end of the function, at which point control is returned to the caller.
```python
    for i in range(100):
        print(next(fsg))
```


```shell
$ python3 main.py
1
2
3
5
# ...
218922995834555169026
354224848179261915075
573147844013817084101
```
Generator behavior is very similar to thread switching, involving steps of execution, context saving, and restoring. Using generators to simulate thread behavior can avoid the switch from user mode to kernel mode, thus improving efficiency.

## 2 Threads

### 2.1 Generator Objects

Through the `tp_name` variable of the type object, we can find that the generator `generator` object corresponds to the structure `PyGenObject` in the source code, whose type object is `PyGen_Type`:
```c++
// Inlcude/genobject.h

/* _PyGenObject_HEAD defines the initial segment of generator
   and coroutine objects. */
#define _PyGenObject_HEAD(prefix)                                           \
    PyObject_HEAD                                                           \
    /* Note: gi_frame can be NULL if the generator is "finished" */         \
    PyFrameObject *prefix##_frame;                                          \
    /* True if generator is being executed. */                              \
    char prefix##_running;                                                  \
    /* The code object backing the generator */                             \
    PyObject *prefix##_code;                                                \
    /* List of weak reference. */                                           \
    PyObject *prefix##_weakreflist;                                         \
    /* Name of the generator. */                                            \
    PyObject *prefix##_name;                                                \
    /* Qualified name of the generator. */                                  \
    PyObject *prefix##_qualname;                                            \
    _PyErr_StackItem prefix##_exc_state;

typedef struct {
    /* The gi_ prefix is intended to remind of generator-iterator. */
    _PyGenObject_HEAD(gi)
} PyGenObject;

// Inlcude/genobject.c


PyTypeObject PyGen_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "generator",                                /* tp_name */
    sizeof(PyGenObject),                        /* tp_basicsize */
    0,                                          /* tp_itemsize */
    /* methods */
    (destructor)gen_dealloc,                    /* tp_dealloc */
    // ...
    (iternextfunc)gen_iternext,                 /* tp_iternext */
    _PyGen_Finalize,                            /* tp_finalize */
};

```
`PyGenObject` structure contains few members, which are:

- The common header `PyObject_HEAD`, including reference count `ob_refcnt` and type object pointer `ob_type`;
- A flag indicating whether the generator is currently running `gi_running`;
- The stack frame object dependent on the generator's execution `gi_frame`;
- The code object corresponding to the generator `gi_code`;
- The list of weak references to the generator `gi_weakreflist`;
- The generator's name `gi_name` and `gi_qualname`;
- The execution state of the generator `gi_exec_state`;

A generator is not executed immediately upon creation, and the last instruction executed in its stack frame is empty.
```python
    print(fsg.gi_running)
    print(fsg.gi_frame.f_lasti)
```
```shell
$ python3 main.py
False
-1
```
Of course, stack frame objects are also associated with code objects:
```python
    print(fsg.gi_code)
    print(fsg.gi_frame)
    print(fsg.gi_frame.f_code)
```
```shell
$ python3 main.py
<code object FibonacciSequenceGenerator at 0x7f8283e49c90, file "main.py", line 89>
<frame at 0x7f8283f17610, file 'main.py', line 89, code FibonacciSequenceGenerator>
<code object FibonacciSequenceGenerator at 0x7f8283e49c90, file "main.py", line 89>
```
### 2.2 Execution

#### next

`next()` is a built-in function in Python, used to drive the execution of a generator, or switch the stack of the current function to the generator; its source code is as follows:
```cpp
// Python/bltinmodule.c
builtin_next(PyObject *self, PyObject *const *args, Py_ssize_t nargs)
{
    PyObject *it, *res;

    if (!_PyArg_CheckPositional("next", nargs, 1, 2))
        return NULL;

    it = args[0];
    if (!PyIter_Check(it)) {
        PyErr_Format(PyExc_TypeError,
            "'%.200s' object is not an iterator",
            Py_TYPE(it)->tp_name);
        return NULL;
    }

    res = (*Py_TYPE(it)->tp_iternext)(it);
    if (res != NULL) {
        return res;
    } else if (nargs > 1) {
        PyObject *def = args[1];
        if (PyErr_Occurred()) {
            if(!PyErr_ExceptionMatches(PyExc_StopIteration))
                return NULL;
            PyErr_Clear();
        }
        Py_INCREF(def);
        return def;
    } else if (PyErr_Occurred()) {
        return NULL;
    } else {
        PyErr_SetNone(PyExc_StopIteration);
        return NULL;
    }
}

```
The core part is `res = (*it->ob_type->tp_iternext)(it)`, which retrieves the type object of the parameter `args[0]` and calls its `tp_iternext` function pointer; then it checks the returned result. When we call the `next` function on the generator object `fsg`, it actually calls the `gen_iternext` function of `PyGen_Type`, which in turn calls `gen_send_ex`.
```cpp
// Objects/genobject.c
static PyObject *
gen_iternext(PyGenObject *gen)
{
    return gen_send_ex(gen, NULL, 0, 0);
}

static PyObject *
gen_send_ex(PyGenObject *gen, PyObject *arg, int exc, int closing)
{
    PyFrameObject *f = gen->gi_frame;
    // ...

    /* Generators always return to their most recent caller, not
     * necessarily their creator. */
    Py_XINCREF(tstate->frame);
    assert(f->f_back == NULL);
    f->f_back = tstate->frame;

    gen->gi_running = 1;
    gen->gi_exc_state.previous_item = tstate->exc_info;
    tstate->exc_info = &gen->gi_exc_state;

    if (exc) {
        assert(_PyErr_Occurred(tstate));
        _PyErr_ChainStackItem(NULL);
    }

    result = _PyEval_EvalFrame(tstate, f, exc);
    tstate->exc_info = gen->gi_exc_state.previous_item;
    gen->gi_exc_state.previous_item = NULL;
    gen->gi_running = 0;

    // ...
}
```
python
def gen_send_ex():
    # Hang the generator object's frame to the current frame
    f->f_back = tstate->frame;
    # Modify the generator object's running state
    gen->gi_running = 1;
    # Execute the generator frame through _PyEval_EvalFrame
    _PyEval_EvalFrame(f, tstate, 0);

About the flow of `_PyEval_EvalFrame` function, it has been discussed in the previous text. In simple terms, it executes bytecode corresponding to the generator object by continuously executing through a loop and a `switch-case` structure.

#### send

And similar to the `next` function, the `send` function can also be used to advance a generator. In terms of invocation, the only difference between them is that the `send` function passes an additional parameter. Upon examining the source code of the `send` function, it can be seen that it similarly calls the `gen_send_ex` function, but this time the second parameter is a pointer to an object (as opposed to a `NULL` in the `gen_iternext` function).
```cpp
// Objects/genobject.c
static PyMethodDef gen_methods[] = {
    {"send",(PyCFunction)_PyGen_Send, METH_O, send_doc},
    // ...
};

PyObject *
_PyGen_Send(PyGenObject *gen, PyObject *arg)
{
    return gen_send_ex(gen, arg, 0, 0);
}
```
View the differences in the byte code instructions corresponding to both:
```python
import os, sys

def FibonacciSequenceGenerator():
    a, b = 0, 1
    while True:
        yield a + b
        a, b = b, a + b

def DriveCo(co):
    r = next(fsg)
    r = fsg.send(1)

if __name__ == "__main__":
    fsg = FibonacciSequenceGenerator()
    DriveCo(fsg)
    import dis
    dis.dis(DriveCo)
```
```shell
$ python3 main.py
 10           0 LOAD_GLOBAL              0 (next)
              2 LOAD_GLOBAL              1 (fsg)
              4 CALL_FUNCTION            1
              6 STORE_FAST               1 (r)

 11           8 LOAD_GLOBAL              1 (fsg)
             10 LOAD_METHOD              2 (send)
             12 LOAD_CONST               1 (1)
             14 CALL_METHOD              1
             16 STORE_FAST               1 (r)
             18 LOAD_CONST               0 (None)
             20 RETURN_VALUE
```
One can see that compared to the bytecode instructions of the `next` function, the `send` function adds a `LOAD_CONST` instruction before the `CALL_FUNCTION`/`CALL_METHOD` to push the parameters onto the stack.

### 2.3 Pause

In Python, one can use the built-in `yield` function to pause the execution of a generator object and return a value. We can investigate the `FibonacciSequenceGenerator` function and the corresponding bytecode instructions for the `yield` statement:
```python
    import dis
    dis.dis(fsg)
```
```shell
$ python3 main.py
  2           0 LOAD_CONST               1 ((0, 1))
              2 UNPACK_SEQUENCE          2
              4 STORE_FAST               0 (a)
              6 STORE_FAST               1 (b)

  4     >>    8 LOAD_FAST                0 (a)
             10 LOAD_FAST                1 (b)
             12 BINARY_ADD
             14 YIELD_VALUE
             16 POP_TOP

  5          18 LOAD_FAST                1 (b)
             20 LOAD_FAST                0 (a)
             22 LOAD_FAST                1 (b)
             24 BINARY_ADD
             26 ROT_TWO
             28 STORE_FAST               0 (a)
             30 STORE_FAST               1 (b)
             32 JUMP_ABSOLUTE            8
             34 LOAD_CONST               0 (None)
             36 RETURN_VALUE
```
Where the assignment operation on variables `a, b` occurs on line 2 (`a, b = 0, 1`) and line 5 (`a, b = b, a + b`), respectively. On line 4, the execution of `yield a + b` involves saving the newly computed value onto the stack via `LOAD_FAST` and `BINARY_ADD` instructions before executing `YIELD_VALUE` and `POP_TOP` instructions. Let's examine the source code for these instructions:
```cpp
// Python/ceval.c

PyObject* _Py_HOT_FUNCTION
_PyEval_EvalFrameDefault(PyThreadState *tstate, PyFrameObject *f, int throwflag)
{
    // ...
        case TARGET(POP_TOP): {
            PyObject *value = POP();
            Py_DECREF(value);
            FAST_DISPATCH();
        }
    // ...
        case TARGET(YIELD_VALUE): {
            retval = POP();

            if (co->co_flags & CO_ASYNC_GENERATOR) {
                PyObject *w = _PyAsyncGenValueWrapperNew(retval);
                Py_DECREF(retval);
                if (w == NULL) {
                    retval = NULL;
                    goto error;
                }
                retval = w;
            }

            f->f_stacktop = stack_pointer;
            goto exiting;
        }
    // ...
    return _Py_CheckFunctionResult(tstate, NULL, retval, __func__);
}
```
One can see that the `YIELD_VALUE` instruction is quite simple. It first extracts the top data on the stack as the return value of the `yield` statement. Then, it modifies the pointer that points to the top of the current stack frame to point to the next frame. Finally, it ends execution with a `goto` statement. Subsequently, the `_PyEval_EvalFrameDefault` function removes the stack frame of the generator object from the current frame "chain" and returns `retval`.

## 3 Summary

This shows that Python's generators fully meet the definition of coroutines (which can be interrupted and resumed in user mode); additionally, since the Python virtual machine is based on a stack (Stack-Based), the implementation of generator objects at the bytecode level is very concise. Coroutines can be driven on top of threads using an event loop based on IO multiplexing like epoll to execute different subroutines and contexts, creating a model that appears blocking but is actually asynchronous, suitable for IO-intensive scenarios.
