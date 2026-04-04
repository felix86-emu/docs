---
title: Thunking
---

Thunking in felix86 is using the host version of some libraries in place of the guest library.

This has a few benefits:
- Better performance, since code in host library is compiled with a compiler
- Better compatibility, since now the userspace driver (USD) has the same memory model as the kernelspace driver (KSD)
- Less stuttering, as the host library doesn't need to be recompiled
- More space in code cache before it needs to be cleared

But also a few considerations:
- Functions almost always have the same signature across architectures, but may have different struct alignment or packing
- Functions that return pointers need to return them in the 32-bit address space for 32-bit apps
- Functions that have variadic arguments need special handling
- If a library uses another library, in many cases both need to be thunked
    - This may require some workarounds, for example libGLX depends on libX11, but libX11 is difficult to thunk
- Callbacks need to be translated to RISC-V code

## How it's done in felix86

In felix86, when a thunked library is `open`ed, the `open` is redirected to a special x86 **overlay** library. This library defines the same functions as the original library, but has special code that tells our emulator to call into host code.

Here's an example function:
```
global glBegin:function
align 16
glBegin:
invlpg [rax]
db "glBegin", 0
ret
```

The `invlpg [rax]` instruction here is the special "call-to-host-code" instruction. When this instruction is reached, the state is saved to memory, `GuestToHostMarshaller` produces a prologue, the call to host code is made, and an epilogue follows. Finally, the state is restored from memory. The name of the thunked function is fetched from the string right after the `invlpg [rax]`.

`GuestToHostMarshaller` takes the name of a function and its signature, and produces a trampoline. This trampoline will setup the arguments and stack (if necessary) and call the host function. After the host function returns, it will move the return value (if any) to the appropriate register.

We keep track of what functions may take a callback. If a function takes a callback, we need to do the reverse: from host code, jump back to the dispatcher to recompile guest code. These `host->guest->host` transitions could happen multiple times. This `host->guest` trampoline is created by `ABIMadness::hostToGuestTrampoline`.

Some libraries may need to do work right after they are loaded. For those, we insert a pointer to a function in the `.init_array`. If some of this work needs to happen in the felix86 side, we need to tell the emulator that the library has loaded. To do this, our `.init_array` function uses `invlpg [rbx]` which notifies the emulator that it has been reached. This is necessary because in felix86 we use the emulated `ld-linux.so` instead of loading the libraries manually.

These overlays are stored by default in `/opt/felix86/lib/x86_64-linux-gnu` and, in the future, in `/opt/felix86/lib/i386-linux-gnu` too. The path to the overlays can be changed with `felix86 -S /path/to/overlays`.

Some libraries are very cumbersome to fully thunk, but we could benefit from thunking only a few independent functions. One such example is `libc.so.6`. Thunking the entirety of libc is an enormous effort, but if we just thunk `memcpy` we could see improved performance in some games. In the future, we can produce an overlay library on the fly, that is a copy of the original library but thunks a few important functions. 