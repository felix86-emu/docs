---
title: Steam
---

Installing Steam using felix86 is easy!

### Installation

Enter a privileged shell and install via `apt`:
```shell
sudo felix86 --shell

# From inside the felix86 shell
apt install steam

# Exit the privileged shell when we are done installing
exit
```

### Running

Enter `felix86 --shell` and run as `steam`. Or as a one-liner: `felix86 --shell="steam"`.

You may also want to make the desktop entry work, this command will make it run with felix86:
```sh
sed -i -E 's/^Exec=(.*)$/Exec=felix86 --shell="\1"/' ~/Desktop/steam.desktop
```

#### Troubleshooting

It shouldn't have any issues, but if it does, try with `-no-cef-sandbox`.

If you're having GPU issues, you can also disable the GPU with `-cef-disable-gpu`.

If you're having issues on Wayland, you may need `SDL_VIDEODRIVER=x11`.
