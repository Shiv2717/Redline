# typo_engine.py

A small kinetic-typography video engine. It renders an animated promo
video entirely from code — no stock footage, no generative AI image or
video model, no sample library for the audio. Pillow draws every frame,
ffmpeg stitches them into an `.mp4`, and numpy/scipy synthesize the
sound effects from sine waves and filtered noise. Same input, same
output, every run.

Built for **Redline** (a browser-based paraphrase/humanize/summarize/
read-check tool), but the engine itself is generic — swap in your own
beats and theme for any other project.

## Requirements

- Python 3 with `pillow`, `numpy`, `scipy`
- `ffmpeg` on your `PATH`
- The DejaVu Serif / DejaVu Sans Mono fonts (ships by default on most
  Linux systems; point the `SERIF`/`MONO` constants elsewhere if not)

```bash
pip install pillow numpy scipy
```

## Usage

```bash
python3 typo_engine.py
```

Writes `redline_promo.mp4` (1280×720, 30fps, H.264 video + AAC audio)
to `/mnt/user-data/outputs/` by default. Change the output path with
`render(out_path="...")`.

## How it's structured

**Beats.** Each scene is a plain function `beat_name(t) -> PIL.Image`,
where `t` goes from 0 to 1 over that beat's duration. Inside, you
animate by computing positions/colors/sizes as a function of `t` (see
`ease_out`, `lerp`, `lerp_color` helpers). A beat's audio cues live in
a parallel list right below it, e.g.:

```python
BEAT_PARAPHRASE_CUES = [(0.02, "scratch", {"dur": 0.35}), (0.46, "chime_soft", {})]
```

Each tuple is `(t_within_beat, cue_name, kwargs)` — `cue_name` must
match a key in `CUE_LIB`.

**Timeline.** `TIMELINE` at the bottom of the beat definitions is the
ordered list of `(beat_function, duration_seconds, cues)`. This is the
only thing you need to touch to reorder, retime, add, or remove scenes.

**Transitions.** Between every pair of beats, `ink_wipe_frames()`
generates a short red-bar wipe (duration set by `TRANSITION_SEC`) from
the outgoing beat's last frame to the incoming beat's first frame, plus
a `"whoosh"` audio cue. This happens automatically — you don't add it
to `TIMELINE` yourself.

**Audio.** `CUE_LIB` maps cue names to zero-argument (or
keyword-argument) functions that return a numpy float array:
- `tone(freq, dur, amp, harmonics)` — a decaying bell/chime tone
- `band_noise(dur, low, high, amp, ...)` — filtered noise (scratches,
  whooshes, clicks)
- `click(amp)` — a short two-noise-band click (typewriter tick)

A quiet two-note background pad (`render_pad`) plays under the whole
video so it's never dead silent. All cue events get mixed onto one
master audio array at the right sample offset, soft-clipped, then
muxed onto the silent video with ffmpeg.

## Customizing

- **Colors/fonts** — edit the `THEME` dict and the `SERIF`/`MONO`
  constants at the top.
- **Canvas size / frame rate** — `W`, `H`, `FPS` constants.
- **Add a beat** — write a new `beat_x(t)` function plus a
  `BEAT_X_CUES` list, then add `(beat_x, duration_seconds, BEAT_X_CUES)`
  to `TIMELINE` wherever you want it to play.
- **Change a sound** — every effect is defined in one place, `CUE_LIB`;
  tweak frequency/duration/amplitude directly, no audio files to source.
- **Transition feel** — `TRANSITION_SEC` controls wipe speed;
  `ink_wipe_frames()` controls its look (currently a left-to-right red
  bar sweep).

## Known limitations

- Text layout is manual (`d.text(...)` with hand-placed coordinates) —
  there's no auto-wrapping, so longer strings need their position
  recalculated by hand.
- No text antialiasing beyond what Pillow/freetype does by default at
  this resolution — fine at 1280×720, would want supersampling at
  higher output resolutions.
- Mono audio only.
- Rendering is frame-by-frame via Pillow (not GPU-accelerated); a
  20-second video takes a few seconds to render but a much longer video
  would scale linearly.
