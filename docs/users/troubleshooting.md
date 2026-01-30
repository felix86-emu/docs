---
title: Troubleshooting
---

## Glitching graphics with AMD GPUs and the `amdgpu` module

There may be problems due to differences in memory models in userspace and kernel drivers, in particular with 64-bit OpenGL applications running with AMD GPUs that use the `amdgpu` kernel module.

Either enable thunking (`FELIX86_ENABLED_THUNKS=glx,vk`), or, if running into issues with that, enable TSO with `FELIX86_ALWAYS_TSO=1`.

## Sub-process apt-key exited unexpectedly

Ensure the rootfs is installed in a path accessible by every user. For example, `/home/myuser/rootfs` is not such a path, as `/home/myuser` is only accessible to `myuser`. Programs like `apt` will switch uid/gid and try to open `apt-key`, which will try to get a file descriptor to the rootfs and fail. Recommended rootfs install directory: `/opt/felix86/rootfs`, default by the install script.