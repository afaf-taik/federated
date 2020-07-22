# Execution

[TOC]

The
[executors](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/executors)
package contains the core executors and runtime modules.

## Runtime

A runtime is the system that executes a computation.

### TFF Runtime

A TFF runtime typically handles federated intrinsics and delegates mathematical
operations to an external runtime.

### External Runtime

An external runtime is any system that the TFF runtime delegates execution to.

#### TensorFlow

TensorFlow is an open source platform for machine learning. Today the TFF
runtime delegates mathematical operations to TensorFlow using an
[eager_tf_executor.EagerTFExecutor]().

## Executor

An
[executor_base.Executor](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/executors/executor_base.py)
is an abstract interface that defines an API for executing computations. The
[executors](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/executors)
package contains a collection of concrete implementations of this interface that
can be composed into a hierarchy, referred to as an
[execution stack](#execution-stack).

## ExecutorFactory

An
[executor_factory.ExecutorFactory](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/executors/executor_factory.py)
is an abstract interface that defines an API for constructing an
[executor_base.Executor](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/executors/executor_base.py).
These factories construct the executor lazily and manage the lifecycle of the
executor; the motivation to lazily constructing executors is to infer the number
of clients from the computation at execution time.

## Execution Stack

An execution stack is a hierarchy of
[executor_base.Executor](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/executors/executor_base.py)
instances. The
[executor_stacks](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/executors/executor_stacks.py)
module contains logic for constructing and composing specific execution stacks.

### Local Execution Stack

The [executor_stacks.local_executor_factory]() function constructs an execution
stack that executes a computation on some number of clients.

### Remove Execution Stack

The [executor_stacks.worker_pool_executor_factory]() function constructs an
execution stack that executes a computation on some service.
