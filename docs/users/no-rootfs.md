---
title: Using without a rootfs
---

!!! Warning
    Using felix86 without a rootfs isn't the recommended way, but it exists for users or developers who are willing to tinker.

felix86 is designed to [use a rootfs](../general/rootfs-information.md) to allow for isolating an x86 environment and its files, while also not requiring root permissions for ease of use. However, the rootfs gives no security guarantees, it may be possible to escape it and modify files outside, as the chroot and mounts are purely emulated.

However, you may be developing a VM environment similar to [muvm](https://github.com/AsahiLinux/muvm) that wants to use felix86, or you are more of a power user, or you want to use some container tool. In these cases, it may be worthy to not use a rootfs.

The `FELIX86_NO_ROOTFS` environment variable will make felix86 not use a rootfs and passthrough all filesystem syscalls to the kernel.

!!! Warning
    While this works, you may run into problems if you try to run an emulated app that runs `chroot` or `pivot_root`, that you wouldn't run into otherwise.

    The reason for this is that felix86 doesn't statically link, and statically linking would come with a set of problems. Because felix86 is dynamically linked, if an emulated app uses `chroot`/`pivot_root` and doesn't copy/mount the felix86 libraries and ld.so over to the new root, it won't be able to run.

Or, you may want to use a rootfs, but you want to use a proper containerization tool like `bwrap`, and real `mount` syscalls.

In this case, you need an existing x86 rootfs, and to copy over the felix86 libraries. On multiarch systems they probably need to go in `/lib/riscv64-linux-gnu`. On non-multiarch systems, they need some way to not conflict with the x86 libraries which may prove harder. These are the libraries felix86 loads on runtime: `libstdc++.so.6`, `libm.so.6`, `libgcc_s.so.1`, `libc.so.6`. Make sure the dynamic linker `ld-linux-riscv64-lp64d.so.1` is also copied over to the rootfs `/lib` directory.

You may want to disable the config file too, otherwise felix86 might try to search in a path that it doesn't own and fail to load the configs.
By disabling the config file, you may still pass configurations via environment variables.

Here's an example `bwrap` command to help, assuming a small rootfs in `/tmp/felix86-root` with the aforementioned libraries copied:
```
sudo bwrap \
  --unshare-all \
  --bind /tmp/felix86-root / \
  --proc /proc \
  --dev /dev \
  --tmpfs /tmp
  --setenv FELIX86_NO_ROOTFS 1 \
  --setenv FELIX86_NO_CONFIG_FILE 1 \
  --setenv FELIX86_BINFMT_MISC_INSTALLED 1 \
  --setenv FELIX86_QUIET 1 \
  -- /bin/bash
```
Since the configuration file isn't loaded, we need to notify felix86 that it should `execve` executables directly, and this is what `FELIX86_BINFMT_MISC_INSTALLED` is for.

Inside this emulated shell, you can confirm felix86 is used with `cat /proc/cpuinfo`. Alternatively, you may remove `FELIX86_QUIET`.