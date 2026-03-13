---
title: Installation guide
---

Installing felix86 is easiest done with the installation script. Simply paste this into a terminal and press Enter.
```sh
bash <(curl -s https://install.felix86.com)
```

This script will guide you through choosing a felix86 version and a rootfs.

If you only need to install a rootfs, use the rootfs installer script by itself:
```sh
bash <(curl -s https://install.felix86.com/rootfs.sh)
```

!!! tip
    Install the rootfs in a path accessible by root, such as the default `/opt/felix86/rootfs`. Installing the rootfs in the home directory may lead to problems.

!!! example
    Both scripts are hosted via GitHub Pages and their source code is available at [https://github.com/felix86-emu/install](https://github.com/felix86-emu/install)

Consult the [usage guide](./usage-guide.md) for further info.

For manual installation, see the [building instructions](../devs/building-instructions.md).

If you want to build your own rootfs, see [the rootfs section in building instructions](../devs/building-instructions.md#rootfs).