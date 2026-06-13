---
title: Usage guide
---

A nice and clean start is using the felix86 shell:
```bash
felix86 --shell
```

This will disable logging and start a bash or zsh shell with a custom prompt.

If you want logging to be enabled, execute the shell manually:
```bash
felix86 /path/to/rootfs/bin/bash
```

If you don't want a shell, you can just run the program directly:
```bash
felix86 /path/to/rootfs/usr/bin/nano
```

If installed in binfmt_misc (handled by install script, or with `sudo -E felix86 -b`) you can simply run x86 programs that are inside the rootfs:
```bash
/path/to/rootfs/MyGame.AppImage
```

If you want to run a program outside the rootfs, you will be prompted to trust the parent directory:
```bash
/path/to/some/dir/MyGame.AppImage
```

If not prompted for whatever reason, create `$HOME/.config/felix86/trusted.txt` and add the absolute path to the trusted directory.

## Compatibility

There's a compatibility list:

[https://felix86.com/compat/](https://felix86.com/compat/)

If you want to try a game that is not listed, you can make a compatibility report here:

[https://github.com/felix86-emu/compatibility-list/issues](https://github.com/felix86-emu/compatibility-list/issues)

## Thunking

There's thunking support for GLX (OpenGL on X11) and Vulkan. You can enable it using `FELIX86_ENABLED_THUNKS=glx,vk`. This should improve performance in games.

## Profiles

You can use different execution profiles that set multiple configurations at once.

Usage:
```
FELIX86_PROFILE=extreme felix86 --shell
```

There's currently the following profiles:    

- `extreme` - Enables all optimizations, may break some games that rely on TSO.
- `safe` - Enables safe optimizations and TSO mode.
- `paranoid` - Disable almost all optimizations and enable some slow safety checks.
- `zink` - Enables Vulkan thunking and Zink usage in Mesa.

Each profile is a .toml file in `$HOME/.config/felix86/profiles`. Each profile is a partial or full version of the `$HOME/.config/felix86/config.toml` file, with some configurations changed.

## DXVK

Installing DXVK is as simple as installing it on your host system:
```bash
# Set $ROOTFS to the rootfs absolute path for convenience
export ROOTFS=$(felix86 -g)

# Copy DXVK release inside rootfs
cp -r /path/to/dxvk-release $ROOTFS/tmp/

# Enter felix86 shell
felix86 --shell

export WINEPREFIX=/path/to/wineprefix
cd /tmp/dxvk-release
cp x64/*.dll $WINEPREFIX/drive_c/windows/system32
cp x32/*.dll $WINEPREFIX/drive_c/windows/syswow64

# Run winecfg from inside felix86 shell and add native DLL overrides for
# d3d8, d3d9, d3d10core, d3d11, dxgi.
winecfg
```

Make sure to enable Vulkan thunking for better performance:
```bash
# Needs to be exported before entering felix86 shell
export FELIX86_ENABLED_THUNKS=vk
felix86 --shell
```

## gl4es

[gl4es](https://github.com/ptitSeb/gl4es) allows for using OpenGL on PowerVR iGPUs in RISC-V boards, as those GPUs only have GLES support. 

For ease of use, a wrapper script `felix86-gl4es` exists. The script will set relevant environment variables and start felix86.

You can install it like so:
```bash
bash <(curl -fsSL https://install.felix86.com/gl4es.sh)
```

This will install the gl4es libraries compiled for RISC-V and the wrapper script.

On current RISC-V distros, many games may also require setting `SDL_VIDEODRIVER=x11`:
```bash
// Enter an emulated shell
SDL_VIDEODRIVER=x11 felix86-gl4es
```

!!! Warning
    There's currently no 32-bit thunking, so you need to try 64-bit games if you want to use the iGPU. A discrete GPU, like an AMD one, will be able to use the x86 drivers and not require thunking, so 32-bit games will work fine with a discrete GPU. 