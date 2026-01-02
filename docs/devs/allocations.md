---
title: Allocations
---

GPRs and XMMs are statically allocated. This is faster, less bug-prone and easier, and we can afford to do it because x86-64 has 16 GPRs/XMMs while RISC-V has 32 GPRs/Vecs.

The four most used x86 flags (CF/ZF/SF/OF) are also statically allocated to a GPR. While it may seem wasteful, there seems to be a noticeable performance improvement over storing all flags in one register and unpacking them when they need to be used.

The AF/PF and other flags are not allocated to a register and immediately written to memory when their value changes.

The x87 registers are not statically allocated. Instead, we assign FPRs to them as they get loaded and keep track of push/pop operations to avoid having to do excessive loading/storing. Same applies to MMX registers, as their data is shared with x87 registers we do not statically allocate them.

When in JIT code, the following allocation scheme takes place:

| Representation | RISC-V register |
| :----: | :----: |
| `rax` | `x5` / `t0` |
| `rcx` | `x26` / `s10` |
| `rdx` | `x12` / `a2` |
| `rbx` | `x8` / `s0` |
| `rsp` | `x9` / `s1` |
| `rbp` | `x18` / `s2` |
| `rdi` | `x11` / `a1` |
| `rsi` | `x10` / `a0` |
| `r8` | `x14` / `a4` |
| `r9` | `x15` / `a5` |
| `r10` | `x13` / `a3` |
| `r11` | `x16` / `a6` |
| `r12` | `x17` / `a7` |
| `r13` | `x22` / `s6` |
| `r14` | `x19` / `s3` |
| `r15` | `x20` / `s4` |
| `rip` (start of current block) | `x3` / `gp`[^1] |
| `cf` | `x21` / `s5` |
| `zf` | `x23` / `s7` |
| `sf` | `x24` / `s8` |
| `of` | `x25` / `s9` |
| `xmm0` - `xmm15` | `v16` - `v31`[^2] |
| `ThreadState*` | `x27` / `s11` |
| TLS | `x4` / `tp` |
| Host stack | `x2` / `sp` |
| Vector mask register | `v0` |
| Allocatable FPRs for x87 | `ft0` - `ft7` |
| Allocatable Vecs for MMX | `v8` - `v15` |
| GPR temporaries | `x1`, `x6`, `x28`, `x29`, `x7`, `x30`, `x31` |
| Vec temporaries | `v1`, `v2`, `v3`, `v4`, `v5`, `v6`, `v7` |
| FPR temporaries | `ft8`, `ft9`, `ft10`, `ft11` |

!!! note
    Most of the GPR related allocations don't have a meaning behind their allocated RISC-V register and are purely random, except for `rdi`/`rsi`/`rdx`/`r10`/`r8`/`r9` which are allocated to `a0` - `a5`, following the x86-64 syscall ABI. This is currently unused but in a future version will allow saving some instructions when inlining certain syscalls.


[^1]: We compile felix86 with `-mno-relax`, which frees up the `gp` register for us to use in the JIT.
[^2]: Using sequential registers starting with a number divisible by eight allows us to do grouping with eight registers for operations that need it.