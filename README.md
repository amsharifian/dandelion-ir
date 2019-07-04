# IR-Lib

IR-lib is a library of hardware components for auto generating highly configurable parallel dataflow accelerator.
IR-lib provides the implementation of the following hardware units:

1. a set of highly configurable and parameterizable computation nodes.
2. a set of control units to support arbitrary control path.
3. A collection configurable Memory structures like Cache, Scratchpad memory, and such.
4. A set of standard flexible set of junctions and interfaces to connect different pieces of the design.


**WARNING:** These units are works in progress. They may not be yet completely free of bugs, nor are they fully optimized.

## What's in the IR-Lib repository?

The IR-Lib repository contains code that is used to implement library modular hardware components to build hardware accelerators. Hardware generation is done using Chisel, a hardware construction language embedded in Scala.
The IR-Lib code base is itself factored into several Scala packages. These packages are all found within the src/main/scala directory. Some of these packages provide Scala utilities for generator configuration, while others contain the actual Chisel RTL generators themselves. Here is a brief description of what can be found in each package:

**Please find each package under src/main/scala folder**

### accel:

This RTL package contains all the accelerator code used to wrap a dataflow scala file. The top level file is *Accelerator.scala*. It instantiates and connects three helper blocks: *SimpleReg.scala*, *Cache.scala* and *Core.scala*.

To read more about IR-lib accelerator design and SoC interface, please read the following document:

* [Accelerator Interface](./doc/Soc-Interface.md)

![Accelerator](https://www.dropbox.com/s/q600al5gt2yo91i/accelerator-resize.png?raw=1)


### node:

This RTL package resembles LLVM IR instruction nodes in our design so that we can target arbitrary dataflow of computation nodes from our input software IR. All these nodes use *Handhsaking* interface from our *interface* package to talk with other nodes. The logic within each node can vary from a simple ALU operation like addition to more complicated operations like memory address calculation and so on.

To read more about IR-lib node's design, please read the following document:

* [IR Node design](./doc/Node.md)

### control:

In this package, we have implemented all of our control logic to support arbitrary dataflow between IR-lib's nodes.

* [Controlflow](./doc/Control-flow.md)


### loop:

This RTL package contains the implementation of our special *loop* nodes for controlling repetitive execution of the same dataflow. Under this package, we have different implementation of the notion of software loops. The loop structure not only should capture all the live-ins and live-outs to the loop but also it has been able to have support for a different type of loops like *serial*, *parallel* and different patterns.

* [Parallel Loop](./doc/Parallel-loop.md)
* [Nested Parallelism](./doc/Nested-loop.md)


### arbiters:

This RTL package contains a parametrizable set of arbiter implementation that's been used in other packages like *memory* or *junctions*.


### concurrent:

This RTL package contains the implementation of our concurrent modules to support higher task level parallelism. For example, our task manager implementation exists under this package. Different implementation of task controller can be found under this package.


### config:

This utility package provides Scala interfaces for configuring a generator via a dynamically-scoped parameterization library.


### dataflow:

This RTL package contains different small dataflow to test the correctness of IR-lib accelerator's design.

### dnn:

This RTL package contains our computation nodes to implement application-specific accelerators for *dnn workloads*. For instance, we have different implementation of *Systolic arraies* in this branch. Another type of computation nodes which exist in this package is our *Typed node* computation nodes.
### FPU:

This RTL package provides wrappers around Floating point operations to be integrated with IR-lib design. At this moment, there are two different wrappers in this package. One is a warper for Berkeley hard floating point unit. Second, is a wrapper for embedding Alter's IP cores in our design during the FPGA mapping process.
### generator:

This RTL package contains different test cases generate using our *front-end* to target IR-lib. There is a wide range of examples to hammer IR-lib for testing different software test cases. The test cases test IR-lib logics in different scenarios such as arbitrary control-flow, nested loops, nested parallel loops, nested parallel loops with arbitrary , and so on.

### interfaces:

This package contains all the different definition of our interfaces between nodes. These interfaces connect different types of the node. For instance, for connecting two compute node *Databundle* is provided, alternatively for connecting control signals *ControlBundle* is provided.


### junctions:

This RTL package includes the implementation of generic 1: N, N:1, and M: N connections.
All the memory operations in the task are time multiplexed over the junction. There are different implementation of junction in this package. For instance, one possible implementation of junctions is a static tree network or a local bus.
We have parameterized the junction implementations in IR-lib so that the designer can control the physical network that the junction is lowered into.


### memory:

This RTL package contains our different memory unit implementation of memory units. These units vary from the different implementation of *Caches* to *Scratchpad memories*. There is also a different implementation of IR-lib memory controllers for a different type of data like *Scalar, Tensor2D* and so such.


### utility:

This utility package provides a variety of common Scala and Chisel constructs that are re-used across multiple other packages,


## How to debug your design?

In this document, we have explained how someone can start debugging his generated accelerator using simulation tools.

* [**Debug design**](./doc/debugging.md)

## How to fit design on an FPGA?

In this document, we explain how someone can build an accelerator using IR-Lib and then using template files; he can instantiate a new project and fit the design on and `FPGA`. Moreover, we explain the software interface to talk with the accelerator.

* [**Test Fitting**](./doc/test-fit.md)



## Getting Started on a Local Ubuntu Machine

This will walk you through installing Chisel and necessary dependencies:

* **[sbt:](https://www.scala-sbt.org/)** which is the preferred Scala build system and what Chisel uses.

* **[Verilator:](https://www.veripool.org/wiki/verilator)**, which compiles Verilog down to C++ for simulation. The included unit testing infrastructure uses this.


## (Ubuntu-like) Linux

Install Java

```
sudo apt-get install default-jdk
```

Install sbt, which isn't available by default in the system package manager:

```
echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 642AC823
sudo apt-get update
sudo apt-get install sbt
```

Install Verilator. We currently recommend Verilator version 3.922. Follow these instructions to compile it from the source.

Install prerequisites (if not installed already):

```
sudo apt-get install git make autoconf g++ flex bison
```

Clone the Verilator repository:

```
git clone http://git.veripool.org/git/verilator
```
In the Verilator repository directory, check out a known good version:

```
git pull
git checkout verilator_4_202
```

In the Verilator repository directory, build and install:

unset VERILATOR_ROOT # For bash, unsetenv for csh
```
autoconf # Create ./configure script
./configure
make
sudo make install
```


## IR dependencies
IR-lib depends on _Berkeley Hardware Floating-Point Units_ for floating nodes. Therefore, before building IR you need to clone hardfloat project, build it and then publish it locally on your system. Hardfloat repository has all the necessary information about how to build the project, here we only briefly mention how to build it and then publish it.

```
git clone https://github.com/ucb-bar/berkeley-hardfloat.git
cd berkeley-hardfloat
sbt "publishLocal"
```
