---
title: Frequently Asked Questions
---

# What is a rootfs and how is it used?

In order to use felix86, you need a rootfs -- a directory that contains all the x86 libraries that your programs may need. Upon running, felix86 will perform a fake, **rootless** mount of folders such as `/dev`, `/proc`, `/sys` and others inside the rootfs. Any guest application will solely exist inside the rootfs. For example, if your rootfs is at `/opt/felix86/rootfs` and a program accesses `/home/ubuntu/myfile`, this access will be translated to `/opt/felix86/rootfs/home/ubuntu/myfile` inside the rootfs. This is accomplished by doing full path resolution in userspace, before passing the final resolved and symlink-free path to the kernel. As a consequence, filesystem accesses such as `/..`, or symlinks that point backwards will not escape the rootfs.[^1] From the prespective of the application, it is equivalent to being chrooted inside the rootfs.

This means that the rootfs will also contain all the data that applications produce. For example, if a game normally saves to `/home/ubuntu/.config/MyGame/data.toml`, when ran through felix86 the actual location will be at `/opt/felix86/rootfs/home/ubuntu/.config/MyGame/data.toml`.

You can easily install new packages inside the rootfs. Once inside a privileged felix86 shell (run `sudo felix86 --shell`) you can use the package manager as normal. For example, if you installed the Ubuntu rootfs, `apt` can be used.

The [installation guide](../users/installation-guide.md) provides info on automatic and manual rootfs installation.

## Does this mean I can only run applications that are inside the rootfs?

Yes, or ones in $HOME if `FELIX86_MOUNT_HOME` is enabled, which it is by default.

## How can I change my rootfs?

As with most configurations, there's two options:

- Set the `rootfs` option in the config file
    * Can be done using `sudo felix86 --set-config general.rootfs=/path/to/rootfs`
- Set the `FELIX86_ROOTFS` environment variable (overrides config value)


## But why a rootfs at all?

If we don't use a rootfs and passthrough all the filesystem syscalls, then `chroot` and `pivot_root` syscalls would hide the path of the emulator. This means that if from inside a chrooted environment `execve` is called, the emulator won't be found. `binfmt_misc` fixes this with the `F` flag, however in the case of felix86, unless if we statically link, we won't be able to load the shared libraries. Even if we statically link, we won't be able to load host dynamic libraries for thunking.

While applications that `chroot`/`pivot_root` are rare, `bwrap` which is used by Steam is one of them, and we want to support it by default.

Additionally, we think that isolation of the x86 and RISC-V environments is neat, especially features such as being able to update your x86 rootfs via `apt` or similar, without risk of changing stuff in your host system. Without a rootfs, this is harder to do as you need to ensure the x86 binaries get installed in a different directory to not mess with your host binaries.

There's ways to use felix86 [without a rootfs](../users/no-rootfs.md).

[^1]: That being said, felix86 is not a security application. Only run applications you trust, not malware. Escapes are possible.

# Why does the installer script install a custom glibc fork?

For some libraries, felix86 will call the host version of a guest function. This improves performance and was a feature for a long time. An issue with this is that these host libraries almost always rely on libc. When a library calls into a libc function, for example `malloc`, that function assumes a valid TLS is present at the architecture's thread pointer. This means that we can't **not** use libc, and since most distros rely on glibc, we use glibc.

But what happens when a process is cloned with the `CLONE_VM` flag? On the guest side, the guest will prepare a new TLS and pass it to the clone arguments. But the host now needs to run the equivalent clone syscall. Problem is, the host hasn't prepared a TLS, or a stack. There is no function like `allocateStackAndTLS` in glibc, so we'd have to create the TLS ourselves from within felix86. However, the TLS details are internal to libc and would require us looking into the internal implementation and rewriting it. If the internal structs, which weren't meant to be recreated from outside, were to add or remove members, our implementation would suddenly become incompatible with those versions of glibc. In regular programs, the TLS is either created on initialization code or during `pthread_create`.

The majority of clone syscalls called will either not have `CLONE_VM`, which means we don't need to care about the TLS as the new process will be on its own address space, or their flags will match up exactly with the flags used by `pthread_create`. Since the vast majority of `clone` syscalls with the `CLONE_VM` flag are from `pthread_create`, we can just the host side `pthread_create` with no modification. But we do care about supporting the rare cases of the flags not matching up perfectly. Previously we would run a `clone` with the correct flags and a `pthread_create` inside the `clone` to create the TLS. The original process would then die and from the guest point of view the cloned process is the one created by the host `pthread_create`. This can lead to issues though, as the thread created inside the clone is not a thread group leader as it was supposed to be if `CLONE_THREAD` wasn't passed.

For this reason, felix86 26.07 can make use of a custom pthread attribute that exists in [the glibc fork](https://github.com/felix86-emu/glibc). The pthread attribute allows for specifying custom flags to pass to the `clone` syscall called by `pthread_create`. This fork is automatically installed with the installer script, but the emulator will continue to work without it. If the custom attribute is not present the old method of supporting rare clone flags is used.

Instead of forking glibc, other options were considered. As mentioned, creating the TLS ourselves is not worth pursuing. Binary patching was considered as a way of capturing the clone syscall called from the `pthread_create` handler. This would be theoretically possible, however it is difficult. You would want to replace the `ECALL` with a jump to a handler. It would need to be a `C.LD` + `C.JALR`. However, `C.LD` can't load from `$gp`, which holds our `ThreadState`. We can't `C.LD` from the stack like we do in `invalidate_caller_thunk` because we don't control the stack at that point since we aren't in JIT code. A faulting instruction could be used to handle the syscall in a signal handler. However, `pthread_create` will block all signals before calling clone, which means we'd need to patch the `sigprocmask` too, and it gets too hacky. It's possible the syscall user dispatch feature could be used, but it doesn't exist on RISC-V. And we can't `LD_PRELOAD` the `clone` function, because `pthread_create` tries a raw `clone3` first, which can't be preloaded. For these reasons, forking glibc was the decision.

## Does that mean that felix86 will not work with upstream glibc?

Felix86 will **always work** with an upstream glibc, and >99% of programs will not care. It is rare that a program uses `CLONE_VM` without `CLONE_VFORK` and without the flags being the exact same as the ones used by `pthread_create`, because 99% of cases a process calls `clone` with `CLONE_VM`, it is creating a thread, or using `CLONE_VFORK` which is handled differently.
