---
title: Steam
---

Installing Steam using felix86 is easy!

### Installation

Download the `steam_latest.deb` package from Steam's website.

Enter `felix86 --shell` and install it via dpkg: `sudo dpkg -i /path/to/steam_latest.deb`

### Running

Run as `steam -no-cef-sandbox`.

If you're having GPU issues, you can also disable the GPU with `-cef-disable-gpu`

If you're having issues on Wayland, you may need `SDL_VIDEODRIVER=x11`.

!!! tip
    Since felix86 can use `apt`, it may be possible to install through the package manager. This is untested.