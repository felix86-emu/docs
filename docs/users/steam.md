---
title: Steam
---

Installing Steam using felix86 is easy!

### Installation

Download the `steam_latest.deb` package from Steam's website.

Enter a privileged shell to install the .deb:
```shell
sudo felix86 --shell

# From inside the felix86 shell
dpkg -i /path/to/steam_latest.deb

# Exit the privileged shell when we are done installing
exit
```


### Running

Enter `felix86 --shell` and run as `steam -no-cef-sandbox`.

If you're having GPU issues, you can also disable the GPU with `-cef-disable-gpu`.

If you're having issues on Wayland, you may need `SDL_VIDEODRIVER=x11`.

!!! tip
    Since felix86 can use `apt`, it may be possible to install Steam through the package manager. This is untested.

You may also want to make the desktop entry work, this command will make it run with felix86:
```sh
sed -i -E 's/^Exec=(.*)$/Exec=felix86 --shell="\1"/' ~/Desktop/steam.desktop
```