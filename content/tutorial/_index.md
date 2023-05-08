+++
title = "DSL-Based Hardware Generation"
template="tutorial.html"
+++


**When**: **June 18, 2023** <br/>
**Where**: **TK**

In the last four decades, the easiest way to improve performance of programs has been to simply wait;
Processor and process scaling took care of the rest.
Sadly, Moore's law is over, there are no free lunches left, and everyone at the table is desperate.
The only way forward is to build specialized hardware accelerators, that can be customized to the needs of the application.

So how, then, does an enterprising speed devil [**TK: better term**] like yourself build an accelerator?
Teach yourself a hardware design language?
Stare at inscrutable errors for weeks?
Just convert everything into a matrix multiply?
No, thank you.

Welcome to the DSL-based Hardware Generation at FCRC 2023.
We'll show you how to stay within the comforts of your domain specific language (DSL) and turn programs written in your language into accelerated hardware designs.
Your performance graphs will be more up and more to the right than ever before!

## Logistics

If you're interested in attending, please register yourself for the **second day of tutorial** when filling out your [PLDI '23 registeration][pldi-reg].
If you're attending another co-located conference at FCRC and would like to attend, please reach out to [Rachit Nigam][rachit-email].

## Expectations

We're big believers in learning by doing, so we'll be teaching you how to write your own Calyx frontend from scratch.
Please bring your laptops and expect to be programming for most of the tutorial.

We'll provide a docker container (**TK**) with all the tools needed to make Calyx program compile and run locally.
If you know you'll be attending the tutorial for sure, please install them beforehand to save time.


## Schedule

Here is an extremely tentative schedule for the tutorial.
For the most part, we'll be helping you write your own Calyx frontend and have short, interspersed talks.

| Time | Topic |
| ---- | ----- |
| 5 mins | Introduction to Calyx |
| 15 mins | Setting Up |
| 35 mins | Your first Calyx Program |
| 10 mins | Break |
| 5 mins | Pollen: A Pangenome Analysis DSL |
| 15 mins | MrXL: A `map`-`reduce` frontend |
| 60 mins | Compiling MrXL to Calyx |
| 10 mins | Break |
| 5 mins | `fud`: The Hardware Tool Composer |
| 60 mins | Hardware Performance 101 / MrXL competition |
| 10 mins | Closing remarks |


[rachit-email]: mailto:rnigam@cs.cornell.edu
[pldi-reg]: https://pldi23.sigplan.org/venue/pldi-2023-venue