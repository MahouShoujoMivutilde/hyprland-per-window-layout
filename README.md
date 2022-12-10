# Hyprland per window xkb layout

The script maintains associative array `windows` that maps windows' addresses to selected layouts.

## How it works

At script's launch every window (if any) gets currently selected xkb layout.

```python
windows = {
    "0x7cd6bc40": "us",
    "0x7cd93c40": "us",
    "0x7cd42830": "us",
    "0x7cd2a610": "us",
    "0x7e95cb30": "us"
}
```

When new windows is opened the current layout is used.

When existing window is selected - layout from `windows` is used.

When you change layout when one of the windows is in focus - its layout gets redefined:


```python
windows = {
    "0x7cd6bc40": "us",
    "0x7cd93c40": "ru",
    "0x7cd42830": "us",
    "0x7cd2a610": "us",
    "0x7e95cb30": "us"
}
```

## Requirements

* bash 4.0+ (for associative arrays).
* socat (for listening for Hyprland socket2 events).
* [gojq](https://github.com/itchyny/gojq) (for working with `hyprctl`'s json. `jq` could work, but it is much slower, so quit using it already).
* [lolcat](https://github.com/jaseg/lolcat) (optional, for logs)

Tested and works on Hyprland v0.19.0beta.

## Setup

Download [the script](./hyprland-per-window-xkblayout), save it somewhere in your `$PATH`, make sure it is executable.

The script assumes that you setup layout in dedicated `device:...` section in `hyprland.conf`, **not** inside general `input` block.

Let's say we have this in `hyprland.conf`:

```
device:Logitech USB Keyboard {
    kb_layout = us,ru
    kb_options = caps:escape,grp:toggle
}
```

Then there are 2 options - (1) autodetection and (2) manual configuration.

1\. You could try running the script with debug output on, see if it guesses your setup correctly

```
❯ HPWX_DEBUG=true hyprland-per-window-xkblayout

no config file, trying to guess options...
ok
found: us (0) is English (US)
ok
found: ru (1) is Russian
ok
...

options set:

DEBUG = true, keyboard = logitech-usb-keyboard
declare -A layouts_short=([Russian]="ru" ["English (US)"]="us" )

(this is an index of a given layout in `kb_layout`)
declare -A kb_layout=([us]="0" [ru]="1" )

```

If it is correct - good, you don't need to write the config file.

2\. If it isn't - you can use the config file to set it up manually.

Then our config, stored in `~/.config/hypr/xkb_layout.conf`, will be

```sh
# config for hyprland-per-window-layout script
# use it to overwrite default values

# print events and actions taken
DEBUG=true

# keyboard device from hyprland.conf where you define `kb_layout`
# in lowercase, spaces as `-`
keyboard="logitech-usb-keyboard"

# map long layout names to short ones, as displayed in
# `hyprctl devices -j | gojq -r '.keyboards | .[] | .active_keymap' | sort -u`
# when selecting different layouts
declare -A layouts_short=(["English (US)"]="us" ["Russian"]="ru")
```

Once you set this up, the script is ready for use.

You can also add it to startup

```
# hyprland.conf

exec-once = hyprland-per-window-xkblayout
```
