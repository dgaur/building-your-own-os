# Bibliography

Various book, papers and other links relevant to OS development.  I've looked at
most (but not all) of these.  In a few instances, I've added a brief assessment,
too.



## OS design and theory
+ OS Concepts, Silberschatz and Galvin
+ OS Design + Implementation, Tannenbaum.  Less theory than Silberschatz + Galvin, more practical, hands-on.  Micro-kernel focus, obviously.  Uses all four of the x86 protection rings, which is unusual.
+ The Art of Unix Programming, Eric Raymond.  Not really a book on OS development, but it contains some interesting ideas on OS design philosophy.
+ Communicating Sequential Processes, C A R Hoare.  <http://www.usingcsp.com>.  And also: How to Write Parallel Programs.  <http://www.lindaspaces.com/book/>.  Mathematics and formal theory behind cooperating processes + msg-passing.
+ ACM SIGOPS?  <http://www.sigops.org>  Are papers/proceedings online anywhere?



## Analysis + innards of specific OS
+ Linux
	- Understanding the Linux Kernel, Daniel Bovet and Marco Cesati
	- Linux Device Drivers, Corbet, Rubini, GKH.  <http://lwn.net/Kernel/LDD3>
	- IBM's DeveloperWorks series has a number of good articles on Linux internals (<http://www-128.ibm.com/developerworks/views/linux/libraryview.jsp?type_by=Articles>).  Scheduling, memory management, threading, boot process.
	- In particular, all of Tim Jones' "Anatomy of Linux ..." articles. <http://www.ibm.com/developerworks/views/linux/libraryview.jsp?topic_by=All+topics+and+related+products&sort_order=desc&lcl_sort_order=desc&search_by=anatomy&search_flag=true&type_by=Articles&show_abstract=true&start_no=1&sort_by=Date&end_no=100&show_all=false&S_TACT=105AGX03&S_CMP=EDU>.  These cover ELF format, scheduling, loadable libraries, kernel, slab allocator, network stack, etc.
	- IBM DeveloperWorks article on ELF format and library format <http://www.ibm.com/developerworks/linux/library/l-dynamic-libraries/index.html?S_TACT=105AGX03&S_CMP=EDU>.
	+ Linux HOWTO: "From power-up to bash prompt"
+ Windows
	+ Inside Windows NT.  Doesn't discuss all of the low-level details, but still a good description of the NT architecture.  Too much discussion/examples using kernel debugger to examine real system.  Includes specific numbers for scheduling quanta, memory/pool sizes, etc.
	+ The NT Insider.  Available as periodic journal as well as online at <http://osronline.com>.  Articles geared more towards Windows drivers than kernel development, but still some good topics + discussion here.  Includes discussion of various kernel features, etc.
+ L3/L4
	+ "Toward Real Microkernels"
	+ "On uKernel Construction"
	+ "Improving IPC by Kernel Design"
	+ All by Jochen Liedtke.  These are all quite good, but these last two papers in particular should be mandatory reading if you're building a microkernel.  Discussion of core/minimal set of micro-kernel features.  Specific discussion and ideas about implementation.  Not well known, perhaps, but the L4 kernels are/were probably the best proof that micro-kernels need not be slower than monolithic kernels.
+ Other Unixen
	+ Lion's Commentary on UNIX.
	+ Design and Implementation of the 4.4 BSD Operating System by Marshall Kirk McKusick, Keith Bostic, Micahel J. Karels and John S. Quarterman
	+ FreeBSD developer's handbook here: <http://www.freebsd.org/doc/en_US.ISO8859-1/books/developers-handbook>
	+ OSX innards.  <http://www.kernelthread.com/publications/> and <http://www.osxbook.com>.  Amit Singh.
	+ More OSX.  <http://0xfe.blogspot.com/2006/03/how-os-x-executes-applications.html>
