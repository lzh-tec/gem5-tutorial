---
title: Debugging gem5
doc: Learning gem5
author: Jason Lowe-Power (modified by Siddharth Sahay)
---


Debugging gem5
==============

In the previous chapters we covered how to
create a very simple SimObject. In this chapter, we will replace the
simple print to `stdout` with gem5's debugging support.

gem5 provides support for `printf`-style tracing/debugging of your code
via *debug flags*. These flags allow every component to have many
debug-print statements, without all of them enabled at the same time.
When running gem5, you can specify which debug flags to enable from the
command line.

Debug flags are only enabled on *debug* and *opt* variants of gem5 (and not on the *fast* variant). We use the the *opt* variant to get both debug functionality and code optimization (and now you know why it's called *gem5.opt*).

Using debug flags
-----------------

For instance, when running the first simple.py script from
simple-config-chapter, if you enable the `DRAM` debug flag, you get the
following output. Note that this generates *a lot* of output to the
console (about 7 MB).

```
./gem5.opt --debug-flags=DRAM configs/simple.py | head -n 50
```

    gem5 Simulator System.  http://gem5.org
    DRAM device capacity (gem5 is copyrighted software; use the --copyright option for details.

    gem5 compiled Jan  3 2017 16:03:38
    gem5 started Jan  3 2017 16:09:53
    gem5 executing on chinook, pid 19223
    command line: build/X86/gem5.opt --debug-flags=DRAM configs/learning_gem5/part1/simple.py

    Global frequency set at 1000000000000 ticks per second
          0: system.mem_ctrl: Memory capacity 536870912 (536870912) bytes
          0: system.mem_ctrl: Row buffer size 8192 bytes with 128 columns per row buffer
          0: system.remote_gdb.listener: listening fort remote gdb #0 on port 7000
    Beginning simulation!
    info: Entering event queue @ 0.  Starting simulation...
          0: system.mem_ctrl: recvTimingReq: request ReadReq addr 400 size 8
          0: system.mem_ctrl: Read queue limit 32, current size 0, entries needed 1
          0: system.mem_ctrl: Address: 400 Rank 0 Bank 0 Row 0
          0: system.mem_ctrl: Read queue limit 32, current size 0, entries needed 1
          0: system.mem_ctrl: Adding to read queue
          0: system.mem_ctrl: Request scheduled immediately
          0: system.mem_ctrl: Single request, going to a free rank
          0: system.mem_ctrl: Timing access to addr 400, rank/bank/row 0 0 0
          0: system.mem_ctrl: Activate at tick 0
          0: system.mem_ctrl: Activate bank 0, rank 0 at tick 0, now got 1 active
          0: system.mem_ctrl: Access to 400, ready at 46250 bus busy until 46250.
      46250: system.mem_ctrl: processRespondEvent(): Some req has reached its readyTime
      46250: system.mem_ctrl: number of read entries for rank 0 is 0
      46250: system.mem_ctrl: Responding to Address 400..   46250: system.mem_ctrl: Done
      77000: system.mem_ctrl: recvTimingReq: request ReadReq addr 400 size 8
      77000: system.mem_ctrl: Read queue limit 32, current size 0, entries needed 1
      77000: system.mem_ctrl: Address: 400 Rank 0 Bank 0 Row 0
      77000: system.mem_ctrl: Read queue limit 32, current size 0, entries needed 1
      77000: system.mem_ctrl: Adding to read queue
      77000: system.mem_ctrl: Request scheduled immediately
      77000: system.mem_ctrl: Single request, going to a free rank
      77000: system.mem_ctrl: Timing access to addr 400, rank/bank/row 0 0 0
      77000: system.mem_ctrl: Access to 400, ready at 101750 bus busy until 101750.
     101750: system.mem_ctrl: processRespondEvent(): Some req has reached its readyTime
     101750: system.mem_ctrl: number of read entries for rank 0 is 0
     101750: system.mem_ctrl: Responding to Address 400..  101750: system.mem_ctrl: Done
     132000: system.mem_ctrl: recvTimingReq: request ReadReq addr 400 size 8
     132000: system.mem_ctrl: Read queue limit 32, current size 0, entries needed 1
     132000: system.mem_ctrl: Address: 400 Rank 0 Bank 0 Row 0
     132000: system.mem_ctrl: Read queue limit 32, current size 0, entries needed 1
     132000: system.mem_ctrl: Adding to read queue
     132000: system.mem_ctrl: Request scheduled immediately
     132000: system.mem_ctrl: Single request, going to a free rank
     132000: system.mem_ctrl: Timing access to addr 400, rank/bank/row 0 0 0
     132000: system.mem_ctrl: Access to 400, ready at 156750 bus busy until 156750.
     156750: system.mem_ctrl: processRespondEvent(): Some req has reached its readyTime
     156750: system.mem_ctrl: number of read entries for rank 0 is 0

Or, you may want to debug based on the exact instruction the CPU is
executing. For this, the `Exec` debug flag may be useful. This debug
flags shows details of how each instruction is executed by the simulated
CPU.

```
./gem5.opt --debug-flags=Exec configs/simple.py | head -n 50
```

    gem5 Simulator System.  http://gem5.org
    gem5 is copyrighted software; use the --copyright option for details.

    gem5 compiled Jan  3 2017 16:03:38
    gem5 started Jan  3 2017 16:11:47
    gem5 executing on chinook, pid 19234
    command line: build/X86/gem5.opt --debug-flags=Exec configs/learning_gem5/part1/simple.py

    Global frequency set at 1000000000000 ticks per second
          0: system.remote_gdb.listener: listening for remote gdb #0 on port 7000
    warn: ClockedObject: More than one power state change request encountered within the same simulation tick
    Beginning simulation!
    info: Entering event queue @ 0.  Starting simulation...
      77000: system.cpu T0 : @_start    : xor   rbp, rbp
      77000: system.cpu T0 : @_start.0  :   XOR_R_R : xor   rbp, rbp, rbp : IntAlu :  D=0x0000000000000000
     132000: system.cpu T0 : @_start+3    : mov r9, rdx
     132000: system.cpu T0 : @_start+3.0  :   MOV_R_R : mov   r9, r9, rdx : IntAlu :  D=0x0000000000000000
     187000: system.cpu T0 : @_start+6    : pop rsi
     187000: system.cpu T0 : @_start+6.0  :   POP_R : ld   t1, SS:[rsp] : MemRead :  D=0x0000000000000001 A=0x7fffffffee30
     250000: system.cpu T0 : @_start+6.1  :   POP_R : addi   rsp, rsp, 0x8 : IntAlu :  D=0x00007fffffffee38
     250000: system.cpu T0 : @_start+6.2  :   POP_R : mov   rsi, rsi, t1 : IntAlu :  D=0x0000000000000001
     360000: system.cpu T0 : @_start+7    : mov rdx, rsp
     360000: system.cpu T0 : @_start+7.0  :   MOV_R_R : mov   rdx, rdx, rsp : IntAlu :  D=0x00007fffffffee38
     415000: system.cpu T0 : @_start+10    : and    rax, 0xfffffffffffffff0
     415000: system.cpu T0 : @_start+10.0  :   AND_R_I : limm   t1, 0xfffffffffffffff0 : IntAlu :  D=0xfffffffffffffff0
     415000: system.cpu T0 : @_start+10.1  :   AND_R_I : and   rsp, rsp, t1 : IntAlu :  D=0x0000000000000000
     470000: system.cpu T0 : @_start+14    : push   rax
     470000: system.cpu T0 : @_start+14.0  :   PUSH_R : st   rax, SS:[rsp + 0xfffffffffffffff8] : MemWrite :  D=0x0000000000000000 A=0x7fffffffee28
     491000: system.cpu T0 : @_start+14.1  :   PUSH_R : subi   rsp, rsp, 0x8 : IntAlu :  D=0x00007fffffffee28
     546000: system.cpu T0 : @_start+15    : push   rsp
     546000: system.cpu T0 : @_start+15.0  :   PUSH_R : st   rsp, SS:[rsp + 0xfffffffffffffff8] : MemWrite :  D=0x00007fffffffee28 A=0x7fffffffee20
     567000: system.cpu T0 : @_start+15.1  :   PUSH_R : subi   rsp, rsp, 0x8 : IntAlu :  D=0x00007fffffffee20
     622000: system.cpu T0 : @_start+16    : mov    r15, 0x40a060
     622000: system.cpu T0 : @_start+16.0  :   MOV_R_I : limm   r8, 0x40a060 : IntAlu :  D=0x000000000040a060
     732000: system.cpu T0 : @_start+23    : mov    rdi, 0x409ff0
     732000: system.cpu T0 : @_start+23.0  :   MOV_R_I : limm   rcx, 0x409ff0 : IntAlu :  D=0x0000000000409ff0
     842000: system.cpu T0 : @_start+30    : mov    rdi, 0x400274
     842000: system.cpu T0 : @_start+30.0  :   MOV_R_I : limm   rdi, 0x400274 : IntAlu :  D=0x0000000000400274
     952000: system.cpu T0 : @_start+37    : call   0x9846
     952000: system.cpu T0 : @_start+37.0  :   CALL_NEAR_I : limm   t1, 0x9846 : IntAlu :  D=0x0000000000009846
     952000: system.cpu T0 : @_start+37.1  :   CALL_NEAR_I : rdip   t7, %ctrl153,  : IntAlu :  D=0x00000000004001ba
     952000: system.cpu T0 : @_start+37.2  :   CALL_NEAR_I : st   t7, SS:[rsp + 0xfffffffffffffff8] : MemWrite :  D=0x00000000004001ba A=0x7fffffffee18
     973000: system.cpu T0 : @_start+37.3  :   CALL_NEAR_I : subi   rsp, rsp, 0x8 : IntAlu :  D=0x00007fffffffee18
     973000: system.cpu T0 : @_start+37.4  :   CALL_NEAR_I : wrip   , t7, t1 : IntAlu :
    1042000: system.cpu T0 : @__libc_start_main    : push   r15
    1042000: system.cpu T0 : @__libc_start_main.0  :   PUSH_R : st   r15, SS:[rsp + 0xfffffffffffffff8] : MemWrite :  D=0x0000000000000000 A=0x7fffffffee10
    1063000: system.cpu T0 : @__libc_start_main.1  :   PUSH_R : subi   rsp, rsp, 0x8 : IntAlu :  D=0x00007fffffffee10
    1118000: system.cpu T0 : @__libc_start_main+2    : movsxd   rax, rsi
    1118000: system.cpu T0 : @__libc_start_main+2.0  :   MOVSXD_R_R : sexti   rax, rsi, 0x1f : IntAlu :  D=0x0000000000000001
    1173000: system.cpu T0 : @__libc_start_main+5    : mov  r15, r9
    1173000: system.cpu T0 : @__libc_start_main+5.0  :   MOV_R_R : mov   r15, r15, r9 : IntAlu :  D=0x0000000000000000
    1228000: system.cpu T0 : @__libc_start_main+8    : push r14

In fact, the `Exec` flag is actually an agglomeration of multiple debug
flags. You can see this, and all of the available debug flags, by
running gem5 with the `--debug-help` parameter.

```
./gem5.opt --debug-help
```

    Base Flags:
        Activity: None
        AddrRanges: None
        Annotate: State machine annotation debugging
        AnnotateQ: State machine annotation queue debugging
        AnnotateVerbose: Dump all state machine annotation details
        BaseXBar: None
        Branch: None
        Bridge: None
        CCRegs: None
        CMOS: Accesses to CMOS devices
        Cache: None
        CacheComp: None
        CachePort: None
        CacheRepl: None
        CacheTags: None
        CacheVerbose: None
        Checker: None
        Checkpoint: None
        ClockDomain: None
    ...
    Compound Flags:
        All: Controls all debug flags. It should not be used within C++ code.
            All Base Flags
        AnnotateAll: All Annotation flags
            Annotate, AnnotateQ, AnnotateVerbose
        CacheAll: None
            Cache, CacheComp, CachePort, CacheRepl, CacheVerbose, HWPrefetch
        DiskImageAll: None
            DiskImageRead, DiskImageWrite
    ...
    XBar: None
        BaseXBar, CoherentXBar, NoncoherentXBar, SnoopFilter

Adding a new debug flag
-----------------------

In the previous chapters, we used a simple
`std::cout` to print from our SimObject. While it is possible to use the
normal C/C++ I/O in gem5, it is highly discouraged. So, we are now going
to replace this and use gem5's debugging facilities instead.

When creating a new debug flag, we first have to declare it in a
SConscript file. Add the following to the SConscript file in the
directory with your hello object code (`work/src`).

```python
DebugFlag('HelloExample')
```

This declares a debug flag of "HelloExample". Now, we can use this in debug
statements in our SimObject.

By declaring the flag in the SConscript file, a debug header is
automatically generated that allows us to use the debug flag. The header
file is in the `debug` directory and has the same name (and
capitalization) as what we declare in the SConscript file. Therefore, we
need to include the automatically generated header file in any files
where we plan to use the debug flag.

In the `hello_object.cc` file, we need to include the header file.

```cpp
#include "base/trace.hh"
#include "debug/HelloExample.hh"
```

Now that we have included the necessary header file, let's replace the
`std::cout` call with a debug statement like so.

```cpp
DPRINTF(HelloExample, "Created the hello object\n");
```

`DPRINTF` is a C++ macro. The first parameter is a *debug flag* that has
been declared in a SConscript file. We can use the flag `HelloExample` since we
declared it in the `work/src/SConscript` file. The rest of the
arguments are variable and can be anything you would pass to a `printf`
statement.

Now, if you recompile gem5 and run it with the "HelloExample" debug flag, you
get the following result.

```
./gem5.opt --debug-flags=HelloExample configs/run_hello.py
```

    gem5 Simulator System.  http://gem5.org
    gem5 is copyrighted software; use the --copyright option for details.

    gem5 compiled Jan  4 2017 09:40:10
    gem5 started Jan  4 2017 09:41:01
    gem5 executing on chinook, pid 29078
    command line: build/X86/gem5.opt --debug-flags=HelloExample configs/learning_gem5/part2/run_hello.py

    Global frequency set at 1000000000000 ticks per second
          0: hello: Created the hello object
    Beginning simulation!
    info: Entering event queue @ 0.  Starting simulation...
    Exiting @ tick 18446744073709551615 because simulate() limit reached

You can find the reference implementation in `run_hello22.py`, `hello_object22.hh`, `hello_object22.cc`, and `HelloObject22.py`.

Debug output
------------

For each dynamic `DPRINTF` execution, three things are printed to
`stdout`. First, the current tick when the `DPRINTF` is executed.
Second, the *name of the SimObject* that called `DPRINTF`. This name is
usually the Python variable name from the Python config file. However,
the name is whatever the SimObject `name()` function returns. Finally,
you see whatever format string you passed to the `DPRINTF` function.

You can control where the debug output goes with the `--debug-file`
parameter. By default, all of the debugging output is printed to
`stdout`. However, you can redirect the output to any file. The file is
stored relative to the main gem5 output directory, not the current
working directory.

Using functions other than DPRINTF
----------------------------------

`DPRINTF` is the most commonly used debugging function in gem5. However,
gem5 provides a number of other functions that are useful in specific
circumstances.

> These functions are like the previous functions `DDUMP`,
> `PRINTF`, and `DPRINTFR` except they do not take a flag as a
> parameter. Therefore, these statements will *always* print whenever
> debugging is enabled.

### Next Page
[Part 2.3: Events](part2_3_events.md)
