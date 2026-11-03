English | [繁體中文](README.zh-TW.md)

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

## Build it yourself

Everything below follows the upstream project
[wobsoriano/muse-gadget-psp](https://github.com/wobsoriano/muse-gadget-psp)
(Apache-2.0). You build the app from source with your own credentials.
Nothing here is prebuilt, and no one else's keys or pairing files are
involved.

### What you need

- A PSP-3000 with a built-in microphone (the PSP-1000 and PSP-2000 do
  not have one). The upstream author built and tested on a PSP-3001.
- Custom firmware: system software 6.61 with ARK. ARK is what lets the
  PSP join a WPA2 Wi-Fi network, which the stock firmware cannot.
- A Memory Stick with about 5 MB free.
- A Muse account, the Muse app on your phone, and an SDK token from
  gadgets.muse.ai.
- A computer to build the app (the upstream author used macOS; Linux
  should work).
- A second device with Bluetooth, used once for pairing (a Raspberry Pi
  or a Linux computer running the SDK's Linux client).
- An OpenAI API key, for the voice. Without one, the PSP still hears you
  and Muse still answers, but the PSP stays silent.

### Step 1. Get the tools

Install Rust with rustup, then the PSP build tool:

```sh
cargo install cargo-psp
```

The project pins a nightly toolchain in `rust-toolchain.toml`, which
rustup fetches the first time you build. You also need a C compiler
for one step.

### Step 2. Make the avatar

The app plays the avatar that ships with the Muse Gadget SDK. That
avatar is Meta's and the SDK's licence does not cover it, so it is not
in the upstream repository. You build it from your own copy of the SDK:

```sh
git clone https://github.com/facebookincubator/muse-gadget-sdk vendor/muse-gadget-sdk
tools/avatar.sh
```

What this produces:

- `assets/avatar/avatar.bin`, one short clip for each mood.

Git ignores both the SDK clone and the generated avatar file.

### Step 3. Build the EBOOT yourself

```sh
cargo psp --release
```

What this produces:

- `target/mipsel-sony-psp/release/EBOOT.PBP`

This is the PSP executable. You generate it on your own computer; no
prebuilt EBOOT is provided.

### Step 4. Pair it with Muse over Bluetooth (one time only)

Muse pairs a gadget over Bluetooth, and the PSP has no Bluetooth. So a
device that has Bluetooth does that conversation once. What comes out
of it is small: an identity and a pair of tokens, which you hand to the
PSP. After that, the PSP connects to Muse by itself and renews its own
credentials.

Using the Linux client from the Muse Gadget SDK:

1. Install it on a Raspberry Pi or a Linux computer with Bluetooth,
   and pair it from the Muse app, following the SDK Linux client's
   README.
2. Stop it, so it is no longer using the pairing:

```sh
sudo systemctl disable --now musegadget
```

3. Copy `identity.json` and `pairing.json` out of `/var/lib/musegadget`
   to your computer.
4. Hand them to the repository:

```sh
tools/pair.py path/to/the/folder
```

What this produces:

- A pairing saved in `state/`, which git ignores. Treat it like a
  password.

Two rules matter. Only one device can use an identity at a time, and
every renewal replaces the tokens. So once the PSP has the pairing,
leave the client it came from switched off, or remove it.

### Step 5. Copy it to the PSP

Put your SDK token in a file named `.sdk_token` in the repository
folder, and your OpenAI key in `.openai_key`. Git ignores both.

Put the PSP in Settings > USB Connection, then:

```sh
tools/install.py
```

What this does:

- Copies the app and the avatar to `PSP/GAME/Muse` on the Memory Stick
  and ejects it.

Run it again after every rebuild. It never overwrites a pairing that
is already on the stick.

### Step 6. Connect Wi-Fi (this is not pairing)

Wi-Fi setup is separate from Bluetooth pairing. Start your custom
firmware, join Wi-Fi once in the PSP's Network Settings, make sure the
date and time are set, and run Muse from the Game menu.

The app joins the first connection saved in the PSP's Network Settings.
Hold L while it starts and it joins the second one.

### Step 7. Boot and test the connection

The corner of the screen shows whether the PSP can reach Muse, and the
avatar shows what it is doing:

- An amber light and "connecting": joining Wi-Fi or reaching Muse.
- A green light and "online": ready. Hold R to talk.
- A red light and "offline": Muse has been out of reach for 45 seconds,
  or the PSP is not paired. The app keeps trying.
- The avatar in blue: listening while you hold R.
- The avatar in pink, with thought dots: your question is on its way,
  or Muse is answering.
- The avatar in green, with sound waves: speaking the answer.
- Hearts for a few seconds, no sound: Muse answered and the PSP could
  not speak it (for example, no voice key was provided).

Test it: hold R to talk and release to send. Press HOME to quit.

If something misbehaves, read `log.txt` next to the app on the Memory
Stick (the previous run is kept as `log-before.txt`). The log names
the Wi-Fi network and its signal strength, which is the first thing to
check. Note that the PSP's radio is weak and needs a strong Wi-Fi
signal, and that long answers are spoken only up to about the first
420 characters.

You can also ask Muse from any device to act on the PSP, naming the
device: `psp.say` speaks a message through the PSP, `psp.dance` makes
the avatar dance, and `psp.status` reads the battery and Wi-Fi.

## Status

- [x] EBOOT builds and boots on real hardware (Rust port)
- [x] Custom avatar generated and rendering
- [x] SDK token issued; one-time BLE pairing completed
- [x] Install device identity to the memory stick and verify online state
- [x] First end-to-end voice conversation on the PSP
- [x] Demo video (70 seconds, checked)

## Security note

`identity.json`, `pairing.json` and the SDK token are per-device secrets.
They are listed in `.gitignore` and are never committed to this repository.

## Credits and license

Based on the open-source muse-gadget-psp project by wobsoriano
(Apache License 2.0). See `LICENSE` and `NOTICE` in the upstream project.
