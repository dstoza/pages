---
layout: post
title: FXM ISA Almost Complete
tags: fxm
---

*This is part of the [FXM Project][fxm-overview]*

## Who you gonna call?

This blog has moved from a self-hosted instance of [Ghost][ghost] to [GitHub
Pages][github-pages]. I decided to do this because I spent a couple of days
putting together a long status update and a document on the FXM ISA and posted
them to my site earlier this week. After I restarted my machine for some
routine maintenance, the two new posts were gone. The post I wrote over a month
ago was still there, so it wasn't a total data loss, but it seems like the
posts were in memory somewhere but never got flushed to the on-disk database.

As much as I liked the Ghost platform, if it's going to drop stuff on the floor
without warning, I'm going to go elsewhere. With Pages, I can keep all my stuff
in a GitHub repo and trust that I can reboot my machine without losing stuff,
since there's a copy in The Cloud. It's a shame, though, because the realtime
Markdown preview was slick.

## So where was I...

Over the last month or so, I've put a lot of thought into the specifics of the
FXM instruction set architecture. It's a tough set of constraints. The most
important consideration is how easily I'll be able to actually build the
hardware. Even though that's one of the last steps, I need to keep it in mind
so I don't try to do something stupid, like adding a single cycle 32-bit x
32-bit multiply instruction.

Obviously that can't be the only constraint, because the simplest architecture
to build would probably be a pain to program for, and wouldn't achieve the
performance targets I've set (which I will go into later). With that in mind,
I'm also trying to add enough instructions to create a relatively quirk-free
programming environment. For example, I'm going to have PC-relative memory
addressing so that I can write position-independent code without having to use
the x86-style jump-and-link + load link register thunk.

My third constraint is that while I've drawn a lot of inspiration from RISC
processors, especially MIPS, I want to add a few instructions that no
self-respecting RISC architecture would have, but that will hopefully speed up
common operations. Here again I'm constrained by hardware cost, but I think
there's a little room for exploration. Instructions in this category include a
single-cycle memory copy with offset post-increment, which will let me copy up
to 100MB/s but incurs a moderate hardware cost, and a leading-zero count, which
should speed up emulated floating-point math, and can be implemented cheaply
with a priority encoder.

## The Brass Ring

I mentioned above that performance was one of my priorities for this system.
It's obviously never going to be competetive with even a basic smartphone, but
I still want to push the envelope as much as I can with discrete logic ICs and
a finite budget. Hitting my 25MHz target isn't going to be easy, since the
fastest ALU I can design will be *just* under the ~40ns cycle time, which
doesn't leave much time for control logic. Assuming I do hit the target,
though, I'll end up with a CPU that can execute about 25 million instructions
per second. The first Intel processor to hit that throughput was a 486DX at
somewhere between 33MHz and 50MHz (reports vary), and I'll have the advantage
of single-cycle access to memory.

While a lot of homebuilt CPU projects tend to end up with the computer running
as a web server or some other kind of general purpose device, I realized that
the early '90s, when the 486 was king, produced some of the computer games I
remember most fondly from my early days. The first one that came to mind was
King's Quest VI, which came bundled with my family's first computer: a
486-powered IBM PS/1. Getting KQVI running on a new architecture would even be
relatively straightforward, thanks to all of the work developers have put into
ScummVM. I could take the relevant bits of ScummVM, hook them directly up to my
hardware, and be done.

But then I started looking into some of the other games from that era. DOOM
would be an option, since it's an icon, and its source code is also available,
but I never really got into it. SimCity 2000 would be a bit of a stretch,
because even though I would meet the minimum requirements, I remember it
slowing down at times on my Pentium 133. Then I remembered TIE Fighter. It's a
great game that I put tons of time into. The only real issue was one of
compatibility.

The version that I'd like to run is the DOS Collector's Edition CD-ROM. It add
voiceovers and increases the resolution of the flight mode from 320 x 200 to
640 x 480. Since it was written for DOS on a 486+, it obviously won't run on
whatever OS I end up creating for this system. The easiest approach would be to
do what DOSBox does and emulate a DOS machine, except that emulation is *way*
too slow. Currently I'm torn between two other options:

