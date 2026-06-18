---
title: Troubleshooting
---

## Privileged executables don't work

By default since felix86 26.07, binfmt_misc installation through the script is done without the `C` flag. The `C` flag allows for passing the permissions from a privileged executable (like `sudo`) to felix86. If a vulnerability exists in felix86, an attacker could perform privilege escalation by exploiting felix86. This type of attack would require either you or an attacker with access to your computer to run malicious code. 

If you understand the risk and want to run privileged executables through felix86, you can enable privileged executable execution:
```
sudo felix86 --binfmt-misc-setuid
```

Alternatively, if you just want to run a privileged command once, you can enter a privileged shell:
```
sudo felix86 --shell
```

## AppImages don't work

See [Privileged executables don't work](#privileged-executables-dont-work), since AppImage uses `fusermount3` which is marked as setuid.

## Glitching graphics with AMD GPUs and the `amdgpu` module

There may be problems due to differences in memory models in userspace and kernel drivers, in particular with 64-bit OpenGL applications running with AMD GPUs that use the `amdgpu` kernel module.

Either enable thunking (`FELIX86_ENABLED_THUNKS=glx,vk`), or, if running into issues with that, enable TSO with `FELIX86_ALWAYS_TSO=1`.

## Sub-process apt-key exited unexpectedly

Ensure the rootfs is installed in a path accessible by every user. For example, `/home/myuser/rootfs` is not such a path, as `/home/myuser` is only accessible to `myuser`. Programs like `apt` will switch uid/gid and try to open `apt-key`, which will try to get a file descriptor to the rootfs and fail. Recommended rootfs install directory: `/opt/felix86/rootfs`, default by the install script.

## How can I use my iGPU?

The iGPUs in current RISC-V hardware aren't great, so you are likely to run into issues. If you're not afraid of tinkering you can try installing the open source drivers for the PowerVR GPU (likely requires a custom kernel build) but your best bet is trying to get an AMD GPU to work until the iGPU support gets better. [gl4es can be used](https://felix86.com/docs/users/usage-guide/#gl4es) to run some games.

Installing a discrete GPU isn't plug-and-play on current RISC-V hardware. For some help, [check out the amdgpu page](https://felix86.com/docs/users/amdgpu/).

## Unity games exit without doing anything

Make sure you don't have `SDL_VIDEODRIVER=wayland`, set it to `SDL_VIDEODRIVER=x11` or `SDL_VIDEODRIVER=wayland,x11`. This isn't a felix86 problem, some RISC-V boards define this in `/etc/environment`.
