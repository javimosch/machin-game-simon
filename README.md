# machin-game-demo-simon

**Simon Says** as a real **native desktop app** — written in **[machin](https://github.com/javimosch/machin)** (MFL) and drawn with [raylib](https://www.raylib.com/) through machin's C FFI. The machine flashes a growing sequence of colored pads, each with its own **tone**; you repeat it back by clicking. Miss one and it buzzes. How long a sequence can you hold?

Part of [**awesome-machin**](https://github.com/javimosch/awesome-machin) — the machin ecosystem.

> **Agents:** [`SKILL.md`](SKILL.md) covers the build (incl. no-root raylib), the opaque-handle FFI for audio, and the state machine.

```
+-------------------------+
|  green    |    red      |
|        ( 3 )            |   ← round number in the hub
|  yellow   |    blue     |
+-------------------------+
        watch / repeat
```

## Why it exists

The machin north star is "build real things, let usage drive features." This is the **audio** dogfood — the third game in the set, after the terminal [machin-game-demo-snake](https://github.com/javimosch/machin-game-demo-snake) and the sprite-driven [machin-game-demo-flappy](https://github.com/javimosch/machin-game-demo-flappy). Simon is the game where **sound *is* the mechanic**, so it can't fake the requirement.

And the requirement drove a real language feature. raylib's `Sound` is a by-value struct that **contains pointers**, so it can't be a numeric `cstruct` — it needs an **opaque handle**, which became **machin v0.44.0**:

```
cstruct Sound {}            // empty body = opaque: machin holds the real C
fn LoadSound(string) Sound  // struct by value and passes it back to PlaySound
fn PlaySound(Sound)         // without ever naming its fields
```

machin loads five `Sound`s into a `[]Sound`, indexes them by pad, and plays them — all over the new opaque-handle FFI. That unlocks every "load a handle, pass it back" C library, not just audio.

## Build

Needs the `machin` compiler (**v0.44.0+**), a C compiler, **raylib**, a display, and an audio device.

```bash
./build.sh            # → ./machin-game-demo-simon
./machin-game-demo-simon   # run from the repo root so it finds assets/
```

`build.sh` uses a **system raylib** if installed (`sudo apt-get install libraylib-dev`, `brew install raylib`, …); otherwise it **vendors raylib's prebuilt static release** into `vendor/` automatically — no root required.

## Play

- **Watch** the machine flash + sound a sequence of pads.
- **Repeat** it: click the pads (or press **1**/**2**/**3**/**4** for green/red/yellow/blue) in the same order.
- Complete it and the sequence grows by one. Miss and it buzzes — **R** to play again, **Esc** to quit.

## How it works

- **State machine.** `show` (flash the sequence, frame-timed so the window never blocks) → `input` (match clicks against the sequence) → `over`. Each pad lights its bright color and plays its tone; an input feedback flash lasts a few frames.
- **Audio.** `assets/` holds five WAVs (four pad tones + a buzzer), loaded once into `[]Sound`. `rand_bytes` grows the sequence.
- **Layout.** Four quadrant pads with a central hub showing the round number.

See [`simon.src`](simon.src). `build.sh` runs `machin encode` to produce the canonical `simon.mfl`, then `machin build`.

## License

MIT (audio assets included under the same license)
