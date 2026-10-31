# Changelog

## V1.0

First stable release: HexedVoiceMod 1.0.0 (library API 9, menu 1.0). It replaces the VoiceFX v8 beta and ends the VoiceFX v1..v8 numbering. The parameter ids 0..10 and every earlier id are unchanged; everything new is appended (ids 43..66). Every new stage is off or transparent by default, with these exceptions that change the default sound on purpose: the 75 Hz input high-pass, the lookahead limiter (instead of the tanh knee), the adaptive Auto-Tune analysis window, the 8-line reverb (instead of Freeverb), the 20-band vocoder, the oversampled drive. 1.0 does not reproduce the v8 sound and has no switch back to it (an A/B "Compare" switch was built during development and removed again before release, so the engine has one code path).

### What is new
* **One code path (final cleanup)**: during development an A/B "Compare" switch (`legacy_mode`) kept the previous algorithms alive next to the new ones. It was removed again: no cross-fades, no second reverb / vocoder / drive / limiter / gate, no reference engine in the tests. Measured on the host (whole engine, x86, 48 kHz, 20 s of voice): Normal preset 25.9 -> 21.5 ns per sample, Lite pitch 41.8 -> 32.4, cyber vocoder + reverb 292 -> 275, everything on 373 -> 363. Parameter slot 43 stays reserved so the ids after it never change.
* **Rename** to HexedVoiceMod (`libHexedVoiceMod.so`, `HexedVoiceMod_Menu.lua`, `/hvm` and `/hexedvoice`, `/vfx` and `/voicefx` kept), double-load guard against the old `libvoicefx.so`, one-time import of the old config, menu and favorites files.
* **Conditioning and dynamics**: DC blocker + 75 Hz high-pass, gate with hysteresis / hold / soft expander / lookahead, compressor with soft knee and auto makeup, split-band de-esser, 3 ms lookahead limiter with a ceiling.
* **Auto-Tune**: lag compensation (`tune_lookahead_ms`), adaptive analysis window, vibrato preservation (`tune_keep_vibrato`), octave protection kept.
* **Hear myself**: servo (+-1 % playback rate) instead of dropping audio, measured latency, hard reset only on a stall.
* **Quality**: AI noise suppression (RNNoise), formant-preserving chord voices, oversampled drive, 8-line FDN reverb with pre-delay and damping, echo damping / low cut, 20-band vocoder with a carrier that follows your pitch or snaps to the scale.
* **CPU**: lookup tables instead of per-sample `sinf` / `cosf`, automatic return from Lite to RubberBand, per-stage CPU meters.
* **Bug fix**: a rare out-of-bounds read in the echo (one float past the buffer) is fixed.
* **Docs and licence**: README, GPL-2.0-or-later licence, third-party notices.

### New parameters (appended; ids 0..42 unchanged)
| id | name | default | range | notes |
|---|---|---|---|---|
| 43 | reserved | - | - | unused slot (ids never change) |
| 44 | `gate_hold_ms` | 80 | 0..500 | |
| 45 | `gate_range_db` | -60 | -80..-6 | |
| 46 | `gate_hyst_db` | 4 | 0..20 | |
| 47 | `comp_on` | 0 | 0/1 | |
| 48 | `comp_thresh_db` | -18 | -60..0 | |
| 49 | `comp_ratio` | 3 | 1..20 | |
| 50 | `comp_attack_ms` | 10 | 1..100 | |
| 51 | `comp_release_ms` | 150 | 20..1000 | |
| 52 | `comp_makeup_db` | 0 | 0..24 | |
| 53 | `comp_auto` | 1 | 0/1 | auto makeup |
| 54 | `deess_on` | 0 | 0/1 | |
| 55 | `deess_amt` | 0.5 | 0..1 | |
| 56 | `deess_freq` | 6500 | 4000..9000 | Hz |
| 57 | `limit_ceil_db` | -0.9 | -12..0 | |
| 58 | `tune_lookahead_ms` | 0 | 0..60 | 0 = previous Auto-Tune timing |
| 59 | `tune_keep_vibrato` | 0 | 0..1 | 0 = previous behaviour |
| 60 | `ns_engine` | 0 | 0/1 | 0 Classic, 1 AI (RNNoise) |
| 61 | `chord_formant` | 0 | 0..1 | 0 = previous chord voices |
| 62 | `rev_predelay_ms` | 0 | 0..100 | |
| 63 | `rev_damp` | 0.5 | 0..1 | |
| 64 | `echo_damp` | 0 | 0..1 | 0 = previous echo |
| 65 | `echo_hp_hz` | 0 | 0..800 | 0 = off |
| 66 | `voc_carrier` | 0 | 0..2 | 0 fixed, 1 follow, 2 snap to scale |

