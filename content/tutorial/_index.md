+++
title = "DSL-Based Hardware Generation"
template="tutorial.html"
+++

*At [PLDI 2023][pldi-home] in Orlando*

**When**: **Sunday June 18, 2023; morning session** <br/>
**Where**: **Magnolia 9, with a virtual option (see below)**

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

There are three primary ways to attend:
* If you're attending PLDI '23, please register for the **Sunday tutorials and workshops** when registering for [PLDI '23][pldi-reg].
* If you're attending a colocated conference at FCRC '23, please reach out to [Rachit Nigam][rachit-email].
* If you'd like to attend virtually, please [pre-register for this Zoom meeting][zoom] to get the link to join.

## Expectations

We're big believers in learning by doing, so we'll be teaching you how to write your own Calyx frontend from scratch.
Please bring your laptops and expect to be programming for most of the tutorial.

We'll provide a [docker container][calyx-docker]  with all the tools needed to make Calyx program compile and run locally.
If you know you'll be attending the tutorial for sure, please install them beforehand to save time.

**Performance Competition**: Towards the end of the tutorial, we'll be running a performance competition, so you can show off your new skills with Calyx. There is a small prize as well!

## Schedule

Here is the tentative schedule for the tutorial.
All times are in US Eastern Time, GMT-4.
For the most part, we'll be helping you write your own Calyx frontend and have short, interspersed talks.
If you'd like to follow along at your own pace, ([here][slide-deck]) are the slides we'll be using.

| Time | Topic |
| ---- | ----- |
| 9-9:20am | Introduction to Calyx, and [setting up][calyx-start] |
| 9:20-9:55am | [Your first Calyx program][calyx-prog] |
| 9:55-10am | [`fud`, the hardware tool composer][calyx-fud] |
| 10-10:10am | [MrXL, a `map`-`reduce` frontend][mrxl] |
| 10:10-10:50am | Implement a `map` operation for MrXL |
| 10:50-10:55am | [Cider, the Calyx interactive debugger][cidr] |
| 10:55-11am | [Pollen, a pangenome analysis DSL][pollen] |
| 11-11:20am | Break |
| 11:20am-12:20pm | Contest: extensions to MrXL! |
| 12:20-12:30pm | Award ceremony and closing remarks |


[calyx-prog]: https://docs.calyxir.org/tutorial/language-tut.html
[calyx-start]: https://docs.calyxir.org/
[calyx-fud]: https://docs.calyxir.org/fud/index.html
[mrxl]: https://docs.calyxir.org/tutorial/frontend-tut.html
[cidr]: https://docs.calyxir.org/debug/cider.html

[rachit-email]: mailto:rnigam@cs.cornell.edu
[pldi-reg]: https://fcrc.acm.org/
[pldi-home]: https://pldi23.sigplan.org/
[calyx-docker]: https://github.com/cucapra/calyx/pkgs/container/calyx
[pollen]: https://github.com/cucapra/pollen
[zoom]: https://cornell.zoom.us/meeting/register/tJYud-mrqD0iE9XLs5Ms6vxFI-wSngje6AEh
[slide-deck]: /pdf/competition_hints.pdf
