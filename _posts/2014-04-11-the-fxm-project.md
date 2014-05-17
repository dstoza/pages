Based on the entirely awesome [Homebrew CPU][homebrew] project, I decided that it would be fun to
build my own (primarily) 74-series CPU and a computer system around it. I've done some preliminary
design, but this is sort of a daunting task, so it's going to take a while.

I'd like the computer to be fairly well-rounded and as modern as is reasonable for something
hand-built, so my initial list of features looks something like this:

* Fairly minimal RISC instruction set (it's looking rather similar to MIPS right now)
* 3-stage pipeline
  * Fetch/decode
  * Register read/execute
  * Memory/writeback
* 32-bit instruction and native word size
* 8 addressable registers: SP, FP, return address, and some GPRs
* 24-bit virtual addressing
* Modified Harvard architecture (instruction "cache" copies pages from main memory)
* Copy-on-write and demand paging
* Up to 16MB physical memory, banked for fast memory-to-memory copies
* Several expansion slots (MMIO + IRQ lines)
  * Ethernet
  * UART
  * Programmable timer
  * Random number generator
  * Flash storage
  * VGA graphics (character mode, not full framebuffer)
* ~25MHz to use the system clock to drive VGA

I'm currently calling it FXM for the three pipeline stages: **F**etch, e**X**ecute, **M**emory, but
I'm not completely in love with the name, so consider it a codename for now, pending a better name.

Instead of trying to settle on the final instruction set before everything is pinned down, I'm
going to start by creating a simulator in [SystemC][systemc]. That should help me figure out what
signals I'm going to need and what the general block layout of the system is going to be.

[homebrew]: http://www.homebrewcpu.com
[systemc]: http://en.wikipedia.org/wiki/SystemC