+ Misc
	+ Dash OS.  "Performance of Message-passing using Restricted Virtual Memory Remapping".  Another microkernel design with special IPC region.  Distributed system, which makes it more complicated and perhaps less useful.
	+ "An Architectural Overview of QNX".  Dan Hildebrand.  Discussion of QNX design.  Relatively high-level, not much low-level detail and not as useful as L4 papers, but mentions some interesting ideas: multi-part messages, simple IPC mechanism, priority inheritance tricks.
	+ "An Overview of the Spring System".  Mitchell, Gibbons, Hamilton, etc.  The paper + design are OK, but the intro (first page) has some good ideas on design + goals.  What are you trying to solve?  What features of other OS do you keep?  Which do you throw out?
	+ "Roadmap through Nachos".  Narten.  Talks about Nachos OS internals.  A lot of the discussion is around Nachos hardware emulation, assigning Nachos as a class project, etc.  But the sections on Nachos threads + VM pitfalls are useful and applicable to other OS.
	+ Developing your own 32-bit OS (MMURTL book), Richard Burgess.  Original edition appears to be out-of-print but (new?) edition available through IP DataCorp.  Reviews online are mixed: some like it, some are mixed.  Includes own compiler/linker/development tools.  Assessment: some occasional good ideas + content, but weak overall.  Nearly all code is raw x86 assembly, which seems unnecessarily painful (although, isn't this how SkyOS is implemented?).   Terminology is just flat-out wrong in places.  The actual book mechanics are poor: layout, editing, etc.  Hardly any figures/diagrams, just long sections of text + code.  Burgess' has no real (academic/theory) CS background; and it shows.
	+ Netware Overview (<http://www.usenix.org/publications/library/proceedings/sf94/full_papers/minshall.a>).  A little dated but interesting, in part, because it deviates from the "traditional" OS design.  Maybe helps you clarify your own goals/needs, etc.
	+ <http://rebol.com> and <http://rebol.com/article/0375.html>.  Different form of program style?  And associated OS?  OS without installation.
	+ Squeek OS.  Different model - single flat image + hot editing/modification on the fly.
	+ <http://www.research.ibm.com/K42> and <http://www.research.ibm.com/K42/papers/eurosys06.pdf> in particular.  Research OS at IBM.  Heavy SMP/NUMA focus.
	+ JX and Singularity OS.  Memory protection based on managed languages (Java + C#, respectively) instead of relying on hardware MMU.  JX appears to be dead?  Singularity is ongoing at MS.
	+ Unununium.  Python-based OS.  Dead.
	+ Amiga ROM Kernel Manuals -- both "Libraries and Devices" and "Includes and Autodocs."
	+ "Writing Systems Software in a Functional Language".  Haskell, House OS.
	  Interesting.  Mostly an overview, so a little thin at points, but
	  touches on some obvious concerns: structure/register layout; performance
	  and predictability.  <http://yav.purely-functional.net/publications/plos07.pdf>




## Memory management
+ "Elements of Cache Programming Style" paper, Chris B. Sears.   A little dated, but still a good list of techniques + tricks for optimizing cache + TLB usage.  <http://static.usenix.org/publications/library/proceedings/als00/2000papers/papers/full_papers/sears/sears_html/>
+ Memory Management: Algorithms and Implementation in C++, Bill Blunden. Discussion of different allocation algorithms, sample code and performance results/analysis.  The examples are all application-layer allocators, so while the discussion is still applicable to kernel allocators, the code isn't necessarily reusable.  Talks about the x86 memory management architecture (GDT, paging, etc).  Includes basic descriptions of DOS, MMURTL, Linux + Windows memory subsystems, including GDT layout.  Not critical, but interesting.
+ "Reconsidering Custom Memory Allocation" paper.  Emery Berger et al.  Compares the performance of a few different allocators.  Basic idea is that a good general-purpose allocator (the Doug Lea allocator) beats most custom allocators.  Counter-intuitively, they also propose a hybrid allocator of their own design.  Not as useful for kernel-level allocators unless you're planning on something complicated; but it's a good data-point for user-level allocators.
+ "Dynamic Storage Allocation: A Survey and Critical Review" paper.  Paul Wilson et al.  Overview and discussion of various allocator strategies.  Includes some numbers/results.  Portions of the paper are interesting but irrelevant: the history of allocator designs, etc.  Again, probably more useful for application-level allocator, but it's a good read if you want to roll your own allocator.  Later editions of Knuth, vol 1, also refer to this paper.
+ LWN.net series on Cache Management.  <http://lwn.net/Articles/250967>
+ "What every programmer should know about memory".  Drepper.  <http://people.redhat.com/drepper/cpumemory.pdf>.  PDF of LWN series.  GCC optimizations.  Cache + TLB optimizations.  Discussion of physical memory implementation (circuits, etc) is less useful, but still a good read.  Lots of data/numbers.
+ Doug Lea's malloc.  Well-known memory allocator (malloc drop-in replacement).  Supposedly the basis for some of Linux allocation schemes.  Code is public domain. <http://g.oswego.edu/dl/html/malloc.html>
+ Hoard.  Another memory allocation library.  <http://www.hoard.org>
+ <http://www.ibm.com/developerworks/linux/library/l-memory/>.  Very brief overview of memory allocators and various implementations (Lea allocator, Hoard, ref counting, garbage collection, etc).
+ "Optimizing the Idle Task and Other MMU Tricks" paper, Cort Dougan, Paul Mackerras, Victor Yodaiken


## Filesystems
In fairness, I've done virtually no filesystem work to date, so I haven't really
looked at any of these:

+ Practical File System Design with the Be File System, Dominic Giampaolo.  Out-of-print but available online: <http://www.nobius.org/~dbg/practical-file-system-design.pdf>
+ Filesystem hierarchy standard: <http://www.pathname.com/fhs/>.  Reference on standard Unix filesystem layout.


## Hardware management

+ Intel x86 CPU manuals.  <http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html>
+ Intel AP-485: CPUID instruction, identifying processor <http://www.intel.com/content/www/us/en/processors/processor-identification-cpuid-instruction-note.html>
+ Indispensable PC Hardware Book, Hans-Peter Messmer.  Assessment: this book is key.  Describes various system devices that you'll need to work with (PIC, PIT, RTC, disk drives, etc). You can probably find all of this information online somewhere, but this book explains it all at once.  If you only buy one book, this is the one.  @my copy is old now, but presumably newer editions cover newer hardware/devices?
+ Protected Mode Software Architecture (MindShare book).
+ Pentium Pro and Pentium II System Architecture.  Mindshare book.  The Mindshare books seem to be hit-or-miss.  Some are quite good + useful, some are useless.  This one seems pretty good.  It rehashes much of the Intel processor documentation, but seems a little clearer and easier to understand.  Thought the section on memory management + paging enhancements was pretty good.  As a minor nit, the formatting (the section headers, specifically) make it harder to determine where new topics start.
+ Interrupt-Driven PC System Design, Joe McGivern.  Assessment: This book is very dated and a little difficult to read in places, but it covers some of the more subtle aspects of 8259 programming and interrupt handling in general.  If you're doing something unusual, embedded, etc, this might still be a good option.  @Compare to Messmer book?
+ Programmer's Guide to EGA and VGA Cards, Richard Ferraro.  Dated, but good low-level information.  Newer revisions might be better?
+ <http://agner.org/optimize>.  Various references/papers on code optimization, low-level x86 performance tricks.



## Tools
+ Multiboot specification <http://www.gnu.org/software/grub/manual/multiboot/multiboot.html>
+ Linkers and Loaders, John Levine (manuscripts available at <http://www.iecc.com/linker/>).  Details of dynamic linking + shared library management.  Some discussion about the various file formats.
+ <http://www.ibm.com/developerworks/linux/library/l-gcc-hacks/index.html>.  List of "GCC hacks in the Linux kernel".  Some GCC features/tricks/extensions used in the Linux kernel.  Includes the prefetch + likely/unlikely branch prediction extensions, others.
+ "Month of Kernel Bugs".  Discusses various tools for kernel testing.  "Fuzz" testing/tools.



## Data Structures
Interesting, non-obvious or non-traditional data structures.

+ "Performance Analysis of BST's in System Software", Ben Pfaff.  Straightforward paper, but good data.  I selected the tree algorithms + structure for the VM subsystem in part based on this paper.
+ Network Algorithmics: An Interdisciplinary Approach to Designing Fast Networked Devices.  George Varghese.  Mentions interesting ideas about "timing wheels", sort of like calendar queues.
+ "Split-Ordered lists".  Shalev and Shavit.  Lock-free, extensible hash tables.
	<http://www.cs.tau.ac.il/~shanir/nir-pubs-web/Papers/Split-Ordered_Lists.pdf>



## Misc
+ POSIX, UNIX application portability requirements.  <http://www.unix-systems.org>
+ POSIX standards:
	+ IEEE 1003.1 1990, POSIX 1: System API
	+ IEEE 1003.2 1990, POSIX 1: Shell utilities
	+ IEEE 1003.3 1990, POSIX 1: Measuring conformance
	+ IEEE 1003.4 1990, POSIX 1: System API/realtime
+ MIT OpenCourseWare, EE/CS class 6.828, OS Engineering. <http://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-828-operating-system-engineering-fall-2006/>
+ Virtual Machines: Versatile Platforms for Systems and Processing.  Jim Smith.  Haven't looked at this, but it's supposedly the definitive reference on virtual machines.  Aimed more at dynamic/scripting languages?
+ <http://www.cs.ucla.edu/~kohler/class/04f-aos/index.html>.  "Advanced OS" class.  Provides OS skeleton/labs, but intended for real hardware (Bochs) unlike Nachos.  The lab documentation is actually pretty good, talks about x86 architecture, design issues, etc.
+ <http://lwn.net/Articles/217366/>.  Tolerating failure/faults in device drivers.  Includes links to a couple of papers + "Nooks" isolation system.
+ Assembly/HLL interface: <http://lambda-the-ultimate.org/node/2283>
+ Michael Abrash books on graphics + assembly programming.
+ "Passing a Language through the Eye of a Needle".  Lua design +
	implementation.  Not strictly OS-related, but found this interesting; in
	part because Lua is (will be) the preferred tongue of user mode
	applications.  <http://queue.acm.org/detail.cfm?id=1983083>

