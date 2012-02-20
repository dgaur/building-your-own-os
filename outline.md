


# SECTION 1: GETTING STARTED


## Introduction

### Why would you want to do this?
- It's fun!
- Learning
- Sheer power of controlling the entire system (mostly)



### What this document is:
- Writing your own OS for a single-processor x86 PC.  Focus is on kernel
  development and application libraries here; once those are done,
  everything else is "basic" application or driver development.
- Fairly simple OS design, designed for exploration + modification
- Simple algorithms + policies
- Mechanism vs. policy.  Mainly about mechanism here, with some discussion
  of possible policies.  It's your OS, so I leave the details of the
  policies to you.




### What this document is NOT:
- A book on OS theory.  There are plenty of those already.
- A portable, multi-proc, real-time, massively-parallel NUMA OS, etc.
- An in-depth examination of the x86 architecture.
- An in-depth examination of any 3rd-party piece of hardware.  This is
  about getting your own OS off the ground, not the intricacies of your
  particular video adapter, etc.



### Target audience:
- Experienced system programmers + enthusiasts.  Students?
- Assume basic knowledge of x86 assembly, C/C++
- Assume basic hardware operation (interrupts, DMA, etc., ...)
- Assume basic knowledge of OS internals (VM, scheduling)
- Assume basic knowledge of algorithm performance + analysis.
  O(1) vs O(N), etc.  This is perhaps less critical than the other
  assumptions, but it's helpful for choosing algorithms + data structures.




## Basic Hardware Overview

[Not sure if this section is useful.  Cannot describe the entire CPU
architecture + features here; there are better references, etc]


### x86 Architecture
Just an overview of key features:

- Registers?  Probably only mention unusual or esp. important ones (CR0,
  CR3, GDTR, IDTR, etc., ...)
- Segmentation + memory models. Linear ("virtual") vs. logical vs.
  physical addresses.
- Real mode vs. Protected mode
- Descriptor Tables
- VM
- Stack management.  Stack layout for function calls, parameters, etc.
  Include a figure.


### Basic PC architecture
Again, just give an overview of some of the important features that OS
has to handle:

- PIC, APIC
- PIT
- PCI, ISA.  Bus basics.
- A20 line.  If you're using GRUB, this might be transparent.
- Commonly used/reserved memory regions (memory map).  E.g., ISA DMA,
  PCI mapping, BIOS area, video ...
- Boot process.  Basic boot-loader fundamentals.  From BIOS to OS.  Memory
  map, reserved addresses and well-known I/O ports.




## Basic OS Architecture

It's important to answer some fundamental questions first:

- Type of OS: general-purpose, embedded, networking, etc.
- Multi-threaded kernel?  Older Unix/Linux kernels were single-threaded
  (only one active kernel thread at a time).  This is probably simplest,
  but limits performance somewhat.  Newer kernels support multi-threading.
  More complicated (lock contention, need to be careful with your locks),
  but better performance.
- Defining a memory map/layout for your OS.  Which components go where?
  Kernel placement, stack placement.  OS mapped in high-memory?  Or
  identity-mapped in low memory?  Or something else?  Brief discussion of
  why this matters.
- Monolithic vs. micro-kernel.  This is a major decision.  Monolithic is
  probably (?) easier up-front, since "everything" is readily available to
  your kernel components, but might harder to maintain and extend in the
  long-run.  Linux/Unix/Windows vs Mach/L4/etc.  Historically,
  micro-kernels have not performed as well, but L4 results show this to be
  false or artifact of poor design/implementation.
- Layering of kernel functions + impact on functionality.  Not all
  layers have access to the same functions.
- Device driver model.  Loadable modules?  Loadable modules
  require a file format that is friendly towards runtime/dynamic
  linking (ELF, etc).  ISR/DPC (bottom-half, top-half, etc).
- Virtual memory?
- Scheduling: native mechanism (TSS) vs. software-only.
- Single vs. multi-threaded?
- Multi-user?
- Interrupt-handling philosophy.  Handle inline vs. deferred/DPC
  model.  See also chapter/section on interrupt-handling.

As an aside, feel free to introduce your own conventions + design ideas.  It's
your OS; don't be constrained by existing or conventional ideas.  It's not
necessary to re-implement Unix, e.g.  In some ways, doing something
"different" is the most interesting + entertaining.  The papers on
"alternative" OS designs (L4, Netware, etc) are more intriguing than those on
Windows, Linux - different goals and decisions, etc.

This is probably a good place to mention the Spring OS paper (which
has good intro on writing your own OS).  "If you don't know where
you're going, you'll never get there."

Also: the Pebble OS paper - good intro discussion/quotes on novelty of new OS.



## Choose your weapons


### What's your objective?
In practice, no project like this is ever really "finished", but it's helpful
to have some target/goal in mind.  Perhaps you just want a simple command
shell.  Maybe port a "big" application like gcc, which would then let you
develop + build your OS from within itself.  Perhaps you want to support X.


