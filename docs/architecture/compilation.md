# Compilation

[TOC]

The
[compiler](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/compiler)
package contains the data structures defining Python representation of the TFF
AST as well as the core transformation and compiler modules.

TODO(b/148163833): Some of the modules have not yet been moved from the
[impl](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl) package
to the
[compiler](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/compiler)
package.

## AST

An abstract syntax tree (AST) in TFF describes the structure of a federated
computation.

### Building Block

A
[building_block.ComputationBuildingBlock](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/compiler/building_blocks.py)
is an abstract interface that defines the Python representation of the TFF AST.

#### CompiledComputation

A
[building_block.CompiledComputation](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/compiler/building_blocks.py)
is a
[building_block.ComputationBuildingBlock](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/compiler/building_blocks.py)
that defines work that will be delegated to an external runtime. Currently TFF
only supports [TensorFlow Computations](#tensorFlow-computation).

### Computation

A
[pb.Computation](https://github.com/tensorflow/federated/tensorflow_federated/proto/v0/computation.proto)
that defines the Proto or serialized representation of the TFF AST.

#### TensorFlow Computation

A
[pb.Computation](https://github.com/tensorflow/federated/tensorflow_federated/proto/v0/computation.proto)
that defines work that will be delegated to the TensorFlow runtime in the Proto
or serialized representation of the TFF AST.

## Transformation

A transformation constructs a new object for the given input after applying a
collection of mutations. Transformations can operate on
[Building Blocks](#building-block) in order to transform a TFF AST or on
[TensorFlow Computations](#tensorFlow-computation) in order to transform a
`tf.Graph`.

An **atomic** transformation is one that applies a single mutation (possibly
more than once) to the given input.

A **composite** transformation is one that applies multiple transformations to
the given input in order to provide some feature or assertion.

Note: Transformations can be composed in serial or parallel, meaning that you
can construct a composite transformation that performs multiple transformations
in one pass through a TFF AST. However, the order in which you apply
transformations and how those transformations are parallelized is hard to reason
about; as a result, composite transformations are hand-crafted and most are
somewhat fragile.

The
[tree_transformations](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/compiler/tree_transformations.py)
module contains atomic [Building Block](#building-block) transformations.

The
[transformations](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/compiler/transformations.py)
module contains composite [Building Block](#building-block) transformations.

The
[tensorflow_computation_transformations](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/compiler/tensorflow_computation_transformations.py)
module contains atomic [TensorFlow Computation](#tensorflow-computation)
transformations.

The
[compiled_computation_transforms](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/compiler/compiled_computation_transforms.py)
module contains atomic and composite
[Compiled Computation](#compiled-computation) transformations.

TODO(b/148163833): Refactor the `compiled_computation_transforms` into
[Building Block](#building-block) transformations operating on a TFF AST
independently from the local execution platform and
[TensorFlow Computation](#tensorflow-computation) transformations operating on
TF Graphs.

The
[tree_to_cc_transformations](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/tree_to_cc_transformations.py)
module contains composite [Building Block](#building-block) transformations
representing the syntax-directed definition (SDD) logic expressed in
go/tff-to-tf-parser.

TODO(b/148163833): Rename the `tree_to_cc_transformations` module to something
more descriptive, possibly `sdd_transformations`.

The
[transformation_utils](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/compiler/transformation_utils.py)
module contains functions, traversal logic, and data structures used by other
transformation modules.

## Compiler

A compiler is a collection of [transformations](#transformations) that construct
a form that can be executed.

## CompilerPipeline

A
[compiler_pipeline.CompilerPipeline](https://github.com/tensorflow/federated/tensorflow_federated/python/core/impl/compiler/compiler_pipeline.py)
is a data structure that compiles computations and caches the results as the
computations are compiled. The performance of comipling a computation is
dependent on the complexity of the compilation function; the
`compiler_pipeline.CompilerPipeline` ensures that compiling the same computation
multiple times does not impact the performance of the system.
