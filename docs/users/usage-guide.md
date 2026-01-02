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

## Compatibility

There's a compatibility list:

[https://felix86.com/compat/](https://felix86.com/compat/)

If you want to try a game that is not listed, you can make a compatibility report here:

[https://github.com/felix86-emu/compatibility-list/issues](https://github.com/felix86-emu/compatibility-list/issues)

## Thunking

There's thunking support for GLX (OpenGL on X11). You can enable it using `FELIX86_ENABLED_THUNKS=glx`. It is not perfect and may lead to problems.