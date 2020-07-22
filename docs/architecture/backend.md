# Backend

[TOC]

A backend is the composition of a compiler and a runtime in some context
required to evaluate a computation.

The
[backends](https://github.com/tensorflow/federated/tensorflow_federated/python/core/backends)
package contains backends which may require extending the TFF compiler and
runtime. These extensions can be found in the corresponding backend.

If the runtime of a backend is implemented as an [execution stack](), then the
backend can construct [ExecutionContext]() to provide TFF with a context in
which to evaluate a computation. In this case, the backend can be thought of as
integrating with TFF at a higher-level of abstraction. However, if the runtime
is *not* implemented as an [execution stack](), then the backend will need to
construct a [Context]() and can be thought of as integrating with TFF at a
lower-level of abstraction.

```dot
<!--#include file="backend.dot"-->
```

The **blue** nodes are provided by TFF [core]().

The **green**, **red**, **yellow**, and **purple** nodes are provided by the
[native](), [mapreduce](), [iree](), and [reference]() backends respectively.

The **dashed** nodes are provided by an external system.

The **solid** arrows indicate relationship and the **dashed** arrows indicate
inheritance.

## Native

The
[native](https://github.com/tensorflow/federated/tensorflow_federated/python/core/backends/native)
backend composes of the TFF compiler and runtime in order execute computations
in a way that is reasonably efficiant and debuggable.

### Native Form

A native form is a TFF AST that is topologically sorted into a directed acyclic
graph (DAG) of TFF intrinsics with some optimizations to the dependency of those
intrinsics.

### Compiler

The [compiler.transform_to_native_form]() function compiles a computation into a
native form.

### Runtime

The native backend does not contain backend specific extentions to the TFF
runtime, instead an [execution stack](execution.md#execution-stack) can be used
directly.

### Context

A native context is an [execution_context.ExecutionContext]() constructed with a
native compiler (or no compiler) and a TFF runtime:

```python
executor = eager_tf_executor.EagerTFExecutor()
factory = executor_factory.create_executor_factory(lambda _: executor)
context = execution_context.ExecutionContext(
    executor_fn=factory,
    compiler_fn=None)
set_default_context.set_default_context(context)
```

However, there are some common configurations:

The [execution_context.set_local_execution_context]() function constructs an
[execution_context.ExecutionContext]() with a native compiler and a
[local execution stack](execution.md#local-execution-stack).

## MapReduce

The
[mapreduce](https://github.com/tensorflow/federated/tensorflow_federated/python/core/backends/mapreduce)
backend contains the data structures and compiler required to construct a form
that can be executed on MapReduce-like runtimes.

### CanonicalForm

A
[canonical_form.CanonicalForm](https://github.com/tensorflow/federated/tensorflow_federated/python/core/backends/mapreduce/canonical_form.py)
is a data structure defining the representation of logic that can be executed on
MapReduce-like runtimes. This logic is organized as a collection of TensorFlow
functions, see the [canonical_form]() module for more information about the
nature of these functions.

### Compiler

The
[transformations](https://github.com/tensorflow/federated/tensorflow_federated/python/core/backends/mapreduce/transformations.py)
module contains [Building Block](compilation.md#building-block) and
[TensorFlow Computation](compilation.md#tensorflow-computation) transformations
required to compile a computation to a
[canonical_form.CanonicalForm](https://github.com/tensorflow/federated/tensorflow_federated/python/core/backends/mapreduce/canonical_form.py).

The
[canonical_form_utils](https://github.com/tensorflow/federated/tensorflow_federated/python/core/backends/mapreduce/canonical_form_utils.py)
module contains the compiler for the MapReduce backend and constructs a
[canonical_form.CanonicalForm](https://github.com/tensorflow/federated/tensorflow_federated/python/core/backends/mapreduce/canonical_form.py).

### Runtime

A MapReduce runtime is not provided by TFF, instead this should be provided by
an external MapReduce-like system.

### Context

A MapReduce context is not provided by TFF.

## IREE

[IREE](https://github.com/google/iree) is an experimental compiler backend for
[MLIR](https://mlir.llvm.org/).

The
[iree](https://github.com/tensorflow/federated/tensorflow_federated/python/core/backends/iree)
backend contains the data structures, compiler, and executor required to
construct a form that can be executed on an IREE runtime.

### Compiler

The
[compiler](https://github.com/tensorflow/federated/tensorflow_federated/python/core/backends/iree/compiler.py)
module contains transformations required to comiple a computation to a form that
can be exected using an
[executor.IreeExecutor](https://github.com/tensorflow/federated/tensorflow_federated/python/core/backends/iree/executor.py).

### Runtime

The
[executor.IreeExecutor](https://github.com/tensorflow/federated/tensorflow_federated/python/core/backends/iree/executor.py)
is an [executor_base.Executor]() that executes computations by delegating to an
IREE runtime. This executor can be composed with [executor_base.Executor]()s
from the TFF runtime in order to construct an [execution stack]() representing a
runtime.

### Context

An iree context is [execution_context.ExecutionContext]() constructed with an
iree compiler and an [execution stack]() with an [executor.IreeExecutor]
delegating to an IREE runtime.

## Reference

A [reference_executor.ReferenceExecutor]() is a [context_base.Context]() that
compiles and executes computations. Note that the
[reference_executor.ReferenceExecutor]() does not inherit from
[execution_context.ExecutionContext]() and the runtime is not implemented using
an [execution stack](); instead the compiler and runtime are trivially
implemented inline in the context. As a result, the
[reference_executor.ReferenceExecutor]() is more accurately a "context" than an
"executor".

TODO(b/148163833): Rename the `ReferenceExecutor` module to something more
descriptive, possibly `DebugContext`.
