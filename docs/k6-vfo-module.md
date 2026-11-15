# Compact FD6818B Explore VFO module

`c/k6/mod/vfo.c` adds a small receive VFO to the Explore-only FD6818B firmware. It runs as a RAM module, so it does not take space in the MCU application image. It provides two volatile VFO frequencies and ten saved memory frequencies in the existing repeater-table flash region. VFO records are tagged and excluded from repeater matching and Explore's surf list.

This first version is receive-only. The current module API has a guarded transmitter for generated signals, but no voice-microphone PTT path. It is a compact tuner and memory screen, not a complete replacement for Aura's transmitting VFO firmware.

## Build

Build the 91 KB Explore host and the module bundle with the radio's module key:

```sh
K6_AURA_FEATURES=pager-explore,fd6818b,spilib,txdog \
K6_IMAGE_NAME=aura-fd6818b-vfo-host \
K6_MOD_KEY_FILE="$HOME/.config/blowfang/k6-mod.key" \
sh firmware/k6-aura/build-fd6818b.sh
```

The host is written to `firmware/k6-aura/out-fd6818b/aura-fd6818b-vfo-host.bin`; the module-only bundle is written to `firmware/k6-aura/out-fd6818b/vfo-module-bundle/`. The build rejects firmware over the KD32F328's 122,880-byte application limit.

## Install

With the radio already showing `UPDATE`:

```sh
python3 tools/k6_install.py --port /dev/ttyUSB2 \
  --key-file "$HOME/.config/blowfang/k6-mod.key" \
  flash firmware/k6-aura/out-fd6818b/aura-fd6818b-vfo-host.bin --already-update
```

After it restarts into Explore, load the VFO module and the updated Surf module (which hides VFO memories from its repeater list):

```sh
python3 tools/k6_install.py --port /dev/ttyUSB2 \
  --key-file "$HOME/.config/blowfang/k6-mod.key" \
  load firmware/k6-aura/out-fd6818b/vfo-module-bundle --only 0,30 --verify
```

Run `VFO RADIO` from the module menu (slot 30). Slot 0 refreshes Surf. This replaces the existing contents of those slots.

## Keys

- Number keys enter six digits as kHz, so `145500` tunes 145.500 MHz.
- Up/down step by 12.5 kHz.
- `N` switches between VFO A and B.
- `W` selects one of ten memory slots.
- `#` saves the active frequency; `*` recalls it.
- `X` returns to Explore. `V/M` and side keys A/B use the firmware's escape path when available.
