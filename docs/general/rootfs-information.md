---
title: Rootfs information
---

## What is a rootfs and how is it used?

In order to use felix86, you need a rootfs -- a directory that contains all the x86 libraries that your programs may need. Upon running, felix86 will perform a fake, **rootless** mount of folders such as `/dev`, `/proc`, `/sys` and others inside the rootfs. Any guest application will solely exist inside the rootfs. For example, if your rootfs is at `/opt/felix86/rootfs` and a program accesses `/home/ubuntu/myfile`, this access will be translated to `/opt/felix86/rootfs/home/ubuntu/myfile` inside the rootfs. This is accomplished by doing full path resolution in userspace, before passing the final resolved and symlink-free path to the kernel. As a consequence, filesystem accesses such as `/..`, or symlinks that point backwards will not escape the rootfs.[^1] From the prespective of the application, it is equivalent to being chrooted inside the rootfs.

This means that the rootfs will also contain all the data that applications produce. For example, if a game normally saves to `/home/ubuntu/.config/MyGame/data.toml`, when ran through felix86 the actual location will be at `/opt/felix86/rootfs/home/ubuntu/.config/MyGame/data.toml`.

You can easily install new packages inside the rootfs. Once inside the felix86 shell (run `felix86 --shell`) you can use the package manager as normal. For example, if you installed the Ubuntu rootfs, `apt` can be used.


## Does this mean I can only run applications that are inside the rootfs?

No. Recent versions of felix86 implement the concept of "trusted directories". The first time you run an application outside the rootfs you will be prompted to trust the parent directory. Doing so will allow you to run the application and any subdirectories and files will be visible to the application. Since guest programs can truly only see inside the rootfs, the fake mount mechanism used for `/dev` & co is used for trusted directories as well.

## How can I change my rootfs?

As with most configurations, there's two options:

- Set the `rootfs` option in `$HOME/.config/felix86/config.toml`
    * Can be done using `felix86 -s /path/to/rootfs`
- Set the `FELIX86_ROOTFS` environment variable (overrides config value)


[^1]: That being said, felix86 is not a security application. Only run applications you trust, not malware. VM escapes are possible.