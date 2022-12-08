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

Tested and works on Hyprland v0.19.0beta.

## Setup

Open the script, edit values under `OPTIONS` line.

The script assumes that you setup layout in dedicated `device:...` section in `hyprland.conf`, **not** inside general `input` block.

e.g.

```
device:Logitech USB Keyboard {
    kb_layout = us,ru
    kb_options = caps:escape,grp:toggle
}
```

So under `OPTIONS` it will look like:

```sh
# keyboard device from hyprland.conf where you define `kb_layout`
# in lowercase, spaces as `-`
keyboard="logitech-usb-keyboard"

# map long layout names to short ones, as displayed in
# `hyprctl devices -j | gojq -r '.keyboards | .[] | .active_keymap' | sort -u`
# when selecting different layouts
declare -A layouts_short=(["English (US)"]="us" ["Russian"]="ru")
```

Once you set this up, the script is ready for use.

Make sure it is executable and run it.

You can also add it to startup

```
# hyprland.conf

exec-once = path/to/script/hyprland-per-window-xkblayout
```