1. Write a binary translation system. This would parse the original executables
   and generate new executables that target my CPU and OS instead of x86 and
   DOS. Such systems do exist, and the benefit would be that I could use the
   same system for essentially any other DOS game. If I wrote a system that
   emitted LLVM bytecode, I could even optimize the crap out of it in the
   process. The drawback is that it's a lot of work.

2. Reimplement the TIE Fighter engine. With this approach, I would basically
   create a clone of the game in a language that I could then target at my
   system. The advantages of doing this are that I have the flexibility to
   optimize for things that differ between my system and a 486 DOS machine, and
   I don't end up supporting anything I don't actually need. The drawback is
   that I can't reuse any of the resulting code to help get other games running
   (which I would like to do eventually).

I was leaning toward the second option, but I may take a crack at the first to
see how feasible it seems. The fact that I would like to be able to play other
games makes me a bit hesistant to commit to reimplementing every game engine
I'm interested in running.

## The Gory Details

I already covered what I see as the project constraints, but I'd also like to
document some of the specific changes I've made since my initial post.

### Don't You Forget About Me

Perhaps the biggest change I've made is in how I'm going to organize the memory
and register file. Since my target cycle time at 25MHz is 40ns and the SRAMs
I'm going to use for main memory have an access time of just 10ns, I'm going to
perform one write and one read every cycle. I had initially planned to create
multiple banks of memory, with the thought that in most cases, I could set one
to read and one to write and achieve a single cycle memory copy. The problem is
that in the presence of bank conflicts (where both the read and write addresses
reside in the same bank), I would have to buffer the read value and stall the
pipeline for a cycle while I write it. This isn't particularly bad, since on
average I'd be unlikely to hit such a conflict, but it adds hardware cost. By
allowing a read and a write in the same cycle, I no longer have to deal with
memory banking for performance.

The bigger revelation is that I can use similar SRAMs to implement my register
file. I'll have to replicate it three times, since I have some instructions
that require three source operands, but the overall hardware cost shouldn't be
much worse than having discrete register chips. It also comes with the benefit
that my register file can be huge. SRAMs are readily available in the 256kB
range, which would be 64k registers if I could index them all. Since I only
have 32 bits in my instruction words and I want up to three operands, I settled
on 128 x 32-bit hardware registers. Not every instruction can access all 128
registers, since there's not enough room for an opcode, two register
specifiers, and a 16-bit immediate in a 32-bit instruction word, but standard
three-register-operand instructions can use them all. Hopefully my compiler
will be smart enough to take advantage of the full set.

Another benefit of keeping the registers in SRAM is that since the SRAM is so
much larger than 128 x 32 bits, it can keep multiple independent sets of
registers resident at the same time. I intend to exploit this by adding a
*register file base address*, which will set the upper bits of the "register
address". This will give me fast context switching, since I won't have to flush
and restore most registers.

### 16MB Ought To Be Enough For Anybody

Another change I've decided to make in my architecture is to expand memory
addresses from 24 bits to 32 bits. While I'll still only be able to have 16MB
of physical memory (which is about as much as I can afford in SRAM, anyway), I
was a bit worried about programs that like to `mmap` large virtual spaces.

My initial decision to limit things to 24 bits was based on some calculations
on how fast I could get the PC increment logic to work. Even though `PC + 4`
seems like a relatively simple calculation, addition implies a carry chain,
which is the limiting factor in making wider addresses. After sitting down with
the datasheets a bit longer, though, I decided that I can tolerate up to a
28-bit adder, and since my instructions are 4 bytes wide, the only constraint
will be that code has to live in the bottom 30-bits (1GB) of memory. Data can
live anywhere in the 32-bit space, since the address calculation will be
performed in the main 32-bit ALU(s).

## The Next Step

I had started working on the SystemC model of the system, but decided that I
didn't want to waste time implementing things that I might end up changing
before the final design at such a low level. That spurred me to try to nail
down the ISA as much as possible and led to some of the changes above. I think
my next step will be to start working on a lightweight model that just emulates
the instruction set without getting into the details of the hardware. I'll also
be able to use the lightweight model to validate the compiler and operating
system designs, once I start on those. I have some thoughts on the operating
system and compiler, but I'll save those for their own posts after I do a bit
more investigation.

[fxm-overview]: /fxm-overview
[ghost]: http://ghost.org
[github-pages]: https://pages.github.com
