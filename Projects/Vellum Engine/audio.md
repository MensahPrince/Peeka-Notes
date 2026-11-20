---
id: 5iz8644rwn
title: "Audio"
sort: 9000
version: 1
---

> **Parked.** Vellum is now a non-photorealistic 3D renderer, not a game engine ([Engine Overview](page:3vibzoegdx)), so audio is out of scope. These notes are kept in case Vellum grows into a stylised game, or its animations need sound later.

Sound effects, music and the audio thread: what game audio needs, which crates to use, and the rules the audio callback lives by.

---

## 1\. How sound reaches the speakers

```
main thread                       audio thread (real-time)
  play("hit.ogg") ── command ──►  mixer: sum every playing sound, apply volume/pan/effects
  set_volume(music, 0.5)            │  fills a buffer of ~256–1024 samples
                                     ▼
                                  OS audio device (cpal: ALSA/PulseAudio/PipeWire, CoreAudio, WASAPI)
```

The device asks for the next buffer every few milliseconds (1024 samples at 48 kHz ≈ 21 ms). If the buffer is late you hear a **click or dropout**: the audio thread's deadline is far stricter than the frame's ([Game Loop and Timing](page:kom1zxtzlw)).

### The rules of the audio callback

- **No allocation** (`Vec::push`, `Box::new`, `String` formatting).
- **No locks** that the main thread might hold (`Mutex`), no file or network I/O.
- Talk to it through **lock-free queues** (ring buffers) of commands.

These are the same ownership-across-threads questions as in [Rust Patterns for Engines](page:7nta66y9tk).

---

## 2\. Crates

| Crate | Level | For |
| --- | --- | --- |
| `cpal` | Lowest: a callback that fills sample buffers | Writing your own mixer |
| `rodio` | Simple playback on top of cpal | Tools, small apps (Peeka itself plays its notification sound with it) |
| `kira` | Game audio: mixer tracks, tweens, clocks, spatial sound, effects | The pick if this comes back |
| `symphonia` | Decoding (MP3, OGG, FLAC, WAV…) | Used under the others |
| `fundsp`, `oddio` | Synthesis, DSP graphs, 3D audio | Procedural sound |

---

## 3\. What a game needs

| Need | How |
| --- | --- |
| Fire-and-forget effects | Play a sound, forget it; limit how many of one sound play at once |
| Music | Streamed from disk; crossfade between tracks with tweens |
| Buses / tracks | Master → Music, Effects, UI, Voice |
| Ducking | Lower music while dialogue plays |
| Spatial sound | Volume and pan from the listener (camera) and the emitter ([Math for Games](page:l0adi79jhz)) |
| Timing | Sounds started on a musical clock (kira's clocks), so stingers land on the beat |

---

## 4\. If it comes back

- For **animation playback**: a soundtrack synced to scene time ([Game Loop and Timing](page:kom1zxtzlw)); scrubbing seeks the audio.
- Short effects decoded at load into memory; music streamed ([Asset Pipeline](page:s79wv0aek9) would give `Handle<Sound>`).
- Systems push `PlaySound` events; one audio system plays them ([Entity Component System](page:vj3mm21ptz)).

---

## 5\. Pitfalls

- **Latency vs safety**: smaller buffers mean less delay and more risk of dropouts; 256–512 frames is a usual compromise.
- **Sample rate**: resample everything to the device's rate once (at load), not per buffer.
- **Clipping**: many loud sounds summed exceed 1.0; a limiter on the master bus.
- **Profiling**: audio dropouts don't show in frame time; watch the callback's own timing ([Performance and Profiling](page:tmmt1jyud5)).