New library stats: `vfx_stat` 17 (version x100), 18 (old library found), 19 (settings imported), 21 / 22 (monitor resets / dropouts), 23 (RNNoise built in), 24 / 25 (CPU-recovery probes / recoveries); `vfx_statf` 9..12 (gate, compressor, de-esser, limiter), 13 (analysis window), 14 / 15 (monitor queue ms / rate offset %), 16 (noise-suppression delay), 17..21 (CPU % per stage).

### New commands
`/hvm comp on|off|auto`, `/hvm deess on|off`, `/hvm tune look <ms>`, `/hvm tune vibrato <0..1>`, `/hvm noise ai|classic`; every parameter also works with `/hvm set <name> <value>`.

### Expected extra latency per feature
| feature | extra delay |
|---|---|
| gate lookahead + limiter lookahead (V1, always) | 8 ms (5 + 3) |
| `tune_lookahead_ms` | exactly the value you set (default 0) |
| Classic noise suppression | 10.7 ms (512 samples; now included in the reported latency) |
| AI noise suppression | 20 ms (960 samples, measured) |
| formant chord voices | none (same 770 samples as the Lite voice) |
| hear myself | queue of about 50 ms + the device output buffer (not measured) |
| everything else | none |

### Known limits
* 32-bit `armeabi-v7a`, 48 kHz mono 16-bit microphone; AI noise suppression needs 48 kHz.
* The always-on delay (8 ms) and the noise-suppression delay are part of V1.
* Chord voices with preserved formants: about 96 Hz to 1.1 kHz only (deeper voices fall back to the normal voice).
* RNNoise passes loud steady noise for the first second or so; Classic needs about a second of quiet to learn the room.
* A failed CPU-recovery probe costs a few seconds of RubberBand load every 20 s while the phone struggles.
* The best `tune_lookahead_ms` for the RubberBand engine is unknown (host tests only have a stand-in); with the Lite engine 20-25 ms flattened vibrato best.
* `menu/HEXED/Fonts/Arial_Bold.ttf` is a proprietary font bundled from the earlier releases (see THIRD_PARTY_NOTICES.md).

### Not verified
Nothing has been tested on a real device or built with the real `ndk-build` by the author of these changes; no test results from your phone had been reported when this was written, so update this list after you test. Everything below needs a device or the real toolchain:
* The real `ndk-build` for `armeabi-v7a` (the host tests compile the DSP with g++ on x86; `main.cpp` was only syntax-checked against stubbed Android headers).
* **RNNoise on armeabi-v7a**: the source list (`denoise.c rnn.c rnn_data.c pitch.c kiss_fft.c celt_lpc.c`) and flags (`-O3 -DNDEBUG -fno-fast-math`, as a separate C static library) were compiled and run only with gcc on x86.
* The BASS hook on the Nezuko build, the double-load guard with both libraries present, the one-time import of the old settings, and the storage paths on your phone.
* BASS behaviour the monitor servo assumes: `BASS_DATA_AVAILABLE` on a push stream, click-free `BASS_ATTRIB_FREQ` changes, freeing and recreating the stream from the record thread.
* CPU load of every stage (the stage meters, the 20-band vocoder, the FDN reverb, RNNoise, the sine tables), the real effect of the CPU recovery probes, and latency numbers (monitor queue + device buffer, total chain).
* How the 1.0 sound behaves on real voices and your microphone (gate, compressor and de-esser defaults, the new reverb, formant chord voices, RNNoise settling time, vocoder following your pitch).
* Menu layout on different screens (the Tune, Chords, Clean, Space and Status pages were tightened to fit; 10 effect tabs), and the Lua code was only syntax-checked.
* The best `tune_lookahead_ms` for RubberBand.

