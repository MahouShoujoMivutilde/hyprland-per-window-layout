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

When new windows are opened the current layout is used (by default) or the first defined in `kb_layout` (when `HPWX_PREFER_FIRST=true`).

When existing window is selected - layout from `windows` is used.

When you change layout with one of the windows in focus - its layout is redefined:


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

Tested and works since Hyprland v0.19.0beta.

## Setup

Download [the script](./hyprland-per-window-xkblayout), save it somewhere in your `$PATH`, make sure it is executable.

The script assumes that you setup layout in dedicated `device` section in `hyprland.conf`, **not** inside general `input` block.

Let's say we have this in `hyprland.conf`:

```
device {
    name = logitech-usb-keyboard
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
found layout: 'us' (0) is 'English (US)'
ok
found layout: 'ru' (1) is 'Russian'
ok
define window 0x5614683c7ab0 layout as 'us'
define window 0x5614683a8940 layout as 'us'
define window 0x561469fcf2f0 layout as 'us'
define window 0x561469fc7dd0 layout as 'us'
define window 0x561469fd9440 layout as 'us'

Options set:

DEBUG = true, PREFER_FIRST = false, keyboard = 'logitech-usb-keyboard'
Long layout names to short names:
    declare -A layouts_short=([Russian]="ru" ["English (US)"]="us" )
Index of a given layout in Hyprland's 'kb_layout':
    declare -A kb_layout=([us]="0" [ru]="1" )

...waiting for new events...
```

If it is correct - good, you don't need to write the config file.

If you prefer windows to open with default layout (first in hyprland config) **instead of
inheriting the current** - set `HPWX_PREFER_FIRST=true` either as environment variable
(if you don't need to configure anything else) or in config (when you want to
configure _everything_ yourself).

```sh
❯ HPWX_PREFER_FIRST=true hyprland-per-window-xkblayout
```

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
