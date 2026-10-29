# KOICE

Native macOS dictation, written in [Koru](https://github.com/korulang/koru).
Hold a key, talk, text lands at your cursor.

One `.k` file. A 1.1MB inference dylib. 17MB of model weights.
No Electron, no Python, no runtime — the product is the toolchain's showcase.

## Stack

| Layer | Piece |
|---|---|
| Language | [koru](https://github.com/korulang/koru) (`koruc`) |
| Platform | [oshun](https://github.com/korulang/oshun) — AppKit, Carbon hotkeys, Metal overlay, CoreAudio, NSPasteboard |
| Inference | [koru-libs](https://github.com/korulang/koru-libs) `needle` — FFI to `libneedle.dylib` (C ABI) |
| Model | [cactus-needle](https://crates.io) Whistle weights (`~/.cache/cactus-needle/whistle/2.0.0/whistle.cact`) |

## Build

Clone all four repos side by side:

    git clone https://github.com/korulang/koru
    git clone https://github.com/korulang/koru-libs
    git clone https://github.com/korulang/oshun
    git clone https://github.com/korulang/koice

Drop `libneedle.dylib` into `koru-libs/needle/vendor/` (ships with the
cactus-needle `needle` package), place `whistle.cact` under
`~/.cache/cactus-needle/whistle/2.0.0/`, then:

    cd koice/app && koruc run koice.k

## Controls

- **Ctrl+Option+Period** (hold) — dictate: pill appears, bar tracks your voice; release pastes the transcript.
- **Ctrl+Option+Q** — quit.

Needs microphone permission; paste (synthetic Cmd+V) needs an
Accessibility/Input Monitoring grant for your terminal.

## How it works

`app/koice.k` is the whole application — read it in one sitting.
`host.run` idles on a Carbon hotkey ring; press-down drains the mic ring
(discarding idle ambient), mints a borderless Metal pill, and enters a
nested `frames` pump drawing live RMS; release hides the pill, drains
the VoiceProcessingIO ring into `needle.transcribe`, and pastes the
`text` field via NSPasteboard + CGEvent. All OS glue lives behind `.kz`
obligation boundaries in oshun; all FFI behind `koru-libs/needle`.

## Roadmap

- `keywords:` vocabulary dictionary (the FFI already takes it)
- `needle_complete` rewrite pass with rolling-conversation context
- Tray icon + resident agent
- `surface` backend abstraction (challenge: extract the Metal concretization)
