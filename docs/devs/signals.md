---
title: Signals
---

There's important considerations to make when handling signals in userspace emulation.

## Differences

Signal handlers are pointers to machine code. As such, the signal handlers need to be recompiled. This comes with a whole set of problems described below.

## Asynchronous signals

Asynchronous signals can happen at any time, because a different process or thread decided to send a signal to this one, or from `sigqueue`, `tgkill`, or similar syscalls.

### Considerations

We don't want to handle asynchronous signals immediately. We might be inside host C++ code that holds locks, for example. Or, we might be executing recompiled code that is halfway into an instruction.

For example, consider an `add rax, rbx` that needs to compute the `ZF` flag:
```
add t0, t0, s0
seqz s7, t0
```
If a guest signal happens after the `add` and before the `seqz`, the instruction will have partially updated the x86 state. When the guest signal handler returns, it needs to return to a guest address. But since the instruction is halfway finished, we can't return to before or after the `add`.

Another consideration is the fact that if we handle signals immediately, even if we aren't holding any locks and they are safe to handle, if we are currently in C++ code we may end up leaking the stack. We can't safely jump to the dispatcher. It could be the case the stack was decremented. We *could* mask signals before running any C++ code, but then we'd have performance considerations.

Finally, it's important to note that we don't want to handle signals inside the host signal handler, even if we are in JIT code at the time of the signal. If the guest signal handler doesn't `sigreturn`, but instead uses `longjmp` (or similar context switch instead of returning), this is undetectable from the emulator side. This means that the guest signal handler never returns, which would mean the host signal handler would never return, leaking stack.

### Current solution

Currently, we always defer asynchronous signals until **safepoints**. The `siginfo_t` struct is added to a flat array[^1], and the pending signal is a set bit in a `uint64_t`. A page allocated right before `ThreadState` is set to `PROT_NONE`. This allocation allows us to access it via `-8($s11)`. Each safepoint is an instruction that stores `x0` to the page. When there's no deferred signals, the page is set to `PROT_READ | PROT_WRITE`, and the safepoints do nothing. When signals are deferred, a SIGSEGV happens. In it, the page is unprotected, the signal frame is prepared, the signal mask for inside the signal handler is set, and the x86 registers are set. The statically allocated registers are updated to reflect these changes (e.g. `a0` which maps to `rdi` becomes equal to `sig`, `s1` which maps to `rsp` points to the signal frame). The guest `rip` is set to the guest signal handler. Finally, the host `pc` is set to the [dispatcher](../general/glossary.md#d). This way, when the host signal handler returns, the dispatcher will jump to the signal handler, and the x86 program will think it is now inside the signal.

If a signal happens during a syscall like `sigsuspend`, the host `sigsuspend` syscall will return `-EINTR`. We still defer as usual. Right after the syscall, we insert a safepoint and update the `rip`. If a signal happened during the `sigsuspend`, the safepoint will trigger. The `sigsuspend` will have set `rax` to `-EINTR` already inside the syscall function. Since we update the `rip` before the safepoint, from the guest signal handlers perspective the behavior is correct. The `rip` in `ucontext_t` points to right after the x86 syscall instruction, and `rax` is `-EINTR`.

When a signal handler is finished, it returns using the `rt_sigreturn` syscall. We handle this syscall by setting `ThreadState` values using the `ucontext_t` in the stack. Register values and the signal mask are restored, and the `rip` is set too. After each `syscall` or `int 0x80` instruction we check if the `rip` changed inside the syscall (which only happens in sigreturn) and if so, jump to the dispatcher.

With this solution, there's no problem of state being torn because the signal happened during recompiled code. There's also no problems when we are in host code, as asynchronous signals are always deferred and only handled in safepoints. Finally, there's no issues if a signal handler uses `longjmp`, as we always handle signals outside the host signal handler or C++ code.

### Problems

This solution isn't perfect.

#### Not handling the signal immediately is not accurate

If a program spawns a thread, and that thread spams its parent with signals, it could be the case that the parent has a hard time reaching the safepoint. It spends a lot of time on deferring signals, and by the time it's finished deferring one, another one is queued. This happens because there's a period from receiving the signal to servicing it at the safepoint, where signals of the same variety aren't masked as they should be. We mask signals right before enterring the signal handler at the safepoint.

**But why not mask them right when you defer?**

While this might solve this particular problem, it's hard to properly implement. What if an asynchronous signal of another variety happens before you reach the safepoint? What if a synchronous signal happens? What if a process is spamming us with asynchronous SIGSEGV signals which we can't mask because we use them for emulator purposes?

**What can be done about this?**

Not much for a perfect solution.

One idea could be to use a RISC-V interpreter to interpret code until a safepoint, inside the host signal handler, and immediately service the guest signal upon return from the host handler. This could work for when a signal happens during JIT code, but not when it happens outside it. For a perfect solution, we would perhaps have to mask signals during non-JIT code segments.

Our current implementation can run programs that execute signals very frequently, like **Golang applications** with async preemption enabled. Since this problem (AFAIK) only happens in tests and not real-world applications, it is ignored for now.

## Synchronous signals

Synchronous signals happen due to an instruction. For example, dividing by zero or accessing a nullptr. Synchronous signals are rarer, but programs like emulators may use signals like SIGSEGV to help with emulation, for example Dolphin can use SIGSEGV to detect memory accesses to I/O components and patch them with a function call.

### Considerations

Synchronous signals can't be deferred, they need to be handled immediately, as they happen because of a specific instruction. This gives us an interesting problem. Our `gp` register points to the `rip` at the start of the block (usually, it may point to right after a syscall for reasons mentioned above), so how do we get the `rip` of the actual instruction that faulted?

### Current solution

Currently, we use a vector that maps each x86 `rip` to a RISC-V `pc` for each block. The faulting block is fetched using a map and `lower_bound`.

### Problems

While this allows us to fetch the actual `rip` from the faulting `pc`, it has a memory overhead. Perhaps a better solution would be identifying all possible faulting points and updating the `gp` register right before they happen. However, this could incur a performance penalty. Another possibility would be mapping all possible faulting `pc` values to their `rip` directly. This should have lower memory overhead, at the expense of having to identify all possible faulting locations.


[^1]: Realtime signals may have multiple of the same signal being queued. We currently don't handle this behavior and only queue the latest, which is the correct behavior for non-realtime signals.