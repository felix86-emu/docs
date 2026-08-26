---
title: Uninstalling
---

To properly uninstall felix86, you need to remove the felix86 binary, the rootfs, and potential binfmt_misc installations, among other things.

The uninstallation script can handle all this for you:
```shell
bash <(curl -fsSL https://install.felix86.com/uninstall.sh)
```

# Manual uninstallation

## Uninstalling from binfmt_misc

The `felix86 -u` command should uninstall from binfmt_misc:
```bash
sudo felix86 -u
```

Or, if you prefer doing it manually:
```bash
sudo rm /etc/binfmt.d/felix86-*
sudo rm /usr/lib/binfmt.d/felix86-*
sudo rm /usr/local/lib/binfmt.d/felix86-*
sudo rm /run/binfmt.d/felix86-*
```

Restart systemd-binfmt to finalize in the current session:
```bash
sudo systemctl daemon-reload
sudo systemctl restart systemd-binfmt
```

## Deleting the rootfs

If you don't remember the rootfs path, it can be obtained with `felix86 --get-config general.rootfs_path`:
```bash
felix86 --get-config general.rootfs_path
```

If the rootfs lives inside `/opt/felix86` you can skip this step.
```
sudo rm -rf /path/to/my/rootfs
```

## Deleting configuration files

The felix86 profile files are stored in `$HOME/.config/felix86`
```bash
rm -rf ~/.config/felix86
```

By default, felix86 will have its own history file that should be removed:
```bash
rm -f ~/.felix86_history
```

The global configuration file is stored in `/etc/opt/felix86/config.toml`
```bash
sudo rm -rf /etc/opt/felix86
```

## Deleting /opt/felix86

The felix86 binary itself, thunks, and any other tools exist in `/opt/felix86`:
```bash
sudo rm -rf /opt/felix86
```

## Deleting symlinks

```bash
sudo rm /usr/local/bin/felix86
```

## Deleting icons

```bash
for size in 16 24 32 48 96 128; do
  xdg-icon-resource uninstall --mode user --context apps --size "$size" offtkp-felix86
done
```

## Deleting Vulkan layer, if felix86 MangoHud is installed

If you installed the felix86 MangoHud fork using [https://install.felix86.com/mangohud.sh](https://install.felix86.com/mangohud.sh), you will need to remove the installed Vulkan layer:
```bash
sudo rm /usr/local/share/vulkan/implicit_layer.d/felix86-MangoHud.riscv64.json
```