# Control Flow Integrity

This project captures research to effectively fix the lack of underlying control flow enforcement that would prevent memory corruption exploitation. This mechanism does not exist today but could be implemented in the future by the IT industry. 

## Hardware Control Flow Integrity (CFI) for an IT Ecosystem
This paper is a brief introduction to the problem of memory corruption and a description of one way to prevent control flow hijacking. It also includes a discussion on the issues that may be encountered in the IT ecosystem when an architectural change like this is introduced.

* [Download PDF](https://github.com/iadgov/Control-Flow-Integrity/raw/master/paper/Hardware Control Flow Integrity for an IT Ecosystem.pdf)
* [View PDF on GitHub](https://github.com/iadgov/Control-Flow-Integrity/blob/master/paper/Hardware Control Flow Integrity for an IT Ecosystem.pdf)

Additionally, Intel recently disclosed an x86 instruction specification (CET) that closely resembles Landhere. 

* [View spec at Intel](https://software.intel.com/en-us/isa-extensions/cet-preview)

Questions or comments can be sent to control-flow-integrity@nsa.gov or [submitted](https://github.com/iadgov/Control-Flow-Integrity/issues/new) to [our GitHub issue tracker](https://github.com/iadgov/Control-Flow-Integrity/issues).

## Code 

The [code folder](https://github.com/iadgov/Control-Flow-Integrity/tree/master/code) has examples of software that would leverage the hardware described in the paper. It is hoped that researchers can learn more about the effect and strength of the proposal by reverse engineering and performing static analysis on them. Perhaps demonstrate a way to bypass the mitigation and report via mechanisms described above. The files are:

* 2 source files that have a buffer overflow of either a return address or a function pointer (i.e. stack-return.c and stack-functionptr.c).
* 2 Linux binaries of the overflow source files instrumented with new landing point opcodes (i.e. stack-return and stack-functionptr).
* 2 instrumented Linux libraries that were linked to the example buffer overflow binaries (ld-linux-x86-64.so.2 and libc.so.6).
* 1 binary of ls, just as a typical Linux utility that has been instrumented with landing point opcodes.

These simple examples should allow one to explore the impact of the CFI countermeasure in a process address space. The binaries should run on any x86 Linux machine. The opcodes will work as NOP's. The landing point opcodes are as follows:

* CLP: 0x0f,1f,40,aa
* JLP: 0x0f,1f,40,bb
* RLP: 0x0f,1f,40,cc

The binaries contain no fine grained label checks. They only have a corresponding Landing Point instruction to any indirect branch as a label, which is coarse grained. If other binaries are desired, we can produce them, if source is provided.

One can extract the gadgets (as defined on line 271/2 in the paper) from the binaries and attempt to chain them together. Note: RLP gadgets are of no semantic use due to the (imaginary) shadow stack. So gadget chains can only contain CLP and JLP based gadgets. To test validity of a claim, one can use gdb to "run" a gadget chain. First one can manually change the memory as an exploit might do by setting a breakpoint at the appropriate place and performing the overwrite(s). Then continue using single stepping. As a substitute for the HW enforcement, whenever an indirect branch occurs one can visually validate it lands on a landing point. If it reaches the goal (e.g. exec("Your string")) without crashing the application, you win and this form of CFI loses. There is no secret right answer. We believe it's not possible or extremely unreliable to bypass the minimal CFI design (line 116 in the paper) with these code samples (and others like them). Unfortunately, this is the best dynamic tool we can offer for now to allow independent validation.

# Other research:

Galois has taken the time and effort to implement a full Linux build of Landhere+ShadowStack concept. This includes instrumented binaries and a VM to create Landhere binaries on one's own. http://landhere.galois.com/

Also at the Galois site is a fully instrumented landing point Linux file system that we have used as an exemplar for gadget analysis. As a convenience, we have extracted all gadgets from the Galois file system (https://github.com/iadgov/Control-Flow-Integrity/releases/tag/v1.0.0). The resulting gadgets are 50 instructions or less and contain 5 or less pre-conditions (e.g. conditional branches). A gadget is effectively a trace as it would happen if certain conditions were true. Each gadget is represented as a textual assembly listing. There can be multiple traces from a single landing point. If there's a gadget with an indirect function in the middle, that branch is treated as a stub and the trace falls through to the subsequent instruction, an RLP. Gadgets can have nested function calls/flows.

The tarball contains two main subdirectories, exes and libs which reflect the executables and libraries from the Galois set. Each executable is further broken down into raw and filtered. The raw files contain all possible gadgets. The filtered files are de-duped (logical and binary) and are labeled for their chaining purpose (return, link, prestitch, poststitch, dispatch loop, atomic). Return is a gadget that "returns", typically it's a real function. It's intended to be used with the dispatch loop but can also be substituted for a call\*. Dispatch loop is a quasi gadget that is used to chain multiple return gadgets (i.e. "functions"). Link is a forward flowing gadget that does not return (gadget block exit is either a call\* or jmp\*). Atomic is the minimal size gadget for an exiting free branch (return, call\* or jmp\*). Namely it has the closest preceding landing point. Many of the gadgets contain multiple landing points because the size limit was upto 50 instructions. These non-atomic gadgets are included to illustrate side effects that might, or might not, be useful since one is not restricted to branching to the nearest landing from a free branch. Pre and post stitch gadgets are a sub-genre of link gadgets that can be combined if the stack pointer change is neutral to the shadow stack.

## License
This Work was prepared by a United States Government employee and, therefore, is excluded from copyright by Section 105 of the Copyright Act of 1976.

Copyright and Related Rights in the Work worldwide are waived through the [CC0 1.0 Universal license](https://creativecommons.org/publicdomain/zero/1.0/).

## Disclaimer of Warranty
This Work is provided "as is". Any express or implied warranties, including but not limited to, the implied warranties of merchantability and fitness for a particular purpose are disclaimed. In no event shall the United States Government be liable for any direct, indirect, incidental, special, exemplary or consequential damages (including, but not limited to, procurement of substitute goods or services, loss of use, data or profits, or business interruption) however caused and on any theory of liability, whether in contract, strict liability, or tort (including negligence or otherwise) arising in any way out of the use of this Work, even if advised of the possibility of such damage.

The User of this Work agrees to hold harmless and indemnify the United States Government, its agents and employees from every claim or liability (whether in tort or in contract), including attorneys' fees, court costs, and expenses, arising in direct consequence of Recipient's use of the item, including but not limited to, claims or liabilities made for injury to or death of personnel of User or third parties, damage to or destruction of property of User or third parties, infringement or other violations of intellectual property or technical data rights.

Nothing in this Work is intended to constitute an endorsement, explicit or implied, by the United States Government of any particular manufacturer's product or service.

## Disclaimer of Endorsement
Reference herein to any specific commercial product, process, or service by trade name, trademark, manufacturer, or otherwise, in this Work does not constitute an endorsement, recommendation, or favoring by the United States Government and shall not be used for advertising or product endorsement purposes.