### Choosing a File Format
This might seem an odd topic here, but it has implications for the tool-chain;
boot-loader; etc.  If you're using GRUB, then kernel must be either ELF or
(maybe?) a.out format.  ELF is better for loadable modules than a.out (both
kernel modules + user-mode shared libraries, etc).  The "Linkers and Loaders"
book provides some insight here.

If not using GRUB, then list some other possibilities (PE/COFF).  Some folks
appear to prefer COFF, but is it compatible with GRUB?



### Choosing your Development Tools

Use the GNU tools unless you have a solid reason not to.  They aren't the
best tools out there, but they're solid, they're easily obtained and they're
free.  Note that a basic Linux system will likely include almost all of the
necessary tools.  Conversely, MSVC is probably not the best choice since it
lacks some of the low-level tricks that you'll probably need.

You'll want:

- Compiler, assembler, and linker (all 32/64-bit capable; must be 16-bit
  capable or have 16-bit counterparts if you intend to write your own boot
  loader).  These must be capable of generating your desired file format;
  in some cases (e.g., building ELF binaries under cygwin), you may need
  to build a cross-compiler.  Maybe mention the special libgcc.a library
  that includes the gcc "helper" functions?
- Make.
- A dis-assembler and symbol-table-viewer (or
  linker that emits cross-reference maps).  Incorporate these steps right
  into your build process.  These are useful for debugging and validating
  tables (e.g., the GDT or IDT, if you're building it a compile-time).  At
  some point, you'll probably want to do this.  I found it useful
  occasionally to examine emitted assembly to ensure I had
  references/pointers correct (memory references vs. inline literals,
  etc).
- GRUB.  Again, use GRUB unless you have a good reason not to.  Boot
  loaders are everything that modern OS' are not: 16-bit real-mode
  assembly code.
- VCS.
- Raw disk I/O utilities.
- Debugger?  GDB?
- Virtual machines?  Handy for testing, but ultimately there's no
  substitute for real hardware.  Running on real hardware is more fun,
  too - you're controlling the physical machine, as opposed to running in
  some fake software environment.  Ultimately, the problem with emulation
  is inherent in the concept: it's emulation, not real hardware.

	I've tried a few different options:

	- QEMU seems the best for OS development (scriptable, built-in GDB hooks)
	- Bochs and VirtualPC are ok, provide some nice features, limited
	  debugging support
	- vmware, MS VirtualPC are ok, but not good for debugging



### Hardware
- At least one test system that matches your hardware target.  Initially,
  you probably want: single-processor PC, floppy disk, video,
  keyboard, BIOS with E820 support?
- Development system.  Unix/Linux is probably best bet, since it will
  include the necessary build tools. Alternatively, cygwin under Windows.
  If you're using a virtual machine for testing, then one system may be
  enough for both building/testing.
- POST card for testing on real hardware?
- For debugging real hardware, possibly serial cable back to development
  machine.


### Where to start?
There are various approaches.  Here is roughly how I worked:

- Boot.  Get GRUB working.  Tinker with their sample "kernel".
- Basic console I/O support.
- Basic kernel-mode C library.  This will likely be an ongoing task; you
  will need same basic functions (memset, printf, etc) at first, but will
  probably need to add additional functions later.
- Rudimentary memory management.  At least enough to "allocate" some of
  the fundamental structures.
- GDT.  Test by loading/dumping selectors.
- IDT.  Test using `INT x` instructions.  Validate gate addresses against
  the address in the kernel image.
- PIC setup.
- PIT setup.  Test PIC/PIT by handling timer interrupts.
- Calculate processor speed and timing delays.
- Thread allocation/deletion.
- Thread switching.  Should be able to do this manually prior to writing
  an actual scheduler (e.g., create two threads, switch back and forth,
  etc).
- Scheduling.  Once the basic Scheduler is working, then start adding
  additional scheduling-related support items: timers; semaphores; etc.
- Basic IPC + messaging.
- Address space/containers.  Move threads out into separate address spaces.

Expect some circular dependencies here (e.g., how do you allocate storage or
context for the memory manager?), especially early in the boot process.



## Basic Coding Issues

### Choosing a kernel language
* Safe choices: assembly, C/C++.
* Weirder options: scheme (schemix in Linux), kpForth (likewise).

Most folks opt for C with some assembly bits here and
there.  As always, this is a contentious decision and there is no
simple/single answer.  Include a list of criteria for the language:

- bit-level access;
- structure alignment and packing;
- integration with assembly code and/or inline assembly;
- compiled vs. interpreted;
- low-level hardware access.

Tools are important: a language like D might look appealing (OOP with some nice
high-level constructs and features), but if the compiler/tools are lousy, then
you're constantly fighting/debugging the tools instead of building your OS.


### Assembly
Assembly-interface to high-level language can be difficult; you must know and
understand how the high-level language/constructs are implemented in assembly,
calling conventions, etc.  The aspects of higher-level languages that make
them/you more productive also make them more difficult to interface with
assembly.

Some languages define "foreign" interfaces for calling other
code/other languages.  See
<http://lambda-the-ultimate.org/node/2283> and read the comments.
HLL might be harder to analyze emitted assembly (for debugging, crash dumps,
etc) since the HLL-to-ASM mapping is less obvious.  Same goes for structure
layout - cache-line optimization is harder, etc.


### C++-specific issues.
I wrote my kernel in (a subset of) C++, so some specific notes:

- C++ name mangling.  Using `"extern C"` to avoid linker/naming issues.
- Code + performance overhead of: static methods (no `this` ptr)
  vs. nonstatic nonvirtual methods (`this` ptr, but compile-time resolution)
  vs. methods (`this` ptr, runtime `vtbl` lookup).
- Implementing `new`, `delete`, `throw`, etc.
- Compiler-dependencies for exception-handling, RTTI, etc.  Best to avoid
  these features.
- In general, if you're going to code in C++, it's important to understand
  how the compiler will translate your C++ into machine code.  Although,
  really: this is important regardless of the language you choose, with
  the obvious exception of pure assembly.
- Lesson learned: C++ does not integrate well with inline assembly in some
  instances (e.g., atomic operations should just be a couple of
  instructions; but `this` pointer adds overhead, etc).
- Lesson learned: without support for exceptions + exception-handling,
  there's no clean way to handle errors in object constructors.  Panic the
  kernel; or use some additional object flag (`thread.State`) or separate
  init() method in addition to constructor.
- Lesson learned: class hierarchies complicate cache-optimization of
  object layout because offsets of class members are not necessarily
  obvious.  Likewise, virtual methods add `vtbl`, which complicates layout
  optimization.
- Some gcc/g++ specific issues:
    - Code bloat from "in-charge" and "not-in-charge" ctors and dtors.
    - Need stub for `__cxa_pure_virtual()`.
    - Problems with template instantiation.  May need to instantiate some
      templates directly to include vtable and "key method" in kernel
      image.  Never really figured this out: native compiler worked fine,
      cross-compiler did not.


### Build process
No external libraries or headers allowed.  Language features (C++ exception
handling) that potentially require extra libraries.  No relocations/imports
allowed.  Maybe list some sample GCC options for this.


---


# SECTION 2: KERNEL MODE

## Boot Process, Booting Your OS
@might be better to frame this as "the skeleton/basic framework for booting your
kernel", etc

- Initial memory map.  Reserved regions: ROMs, BIOS Data Area + Extended,
  video memory, etc.
- A20 line.
- Role of boot loader, GRUB, etc.
- Use GRUB!
- Basic discussion/figure of boot process, too: BIOS, boot loader
  (possibly multiple stages), etc.
- List basic boot steps if you aren't using GRUB:
    - Loading sectors from disk
    - Read data from BIOS, etc.
    - Enable A20 line
    - Setup GDT, stack, etc
    - Switch to protected mode.


## Video/Console management
Starting here, or perhaps in the HAL chapter, assume that you have a
rudimentary kernel booting from floppy.  Maybe just the GRUB sample, but
that's enough for now.

- Basic I/O for testing/debugging initialization
- Putting text on the screen.  Basic colors.  Attribute + character code
  usage.  Scrolling, via memory copies and/or hardware manipulation.
- Cursor control.



## Kernel C/C++ Library
Basic library functions, probably need this early in development:

- `memset()`
- `memcpy()` and/or `memmove()`
- `printf()` or `sprintf()`, for debugging.  This one requires a fair amount
  of logic, unlike the other routines.
- `strlen()`, `strcpy()` if any string-processing in kernel.  Once you add
  console I/O, you'll need several string-handling routines to support
  `printf()`.

See also the user-mode libc section below.



## HAL/Hardware Management

Not necessarily for portability, but simply collecting assorted x86
weirdness/quirks into a single location.

If you are actually looking for portability, to a different architecture,
multi-processor, etc., then this is how to do it.  Whether it's an explicit
HAL like this, or just a library of "architecture-specific" functions, or
whatever, you need some sort of abstraction logic.

Sole location for assembly.  If it has to be written in assembly code, then it
probably belongs in the HAL.

- Special processor registers (CR0, CR3, ...)
    - Enable CR0.WP (protect RO pages from kernel access) is good for
      debugging, even if you aren't implementing fork/COW semantics. It
      catches writes to RO pages, which is sometimes (always?) an error.
      Personal experience: I found a couple of bugs this way, where the
      permissions on a couple of pages were incorrect.
- TSS management:
    - Context switch using TSS vs stack.  Conventional wisdom is that TSS
      is slower, but no actual data?  TSS is not portable.
    - Actual TSS structure.  At least one per CPU.
    - Minimal fields required: SS0, ESP0, BitmapAddress.  More if using
      ring-1 or -2.  Bitmap allows I/O from ring-3.  Weird size = 104 +
      8K + 1.  More if using 8086 emulation + IRQ redirection.
    - GDT entry + selector
    - `ltr` instruction + selector
- VM details: page-fault address, page dir/table layout?
- I/O ports
- Memory-mapped I/O.  How to map memory to device registers.
- PIC management
- PIT management
- GDT.  Reload the segment registers or dump them out to verify their
  validity.  GDT construction/initialization can be done at
  compile-time: it's essentially a table/structure of constants.
  Contrast this with IDT + TSS, which (potentially) require
  manipulation of link-time addresses.

	Can you use only
	user-selectors with page-level protection?  No, probably not,
	because selectors implicitly contain/set the CPL?  Double-check this.

- IDT.  Once a basic IDT is installed, test a couple of the processor
  interrupts using `INT x` instructions.

	Figure: show connection between IDT entry, selector in GDT and handler
	address in host memory.

	Unlike GDT construction, IDT construction cannot easily be
	done at compile-time because each entry contains the (possibly unique)
	address of the corresponding entry point, and furthermore
	the address must be broken into two 16bit pieces.  These addresses
	are not available until link-time, and so the IDT requires some
	runtime initialization/fixup to correctly populate the table (or
	possibly better tools?).  May be possible to patch the IDT structure
	directly in the kernel binary, after link-time but before
	execution-time.

	Impact of the IDT length - truncate IDT to N
	entries, which causes GP if software(?) generates interrupt vector
	outside of the table?  Or populate all 256 entries, some with
	NULL/unhandled routines, to catch all possible interrupts, even
	spurious hardware interrupts?  Probably interchangeable, although
	the latter approach seems more resilient in the face of spurious
	hardware interrupts.

- Synchronization mechanisms.  Semaphores, spin locks, spin-locks with
  interrupt protection, etc.  Implications of uni-processor
  vs. multi-processor systems.  Uni-processor synchronization methods
  may not work on multi-processor hosts.

	Give some assembly-language
	examples on implementing the primitives.  Need to be careful even on
	UP, though: interrupts can still cause race conditions; and cannot
	simply disable/re-enable interrupts because of nested
	synchronization points.  Instead, use `{ pushf; cli; ... popf; }`,
	etc.

	Atomic operations: increment/decrement.

	Build more complicated/higher-level synchronization constructs on top of
	more primitive ones.  @Not sure if this discussion belongs here or in
	Thread Management section.

- Shutdown/reboot support
- Processor speed/frequency.  Important for calculating I/O delays.
  Use the PIT and Pentium TSC to calculate processor speed and the
  microsecond delay for busy waits.  Mention older method on
  pre-Pentium hardware, using timing loops.  Regardless of the method,
  this requires interrupt-handling support and interrupts must be
  enabled.
- RTC.
- Multiprocessor management: keep CPU descriptors, containing CPU
  index, memory penalties for NUMA, etc.
- BIOS data area.
- TLB management.  Shoot down TLB entry whenever page dir/table entry
  changes, is overwritten, deleted, etc.
- MTRR's.  How to handle these?  Interaction with TLB?
- Dealing with floating point.  FP registers, instructions (finit,
  etc), lazy save + restore.  How to handle with context switches.


## Interrupt Management
- Semantics of reprogramming the PIC.
- Overlapping interrupt vectors with the default PIC configuration.
- Interrupt-servicing priorities implied by the PIC.
- Interrupt handlers.
    - Maybe include some figures to illustrate the stack layout for each
      type of interrupt (with/without privilege-level change; with/without
      error code).  Where relevant, include a snippet of the dx stub
      handler that illustrates how to handle that type of interrupt.
    - Low-level handlers (stubs) must be in assembly to allow direct stack
      manipulation, saving register contents, etc.  General-purpose
      registers must be saved, restored on exit.
    - For safety, you should re-load segment registers with values that
      you know are valid.  The processor will load CS automatically, but
      the contents of DS are unknown.  @what about SS/ESP? Loaded from
      TSS?
    - Inline interrupt-handling vs. deferred interrupt-handling.  "Top
      half" vs. "bottom half".  ISR/DPC.  Probably need both; and usage
      probably depends on the exact interrupt.  E.g., some interrupts
      cannot be deferred, because the current process cannot continue
      until the interrupt is serviced (GP, invalid instruction, or page
      fault when page is on disk).  Other interrupts may require
      additional device I/O (disk drives, network cards) and therefore are
      better suited to the deferred model.
- Timer interrupts
- Traps (system calls).  Mention call gates, too, since these are nice for
  system calls.
- Faults/exceptions
    - Once you have interrupts working, install a special handler to catch
      double faults and dump the error code.  Keep it simple, to minimize
      the likelihood of a triple fault.  Assume that you'll generate a few
      double faults, so this will catch them rather than just rebooting.
      Easier to debug, etc.


## Time Management
This seems fairly easy/trivial at first; but becomes important
when you start implementing your scheduling functions.  Sleep for x seconds,
etc.

- Programming the PIT.  Timer interval + interrupts; implications for the
  scheduler.  Include the equation for calculating the timer interval:
  `(desired interrupt rate) = (oscillator freq) / (countdown value)`.
  Mention problems of integer division, truncation leads to slight timing
  errors that can add up.  Disabling interrupts masks clock interrupts,
  potentially skewing tick counts.
- Using the PIT to calculate processor speed and microsecond delay.  See
  earlier discussion in chapter on HAL.
- Brief discussion on PIT interrupt frequency.  Higher frequency allows
  more fine-grained scheduling and benefits highly-interactive
  applications (e.g., any kind of multimedia) because of the faster
  reaction/response times, but introduces more interrupt-handling
  overhead.  Lower frequency is more coarse-grained scheduling, but less
  interrupt overhead.  This is a tradeoff, like many other things.
- Programming the RTC.  RTC inaccuracy (see
  <http://www.codinghorror.com/blog/archives/000760.html>).
- 32/64 bit timestamps.  Rollover issues with 32b.  Tick counts vs
  timestamps.
- Time-related system calls



## Process + Thread Management
- Scheduler + scheduling policy
    - "Classic OS topic/problem".
    - This is a good place to keep statistics to tune/modify the
      scheduling policy.  The Silbershatz + Galvin text has a good
      discussion of this, along with some quantitative examples and rules
      of thumb (e.g., 80% of CPU bursts should not exceed quantum).  It
      also mentions some guidelines for setting scheduling quanta.
      Likewise, the Inside Windows NT book has some example quanta
      guidelines as well.
    - Give ideas/examples of quantum values, how to calculate, etc.
- Mechanics of thread switch
    - Stack-based thread switching.  Portability to non-x86.  But portions
      of this belong in the HAL, nonetheless (e.g., exact details of
      saving/restoring registers, page tables, etc).  Include figures
      illustrating the stack layout of new, suspended and active threads.
    - Relation between thread switching and thread creation.  New threads
      require same stack/state as existing-but-switched threads, etc.
      Switching within interrupt context requires that new threads inherit
      that same context and stack layout.  Also requires deferred
      thread-switch, until after PIC EOI.  Ideally, new thread should look
      like any other, so must provide a consistent state/stack/etc., for
      new threads + existing threads.  Consider the mechanics of context
      switch first, so that the requirements for the stack layout of a new
      thread are more obvious.
    - Switching page tables.  Avoid where possible to minimize TLB misses,
      etc. (e.g., when switching between threads within the same address
      space)
    - Managing the floating-point registers.  Bit in CR0 to indicate stale
      FPU state.  Lazy FPU save/restore, since most threads will not use
      FPU.
- Yielding the processor.  Simple function, but should be one of the first
  thread methods implemented because of its utility in thread deletion,
  suspension, etc.
- Thread deletion.  Threads cannot (completely) destroy themselves; an
  external entity/thread must reclaim them eventually.  Thread suspension
  is also a little tricky.  Once a thread is queued for deletion (or
  suspension), do not re-examine the queue while that thread is still
  executing.  The enqueue-and-cleanup logic must be atomic, or must run in
  the context of another thread.
- Sleeping/suspending.  Data structures: queue; calendar queue; binary
  heap; timing ring.
- Development tip: examine all thread-management methods from two
  perspectives - as if the method was called by the thread itself; and
  also as if the method was called by an external thread manipulating the
  thread in question.  Some methods may not support both models (e.g.,
  yield vs sleep).
- TSS usage
    - Updating: ESP0 at least.  Maybe I/O bitmap.
- Data structures: multiple run queues vs data heap, etc.  Impact on
  performance.  Timing rings.
- Idle thread.  Maybe required, depending on how Scheduler is implemented.
  Need one idle thread per processor.
- Kernel timers.  Deferred execution, but execute in arbitrary thread
  context.  Similarity/reuse for deferred interrupt handling.
- Synchronization mechanisms.  Need at least two: busy-wait (spinlock) and
  sleep-wait (semaphore/mutex).  On SMP, need a interrupt-spinlock as
  well; spinlock and interrupt-spinlock are same for uniprocessor.
  Include chart of which mechanism is appropriate for what usage (e.g.,
  interrupt-spinlocks are for protecting data that is accessed by both
  interrupt- and non-interrupt paths).
- Random implementation detail: the page-fault handler may need to force
  context-switches, so it's helpful if it has direct access to the
  scheduling function.
- fork() vs. creation from scratch.  POSIX API.  Impact on porting
  applications from one model or the other.
- Security.  Capability model?
- POSIX signals?  How to implement?



## Memory Management
A word of warning - this is probably one of the most complicated subsystems.
Lots of data structures + policies.  You really want to think about your
design here.  As opposed to the rest of your OS, where you can just make it up
as you go along.

- List of tasks, problems solved by the memory manager: kernel allocation
  (heap + pages); DMA mapping; address-space management; page-faults; PCI
  mapping; etc.
- Memory layout, memory model. Memory holes (ISA hole at 640KB - 1MB, PCI
  hole at upper end of 4GB, VESA hole at 15-16MB?).  Holes make physical
  memory at those addresses unusable.  Layout of physical memory vs layout
  of virtual memory.  GRUB reports memory regions, but don't know how
  effective/useful this is.
- Portion of 4GB address space reserved for kernel, shared across
  processes; this is not the only possible layout, though.  Kernel in high
  address range vs low address range.  Impact on "virtual 8086" mode.  See
  UCLA OS class notes.  Low range is probably a little easier since
  portions of kernel can be identity-mapped and can enable paging later in
  the boot process; but this can make it slightly more difficult to move
  (raise) kernel boundary later - e.g., if applications are based at the
  low-end of the user address space.  Alternately, put applications at
  highest-end of address space + work downward, etc.  High range requires
  that VM be enabled immediately; low range allows this to occur later.
- Taxonomy of kernel allocation: cache/pool of small blocks vs full pages.
  Paged memory, nonpaged memory, DMA-accessible memory.  Memory
  pools/regions: paged, non-paged, per-CPU.
- Clarify Intel nomenclature: logical address vs linear address vs
  physical address.  Logical = linear if flat memory model.  Linear =
  physical if paging disabled.
- Page directory/table layout + usage.  Page sizes vs page directory usage
  vs TLB usage.  TLB management.
- Mechanics of page replacement.  Random: this handler probably requires
  access to the scheduling function.  Show example flowchart.  More
  complicated than you would expect.
- Disk driver required for VM.  Maybe filesystem driver, too.  Where does
  paged data go?  Special file in existing filesystem?  Dedicated
  partition?  Implications of each.
- Sharing memory between processes
- Auxiliary data structures required for managing pages.  Physical memory:
  stacks of single pages, bitmaps or hierarchical bitmaps.  How well do
  the various structures scale up to cover 4GB?  Virtual memory: list or
  balanced tree of address space regions.  This latter structure
  determines how page faults are handled.  Reverse page table for shared
  memory.
- Preexisting allocators: Hoard, Doug Lea, etc.  Mention Wilson's paper as
  a source for allocator algorithms.  And Berger's paper concludes that
  well-designed general purpose allocators (e.g., the Lea allocator) will
  outperform most custom allocators.
- This is another good place to keep statistics: frequency of page faults;
  pool usage; memory fragmentation (memory requested vs size of block
  allocated), etc.
- Include list of various structures you'll need: page dir/tables; address
  space; "regions" of process address space, organized into lists/trees;
  inverted page table; page descriptors; pools/bitmaps; list of virtual
  pages; physical pages; swap pages.
- Include a small section of numbers:
    - 1024 entries per page dir = 4GB of address space
    - 1024 entries per page table = 4MB of address space
    - 1MB of address space = 256 pages = 256 page-related structures.
      Keep this in mind if your hardware has 256MB of memory, etc.
- Master kernel page directory.  Sharing kernel page tables with process
  page directories.  Include figure of shared kernel page tables (e.g., 2
  different page directories sharing the same kernel page tables).  This
  doesn't necessarily eliminate need for master kernel page dir.
- DMA support: physically contiguous memory.  Disable caching, etc.  No
  map registers on x86, but different architectures implement this
  differently.  Virtual address vs physical address vs bus address.  ISA
  DMA = 64KB aligned, up to 64KB contiguous memory under 16MB @check these
  numbers against floppy specs.
- Some portions of the Intel architecture are fairly simple to understand
  and program.  Other features are more complicated.  The memory
  management architecture is the latter.  It's not necessarily
  complicated, but there are a lot of related bits scattered about the
  micro-architecture and documentation.  GDT.  CR0.  CR3.  CR4.  Page
  dirs/tables. MTRR's.  Intel recommends cache-enabled and write-back
  policies (not write-thru).  The "Memory Cache Control" chapter in the
  Intel CPU docs is very useful.
- TLB management.  Shoot down TLB when updating, modifying, deleting any
  existing page dir/table entries, `invlpg` instruction + syntax (pointer
  dereference).  TLB entries not affected by Global flag in page table
  entry, can still shoot down any entry marked Global (behavior documented
  for P4 and later?).  On SMP, TLB's must be updated on all CPU's, unless
  TLB is known to be local on one CPU.  COW is a good method for testing
  TLB shootdown, because it involves a read-modify-write to a live page
  table entry.  In theory, you can't do COW without a TLB flush.  If you
  get an infinite loop of page faults on COW, then TLB shootdown is
  missing/broken.  @@unsure of this: unable to trigger this behavior in
  either simulation or on real hardware.
- Differences between user mode memory mgmt and kernel mode memory mgmt.
  Different allocators and strategies for each.  User code is less
  important, can request large quantities of memory without actually using
  it, etc.  Just because your .text section is 32MB doesn't mean you
  automatically allocate 32MB of physical RAM.
- Implementing COW.  As a trick, pre-allocate/reserve a virtual address for
  mapping COW entries since you'll always(?) need two virtual pages here -
  one to map the new frame that will replace the old shared frame; and one
  to map the old/current frame that was shared and triggered the COW.  Can
  even cache the PTE pointer, too.



## Device Drivers + Loadable Modules
- Driver model
- Dynamically loading + unloading drivers/modules.  Handling load-time
  linking, import/export fix-up's.  Note that this means that functions
  that are unused in the base kernel image might be used by
  dynamically-loaded drivers.  Do not enable link-time removal of unused
  functions.
- Using GRUB to load modules for you (idea: use GRUB to load native
  file-system drivers along with kernel; simplifies kernel and allows it
  to run on top of multiple file systems.).


## File System
- Disk driver
- File system driver
- File formats (ELF, a.out, PE).  ELF is simple but flexible, allows
  dynamic loading + linking.  PE/COFF seems more complicated.  Mention the
  Linkers + Loaders book.


## Device Management
- Bus management (PCI, ISA)
- Device identification
- System DMA?
- Memory-mapping devices into memory.  I/O ports vs memory mapped.


---


# SECTION 3: USER MODE

## User Mode Init
- Initial jump-to-user logic.  Where is user code loaded from?  RAMDISK vs
  loaded from disk.  Microkernel: former option is required.
- Who sets up the user stack?  User heap?  Where is argument vector (argc,
  argv) stored?  Who clears the .bss?
- Maybe include a figure of typical memory layout: .text + .data sections,
  followed by heap (growing up), stack (growing down).
- Startup logic for user apps.  GNU linker (ld) looks for a "start" symbol
  as the entry point, by default.  So build a small .o that starts with
  "start" symbol, which you link with every executable.  This becomes your
  "crt0" code.  The "start" code can basic setup (stack, heap, etc) and
  then call program `main()`.  Or use .a archive if you force it to be
  included with ld -whole-archive.
- I used a raw binary/executable to load the user mode bits from ramdisk.
  No ELF or other file packaging.  Kernel just jumps blindly into the
  executable code. One tidbit: used objcopy to copy/convert ELF executable
  to raw code, but this loses the .bss information, so you need to be
  smart about this.  Either use no .bss data (hard to enforce if you're
  using third-party lib or `malloc()`) or tack on some additional
  uninitialized space at the end of the raw image.  Something like:
    - `dd if=/dev/zero of=$@ oflag=append bs=4096 count=2 conv=notrunc`


## Application Libraries and System Calls
Here, assume you have a working kernel that can handle interrupts.  So now,
you want to add system calls, too.  You probably need (at least) two
libraries: a standard C library; and a library for issuing system calls to
your kernel.

### System calls:
- List of Required/likely system calls?  Varies depending on OS structure
- Implementing system calls.  User side: data structure, registers or
  stack (call gate).  Portability concerns, ease of usage.  Soft-interrupt
  vector.  Kernel-side: another series of interrupts.  Security concerns -
  user code calling into kernel, so kernel must protect self from
  faulty/malicious user code.  Watch for bad pointers, etc.  Do not touch
  spinlocks, allocate memory, references, until args are known to be valid
  pointers, etc.  Pros and cons of the various approaches:
    - Traps vs call-gates.  Call-gates are interesting -- they eliminate
      some of the argument checking (because data is already on stack:
      can't pass NULL, args are always large-enough), but data flow is
      one-way.  Not portable.
    - Intel has SYSENTER for better performance instead of traps.  AMD has
      SYSCALL?
    - registers (like Linux) have fewer(?) security problems, but
      potentially create weird stack layout and increases register
      pressure - may not enough regs on x86 (Linux has this problem).
    - Stack approach (like BSD).  Probably cleaner.  Harder to
      implement/maintain in both directions?  How to guard against bad
      syscall (too few/many args)?  Can't use call-gates.
    - Ptr to struct (dx approach).  Simple.  But security problems since
      struct is shared with user space.  Pin/unpin pages to prevent
      inadvertent swap or free. Windows has `MmProbeAndLockPages()`?

### libc
- Importance/significance of the C library.  Abstracts your system calls,
  OS details.  Your C/POSIX library is the key to porting existing
  applications to your platform.
- Implementing LIBC.  Free, open-source options.  The risk of
  using open-source libc is that they may bring along excessive baggage or
  implied usage model (e.g., linux system calls, posix support, etc).  But
  look at newlib (supposedly easy to port, and basis for GCC toolchain),
  uClib, dietlibc, etc, anyway before making that decision.  I wrote my own
  libc, mostly grew it organically as I needed additional functionality; but
  I ultimately
  spent a lot of time here, so not sure that this was the best solution.
- Maybe include a table of C functions + how they potentially interact with
  kernel, system calls.  Some pieces of C library can be shared with
  kernel (e.g., memcpy, strlen are just utility functions with no
  user/kernel dependencies).
- User-mode memory allocator.  Doug Lea allocator, Hoard, others.
- POSIX is a good target here, at least in theory.
- math support in particular s difficult; do not even attempt to build your
  own.  Try newlib, or fdlibm, etc.


## Creating New Applications
- Link with your custom startup .o file + C/syscall libraries.


---


# SECTION 4: OTHER TOPICS

## Testing + Debugging Options
- Unit-testing.  Difficult due to all-encompassing nature of the kernel,
  but possible for small pieces here and there: C library, etc.
- POST card (Ultra-X, PC Engines)
- Keyboard LED's
- Serial port debugging?  GDB defines stubs for doing remote debugging -
  must provide some primitives on the host side and then incorporate some
  GDB code; but then this allows remote GDB debugging.
- Bochs.  MS virtual PC.  VMware.  AMD SimNow.  Boot-from-emulated-floppy
  option (see .vfd notes below).  The strength of emulation is also its
  weakness: it's *emulation*, not real hardware.  If you're relying on
  emulation and/or virtualization for testing, then it's probably a good
  idea to use more than one, to get the benefit of multiple
  implementations/interpretations of the Intel architecture.  New IA
  processors provide hooks for virtualization, so the issue is less
  critical there probably.  @@some overlap with the "hardware/emulation"
  discussion above.
- If loadable modules are available, create a "system test" driver that
  exercises various pieces of the kernel.  See notes on
  hardware/emulation, too.
- Look back at "Month of kernel bugs".  "Fuzz-testing" tools.
- For testing, one simple option is to use virtual floppy drives/images
  (.vfd's) + virtualization.  Floppy format is simple - single partition,
  easy layout, etc.  So for example: (a) mount the floppy/FAT images under
  loopback in Unix/Linux; (b) free Windows tools/drivers for mounting
  virtual floppy drives; or (c) floppy FAT12 layout is simple enough to
  write your own editing tools.  Better solution: use floppy image built
  with ext2 filesystem.  Loopback still works in Linux, but can use
  e2fstools package in cygwin without any special drivers.  You lose the
  ability to manage/edit the floppy via Windows, though.


## Potential Enhancements, Ideas, Future Work
Random other topics:

- Multi-proc/SMP support
    - Synchronization, spin-locks
    - Per-CPU descriptors
    - APIC, inter-processor interrupts
    - Cross-processor coordination (e.g., manipulation of thread executing
      on different processor).
    - Performance, avoiding cache contention/thrashing.  Per-CPU
      structures, memory pools, etc.

- Security?

- Tips and tricks?  Cheap optimizations?  Maybe "performance" deserves
  its own chapter?
    - Kernel sections.  Dependency on specific compiler and linker.
    - Cache management.  Use of code sections to improve locality.
    - Data alignment.  E.g., stack alignment for aligned memory access.
      Data alignment for better cache utilization.  Layout of data
      structure to optimize cache-usage (although, see comments on
      drawbacks of C++, re: object layout).  Tool for dumping object
      sizes, field offsets?
    - Careful use of inlining to minimize code footprint.  Saves
      call/return sequence, but increases code size + cache footprint,
      etc.
    - Like inlining: merge common code in interrupt stubs into single
      sub-stub.  Need actual numbers, but roughly: saves ~22 bytes per
      vector, 36 vectors = 800B of instruction space = about 25
      cache-lines.
    - Data structures.
    - Avoiding reloading page tables to prevent TLB invalidation.  4M
      pages save TLB entries if you can use them.  Mark TLB entries as
      "system/common" on newer IA processors.
    - Per-CPU structures.  Hoard memory allocator?
    - gcc tricks: `__builtin_expect()` and `__builtin_prefetch()`.
      `__builtin_return_address()` is handy for debugging and less
      error-prone than walking up the call-stack by hand.
    - g++ issues:
        - If you dump the symbols for a c++ binary, you'll notice two
          copies of every ctor and dtor - the "in-charge" version (direct
          allocation) and the "not-in-charge" version (indirect allocation
          via a subclass).  Bloats C++ image.  Strip unnecessary code if
          not using loadable modules?
    - Push init code or relatively-infrequent code into separate code
      segment to avoid cache pollution.  Then use linker script to
      coalesce these into a single common block.  This is easier with ASM
      code than C/C++.
    - assembly tricks?
        - Smart `pushf; cli; ... sti` sequence
        - `jmp SELECTOR:NEXT_INST` for reloading CS after loading GDT.
        - `push ... and iretd` for reloading CS when jumping to user space.
        - GDT[0] contains own descriptor to self.






