---
title: Usage guide
---

A nice and clean start is using the felix86 shell:
```bash
felix86 --shell
```

This will disable logging and start a bash or zsh shell with a custom prompt.

In felix86 26.08 and later, you can start a program with the --shell command, which runs it via `bash -c` or `zsh -c`, which means it can find your binary in `PATH`:
```
felix86 --shell="steam -no-cef-sandbox"
```

In older verions you can achieve the same like so:
```
felix86 /path/to/rootfs/bin/bash -c "steam -no-cef-sandbox"
```

If you want logging to be enabled, use this command:
```bash
felix86 --shell-debug
```

If you don't want a shell, you can just run the program directly:
```bash
felix86 /path/to/rootfs/usr/bin/nano
```

If installed in binfmt_misc (handled by install script, or with `sudo -E felix86 -b`) you can simply run x86 programs that are inside the rootfs:
```bash
/path/to/rootfs/MyGame.AppImage
```

By default `/home` will be mounted inside the rootfs, so you can also run x86 programs from `$HOME`.

## Compatibility

There's a compatibility list:

[https://felix86.com/compat/](https://felix86.com/compat/)

If you want to try a game that is not listed, you can make a compatibility report here:

[https://github.com/felix86-emu/compatibility-list/issues](https://github.com/felix86-emu/compatibility-list/issues)

## Thunking

There's thunking support for GLX (OpenGL on X11) and Vulkan. You can enable it using `FELIX86_ENABLED_THUNKS=glx,vk`. This should improve performance in games.

## Desktop shortcuts

Desktop shortcuts in Linux are usually XDG Desktop Entry specification files.

The shorcuts usually exist in the host desktop, since the host $HOME is mounted in the guest $HOME, but the binary usually exists inside the rootfs, not the host filesystem. Even if it does exist in the host filesystem, it may be a script that needs to be explicitly run with the x86 bash, so that filesystem accesses are resolved relative to the rootfs.

In order to be able to use it like a normal shortcut, we need to wrap the executable with `felix86 --shell="..."`.

Here's a `sed` one-liner that makes a desktop entry run with felix86:
```sh
sed -i -E 's/^Exec=(.*)$/Exec=felix86 --shell="\1"/' myshortcut.desktop
```

## Configuration

The configuration file exists at `/etc/opt/felix86/config.toml`.

You can get a specific config by specifying its group and value, for example: `ROOTFS=$(felix86 --get-config general.rootfs_path)`

You can set a specific config in a similar fashion, but it requires administrator privileges: `felix86 --set-config logging.print_signals=true`

To list all configs, use the `-c` option.

Each configuration has a respective environment variable. These can be discovered by reading the configuration file.

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

Each profile is a .toml file in `$HOME/.config/felix86/profiles`. Each profile is a partial or full version of the config file, with some configurations changed. You may create new profiles and pass them to `FELIX86_PROFILE` as a name (which will look relative to the profile directory) or as an absolute path.

## DXVK

Installing DXVK is as simple as installing it on your host system:
```bash
# Set $ROOTFS to the rootfs absolute path for convenience
export ROOTFS=$(felix86 -g)

# Copy DXVK release inside rootfs
cp -r /path/to/dxvk-release $ROOTFS/tmp/

# Enter felix86 shell
felix86 --shell

# Usually export WINEPREFIX="$HOME/.wine"
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

[gl4es](https://github.com/ptitSeb/gl4es) allows for using OpenGL on iGPUs in RISC-V boards, as those GPUs don't support desktop OpenGL.

!!! danger "The iGPUs suck"
    Use a discrete GPU, if possible, as they tend to have way better Linux support and performance. When using a discrete GPU gl4es is not necessary.

=== "Downloading gl4es"
    You can download a prebuilt version of gl4es: [https://cdn.felix86.com/misc/gl4es/gl4es.zip](http://cdn.felix86.com/misc/gl4es/gl4es.zip)

=== "Building gl4es"
    To build gl4es from source, follow the instructions below.
    ```shell
    sudo apt update
    sudo apt install libx11-dev
    git clone https://github.com/ptitSeb/gl4es
    cd gl4es
    cmake -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo
    cmake --build build -j$(nproc)
    ```

### Installation 

Install the library to a known host location, such as `/opt/felix86/gl4es/`.
From the directory that contains the unzipped/built `libGL.so.1`, run the following:
```shell
export INSTALLATION_DIR="/opt/felix86/gl4es"
sudo mkdir -p $INSTALLATION_DIR
sudo mv ./libGL.so.1 $INSTALLATION_DIR
```

Make sure to symlink `libGLX.so.0`, as gl4es exports the libGLX symbols:
```shell
sudo ln -s libGL.so.1 "$INSTALLATION_DIR/libGLX.so.0"
```

### Usage

In order to use gl4es you need to set `LD_LIBRARY_PATH` and enable thunking:
```shell
export INSTALLATION_DIR="/opt/felix86/gl4es"
export SDL_VIDEODRIVER=x11 # may be required for some games
export FELIX86_ENABLED_THUNKS=glx,egl
export LIBGL_NOBANNER=1
export LD_LIBRARY_PATH="$INSTALLATION_DIR:$LD_LIBRARY_PATH"
felix86 --shell
```

!!! Warning
    There's currently no 32-bit thunking, so you need to try 64-bit games if you want to use the iGPU. A discrete GPU, like an AMD one, will be able to use the x86 drivers and not require thunking, so 32-bit games will work fine with a discrete GPU. 