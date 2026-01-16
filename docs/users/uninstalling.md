---
title: Uninstalling
---

To properly uninstall felix86, you need to remove the felix86 binary, the rootfs, and potential binfmt_misc installations.

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

If you don't remember the rootfs path, it can be obtained with `felix86 -g`:
```bash
sudo rm -rf /path/to/rootfs
```

## Deleting configuration files

The felix86 configuration files are stored in `$HOME/.config/felix86`
```bash
rm -rf ~/.config/felix86
```

## Deleting /opt/felix86

The felix86 binary itself and thunks exist in `/opt/felix86`:
```bash
sudo rm -rf /opt/felix86
```

## Deleting symlinks in /usr/bin

```
sudo rm /usr/bin/felix86
```

In older versions of felix86 (25.11 and prior) you also need to remove `felix86-mounter`:
```
sudo rm /usr/bin/felix86-mounter
```