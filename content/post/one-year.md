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


## New Optimization Pass: Register Unsharing

Yes you read that right, _unsharing_. But please hold your ire for a moment, I
promise there is a method to my madness. The Calyx compiler already has
optimization passes to share computation resources such as registers and adders.
This pass does the opposite for registers: if a single register has independent
uses then the pass separates each of the uses into their own unique register
without changing the program semantics.

At first blush, this seems like non-sense. For one, what is the point of either
the resource sharing pass or the register unsharing pass if they undo each
other? Why have both? And, secondly, isn't sharing resources good? Why would we
want to use _more_ registers?

These are valid and reasonable concerns; however we must remember that in the
world of hardware there are always trade-offs. When thinking about sharing and
unsharing we really ought to be thinking of a trade-off between computational
units/resources and control wiring. Sharing reduces the resources a given design
requires however it does so at the cost of increasing the wiring complexity and
the control structures needed to arbitrate access to the shared resources.
Unsharing, in contrast, moves the trade-off in an opposite direction. It
increases the resource requirements of a design but reduces the wiring
complexity by requiring fewer MUXes.

There are no universally "correct" places on this trade-off curve. FPGAs have a
set amount of resources per block and so up to a certain point it can be better
to have more registers rather than more MUXes while the opposite holds for
taping out a design.

(XXX: This last bit might not contribute anything interesting. It's all
technical detail)

The optimization pass itself takes the form of a reaching definition analysis
for the register uses, but unlike the conventional form of this analysis, Calyx
assignments present a slight wrinkle. Assignments in Calyx can be guarded, which
means even if a particular group runs, it is not necessarily the case that a
given assignment will actually occur. Furthermore, since assignments are
non-blocking and thus not strictly ordered, disambiguating different definitions
within the same group ends up being far beyond the scope of a static dataflow
analysis. As a final domain wrinkle, Calyx groups can appear multiple times in
the control structure which means any re-writes to the group in one place must
be valid in all places, though this can be sidestepped by deduplicating the
groups.

In order to make the pass appropriately conservative, we have to frame groups in
terms of "must-write" and "may-read". We can only kill definitions if a group
"must-write" a given register, but doesn't read it. Re-writing is similarly
restricted. For the moment "must-write" is an overly simple criterion since it
is only true for unguarded assignments. A more sophisticated form of the
analysis may be able to make this judgment for groups with multiple
(complementary) guarded assignments within a single group.

## Calyx Interpreter

Another major infrastructure tool that's been under-development is a native
interpreter for Calyx. It might seem a bit strange to be working on an
interpreter given that Calyx already has a compiler; typically language
development starts with an interpreter and moves to a compiler later as they are
far more efficient. Indeed from the viewpoint of conventional language design,
this appears entirely backward! Nevertheless, an interpreter holds the
solution to many problems: from simulation complexity and massive tool-chains
to compiler pass validation and semantic under-specification.

This is best illustrated by thinking of the interpreter not as an interpreter,
in the PL sense of the word, but as a native simulator for the Calyx IR. An
interpreter means we have a way to run Calyx code directly rather than needing
to transform it into SystemVerilog and run it through Verilator. This opens a
number of opportunities for tools that work with Calyx in its native form and
can take advantage of the control information present in higher-level Calyx
code.

Of perhaps chief interest is making an efficient simulator for Calyx that is
capable of simulating larger designs from the TVM front-end. And there is reason
to be confident that such a thing is possible. For one, having a shorter
toolchain is good both for overall timing and for usability. It presently takes
running three compilers and a binary in order to get simulated results from an
input Calyx program (Calyx -> SV, SV -> Verilated C++, C++ -> binary). Having
that toolchain contain just the Calyx interpreter means fewer dependencies and
fewer compiler invocations which stand to make the end-to-end simulation time
faster.

But beyond mere toolchain improvements, the control information in Calyx may
make simulation more efficient. A common problem in circuit simulation is that a
large portion of the circuit is not running at any given time but still must be
simulated. There are many clever heuristics for identifying the dormant parts of
the design and saving time by not simulating them; however Calyx needs no such
heuristics. The control structure in Calyx code means that we know precisely
what parts of the design are running at any given moment and needn't waste time
simulating portions of the circuit which are known to be dormant.

Furthermore there are performance gains to be had when we consider that the
Calyx IR itself has less simulation complexity than the RTL it generates. Since
the information available at the IR level is more structured, the simulation
itself is free of the complex details that appear at the RTL layer. While it is
by no means guaranteed, there are strong signals to suggest that a fast
simulator for Calyx programs is possible.

Additionally, we may be able to design better debugging tools which use Calyx
directly, such tools could use control information to be more intuitive than
waveform debugging and hopefully far more approachable. We may even be able to
make a "gdb for hardware"!

As a final note, a simulator is also a boon both for the language itself and the
core compiler infrastructure. Being able to run Calyx programs directly means we
can perform differential testing on compiler passes and verify that the given
changes made by a pass are indeed semantic preserving with respect to the input
program. Plus, the act of making the interpreter itself is an excellent way of
"hardening" the semantics of the language as it requires making clear decisions
on corner cases obscured by the translation to Verilog.

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
