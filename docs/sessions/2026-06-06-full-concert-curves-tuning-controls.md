# Session: 2026-06-06 — Full-concert audio curves + tuning controls + F2-driven manual seed v0.2

**Repo(s):** concert-mvp (primary), sidecar (recordings only)
**Interface:** Claude Code (Opus 4.7 1M)
**Branch(es):** concert-mvp `feat/full-concert-curves-tuning-controls`; umbrella `docs/2026-06-06-full-concert-tuning-controls`
**Duration:** Multi-day spread of short sessions, captured here as one wind-down

## Context loaded

- Umbrella `CLAUDE.md`, `INSTRUCTIONS.md`, `HANDOFF.md`
- `repos/sidecar/CLAUDE.md`, `repos/sidecar/HANDOFF.md`
- `repos/concert-mvp/HANDOFF.md` (touched directly)
- F2 annotation flow: `repos/sidecar/docs/event-schema.md` + `repos/concert-mvp/src/annotations.js`
- Earlier rider note: F2 / concert-mvp pre-existing code was sending all preset tags as `marker` regardless of which digit was pressed (since fixed locally by Codex on `develop`)

## Decisions made

- **Tune the manual curve by *creating a new revision*, not overwriting v0.1.** Preserves the original baseline; both selectable in the dropdown for A/B. New: `bnnIdWzGSYIManualSeedV02Curve` (`model_version: manual-seed-v0.2-motorspirit-mindfuzz`).
- **Use rider's note semantics + displayed v0.4 intensity as baseline** when picking values for the 26 anchor points. Notes like "false drop" raise toward peak; "less intense than Metal Peaks due to genre" cap below the metal ceiling; "banter through mic / should not be a spike" pull toward the breakbeat floor.
- **One intensity smoothing slider governs three signals.** Chart derived overlay, controller target series, *and* the BPM line all read from `derivedIntensityPoints()` → `applySymmetricSmoothing()` (intensity-sampling.js). Single knob, consistent semantics. Can split later if independent windows are wanted.
- **Symmetric (centered) smoothing, not causal exponential.** Music peaks shouldn't drift in time when smoothed. Different from the pre-existing terrain smoothing (causal exponential) on purpose.
- **Style segments stay as a coarse genre prior.** Fine moment-to-moment shaping belongs in the manual seed lineage (v0.2 anchor approach). v0.4 unsegmented regions get `style_prior = 0` — works fine, just no metal boost.
- **Full-concert extraction supersedes the 20-min preview.** No more manual-seed splicing inside the song range; the audio extraction is the sole source of truth for t≥833. Manual seed only covers the 0-833 warmup intro now.
- **Curve persistence is per-video** (`gizzERG:curveSelection:<videoId>`). When the library adds more concerts each remembers its own last-selected curve.
- **BPM is *not* an intensity feature.** Stored unnormalized (absolute musical value) in `audio_features.bpm`. Chart line and tooltip prefer per-window when available; fall back to per-section `cue.bpm` for curves without audio data. Tooltip labels the source explicitly.

## Work done

### concert-mvp — feature branch `feat/full-concert-curves-tuning-controls`

**Source:**
- `src/library/intensity-curves.js` — added `bnnIdWzGSYIManualSeedV02Curve` (26 F2-anchored points across t=1607-2036); renamed audio constants to `bnnIdWzGSYIAudioFeaturesV03` / `bnnIdWzGSYISubjectiveV04`; library ids renamed (`audio-v0.3`/`audio-v0.4-subjective`); manual-seed splice removed for t≥833; default metal `styleSegments` extended (The Balrog 0.40, Iron Lung 0.35, Evil Death Roll 0.50, Hog Calling Contest 0.40).
- `src/audio-derived-curves.js` — fully regenerated from full-concert extraction (3974 points covering t=833→8779 at 2s sample step, each with `audio_features.bpm`).
- `src/intensity-sampling.js` — new `applySymmetricSmoothing(series, windowS)`: symmetric centered moving-average, time-aware, peaks preserved at original timestamps. 4 unit tests.
- `src/app.js` — intensity-smoothing slider wiring (els, handler, applied inside `derivedIntensityPoints()`); curve dropdown localStorage persistence (`curveSelectionStorageKey` / `readStoredCurveSelection` / `persistCurveSelection`); init-order fix (`activateSelectedProfile({ reloadVideo: false })` now runs after `populateCurveSelect` so restored choice actually loads); Authored Cues overlay (`drawAuthoredCuesCurve`) on rose-magenta; `audioBpmAt(time)` helper with smoothing-aware averaging; guidance text + hover tooltip prefer per-window BPM and label source.
- `index.html` — `intensitySmoothing` range input under Curve dropdown; `authoredCuesToggle` checkbox in chart header.

**Tests:**
- `tests/intensity-sampling.test.mjs` — 4 new tests for `applySymmetricSmoothing` (no-op when window=0, centered spike averaging, non-uniform spacing).
- `tests/profile-library.test.mjs` + `tests/subjective-intensity.test.mjs` — updated for renamed library ids and v0.2 entry.
- Total: 65/65 passing.

