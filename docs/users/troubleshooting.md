---
title: Troubleshooting
---

## Glitching graphics with AMD GPUs and the `amdgpu` module

There may be problems due to differences in memory models in userspace and kernel drivers, in particular with 64-bit OpenGL applications running with AMD GPUs that use the `amdgpu` kernel module.

Either enable thunking (`FELIX86_ENABLED_THUNKS=glx,vk`), or, if running into issues with that, enable TSO with `FELIX86_ALWAYS_TSO=1`.

