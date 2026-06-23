---
name: machin-game-simon
description: Build, run, and modify machin-game-simon — Simon Says as a native raylib desktop game with sound, written in machin (MFL). Use when working on this repo, or as a worked example of audio / opaque FFI handles (cstruct Sound {}) and a frame-timed state machine in machin.
---

# machin-game-simon

Simon Says as a native raylib desktop window with sound, written in [machin](https://github.com/javimosch/machin) (MFL). It is the reference example for **audio** and **opaque FFI handles** in machin (the sibling sprite/terminal games are [machin-game-flappy](https://github.com/javimosch/machin-game-flappy) and [machin-game-snake](https://github.com/javimosch/machin-game-snake)).

## Build & run

```bash
./build.sh                 # machin encode simon.src -> simon.mfl, then machin build -> ./machin-game-simon
./machin-game-simon        # run from the repo root so assets/ resolves
```

Needs `machin` **v0.44.0+** (uses `cstruct Sound {}`), a C compiler, **raylib**, a display, and an audio device.

- **System raylib** (`apt-get install libraylib-dev`, `brew install raylib`): detected and built directly.
- **No root:** `build.sh` vendors raylib's prebuilt static release into `vendor/` and injects `cflags`/`:libraylib.a` into a throwaway `.mfl`; the committed `simon.src` stays system-style.

Controls: click a pad or press 1/2/3/4 to repeat the sequence; R restarts, Esc quits.

## Audio over the FFI: opaque handles

raylib's `Sound` is a by-value struct that **contains pointers** (`AudioStream { rAudioBuffer*; rAudioProcessor*; ... }`), so it can't be a numeric `cstruct`. machin v0.44.0 added the **opaque handle** for exactly this:

```
cstruct Sound {}            // empty body = opaque
fn LoadSound(string) Sound  // machin holds the real C struct by value...
fn PlaySound(Sound)         // ...and passes it back without naming its fields
```

What you can do with an opaque value: receive it from a `fn`, store it (a variable **or** a slice — this game keeps `[]Sound` and indexes by pad), and pass it to another `fn`. What you can't: construct it (`Sound{}`) or read a field. (A single pointer is simpler — use the `ptr` FFI type, held as an `int`.)

```
sounds := []Sound{}
sounds = append(sounds, LoadSound("assets/pad0.wav"))
...
PlaySound(sounds[p])
```

## Patterns worth copying

- **Frame-timed state machine.** Immediate-mode means you can't `sleep` mid-game (it freezes the window). `show` advances a per-step frame counter (`LIGHT` lit frames, then `GAP` dark) and walks the sequence; `input` matches clicks; `over` waits for R. One `mode` int + a few counters.
- **Sentinel via `0 - 1`.** `lit := 0 - 1` for "no pad lit" — a plain `-1` literal also works; the subtraction form is here just to read as a sentinel. Compared with `lit == i`.
- **Click hit-testing.** `pad_at(GetMouseX(), GetMouseY())` walks the four pad rects and returns the index or -1; the same `pad_x`/`pad_y` helpers place the rectangles.
- **Random without a PRNG builtin.** `byte_at(rand_bytes(1), 0) % 4` picks the next pad.
- **f32 only via literals.** `DrawCircle(250, 250, 70.0, c)` — the radius is `f32`, so pass a float *literal* (`70.0`); a concrete int there would need `float()` (see the machin int/float rule).

## Modifying

- **Tempo / difficulty:** `LIGHT` and `GAP` (playback speed), the `30`-frame lead-in, the `14`-frame input flash.
- **Sounds:** replace the WAVs in `assets/` (keep `pad0..pad3` + `buzz`). Any format raylib loads (wav/ogg/mp3/flac) works.
- **Colors / layout:** `pad_color`, `pad_x`/`pad_y`, `WIN`/`PAD`.
- After any edit to `simon.src`, re-run `./build.sh` (never hand-edit `simon.mfl` — it is generated).
