---
title: Debugging
---

Debugging is inherently more difficult than most apps, as most of the time you are trying to debug the guest application that is running through an emulator.

## Important info

The emulated state of each process exists in the struct `ThreadState`. "Process" here refers to all cloned/forked processes, whether they are an actual "thread" or not.

This state is statically allocated in the GPR `s11`.

!!! Tip
    To inspect the current thread's ThreadState object, you can run `p/x *ThreadState::Get()`.

!!! Warning
    The register state in this struct may not be fully up to date, if the currently executing code is recompiled code. Some data may be modified in their respective RISC-V registers. To get their value you'd have to inspect the corresponding statically allocated host register. See [allocations](./allocations.md).

When in JIT code, the current block `rip` is stored in `gp` register, viewed as `p/x $gp`. When not in JIT code, it can be viewed as `p/x ThreadState::Get()->rip`.

Logs for every run are stored in `/tmp`. If felix86 is started with `FELIX86_QUIET` or through `felix86 --shell`, no logs will be generated. You can enter a shell with `felix86 /path/to/rootfs/bin/bash` to enable logging.

## Useful functions

When built as `RelWithDebInfo`, some functions can be called from gdb to assist with debugging.

The `disassemble(u64)` function can disassemble x86 code at an address. When in JIT code, you can view the disassembly of the current block using `disassemble($gp)`.

The `print_address(u64)` function can print the symbol, if any, at an address.

## Useful configurations

The `FELIX86_CALLTRACE=1` environment variable can generate a calltrace (pushing on calls, popping on rets) that can be printed through the function `dump_states()`, or when an error happens.

The `FELIX86_SLEEP_ERROR=1` environment variable will sleep the thread and print the PID on exit to allow for gdb to attach and get a backtrace.

The `FELIX86_CAPTURE_SIGSEGV=1` and `FELIX86_CAPTURE_SIGABRT=1` environment variables work similarly. Some applications rely on guest SIGSEGV signals being handled correctly.

The `FELIX86_ABORT_SIGSEGV=1` environment variable will raise a SIGABRT on guest SIGSEGV. This is useful for generating a core dump if core dumps are enabled.

The `FELIX86_ALIGNMENT_CHECK=1` environment variable will insert alignment checks on every atomic instruction to catch unaligned atomics that aren't properly handled.

The `FELIX86_PRINT_SIGNALS` environment variable will print all guest signals.

The `FELIX86_PRINT_ALL_SIGNALS` environment variable will print all signals, including ones used by felix86.

The `FELIX86_STRACE` environment variable will print info about all syscalls.

The `FELIX86_STRACE_ERRORS` environment variable will print info about all syscalls that return an error.