### Details by phase
#### Phase 0 - rename (no DSP changes)
* Product is now **HexedVoiceMod 1.0.0**; the VoiceFX v1..v8 numbering ends. `VFX_API_VERSION` = 9 (separate from the product version).
* New stats: `vfx_stat(17)` = 100 (product version x100), `vfx_stat(18)` = old libvoicefx.so found (1/0), `vfx_stat(19)` = settings imported from the old voicefx.cfg (1/0).
* Library: `libHexedVoiceMod.so` (`MYMOD(com.hexed.voicemod, HexedVoiceMod, 1.0.0, brruham)`, logger tag `HexedVoiceMod`). Workflow `build-hexedvoicemod`, artifact `HexedVoiceMod`.
* Double-load guard: if the old `libvoicefx.so` is loaded in the process (dlopen NOLOAD, then a /proc/self/maps scan) the BASS hook is NOT installed; the log gets a clear line and the menu shows a red warning "Old libvoicefx.so found in the mods folder - delete it".
* Paths: `/storage/emulated/0/HexedVoiceMod.cfg`, `/storage/emulated/0/HexedVoiceMod_log.txt`. One-time import of `voicefx.cfg` (never deleted). Menu files `config/HexedVoiceMod_settings.cfg` / `config/HexedVoiceMod_favorites.txt` are imported once from the `VoiceFX_*` files.
* Menu: `HexedVoiceMod_Menu.lua`, `script_name('HexedVoiceMod')`, version 1.0, chat prefix `[HexedVoiceMod]`. Commands `/hvm`, `/hexedvoice` added; `/vfx` and `/voicefx` still work. New loader (tries `libHexedVoiceMod.so`, then scans for `hexedvoicemod` / `voicefx`). The menu needs API >= 9 and shows "Update libHexedVoiceMod.so" otherwise.
* Icon and font folders unchanged (`HEXED/Icons/VoiceFX/`, `HEXED/Fonts/`). Internal names (`vfx_*`, `P_*`, `voicefx_dsp.h`, `vc_api`) unchanged.
* CI: a naming check fails the build if "beta" or "VoiceFX v8" reappears outside CHANGELOG.md / README.md; host tests now run in both modes.
* Extra latency / CPU: none.

#### Phase 1 - signal conditioning and dynamics
* **Input conditioning** (always on in V1): DC blocker (~10 Hz) + 75 Hz Butterworth high-pass before noise suppression and the tuner; primed on the first sample, so no start-up step.
* **Gate rework**: hysteresis, hold, soft 3:1 expander with a floor, 5 ms lookahead (the detector sees the un-delayed signal). New params `gate_hold_ms` (80), `gate_range_db` (-60), `gate_hyst_db` (4). The threshold is still `gate_db` (-80 = off).
* **Compressor** after gate + gain, before the pitch stage: feed-forward, 6 dB soft knee, ~10 ms mean-square detector. Params `comp_on` (0), `comp_thresh_db`, `comp_ratio`, `comp_attack_ms`, `comp_release_ms`, `comp_makeup_db`, `comp_auto` (auto makeup = half of the reduction a full-scale signal would get).
* **De-esser** after pitch + chords, before the FX rack: split-band, only the band around `deess_freq` is turned down when it dominates. Params `deess_on` (0), `deess_amt`, `deess_freq` (4-9 kHz).
* **Limiter**: lookahead limiter (3 ms) replaces the tanh knee in V1: never exceeds `limit_ceil_db` (default -0.9 dB), smooth 60 ms release. Its latency is included in the reported latency.
* Menu: new Effects > **Dynamics** tab (Gate / Compressor / De-esser / Limiter, live meters via `vfx_statf` 9..12). Commands `/hvm comp on|off|auto`, `/hvm deess on|off`; every new param also works with `/hvm set <name> <value>`.
* **Bug fix:** the echo read position could round to exactly the buffer length and read one float past the end of the buffer (heap garbage, rare click). Fixed in both V1 and legacy_mode (the only intentional difference from the old code).
* Extra latency: **+8 ms** in V1 (5 ms gate lookahead + 3 ms limiter), always. Extra CPU: small; no per-sample sinf/cosf/powf/tanhf anywhere in the audio path.
* Presets never touch the new dynamics settings (like the noise suppression settings).
* CI: the naming check now matches the spec exactly ("beta" and "VoiceFX v8").

