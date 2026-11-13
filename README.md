# lapbook-pro-kali

Files and notes for the generic 7-inch mini laptop (no-name Chinese OEM,
"LapBook Pro"-class). Applies to any modern *nix distro; Kali used here.

Verified 2026-11-13 on kernel **6.16.8+kali-amd64**.

## Specs

- CPU: Intel Celeron J4105 @ 1.50 GHz (Gemini Lake)
- RAM: 12 GB LPDDR4 @ 2133 MHz
- SSD: 512 GB
- Board: "Default string / To Be Filled By O.E.M." (generic DMI)
- Wi-Fi: Intel Wireless 3165
- GPU: Gemini Lake UHD Graphics 600
- Camera: Sunplus SPCA2281
- Touch screen: Goodix GDIX1002 (I2C)
- Nib: HTIX5288 (I2C HID)

## Overview

| Working?      |     | Fix |
| ------------- |:---:| --- |
| Audio         | ✅  | modprobe quirk below |
| Touchscreen   | ✅  | Upstream fixed; Xorg config below |
| Mouse nib     | ⚙️  | Moves the cursor out of the box, but it has **no hardware buttons** — remapping two keyboard keys below is required to click |
| GRUB portrait | 🚧  | Unfixable upstream; hide the menu or use Limine |
| Laptop lid    | ✅  | Locks on lid per logind config |

## Audio

```sh
echo "options snd_soc_sof_es8336 quirk=0x02" | sudo tee -a /etc/modprobe.d/alsa-base.conf
```

Reboot, run `alsamixer -c 0`, raise the `Speakers` slider.

This quirk is not upstreamable: mainline defaults to `quirk=0` for boards with
generic DMI strings (like this one) and cannot know which GPIO your OEM used for
the speaker amp. The line above is the correct permanent fix.

## Touchscreen (Goodix GDIX1002)

On current kernels the driver probes cleanly out of the box — the old
`request IRQ failed: -5` probe failure was fixed upstream years ago. What still
breaks on a fresh install is the X layer: the device starts as a disabled
*floating slave* (not attached to the pointer), and touch coordinates don't
track the rotated display.

Persist the whole fix in one file:

```sh
sudo tee /etc/X11/xorg.conf.d/99-goodix-touch.conf <<'EOF'
Section "InputClass"
    Identifier "goodix touch enable and rotate"
    MatchProduct "Goodix Capacitive TouchScreen"
    Option "TransformationMatrix" "0 1 0 -1 0 1 0 0 1"
EndSection
EOF
```

The matrix shown matches a `right`-rotated display (the panel is portrait
origin, you probably want landscape). Use the matching matrix for your rotation.
Works everywhere X runs, including the LightDM greeter, from next boot.

Live, one-shot, for the currently running session:

```sh
DISPLAY=:0 xinput --enable "Goodix Capacitive TouchScreen"
DISPLAY=:0 xinput reattach "Goodix Capacitive TouchScreen" 2
DISPLAY=:0 xinput --set-prop "Goodix Capacitive TouchScreen" "Coordinate Transformation Matrix" 0 1 0 -1 0 1 0 0 1
```

## Screen orientation

The panel is portrait; you want landscape:

```sh
xrandr -o right
```

Add `video=DSI-1:panel_orientation=right_side_up fbcon=rotate:1` to the kernel
cmdline if you want the console and boot sequence rotated too.

## GRUB menu is portrait

**Cannot be fixed.** The GRUB2 menu renders before any kernel subsystem
exists, so framebuffer rotation can't reach it. Options:

- Hide it: `GRUB_TIMEOUT_STYLE=hidden` + `GRUB_TIMEOUT=0` in
  `/etc/default/grub`, then `sudo update-grub`. Hold Shift/Esc at boot to
  summon the menu when you need a snapshot.
- Or install Limine, which supports `interface_rotation: 90` natively.

## Mouse nib (HTIX5288)

Registers as mouse + hotkeys on current kernels and moves the cursor fine —
but the nib itself has **no hardware buttons**. The only buttons are two
keyboard keys shipped next to it, and you must bind them to get clicks:

- `KP_Begin` (keycode 84 / `0xFF9D`) = left click
- `MENU` (keycode 117 / `0xFF67`) = right click

Remap with `input-remapper` (recommended on modern stack) or
`xmodmap`/`xdotool` as preferred.

## Archaeology (2023 era)

- `KERNEL/goodix.patch` — Hans de Goede's GPIO workaround for the
  5.x–6.3-era probe failure (`request IRQ failed: -5`). Upstreamed long
  since; kept for history, no longer needed.
- `KERNEL/menuconfig` — config used for the one custom kernel built for
  this campaign (`vmlinuz-6.3.7-fbcon-goodix-beantop`, still in /boot of
  the original machine, moot).
- The pre-2026 maintenance notes described the touchscreen as "FIX WIP".
  That saga is extinct: kernel probes it, Xorg is the whole story now.
