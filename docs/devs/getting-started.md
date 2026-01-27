---
title: Getting started
---

felix86 is an emulator, and emulators have endless amount of work to be done.

If you've never used felix86 before, I recommend starting by testing it out. [Use the installation script](../users/installation-guide.md) or [check out the building instructions](./building_instructions.md) and try to run a simple program such as `glxgears`.

There's multiple ways to contribute.

## Communication

Join [our discord server](https://discord.felix86.com) to ask development questions.

## Good first issues

There's a `good-first-issues` label on [our issue tracker](https://github.com/OFFTKP/felix86/issues) which contains easy bugs or feature requests for first time contributors.

## Instruction optimizations

We use an IR-less JIT and we go through a lot of work to try to minimize the amount of RISC-V instructions needed to emulate an x86 instruction.

A good starting point is building with REPL support (set `BUILD_REPL=1` during CMake configuration) and testing what the emitted code for each instruction is.

The REPL environment accepts x86 instructions and prints the generated RISC-V assembly. It can accept multiple instructions separated by semicolons.

Example:
```
felix86 --repl
> mov rdi, rsi
MV a0, a1
```

Try to find a non-optimal instruction sequence and optimize it!

## Debugging tips

As of right now, felix86 cannot emulate programs that use `ptrace`, such as `gdb` or `strace`. As such, you need to use the host versions of these commands. [There's some tips to make your life easier](./debugging.md).

Sometimes the host `strace` is useful when chasing bugs in the emulator itself, or want a more complete trace. Make sure to filter out some syscalls that are frequently use and spammy, such as `riscv_flush_icache` and `rt_sigprocmask`. Make sure to capture all processes, using the `-f` argument in `strace`.

## Profiling games

You can make felix86 emit metadata for `perf` using `FELIX86_PERF_BLOCKS=1`.