+++
title = "DSL-Based Hardware Generation"
template="tutorial.html"
+++

*At [PLDI 2023][pldi-home] in Orlando*

**When**: **Sunday June 18, 2023; morning session** <br/>
**Where**: **TK**

In the last four decades, the easiest way to improve performance of programs has been to simply wait; processor and process scaling took care of the rest.
Sadly, Moore's law is over, there are no free lunches left, and everyone at the table is desperate.
The only way forward is to build specialized hardware accelerators, that can be customized to the needs of the application.

So how, then, does an enterprising performance hound like yourself build an accelerator?
Teach yourself a hardware design language?
Stare at inscrutable errors for weeks?
Just convert everything into a matrix multiply?
No, thank you.

Welcome to the DSL-based Hardware Generation tutorial at FCRC 2023.
We'll show you how to stay within the comforts of your domain specific language (DSL) and turn programs written in your language into accelerated hardware designs.
Your performance graphs will be more up and more to the right than ever before!

## Logistics

If you're interested in attending, please register for the **Sunday tutorials and workshops** when registering for [PLDI '23][pldi-reg].
If you're attending another co-located conference at FCRC '23 and would also like to attend our tutorial, please reach out to [Rachit Nigam][rachit-email].
If you'd like to attend virtually, please also get in touch.

## Expectations

We're big believers in learning by doing, so we'll be teaching you how to write your own Calyx frontend from scratch.
Please bring your laptops and expect to be programming for most of the tutorial.

We'll provide a [docker container][calyx-docker]  with all the tools needed to make Calyx program compile and run locally.
If you know you'll be attending the tutorial for sure, please install them beforehand to save time.

**Performance Competition**: Towards the end of the tutorial, we'll be running a performance competition, so you can show off your new skills with Calyx. There is a small prize as well!

## Schedule

Here is an extremely tentative schedule for the tutorial.
For the most part, we'll be helping you write your own Calyx frontend and have short, interspersed talks.

| Time | Topic |
| ---- | ----- |
| 9am-9.05am | Introduction to Calyx |
| 9.05am-9.20am | [Setting Up][calyx-start] |
| 9.20am-9.55.am | [Your first Calyx Program][calyx-prog] |
| 9.55am-10am | [Pollen: A Pangenome Analysis DSL][pollen] |
| 10am-10.05am | MrXL: A `map`-`reduce` frontend |
| 10.05am-11am | Compiling MrXL to Calyx |
| 11am-11.20am | Break |
| 11.20am-11.25am | [`fud`: The Hardware Tool Composer][calyx-fud] |
| 11.25am-12.25pm | Hardware Performance 101 / MrXL competition |
| 12.25pm-12.30pm | Closing remarks & Competition Winners |

[calyx-prog]: https://docs.calyxir.org/tutorial/language-tut.html
[calyx-start]: https://docs.calyxir.org/
[calyx-fud]: https://docs.calyxir.org/fud/index.html

[rachit-email]: mailto:rnigam@cs.cornell.edu
[pldi-reg]: https://fcrc.acm.org/
[pldi-home]: https://pldi23.sigplan.org/
[calyx-docker]: https://github.com/cucapra/calyx/pkgs/container/calyx
[pollen]: https://github.com/cucapra/pollen
