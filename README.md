# HexedVoiceMod

**A voice changer, effect rack, noise suppressor and Auto-Tune for the SA-MP Nezuko Android client, built as an AML mod with a MoonLoader (mimgui) menu.**

[![build](https://github.com/YOUR-NAME/YOUR-REPO/actions/workflows/build.yml/badge.svg)](https://github.com/YOUR-NAME/YOUR-REPO/actions/workflows/build.yml)
<sub>(replace `YOUR-NAME/YOUR-REPO` in the badge with your GitHub user and repository name)</sub>

HexedVoiceMod 1.0 replaces the VoiceFX v8 beta (it sounds different from it on purpose: new gate, limiter, reverb, vocoder and noise suppression). It sits between your microphone and the game's voice chat: you talk, the mod changes the sound, other players hear the result (and, if you like, you hear yourself).

> **Status:** everything was built and tested on a PC with synthetic voices (see [Running the host tests](#running-the-host-tests)). The pieces that need a real phone are listed in [`CHANGELOG.md`](CHANGELOG.md) under *Not verified*. If something misbehaves, send `HexedVoiceMod_log.txt`.

## Features
* **Voice presets** with favorites, plus manual **pitch** and **formant** (voice "size") controls, optional formant preservation.
* **Auto-Tune and chords**: snap your voice to a key and scale (with adjustable lag compensation and vibrato control), or add a harmony choir (optionally keeping the voice character).
* **Noise suppression**: *Classic* (learns the room noise) or *AI* (RNNoise, handles changing noise), with a click guard.
* **Dynamics**: input clean-up (DC + 75 Hz high-pass), gate with hold and hysteresis, compressor / leveler, de-esser and a lookahead limiter, so loud talking never clips.
* **Effect rack**: vocoder (whisper / cyber, optionally following your pitch), drive, bit crusher, tone filters, robot, tremolo, chorus / flanger / vibrato, echo (with damping) and a reverb with pre-delay.
* **Hear myself** monitor with a servo that keeps the delay steady instead of dropping audio.
* **CPU safety**: Auto pitch engine (high quality RubberBand, falls back to a light engine when the phone is overloaded and tries again later), CPU meters per stage.

## Requirements
* SA-MP **Nezuko** Android client (32-bit, `armeabi-v7a`).
* **AML** (AndroidModLoader / the Nezuko modloader) to load `.so` mods.
* **MoonLoader or MonetLoader** with **mimgui** for the menu.
* Microphone permission for the game (the mod hooks the game's BASS recording).

## Installation
1. Download the build (see [Building on GitHub](#building-on-github)) and put **`libHexedVoiceMod.so`** into the game's mods folder (where your other AML mods are).
2. Copy **`HexedVoiceMod_Menu.lua`** into the MoonLoader folder, together with the **`lib`** and **`HEXED`** folders from `menu/` (they hold the icon font, the font file and the icon images; the folders keep the same names as before, so nothing needs to be moved if you used VoiceFX).
3. Start the game and join a server. In chat run **`/hvm icons`** once so the menu builds its icons, then **`/hvm`** to open the menu.
4. If you used the old VoiceFX: **delete `libvoicefx.so`** and the old `VoiceFX_Menu.lua` first (see [Upgrading](#upgrading-from-the-voicefx-v8-beta)).

Your settings are saved automatically (voice settings in `/storage/emulated/0/HexedVoiceMod.cfg`, menu settings and favorites in `moonloader/config/`).

## Commands
Every command also works as `/hexedvoice`, `/vfx` and `/voicefx`.

| Command | What it does |
|---|---|
| `/hvm` or `/hvm menu` | open / close the menu |
| `/hvm on` / `off` | voice changer on / off |
| `/hvm pitch <-12..12>` | pitch in semitones |
| `/hvm formant <0.5..2>` | formant ("voice size") |
| `/hvm gain <dB>`, `/hvm mix <0..1>` | input gain, dry / wet mix |
| `/hvm gate <-80..-20>` | noise gate threshold in dB (-80 = off) |
| `/hvm preset <name>` | choose a voice preset |
| `/hvm preserve on\|off` | keep the voice character when shifting |
| `/hvm robot on\|off`, `/hvm robothz <Hz>` | robot voice |
| `/hvm tune on\|off`, `key <C..B\|auto>`, `scale <0..7>`, `look <0..60>`, `vibrato <0..1>` | Auto-Tune: on/off, key, scale, lag compensation (ms), vibrato kept (0 = flatten) |
| `/hvm chord <0..8>` | harmony choir style (0 = off) |
| `/hvm noise on\|off\|ai\|classic\|<0..1>` | noise suppression: on/off, engine, strength |
| `/hvm comp on\|off\|auto` | compressor on/off, auto makeup |
| `/hvm deess on\|off` | de-esser |
| `/hvm hear on\|off`, `/hvm hearvol <0..2>` | hear yourself (use headphones!) and its volume |
| `/hvm quality auto\|high\|lite` | pitch engine |
| `/hvm set <name> <value>` | set any parameter by name (the command lists the names) |
| `/hvm status` | one-line status: engine, load, latency, meters |
| `/hvm hook` | retry hooking the game's microphone |
| `/hvm save` | save settings now |
| `/hvm icons` | rebuild the menu icons (run once after installing) |
| `/hvm sdebug` | slider debug line (for bug reports) |

## The menu
* **Voices**: the preset list and the voice panel with the main controls (pitch, formant, gain, gate, mix), *Preserve*, *Robot*, and *Hear myself* with its volume.
* **Favorites**: your starred presets.
* **Effects**: tabs *Clean* (noise suppression, engine, strength), *Voice*, *Tune* (key, scale, speed, strength, lookahead, vibrato), *Chords* (style, mix, width, keep formants), *Tone*, *Modulate*, *Crunch* (drive, bit crusher), *Space* (Echo and Reverb pages), *Robot* (vocoder and carrier), *Dynamics* (gate, compressor, de-esser, limiter with live meters).
* **Status**: library state, pitch engine and latency, CPU load and per-stage CPU, hear-myself queue, mic levels, warnings (for example the old `libvoicefx.so` warning).
* **Settings**: menu options.

## Signal chain
```
 microphone (BASS record, 48 kHz mono)
      |
      v
 DC blocker + 75 Hz high-pass            always on
      |
      v
 noise suppression   (Classic or AI / RNNoise)
      |
      v
 noise gate (hysteresis, hold, 5 ms lookahead) -> input gain
      |
      v
 compressor / leveler
      |
      +--------------------------------------------+
      |                                            |
      v                                            v
 pitch engine (RubberBand or Lite)           Auto-Tune analysis (pitch tracker)
 + chord voices                                   |
      |   <-- optional "tune lookahead" delay ----+
      v
 mix with the dry voice
      |
      v
 de-esser
      |
      v
 effect rack: vocoder -> drive -> bit crusher -> tone filters -> robot -> tremolo
              -> chorus / flanger / vibrato -> echo -> reverb
      |
      v
 lookahead limiter (3 ms)
      |
      v
 BASS  --->  voice chat        and, if enabled, the "hear myself" monitor
```

## Performance tips
* **Pitch engine**: *Auto* (default) uses the high-quality RubberBand engine and drops to the light *Lite* engine if the phone is overloaded; it tries RubberBand again about every 20 seconds. *High* never switches, *Lite* is the cheapest. Lite sounds a bit rougher on big pitch changes.
* The Status page shows **CPU load** (average / peak) and the **CPU of each stage** (input chain, pitch analysis, pitch engine, chords, effects). If the *dropout / overrun* counters climb, turn off the heaviest stage first: chords, AI noise suppression, then reverb.
* **AI noise suppression** costs the most CPU of the optional features and adds about 20 ms of delay; it needs about a second to settle on steady hiss.
* Delay you may add: 8 ms for the always-on gate lookahead and limiter, 10.7 ms (Classic) or 20 ms (AI) for noise suppression, plus *Tune lookahead* if you set it. *Hear myself* adds its own queue of about 50 ms.

## Building on GitHub
1. Create a GitHub repository and push this folder (the workflow is `.github/workflows/build.yml`).
2. Open the **Actions** tab and run **build-hexedvoicemod** (it also runs on every push).
3. The workflow first runs the host tests, then downloads **RubberBand 4.0.0** and **RNNoise v0.1.1**, builds `libHexedVoiceMod.so` for `armeabi-v7a` with `ndk-build`, and uploads it as the **HexedVoiceMod** artifact. Open the finished run and download the artifact (the `.so` is inside).
4. RubberBand (GPL-2.0-or-later) and RNNoise (BSD-3-Clause) are **downloaded by the workflow and are not stored in this repository**. RNNoise is optional: if its download fails the build still succeeds and the "AI" noise engine falls back to Classic. (A build without RubberBand, `-DVFX_NO_RUBBERBAND`, has only the Lite pitch engine; the workflow itself always fetches it.)

### Building locally
You need the Android NDK (the workflow uses the one on the GitHub runner). From the project folder:
```
mkdir -p jni/rubberband-src && curl -fL https://codeload.github.com/breakfastquay/rubberband/tar.gz/refs/tags/v4.0.0 | tar xz --strip-components=1 -C jni/rubberband-src
mkdir -p jni/rnnoise-src    && curl -fL https://codeload.github.com/xiph/rnnoise/tar.gz/refs/tags/v0.1.1 | tar xz --strip-components=1 -C jni/rnnoise-src      # optional
$ANDROID_NDK_HOME/ndk-build NDK_PROJECT_PATH=. APP_BUILD_SCRIPT=jni/Android.mk NDK_APPLICATION_MK=jni/Application.mk -j2
```
The result is `libs/armeabi-v7a/libHexedVoiceMod.so`. The AML headers are already in `jni/mod`. If `jni/rubberband-src` is missing, `Android.mk` prints a message and builds the Lite pitch engine only (`-DVFX_NO_RUBBERBAND`); if `jni/rnnoise-src` is missing it builds without the AI noise engine.

## Config and log files
| File | Purpose |
|---|---|
| `/storage/emulated/0/HexedVoiceMod.cfg` | voice settings (never contains *Hear myself*) |
| `/storage/emulated/0/HexedVoiceMod_log.txt` | library log: hook state, engine switches, errors. **Send this when asking for help.** |
| `moonloader/config/HexedVoiceMod_settings.cfg` | menu settings |
| `moonloader/config/HexedVoiceMod_favorites.txt` | favorite presets |

## Upgrading from the VoiceFX v8 beta
1. Put `libHexedVoiceMod.so` in the mods folder and **delete `libvoicefx.so`**. Two copies would hook the microphone twice, so with both present the new library refuses to hook and the menu shows a red warning.
2. Replace `VoiceFX_Menu.lua` with `HexedVoiceMod_Menu.lua` (delete the old menu file, or both menus would register `/vfx`).
3. Start the game: your old settings (`voicefx.cfg`, `VoiceFX_settings.cfg`, `VoiceFX_favorites.txt`) are imported once; the old files are left in place.
4. The icon and font folders stay where they were. Commands `/vfx` and `/voicefx` still work next to `/hvm` and `/hexedvoice`. The new version does not reproduce the old sound; to go back, remove `libHexedVoiceMod.so` and put the old library back (never both at the same time).

## Troubleshooting
* **The menu does not open**: check that MoonLoader / MonetLoader and mimgui are installed, that `HexedVoiceMod_Menu.lua` is in the MoonLoader folder, and that no old `VoiceFX_Menu.lua` is also there. The menu needs `libHexedVoiceMod.so` of API 9 or newer ("Update libHexedVoiceMod.so" means the library is older).
* **Hook failed / nobody hears the changed voice**: run `/hvm hook`, check `/hvm status`, and send `HexedVoiceMod_log.txt` (it names the hook state). The hook installs when the game starts recording the microphone, so talk once in voice chat.
* **"Old libvoicefx.so found in the mods folder - delete it"**: the old library is loaded; delete it and restart the game.
* **No icons**: run `/hvm icons` once and reopen the menu; check that the `HEXED` folder was copied next to the menu.
* **Crackling or dropouts**: look at the Status page; switch *Pitch engine* to Lite, turn off chords or AI noise suppression.
* **Echo of your own voice**: *Hear myself* is only for headphones.

## Known limits
* 32-bit `armeabi-v7a` and 48 kHz mono 16-bit microphone only; AI noise suppression needs 48 kHz.
* Chord voices with preserved formants work from about 96 Hz to 1.1 kHz; deeper voices use the normal chord voice.
* RNNoise needs about a second to settle on steady noise; Classic noise suppression needs about a second of quiet to learn the room.
* Auto-Tune lookahead values that work best with RubberBand still have to be found by ear on the device.
* Not tested on a phone yet: see the *Not verified* list in the changelog.

## Running the host tests
The DSP core has a PC test suite (no Android needed, g++ with C++17):
```
# with RNNoise (fetched into jni/rnnoise-src as above) and the RubberBand stand-in
mkdir -p rnobj && for f in denoise rnn rnn_data pitch kiss_fft celt_lpc; do gcc -c -O2 -DNDEBUG -fno-fast-math -Ijni/rnnoise-src/include -Ijni/rnnoise-src/src jni/rnnoise-src/src/$f.c -o rnobj/$f.o; done
g++ -std=c++17 -O2 -ffast-math -fno-finite-math-only -I test/fake -Ijni/rnnoise-src/include test/host_test.cpp rnobj/*.o -o host_test && ./host_test
# without RNNoise
g++ -std=c++17 -O2 -ffast-math -fno-finite-math-only -DVFX_NO_RNNOISE -I test/fake test/host_test.cpp -o host_test && ./host_test
# without RubberBand (Lite only): add -DVFX_NO_RUBBERBAND
```
The tests cover the filters, dynamics, tuner, pitch engines (with a RubberBand stand-in that is a pure delay), effects, the monitor servo (simulated) and CPU recovery (simulated load). They do **not** test anything on a real device. The last runs are saved in `test/results/`.

## Credits and license
* HexedVoiceMod by **HEXED Studio** (author string `brruham`).
* Uses **RubberBand** (Breakfast Quay), **RNNoise** (Xiph.Org / Mozilla / J.-M. Valin), **AndroidModLoader** headers (RusJJ), **stb_sprintf** (Sean Barrett) and Font Awesome icons.
* Licensed under **GPL-2.0-or-later** (see [`LICENSE`](LICENSE)); third-party licences and a note about the bundled Arial font are in [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md).