#### Phase 2 - Auto-Tune timing
* **Lag compensation**: new param `tune_lookahead_ms` (id 58, 0..60, default 0 = previous behaviour). The pitch analysis keeps seeing the live signal; the lead pitch engine and the chord voices are fed from a copy delayed by this amount, and the dry path is delayed by the same amount, so Mix, chords and dry stay aligned. Changing the value slews the delay (0.25 samples per sample, no click). The reported latency grows by the lookahead while the pitch engine runs.
* **Adaptive YIN window**: the analysis window is now about two periods of the last detected pitch, minimum 20 ms, maximum the old 40 ms (first detection still uses 40 ms). Measured on the host: the correction is ready 10-20 ms sooner for mid and high voices, low voices not worse. New stat `vfx_statf(13)` = current analysis window in ms.
* **Vibrato preservation**: new param `tune_keep_vibrato` (id 59, 0..1, default 0 = previous behaviour). The pitch is split into a slow part (a 2-pole smoother, ~2 Hz) that is snapped to the scale note and a fast part (4-8 Hz vibrato) of which this fraction is passed through. A real note change (more than 1.2 st away from the slow part) re-anchors it immediately. The note choice follows the slow part, so vibrato no longer makes the target flip between two notes.
* Octave-error protection unchanged (single-hop octave jumps rejected, 3 hops needed for a real jump).
* Menu: Tune page gets "Tune lookahead" and "Keep vibrato" sliders (key and scale buttons made slightly smaller to fit); `/hvm tune look <ms>`, `/hvm tune vibrato <0..1>`. Presets do not touch these two settings.
* Extra latency: exactly `tune_lookahead_ms` (0 by default). The adaptive window is always on in V1 (it lowers the tracker lag, it adds nothing). CPU: slightly lower (smaller YIN window).
* Honest numbers (host, synthetic voice): correction ready about 50-60 ms after a note change with the fixed window, about 40-50 ms adaptive; best vibrato flattening with the Lite engine at 20-25 ms lookahead (swing 0.95 st with 0 ms, 0.43 st with 20-25 ms); the best value for the RubberBand engine has to be found by ear on the device.

#### Phase 3 - hear-myself monitor
* **Servo instead of dropping**: the monitor no longer throws away a mic chunk when more than 0.25 s is queued. A servo keeps the queued audio at about 50 ms (a little more when the mic delivers very long chunks: aim = max(50, chunk/2 + 25) ms, 40-120 ms) by nudging the playback rate (`BASS_ATTRIB_FREQ`) by at most +-1 %, smoothed (largest step 0.05 %). A queue above ~200 ms (an output stall) resets the stream once; nothing is ever dropped in V1.
* **Start-up**: playback starts when the target level is queued (about 60 ms for 20 ms chunks, instead of the fixed 80 ms), so the output cannot run dry between chunks.
* **Stats**: `vfx_statf(14)` measured monitor latency (mean queued audio, ms), `vfx_statf(15)` playback-rate offset in %, `vfx_stat(21)` hard resets, `vfx_stat(22)` underruns. Status page: "Hear myself: playing, NN ms queued, N resets"; `/hvm status` prints the same plus the rate offset.
* The decisions live in a small pure class (`MonitorServo` in voicefx_dsp.h); `main.cpp` only executes them (BASS calls). The audio thread takes no lock except in the rare stall reset.
* Extra latency: none added. Expected hear-myself delay is the measured queue (about 50 ms) plus the device output buffer (unknown, device dependent) plus the engine delay.
* Test infrastructure: the engine's pause-reset threshold is now a member (`gapMs`, 120 ms in production) so a stalled CI machine cannot make the chunk-size tests flaky.

