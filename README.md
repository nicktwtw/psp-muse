# psp-muse

**A 2008 PSP-3000, reborn as a voice terminal for Meta Muse.**

Seventeen years after launch, a pink PSP-3000 with no Bluetooth and no
touchscreen can hold a conversation with an AI agent. Hold the R button,
speak, and Muse answers out loud. This project gives end-of-life hardware
a second life as a physical doorway to Muse, built with the Muse Gadgets
Device SDK (hatch-link BLE setup protocol).

Built for the **Meta Global AI Developer Hackathon** (Indie track).

## How it works

- The PSP runs a native homebrew client (EBOOT) based on
  [wobsoriano/muse-gadget-psp](https://github.com/wobsoriano/muse-gadget-psp)
  (Apache-2.0), using its Rust port.
- A Linux host with a BLE adapter performs the one-time pairing with Muse
  and provisions the device identity onto the PSP memory stick.
- Hold **R** and talk into the microphone: the recorded audio is encrypted
  and sent to Meta Muse; the reply is spoken back through text-to-speech.
- A custom on-screen avatar shows the device state (idle, listening,
  thinking, speaking).

## Hardware

- Sony PSP-3000, firmware 6.61 + ARK-4 custom firmware
- Linux host with a BLE-capable Bluetooth adapter (pairing and install)
- No hardware modifications to the PSP itself

## Status

- [x] EBOOT builds and boots on real hardware (Rust port)
- [x] Custom avatar generated and rendering
- [x] SDK token issued; one-time BLE pairing completed
- [ ] Install device identity to the memory stick and verify online state
- [ ] First end-to-end voice conversation on the PSP
- [ ] Demo video

## Security note

`identity.json`, `pairing.json` and the SDK token are per-device secrets.
They are listed in `.gitignore` and are never committed to this repository.

## Credits and license

Based on the open-source muse-gadget-psp project by wobsoriano
(Apache License 2.0). See `LICENSE` and `NOTICE` in the upstream project.
