# Tracing

[TOC]

Tracing the the process of constructing a TFF AST from a Python function.

TODO(b/153500547): Describe and link the individual components of the tracing
system.

## Tracing a Federated Computation

At a high level, there are three components to tracing a Federated computation.

### Packing the arguments

The arguments provided to the [@tff.federated_computation]() decorator describe
type signature of the TFF AST to construct. TFF uses this information to to
determine how to pack the arguments of the Python function into a single
argument, an [Struct](). This argument is used to construct a new Python
function which declares exactly 0 or 1 parameters, often referred to as
`zero_or_one_arg_fn`.

Note: Using [Struct]() as a single data structure to represent both Python
`args` and `kwargs` is the reason that [Struct]() accepts both named and unnamed
fields.

See [function_utils.wrap_as_zero_or_one_arg_callable]() for more information.

### Tracing the function

The body of this new Python function, i.e. `zero_or_one_arg_fn`, is traced using
the Python dunder methods described by the
[Value](https://github.com/tensorflow/federated/tensorflow_federated/python/core/api/value_base.py)
interface. This is accomplished (in the case there is exactly one argument) by:

First, constructing a [ValueImpl]() backed by a [building_blocks.Reference]()
with appropriate type signature to represent the argument.

Then, invoking the function on the [ValueImpl](). This causes the Python runtime
to invoke the dunder methods implemented by [ValueImpl](), which translates
those dunder methods as TFF AST construction. Each dunder method constructs a
TFF AST and returns a [ValueImpl]() backed by that TFF AST.

For example:

```python
def foo(x):
  return x[0]
```

Here the function’s parameter is a tuple and in the body of the fuction the 0th
element is selected. This invokes Python’s `__getitem__` method, which is
overridden by [ValueImpl](). In the simplest case, the implementation of
`ValueImpl.__getitem__` constructs a [building_blocks.Selection]() to represent
the invocation of `__getitem__` and returns a [ValueImpl]() backed by this new
[building_blocks.Selection]().

Tracing continues because each dunder methods return a [ValueImpl](), stamping
out every operation in the body of the function which causes one of the
overriden dunder methods to be invoked.

### Constructing the TFF AST

The result of tracing the function is packaged into a [building_blocks.Lambda]()
whose `parameter_name` and `parameter_type` map to the
[building_block.Reference]() crated to represent the packed arguments. The
resulting [building_blocks.Lambda]() is then returned as a Python object that
fully represents the user’s Python function.

## Tracing a TensorFlow Computation

TODO(b/153500547): Describe the process of tracing a TensorFlow compuation.
