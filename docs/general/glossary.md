---
title: Glossary
---

Some of these terms have specific meanings for felix86 that may not carry over to other projects.

#### A
* **Address cache**: A faster guest-to-host translation method that is implemented in RISC-V assembly. The following pseudocode explains how it works:
    ```c++
    Translation trans = address_cache[guest_address & 0xFFFF];
    if (trans.guest_address == guest_address) {
        return trans.host_address;
    } else {
        // Perform an unordered_map lookup and fill address cache entry
        return slow_translation(guest_address);
    }
    ```

* **AVX**: Advanced Vector Extensions, the successor to the SSE extensions.

#### B
* **Block**: A basic block. A sequence of x86 instructions with no control flow until the last instruction of the sequence.

* **Block linking**: A common JIT optimization where blocks that jump to a known address jump directly to the next block instead of performing address translation

#### C
* **"Callret"**: A relatively common JIT optimization where return addresses are predicted and the host return stack buffer is used.

* **Code cache**: A buffer containing the recompiled code.

#### F
* **Fake mount**: A mechanism within felix86 that allows directories to appear as mounted without actually mounting them or using root permissions. See [rootfs information](./rootfs-information.md).

#### I
* **Invalidations**: Blocks become invalid when their x86 instructions are modified (see SMC). When invalidated, blocks that link to them need to be unlinked and the invalid block needs to be compiled again.

* **Inline SMC**: SMC that changes instructions in the current block. Needs extra work to properly handle.

#### M
* **MMX**: Older and less frequently used way to do SIMD in x86, replaced by SSE. Uses the same registers as x87 registers. Rarely used but we need to emulate it regardless.

#### P
* **Profiles**: A way to fully or partially override a specific set of configurations in felix86. Set using the `FELIX86_PROFILE` environment variable as a path to a configuration file.

#### R
* **Rootfs**: See [rootfs](./rootfs-information.md). A directory containing the necessary x86/x86-64 libraries and binaries to run your applications and games.

* **RVA23**: A RISC-V profile, which is a set of ISA features, such as RISC-V extensions. Felix86 does not currently require RVA23.

* **RVV**: RISC-V Vector, a shorthand name of the vector extension, usually referring to version 1.0

#### S
* **SIMD**: Single instruction multiple data. A type of extension that defines instructions that operate on multiple packed values at once. Examples include SSE and RVV.

* **SMC**: Self-modifying code. A recompiler (such as Mono found in Unity games) may modify it's own code to improve it. Felix86 uses SMC to perform block linking. SMC is detected with page protection of recompiled code.

* **SSE**: Streaming SIMD Extensions, an x86 extension that adds XMM registers and SIMD instructions.

#### T
* **Thunks**: Special x86 libraries that call into host RISC-V code for performance.

* **Trampolines**: Small pieces of assembly code that perform x86 to RISC-V or RISC-V to x86 ABI translations and jump to the recompiled code. The former is used for thunks, the latter is used for x86 callbacks.

* **TSO**: Total store ordering, the x86 memory model. RISC-V by default has a weak memory model, except in the presence of RVTSO (which doesn't exist in hardware).

#### V
* **VDSO**: The kernel maps some frequently used and simple syscalls like `gettimeofday` on the process address space. This speeds up syscall time. Some programs (notably Go) depend on this feature, so we emulate it.

#### X
* **x87**: Older and less frequently used way to do floats in x86, replaced by SSE. Annoying to implement, as it uses 80-bit floats and the registers are structured as a stack.