#### Phase 4 - quality upgrades
* **AI noise suppression**: new param `ns_engine` (id 60: 0 Classic, 1 AI). RNNoise v0.1.1 (BSD-3-Clause, model embedded) runs on 480-sample frames through an input ring; the strength slider is the wet/dry of the AI result (0.4 + 0.6 x strength). Switching engines runs both for 13 ms, then cross-fades 10 ms. **Measured delay: 960 samples = 20 ms** (one frame of buffering + one inside the model); the noise-suppression delay is now included in the reported latency in V1 (classic 512 samples = 10.7 ms, AI 20 ms; `vfx_statf(16)`; before this phase the classic delay was not counted). Needs 48 kHz; at other rates, or without RNNoise (`vfx_stat(23)` = 0), the selector falls back to Classic. RNNoise is fetched by the workflow into `jni/rnnoise-src` (not stored in the repo); the build works without it (`-DVFX_NO_RNNOISE`). RNNoise needs about a second to settle on steady hiss (e.g. it passes loud white noise for the first ~0.5 s).
* **Chord voices with formants**: new param `chord_formant` (id 61, 0..1, default 0 = the previous voices). A pitch-synchronous overlap-add shifter (grains of two periods around pitch marks, re-spaced by T / ratio, driven by the tuner's period) is blended with the Lite voice by this amount, only while the period is reliable (about 96 Hz to 1.1 kHz, so deep voices fall back to the Lite voice). Same latency as the Lite voice (770 samples), so the voices stay aligned. Measured on the host: pitch accuracy within a few cents, and the formant peak stays near its original place where the Lite voice moves it.
* **Drive**: V1 uses 2x oversampling (4th-order filters) around a rational tanh approximation (no per-sample tanhf); aliased harmonics are lower by more than 6 dB at heavy drive, level within 1.5 dB of the previous drive.
* **Reverb**: Freeverb replaced (in V1) by an 8-line feedback delay network: Hadamard feedback, per-line damping, modulated delay lengths, pre-delay. New params `rev_predelay_ms` (0..100, default 0) and `rev_damp` (0..1, default 0.5); `rev_size` sets the decay time (RT60 0.4-4 s) and `rev_mix` the amount, as before; wet level calibrated to within +-1 dB of the old reverb at the default damping. Note: this changes the default reverb sound compared with the v8 beta.
* **Echo**: new `echo_damp` (0..1, default 0) low-pass and `echo_hp_hz` (0..800, 0 = off) high-pass inside the feedback loop; at the defaults the echo is identical to the previous one.
* **Vocoder**: 20 bands (was 12) in V1; new `voc_carrier` (id 66: 0 fixed pitch, 1 follow my pitch, 2 snap to the scale) for the Cyber carrier, driven by the tuner (the tuner now also runs when this is selected). Measured: fixed 45.0, follow 55.4 (sung 55.4), snap 55.0 (midi numbers).
* Menu: Clean tab gets an Engine selector (Classic / AI) with the real delay; Chords tab gets "Keep formants"; Effects > Space is split into Echo and Reverb pages (damping, low cut, pre-delay); Robot tab gets the carrier selector. `/hvm noise ai|classic`.
* Build: `Android.mk` builds RNNoise as a separate static C library (it refuses `-ffast-math`) when `jni/rnnoise-src/src/denoise.c` exists. Sources: denoise.c rnn.c rnn_data.c pitch.c kiss_fft.c celt_lpc.c, flags `-O3 -DNDEBUG -fno-fast-math`, include paths `include` and `src`. This exact source list and these flags were compiled and run with gcc on x86 (host tests); they were NOT compiled with ndk-build for armeabi-v7a. The workflow fetch step is optional (`continue-on-error`), and the host tests run in four modes (with / without RNNoise, with / without the RubberBand stand-in).
* Extra latency: RNNoise 20 ms when the AI engine is selected; everything else adds none. CPU: not measured on a device. The 20-band vocoder and the FDN cost more than their predecessors; RNNoise is the heaviest new item (host-side only a rough idea: it runs well under real time on a PC).

#### Phase 5 - CPU headroom
* **No more per-sample sinf / cosf**: one 4096-step sine table (linear interpolation, error under 6e-7, 20 KB) now serves the Lite shifter's Hann grain windows and the robot, tremolo, chorus, flanger and vibrato LFOs; the bit crusher's `exp2f` moved out of the sample loop. Output differs from the previous code by about -100 dB (robot, tremolo, Lite shifter and the crusher within 1 LSB; the modulated delays within 16 LSB at their worst sample, 0.1 LSB rms). No per-sample libm calls are left in the audio path.
* Host micro-benchmark (x86, glibc): Lite shifter 15.7 -> 10.6 ns per sample (1.5x), robot + tremolo + chorus 37.3 -> 24.4 ns per sample (1.5x). The phone's libm is usually slower than glibc, so the gain there should be larger, but it was not measured.
* **Automatic return from Lite**: when the Auto quality setting has switched to Lite after an overload, the engine retries RubberBand every ~20 s (audio time) for a 4 s probe. The probe averages the measured load after a 0.5 s warm-up and keeps RubberBand only if it stays at or below 55 %; a probe that overloads (load above 90 % or two slow calls in a row) is cut short and goes back to Lite. Status page and `/hvm status` show "auto-switched" and the probe / recovery counts (`vfx_stat(24)`, `vfx_stat(25)`). Quality High and Lite never probe. Cost of a failed probe: up to a few seconds of RubberBand load every 20 s while the phone is struggling.
* **Per-stage CPU meters** (`vfx_statf(17..21)`, Status page row "CPU by stage"): input chain (conditioning, noise suppression, gate, compressor), pitch analysis, pitch engine, chords, effects + limiter, as percent of real time (smoothed). Measured with one clock read per stage per call.
* Test hook `Engine::simLoadRb` (0 in production) lets the host tests simulate a heavy RubberBand load.

### Upgrading from the VoiceFX v8 beta
1. Put `libHexedVoiceMod.so` in the mods folder and **delete `libvoicefx.so`**.
2. Replace `VoiceFX_Menu.lua` with `HexedVoiceMod_Menu.lua` (delete the old menu file, or both menus would register `/vfx`).
3. Start the game: old settings are imported automatically (old files are left in place).

## v8 beta
Last VoiceFX beta (library API 8, menu 8.0): Auto-Tune styles and Magic Chords, noise suppression, effect rack, RubberBand 4.0 LiveShifter with Lite fallback.