**Profile-builder:**
- `tools/profile-builder/build_profile.py` — added per-frame tempo via `librosa.feature.tempo(..., aggregate=None)`; passes through unnormalized; backwards-compatible (older feature JSON without `bpm` still loads).
- Re-ran extraction on cached webm (2h 26m) trimmed to 833→end. 3974 points produced; BPM coverage 100%; range 68-172.

**Docs:**
- `HANDOFF.md` — comprehensive 2026-06-06 update with feature table + known UX gotcha pointer to gizzERG issue #4.

### sidecar — recordings only (gitignored)

- `docs/recordings/music-tuning-01.jsonl` — 41 Motor Spirit markers from earlier session (pre-Codex F2 fix; notes lost, timestamps preserved).
- `docs/recordings/f2-smoke-test.jsonl` — 10-keypress validation of Codex's F2 note-attach fix. All five preset tags + custom tag came through correctly with notes.
- `docs/recordings/semantic-test-02.jsonl` — 27 rider annotations across late Motor Spirit + Mind Fuzz medley; 26 with rich semantic notes ("false drop", "vocals stop but music at peak", "bayou blues, less intense than Metal Peaks due to genre", etc.). Source data for v0.2 manual seed tuning.
- `docs/recordings/semantic-test-02.annotations.csv` — flattened export for analysis.

### gizzERG GitHub

- Filed issue #4: F2 overlay digit hotkey auto-submits before note can be typed. Three suggested fixes; repro steps; references the lost Motor Spirit annotation set.

## Open threads

- **Task #7** (concert-mvp): per-song music-end vs authored cue boundary. Bandcamp track durations don't precisely match where music ends within songs (banter/applause sits in the section tail). Approaches: (a) auto-detect from existing audio features — sustained loudness drop or onset_density floor; (b) F2-driven manual song-end markers; (c) hybrid auto+confirm. Captures rider's Motor Spirit observation ("Song ends abruptly... should drop to a consistent level for the song break").
- **Browser ES-module cache** keeps biting after JS regeneration. `audio-derived-curves.js` doesn't carry a `?v=...` cache-buster like `app.js` imports do. Hard-refresh after every regen is the workaround. Consider adding a version query string if regens become more frequent.
- **v0.4 styleSegments** still only cover four songs (Gila Monster, Motor Spirit, The Balrog, Iron Lung, Evil Death Roll, Hog Calling Contest). The other ten Night-2 songs get `style_prior = 0`. Extend in `intensity-curves.js:bnnIdWzGSYISubjectiveFull` as F2 tuning reveals which sections want a genre boost. Song boundary table in HANDOFF.
- **gizzERG / Codex local divergence** still pending Codex's rebase (from umbrella HANDOFF 2026-05-25). My commit here is on a feature branch; should compose cleanly with whatever Codex pushes next.
- **Whether to extend the smoothing slider to two independent knobs (intensity vs BPM)** — punted. Single knob is fine while tuning; revisit if rider wants different windows for the two.

## Next session entry point

> "Full-concert v0.3/v0.4 with per-window BPM is live on `feat/full-concert-curves-tuning-controls`; v0.2 carries 26 F2-derived anchors for Motor Spirit + Mind Fuzz; intensity smoothing + Authored Cues overlay + dropdown persistence shipped. 65 tests pass. Open: task #7 (per-song music-end detection — three approaches written up in the task), styleSegments coverage for the other 10 songs as F2 tuning reveals priors, and the gizzERG F2 UX issue #4 (digit auto-submits before note can be typed). Code is committed but unpushed; review and merge/PR at convenience."

## Loose notes

- `imageio_ffmpeg` shipped a bundled ffmpeg at `Lib/site-packages/imageio_ffmpeg/binaries/ffmpeg-win-x86_64-v7.1.exe`. Worked around the "ffmpeg not on PATH" check by copying it to `ffmpeg.exe` and prepending the dir to PATH for the extraction. Worth noting for future re-extractions if system ffmpeg still isn't installed.
- Full extraction on the cached webm took ~13 minutes wall clock (27 chunks at ~30s each). Memory stayed bounded as designed.
- Browser cache caught me twice this session: once after the audio constant rename (browser still asked for old export name), once when verifying BPM smoothing (slider had no visible effect because the loaded curve was actually manual-seed, not v0.4 as the dropdown displayed). Init-order fix landed for the latter; cache-buster query string remains a candidate for the former.
- Rider F2 annotations are *extremely* high-signal data for tuning. The 27 Mind Fuzz / Motor Spirit notes carried enough genre-and-structure context to anchor a full manual revision in one pass. Worth investing in the F2 capture experience before scaling tuning to more videos — issue #4 fix would help.
- The lost-notes incident (`music-tuning-01.jsonl`, all 41 events came in as `tag: "marker"` with no notes) — the codepath was already corrected on the concert-mvp `develop` branch by the time I looked. Notes weren't recoverable from any client-side or sidecar-side persistence; the JSONL only contains what crossed the wire. Documented in the gizzERG HANDOFF's annotations section.
