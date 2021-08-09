+++
title = "Growing an Open-Source Hardware Infrastructure"
date = 2021-07-15
draft = true
+++

It's been roughly one year since the submission of the [original Calyx
paper][calyx-paper].
In that time, we've continued building tons of new tools, frontends, and
integrations.

## Simplifying the Hardware Workflow

Calyx is a compiler infrastructure that needs to interact with frontends
(languages that generate Calyx IR) and backends (mechanisms to run Calyx
generated code).

A simple workflow that goes from [Dahlia][] (a C-like frontend) to simulation
through [Verilator][] involves tediously running several commands.
```bash
dahlia/fuse -b calyx --lower -l error -o out.futil # Generate Calyx IR from Dahlia
calyx/futil -b verilog out.futil -o out.sv         # Generate Verilog from Calyx IR
verilator -cc --trace out.sv --exe testbench.cpp \ # Compile Verilog with Verilator
  --exe --build --top-module main
./Vmain   # Execute the Verilator model
```

Furthermore, every time there is change in the input Dahlia file, all of the
commands need to be re-executed.
It's easy to see why getting into hardware design can be so daunting.

In an effort to provide a seamless command-line tool that makes it easy for
novices to use and interact with hardware design tools, we built [`fud`][fud].
Fud is built around the idea of "stages" which provide a way for transforming
a set of input files (say a Dahlia source file) into output files (say a file
with Calyx IR).
Fud can also chain sequences of stages to transform files several times.
For example, the following `fud` command runs all the necessary steps to
compile a Dahlia source file through Calyx and simulate it with Verilator:
```
fud exec input.fuse --from dahlia --to verilog
```

Fud also makes it easy to switch between outputs making debugging at different
levels a breeze:
```
fud exec input.fuse --from dahlia --to calyx
```

The tool is written in python and can be easily extended to support more
frontends and backends.

## Calyx on FPGAs

Our original paper on Calyx ran Vivado's place and route tool to gather
resource usage estimates.
Since then, we've put in the effort to get Calyx generated designs running on
real FPGA boards.
This represents an exciting step towards realizing Calyx as an infrastructure
for rapid development of domain-specific languages that generate hardware
designs.

In order to support Xilinx FPGAs, we implemented an AXI interface so that host
programs can communicate with kernels running on an FPGA.
We wrote a interface wrapper in Verilog that translates AXI commands into the `go/done`
style interface that Calyx components expect as well as copies memory into
local BRAMs that Calyx components can access them.

The Xilinx tools require several different metadata files alongside the Verilog
source.
We automated the creation of these files with `fud` to make interacting with
these tools more seamless.
The only piece of the puzzle that is manual at the moment is writing OpenCL
host code to integrate the kernel into an application.
While we are not looking to completely automate writing the host code, we would
like to write a generic host that understands the same data format we use for
simulation to enable easier validation.

While we are able to support the specific Xilinx boards, we do not yet have a
general solution.
A missing piece in the puzzle is a generic interface and intermediate
representation that can target arbitrary boards.
Thankfully, our collaborators at the University of Washington have been
developing [exactly this][reticle].

## New Frontends

Calyx's development as a language and an infrastructure is driven by frontends
that make use of it.
Since the publication of our paper, we have implemented two new frontends that
can be used to stress test Calyx.

### TVM Relay

[TVM][] is a compiler for machine learning frameworks that can optimize and
target kernels to several different backends.
Our TVM frontend compiles TVM's internal IR, called Relay, to Calyx IR, and
can successfully simulate small neural networks.
Relay instructions are quite high-level, implementing operators such as a
two-dimensional convolution.
In order to side-step manually generating such operators in Calyx and to
simplify debugging, we instead generate Dahlia programs to implement these
operators.

In [Calyx #504](https://github.com/cucapra/calyx/pull/504), we successfully
simulated a [LeNet][] network model.
Of course LeNet is an extremely simple neural network and our long-term goal
is to support modern networks.
Our next target is the [VGG][] network.
However, we've also realized while building this network that simple,
Verilator-based simulation is going to be too slow so we're also working
on other efforts like running Calyx on an FPGA and building a fast simulator
for Calyx itself.

### Number Theoretic Transform (NTT)

The number theoretic transform (NTT) is a generalization of the fast Fourier
transform, commonly used as a primitive in homomorphic cryptography.
We implemented an NTT pipeline generator produces the NTT transform of some
input with the provided bit width and input array size parameters.


## Calyx + CIRCT

[CIRCT][] (Circuit IR Compilers and Tools) is a cross-community effort to build
an open-source hardware ecosystem using the [MLIR][] framework.
The MLIR framework represents compilation as a series of transformations between
"dialects" or languages at different levels of abstractions.
In the long-term, the CIRCT community is interested in compiling high-level
languages to hardware designs, which is something Calyx can already do today.
In order to align our efforts, we've started implementing a [Calyx dialect][calyx-dialect]
which can be emitted from other dialects in CIRCT, transformed by the Calyx compiler,
and of course, integrated with `fud`.

- Griffin
  + Resource Unsharing
  + Interpreter

[lenet]: https://en.wikipedia.org/wiki/LeNet
[vgg]: https://neurohive.io/en/popular-networks/vgg16/
[calyx-paper]: https://rachitnigam.com/files/pubs/calyx.pdf
[mlir]: https://mlir.llvm.org/
[circt]: https://github.com/llvm/circt
[calyx-dialect]: https://github.com/llvm/circt/tree/46ae6df8eb30c0404a0cc54f92a68a63ef643c89/test/Dialect/Calyx
[reticle]: https://github.com/vegaluisjose/reticle
[dahlia]: https://capra.cs.cornell.edu/dahlia/
[verilator]: https://www.veripool.org/verilator/
[tvm]: https://github.com/apache/tvm
