# Context

[TOC]

## Context

A [context_base.Context]() is an abstract interface that defines the API for
constructing a context or environment in TFF that can **construct**,
**compile**, and **execute** computations.

This API defines a **low-level abstraction** that should be used when an
[executor_base.Executor]() is **not** used for execution; the
[ReferenceExecutor]() backend integrates at this level.

### ExecutionContext

An [execution_context.ExecutionContext]() is a [context_base.Context]() that
compiles computations using a compilation function and executes computations
using an [executor_base.Executor]().

This API defines a **high-level abstraction** that should be used when an
[executor_base.Executor]() is used for execution; the [native]() and [IREE]()
backends integrate at this level.

### FederatedComputationContext

A [federated_computation_context.FederatedComputationContext]() is a
[context_base.Context]() that constructs federated computations. This context is
used trace Python functions decorated with [federated_computation]().

### TensorFlowComputationContext

A [tf_computation_context.TensorFlowComputationContext]() is a
[context_base.Context]() that constructs TensorFlow computations. This context
is used to serialize Python functions decorated with [tf_computation]().

## ContextStack

A [context_stack_base.ContextStack]() is an abstract interface that defines the
API for interacting with a stack of [context_base.Context]() instances.

In TFF, you can set the context that TFF will use to interact with computations
(i.e. construct, compile, and execute) by:

*   Invoking [tff.framework.set_default_context]() to set the default context.
    This API is often used to install a context that will compile or execute a
    computaiton.

*   Invoking [tff.framework.get_context_stack]() to get the current context
    stack and then invoking [context_stack_base.install]() to temporarily
    install a given context onto the top of the stack. For example, the
    [federated_computation]() and [tf_computation]() decorators push the
    corresponding contexts onto the current context stack while the function is
    being traced.

### ContextStackImpl

A [context_stack_impl.ContextStackImpl]() is a
[context_stack_base.ContextStack]() that is implemented as a common thread-local
stack.
