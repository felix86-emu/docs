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

### Profiles

You can make a Steam game load a specific felix86 profile.

Create a file in `~/.config/felix86/profiles/steam/<appid>.toml` where `<appid>` is replaced with the app id of the game.

The app id can be obtained by the Steam store URL of the game. For example, the game Overcooked 2 has the URL `https://store.steampowered.com/app/728880/Overcooked_2/`, so the app id is `728880`.

Inside this file you can set custom configurations. Let's say for example you want to enable TSO, thunking, and reduced precision mode for Overcooked 2.
You would run `vim ~/.config/felix86/profiles/steam/728880.toml` and write the following:
```
[General]
enabled_thunks = "vk,wl,glx"
[Performance]
always_tso = true
reduced_precision = 1
```

The next time you run the game through Steam the profile will automatically be loaded. Steam **doesn't need to be restarted**, the game will pick up on the file.

#### Troubleshooting

It shouldn't have any issues, but if it does, try with `-no-cef-sandbox`.

If you're having GPU issues, you can also disable the GPU with `-cef-disable-gpu`.

If you're having issues on Wayland, you may need `SDL_VIDEODRIVER=x11`.
