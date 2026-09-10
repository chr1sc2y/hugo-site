---
title: "Reading CPython Source (1): Types and Objects"
date: 2021-03-14T16:05:52+08:00
draft: false
categories: ["python"]
description: "A translated technical note on Reading CPython Source (1): Types and Objects, preserving the examples and context of the original article."
---
# Reading CPython Source (1): Types and Objects

> Originally published in Chinese on 2021-03-14; this English edition preserves the original scope and technical context.

Python is an interpreted, dynamically typed, multi-paradigm programming language. When we download and install a version of Python from [python.org](https://www.python.org/), we are actually running the C language-compiled [CPython](https://github.com/python/cpython). In addition to CPython's runtime, there are also Jython, PyPy, Cython, and others; within the source code of CPython, there are a series of libraries, components, and tools.
```shell
$ git clone https://github.com/python/cpython
$ tree -d -L 2 .
.
`-- cpython
|-- Doc			# Document
    |-- Grammar
|-- Include 	# C Header
|-- Lib			# Python Library
|-- Mac			# For macOS Build
|-- Misc		# Misc
|-- Modules		# C Library
|-- Objects 	# Core types and definitions of the object model
|-- PC			# For Windows Build
|-- PCbuild 	# For older versions of Windows Build
|-- Parser		# Python Parser Source Code
|-- Programs	# Python Executables and other
|-- Python		# CPython Compiler Source Code
|-- Tools		# Build Tools
    `-- m4

16 directories
```
## Object Model

Python is an object-oriented language, and we can use the `type()` function in Python to view the class of an object:
```shell
>>> type(1)
<class 'int'>
>>> type(True)
<class 'bool'>
```
The integer object and boolean value object are of type `<class 'int'>` and `<class 'bool'>`, respectively.

While in Python, whether integers, boolean values, or basic data types, even custom classes, are all objects:
```python
class Foo:
    pass

print(type(int))
print(type(Foo))
```


```shell
<class 'type'>
<class 'type'>
```
One can see that the types of `int` and the custom class `Foo` are both `<class 'type'>`, instances of the `type` class; the `type` type is specifically used for defining types, also known as **meta-types**; in fact, the `type` itself is an object, and its class is also `type`.
```shell
>>> type(type)
<class 'type'>
```
**Simultaneously, all types in Python, whether `int`, `type`, or a custom class `Foo`, inherit from a base class called `object`. `object` is the endpoint of the inheritance chain:**

1. `int`
2. `type`
3. `Foo` (custom class)

All of these types inherit from `object`, which serves as the root of the inheritance hierarchy.
```shell
>>> int.__base__
<class 'object'>
>>> type.__base__
<class 'object'>
>>> print(Foo.__base__)
<class 'object'>
>>> print(object.__base__)
None
```
While the `object` base class is also an object of type `type`:
```shell
>>> type(object)
<class 'type'>
```
Above relationship can be expressed in the diagram as follows:

![process](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/python/type-0.png)

One can see that all types of base classes are `object`, and all types of types are `type`. This is the **object model** (object model) of Python, the content included in the source code under the Objects/ directory.

## Core Types and Objects

Although Python has many types at the syntax level (including `int`, `type`, `Foo`, etc.), they are actually struct objects at the source (C language) level.

### 2.1 Object

#### `PyObject`

In Python, all types are extended by the `PyObject` structure, which contains the following members:

1. `Py_ssize_t ob_refcnt` used to save the **reference count** of the object;
2. `PyTypeObject *ob_type` points to the **type object** of the object, used to identify the type of the object and store the **metadata** of the type;
3. `_PyObject_HEAD_EXTRA` macro represents two pointers to`PyObject*`, used to link all objects on the heap via a bidirectional linked list. These are only constructed when the `Py_TRACE_REFS` macro is enabled for debugging purposes.
```c
// Include/object.h

/* Define pointers to support a doubly-linked list of all live heap objects. */
#define _PyObject_HEAD_EXTRA            \
    struct _object *_ob_next;           \
    struct _object *_ob_prev;

/* Nothing is actually declared to be a PyObject, but every pointer to
 * a Python object can be cast to a PyObject*.  This is inheritance built
 * by hand.  Similarly every pointer to a variable-size Python object can,
 * in addition, be cast to PyVarObject*.
 */
typedef struct _object {
   _PyObject_HEAD_EXTRA	// Doubly linked list to track all objects on the heap, useful when Py_TRACE_REFS macro is enabled
    Py_ssize_t ob_refcnt;	// Reference count to facilitate garbage collection
    PyTypeObject *ob_type;	// Pointer to the type object of the current object, used to query the object's type
} PyObject;
```
#### PyVarObject

In Python, there exists the mutable-length `PyVarObject` object, which consists of a `PyObject` object and a variable `ob_size` that stores the length of the variable portion (number of elements).
```cpp
typedef struct {
    PyObject ob_base;
    Py_ssize_t ob_size; /* Number of items in variable part */
} PyVarObject;
```
`PyObject` and `PyVarObject` are typically included as headers in a variable structure, and the choice between the two depends on whether the size of the variable is fixed:
```cpp
// Include/object.h
/* PyObject_HEAD defines the initial segment of every PyObject. */
#define PyObject_HEAD          PyObject ob_base;

/* PyObject_VAR_HEAD defines the initial segment of all variable-size
 * container objects.  These end with a declaration of an array with 1
 * element, but enough space is malloc'ed so that the array actually
 * has room for ob_size elements.  Note that ob_size is an element count,
 * not necessarily a byte count.
 */
#define PyObject_VAR_HEAD      PyVarObject ob_base;
```
Python's typical variable-length object is the list, which is similar to `std::vector`. The list object has three member variables, including:

- The basic variable-length object `PyVarObject ob_base`, where `ob_base.ob_size` represents the current number of elements in the list;
- Pointer to the dynamic array `PyObject **ob_item`;
- Current capacity of the dynamic array `Py_ssize_t allocated`:
```cpp
// Inlucde/cpython/listobject.h

typedef struct {
    PyObject_VAR_HEAD
    /* Vector of pointers to list elements.  list[0] is ob_item[0], etc. */
    PyObject **ob_item;

    /* ob_item contains space for 'allocated' elements.  The number
     * currently in use is ob_size.
     * Invariants:
     *     0 <= ob_size <= allocated
     *     len(list) == ob_size
     *     ob_item == NULL implies ob_size == allocated == 0
     * list.sort() temporarily sets allocated to -1 to detect mutations.
     *
     * Items must normally not be NULL, except during construction when
     * the list is not yet visible outside the function that builds it.
     */
    Py_ssize_t allocated;
} PyListObject;
```
### 2.2 Types

#### PyTypeObject

In `PyObject`, `PyTypeObject *ob_type` points to the object's type—the representation of a class in Python. `PyTypeObject` determines the type of a `PyObject` and also carries extensive metadata.

1. `PyObject_VAR_HEAD` indicates that `PyTypeObject` itself is a **variable-length object**;
2. `const char *tp_name` represents the type's name;
3. `struct _typeobject *tp_base` is a pointer to the base type, storing inheritance information;
4. `Py_ssize_t tp_basicsize, tp_itemsize` represents the amount of memory allocated for creating an instance object;
5. `setattrfunc tp_setattr` sets values, `getattrfunc tp_getattr` gets values, `destructor tp_dealloc` is the destructor, and `hashfunc tp_hash` are function pointers indicating the standard operations supported by this type.
```c
// Include/object.h

/* PyTypeObject structure is defined in cpython/object.h.
   In Py_LIMITED_API, PyTypeObject is an opaque structure. */
typedef struct _typeobject PyTypeObject;

// Include/cpython/object.h
struct _typeobject {
PyObject_VAR_HEAD // PyVarObject ob_base;
    const char *tp_name; /* For printing, in format "<module>.<name>" */
    Py_ssize_t tp_basicsize, tp_itemsize; /* For allocation */

    /* Methods to implement standard operations */

    destructor tp_dealloc;
    Py_ssize_t tp_vectorcall_offset;
    getattrfunc tp_getattr;
    setattrfunc tp_setattr;
    PyAsyncMethods *tp_as_async; /* formerly known as tp_compare (Python 2)
                                    or tp_reserved (Python 3) */

    // Strong reference on a heap type, borrowed reference on a static type
    struct _typeobject *tp_base;

    /* More standard operations (here for binary compatibility) */
    // ...
};
```
In Python, each **type object** is **globally unique**. They exist as **global variables** in the source code, such as `int` type:
```cpp
// Objects/longobject.c
PyTypeObject PyLong_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "int",                                      /* tp_name */
    offsetof(PyLongObject, ob_digit),           /* tp_basicsize */
    sizeof(digit),                              /* tp_itemsize */
    0,                                          /* tp_dealloc */
    0,                                          /* tp_vectorcall_offset */
    0,                                          /* tp_getattr */
    0,                                          /* tp_setattr */
    0,                                          /* tp_as_async */
    long_to_decimal_string,                     /* tp_repr */
    &long_as_number,                            /* tp_as_number */
    // ...
};
```
`PyVarObject_HEAD_INIT` macro initializes `PyVarObject` members `ob_refcnt`, `ob_type`, and `ob_size`:
```cpp
#define PyObject_HEAD_INIT(type)        \
    { 1, type },

#define PyVarObject_HEAD_INIT(type, size)       \
    { PyObject_HEAD_INIT(type) size },

```
One can see that `ob_type` is initialized to `&PyType_Type` in `PyLong_Type`. This is the type of type used for defining type objects, or alternatively, the type of types or base types.

#### PyType_Type

Previously, we learned that the `type` of the `type` object is a class, and thus the `type` object itself is also a class object, which has a pointer to its `type` object `PyTypeObject *ob_type`; for the `type` object itself, its `type` object is based on a class object called `PyType_Type` (i.e., the "meta-type").
```cpp
// Objects/typeobject.c
PyTypeObject PyType_Type = {
   PyVarObject_HEAD_INIT(&PyType_Type, 0)		// PyType_Type initializes the pointer to itself during initialization, constructing a PyVarObject of type, where ob_base->ob_type = &PyType_Type, as per Appendix 1
    "type",                                     /* tp_name */
    sizeof(PyHeapTypeObject),                   /* tp_basicsize */
    sizeof(PyMemberDef),                        /* tp_itemsize */
    (destructor)type_dealloc,                   /* tp_dealloc */
    offsetof(PyTypeObject, tp_vectorcall),      /* tp_vectorcall_offset */
    // ...
};
```
In Python, **type objects** are defined under the `Objects/` directory. For example, the `bool` type:
```cpp
// Objects/boolobject.c

/* The type object for bool.  Note that this cannot be subclassed! */

PyTypeObject PyBool_Type = {
PyVarObject_HEAD_INIT(&PyType_Type, 0)	// PyType_Type, ob_base->ob_type = &PyType_Type
    "bool",										// tp_name = "bool"
    sizeof(struct _longobject),					/* tp_basicsize */
    0,											/* tp_itemsize */
    0,                                          /* tp_dealloc */
    0,                                          /* tp_vectorcall_offset */
    0,                                          /* tp_getattr */
    0,                                          /* tp_setattr */
    0,                                          /* tp_as_async */
    bool_repr,                                  /* tp_repr */
    &bool_as_number,                            /* tp_as_number */
    // ...
    0,                                          /* tp_base */
    // ...
};
```
In Python, both built-in types (such as `int`, `bool`, etc.) and user-defined types (such as `Foo`) are constructed through the `PyTypeObject` structure, and they always satisfy `ob_base->ob_type = &PyType_Type`.

Previously mentioned, `tp_base` is a pointer to the base class, storing inheritance information for a type. However, when defining `PyBool_Type`, it can be seen that `tp_base` is set to `PyBaseObject_Type` in the `PyType_Ready` function, even though it is explicitly `0` in `PyBool_Type` definition:
```cpp
// Objects/typeobject.c

int
PyType_Ready(PyTypeObject *type)
{
    PyTypeObject *base;
    // ...
	/* Initialize tp_base (defaults to BaseObject unless that's us) */
    base = type->tp_base;
    if (base == NULL && type != &PyBaseObject_Type) {
        base = &PyBaseObject_Type;
        if (type->tp_flags & Py_TPFLAGS_HEAPTYPE) {
            type->tp_base = (PyTypeObject*)Py_NewRef((PyObject*)base);
        }
        else {
            type->tp_base = base;
        }
    }
    // ...
}
```
Here, `PyTypeObject *base` is assigned to `PyBaseObject_Type`, which is the **base type** mentioned earlier.

#### PyBaseObject_Type

`PyBaseObject_Type` is defined in the `typeobject.c` file.
```cpp
// Objects/typeobject.c
PyTypeObject PyBaseObject_Type = {
PyVarObject_HEAD_INIT(&PyType_Type, 0)	// PyType_Type, ob_base->ob_type = &PyType_Type
    "object",                                   /* tp_name */
    sizeof(PyObject),                           /* tp_basicsize */
    0,                                          /* tp_itemsize */
    object_dealloc,                             /* tp_dealloc */
    // ...
    object_repr,                                /* tp_repr */
	// ...
    0,                                          /* tp_base */
    // ...
};
```
One can see that both `PyBool_Type`, `PyType_Type`, and `PyBaseObject_Type` share two common points:

They are defined with the same type of `PyVarObject_HEAD_INIT(&PyType_Type, 0)`, thus their types are `PyTypeObject`;
`tp_base` pointer is initialized to `NULL` when they point to the base;

Different places are, when assigning their base class pointer `tp_base` using the `PyType_Ready` function, only `PyBaseObject_Type.tp_base` is not assigned, while the others are assigned to `PyBaseObject_Type`. This confirms that `PyBaseObject_Type` is the end of the inheritance chain.

We can organize the above relationships into a graph:

![process](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/python/type-1.png)

## Appendix

### 1 PyType_Type Constructor
```cpp
struct Bar;

struct Foo
{
	Bar* p_b;
	int ref_count;
};

struct Bar
{
	Foo f;
	const char* name;
};

Bar b
{
	Foo{ &b, 1 },
	"ClassBar"
};

int main()
{
	printf("%s\n", b.f.p_b->name);
	return 0;
}
```
```shell
$ ./test
ClassBar
```
