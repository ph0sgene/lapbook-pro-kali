# lapbook-pro-kali
This is a collection of files / scripts for helping users with their generic 7 inch mini laptop from some obscure chinese company. 

In theory, this should work just fine for debian and ubuntu or any other *nix distribution.

Specs:
- CPU: Intel(R) Celeron(R) J4105 CPU @ 1.50GHz Gemini Lake
- Memory: ABCD 12GB @ 2133 Hhz LPDDR4
- HDD: ShiJi 512GB
- MB: Default string Default string 2.1. To Be Filled By O.E.M.
- WIFI: Intel Corporation Wireless 3165 (rev 81)
- GPU: Intel Corporation GeminiLake [UHD Graphics 600] (rev 03)
- CAM: Sunplus Innovation Technology Inc. SPCA2281 Web Camera

## Issues

| Working?      |     | Notes                                                    |
| ------------- |:---:| --------------------------------------------------------:|
| Audio         | ✅   | `snd_soc_sof_es8336 quirk=0x02` must be added in modprobe.d |
| Touchscreen   | 🚧   | probe of i2c-GDIX1002:00 failed with error -5         |
| Mouse         | 🚧  | Still not satisfied with the remapping                   |
| GRUB portrait | 🚧  | GRUB option `fbcon` ignored for menu but the rest is not |
| Laptop lid    | ❔   | Seems to work but locks up with `i3lock`                 |

## Stuck in portrait mode? 
`xrandr -o right` 

## Mouse not working?
The mouse buttons are actually KEYBOARD KEYS. They will have to be macro'd:

- `KP_Begin` => left click 
- `MENU` => right click

You can use either `input-remapper` or `xdotool`. 

The keycode for `KP_Begin` is `84` and `xmodmap` key `0xFF9D`.

The keycode for `MENU` is  `117` and `xmodmap` key `0xFF67`.

The mouse nib itself is an `HTIX5288` which is known to have problems as of kernel 5.4.

## Whats the Kernel?
A PATCH FOR THE KERNEL IS BEING WORKED ON TO FIX THE TOUCHSCREEN! SOURCE [HERE](https://forums.linuxmint.com/viewtopic.php?t=403825)
## Audio 
- Ensure `alsamixer`, `aplay` and `firmware-sof-signed` are installed. 
- Mess arounc with pulse settings (in xfce or something). Make sure theyre all up and not muted
- Terminal > `echo "options snd_soc_sof_es8336 quirk=0x02" >> /etc/modprobe.d/alsa-base.conf`
- Reboot
- Run `alsamixer -c 0`
- Mess with the sliders until you hear audio. Mostly the `Speakers` slider

# Touchscreen
Identified as Goodix Capacitive TouchScreen GDIX1002. 

```sh
[   11.640471] Goodix-TS i2c-GDIX1002:00: supply AVDD28 not found, using dummy regulator
[   11.640560] Goodix-TS i2c-GDIX1002:00: supply VDDIO not found, using dummy regulator
[   11.640644] Goodix-TS i2c-GDIX1002:00: Unexpected ACPI resources: gpio_count 1, gpio_int_idx 0
[   11.841245] Goodix-TS i2c-GDIX1002:00: ID 911, version: 1060
[   11.930999] input: Goodix Capacitive TouchScreen as /devices/pci0000:00/0000:00:17.0/i2c_designware.4/i2c-5/i2c-GDIX1002:00/input/input8
[   11.944029] Goodix-TS i2c-GDIX1002:00: request IRQ failed: -5
[   12.106752] Goodix-TS: probe of i2c-GDIX1002:00 failed with error -5
```
