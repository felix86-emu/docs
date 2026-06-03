---
title: Troubleshooting
---

## Glitching graphics with AMD GPUs and the `amdgpu` module

There may be problems due to differences in memory models in userspace and kernel drivers, in particular with 64-bit OpenGL applications running with AMD GPUs that use the `amdgpu` kernel module.

Either enable thunking (`FELIX86_ENABLED_THUNKS=glx,vk`), or, if running into issues with that, enable TSO with `FELIX86_ALWAYS_TSO=1`.

## Sub-process apt-key exited unexpectedly

Ensure the rootfs is installed in a path accessible by every user. For example, `/home/myuser/rootfs` is not such a path, as `/home/myuser` is only accessible to `myuser`. Programs like `apt` will switch uid/gid and try to open `apt-key`, which will try to get a file descriptor to the rootfs and fail. Recommended rootfs install directory: `/opt/felix86/rootfs`, default by the install script.

## How can I use my iGPU?

The iGPUs in current RISC-V hardware aren't great, so you are likely to run into issues. If you're not afraid of tinkering you can try installing the open source drivers for the PowerVR GPU (likely requires a custom kernel build) but your best bet is trying to get an AMD GPU to work until the iGPU support gets better.

Installing a discrete GPU isn't plug-and-play on current RISC-V hardware.

- For SpacemiT K1, see this thread: [https://community.milkv.io/t/amd-radeon-rx550-gpu-not-working/2869/23](https://community.milkv.io/t/amd-radeon-rx550-gpu-not-working/2869/23)
- For SpacemiT K3, see this post: [https://www.reddit.com/r/RISCV/comments/1tkrrmo/crysis_on_spacemit_k3_with_felix86/](https://www.reddit.com/r/RISCV/comments/1tkrrmo/crysis_on_spacemit_k3_with_felix86/)

## Unity games exit without doing anything

Make sure you don't have `SDL_VIDEODRIVER=wayland`, set it to `SDL_VIDEODRIVER=x11` or `SDL_VIDEODRIVER=wayland,x11`. This isn't a felix86 problem, some RISC-V boards define this in `/etc/environment`.
