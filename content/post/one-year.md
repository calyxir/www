+++
title = "One year of Calyx"
date = 2021-07-15
draft = true
+++

A blog post to celebrate one year since the submission of the original Calyx
paper.

## Outine

It's been a whole year since we first submitted the Calyx paper! In that time,
we've made progress on several different fronts.

## Improvements with tooling

Because Calyx is a compiler infrastructure, there are many different tools
and compilers that need to be called in order to transform a high-level program
into Verilog that can be synthesized and simulated. We created a tool called `fud`
that drastically streamlines the experience of working with these tools.

It is built around a concept we call "stages". Each stage is responsible for taking
one input format into another. For example, we have a Dahlia stage that is responsible
transforming `.fuse` files into Calyx files, and a Verilog stage that compiles Calyx files
into SystemVerilog.

To use `fud`, simply specify an input file, the input stage, and the desired output stage:
```
fud exec input.fuse --from dahlia --to verilog
```
and `fud` seamlessly stitches all the stages together.

The tool is written in python and can be easily extended to support more frontends and backends.

## Calyx on real hardware

In our paper, we ran Vivado synthesis on various benchmarks to gather resource usage information.
However, we never actually ran these designs on real silicon. In the past year, we have set up
the basic infrastructure to get Calyx running on Xilinx FPGAs. There is still a lot of work to be
done, but what we have so far is exciting step towards realizing Calyx as infrastructure that people can
actually use.

Xilinx FGPA boards expect kernels to implement an AXI slave interface so that a host can start
a kernel and pass memory address arguments. We wrote a relatively simple interface wrapper
(if you know AXI you know that the word relatively is doing a lot of heavy lifting) that translates AXI commands into
the `go/done` style interface that Calyx components expect as well as copies memory into local BRAMs
that Calyx components can access. The wrapper is written in Verilog because Calyx doesn't easily support
these types of interfaces. In the future, we are interested in extending Calyx to support this.
Doing so might allow us to verify the correctness of the AXI implementation more easily.

The Xilinx tools require several different metadata files alongside the (System)Verilog source. We
automated the creation of these files with `fud` to make interacting with these tools more seamless.
The only piece of the puzzle that is manual at the moment is writing OpenCL host code to integrate
the kernel into an application. While we are not looking to completely automate writing the host code,
we would like to write a generic host that understands the same data format we use for simulation
to enable easier validation.

Some day we would like to support more FPGA boards and infrastructures, but that is very contingent
on somebody coming along with the desire to write and maintain these backends.

- Sam
  + Fud
  + FPGA execution
- Chris
  + Relay, NTT, TCAM
  + MLIR integration
- Griffin
  + Resource Unsharing
  + Interpreter
