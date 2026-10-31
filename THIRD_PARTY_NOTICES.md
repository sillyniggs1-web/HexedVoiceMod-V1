# Third-party notices

HexedVoiceMod itself is GPL-2.0-or-later (see `LICENSE`). It uses the components below.

## RubberBand 4.0 - GPL-2.0-or-later
* Real-time pitch shifter (`RubberBandLiveShifter`, "High" pitch engine). Copyright Particular Programs Ltd (trading as Breakfast Quay).
* Source: https://github.com/breakfastquay/rubberband . **Not stored in this repository**: the GitHub workflow downloads it into `jni/rubberband-src` and the build links it. Because the built `libHexedVoiceMod.so` contains RubberBand, the library is distributed under the GPL (a commercial RubberBand licence exists from Breakfast Quay for closed-source use).
* A build without RubberBand (`-DVFX_NO_RUBBERBAND`) uses only the built-in Lite shifter and contains no RubberBand code.

## RNNoise v0.1.1 - BSD-3-Clause
* Recurrent-network noise suppression ("AI" noise engine). Source: https://github.com/xiph/rnnoise (tag v0.1.1, the trained model is embedded in `src/rnn_data.c`). **Not stored in this repository**: the workflow downloads it into `jni/rnnoise-src`; the build works without it (`-DVFX_NO_RNNOISE`).
* Licence text (from the RNNoise `COPYING` file):

```
Copyright (c) 2017, Mozilla
Copyright (c) 2007-2017, Jean-Marc Valin
Copyright (c) 2005-2017, Xiph.Org Foundation
Copyright (c) 2003-2004, Mark Borgerding

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:

- Redistributions of source code must retain the above copyright
notice, this list of conditions and the following disclaimer.

- Redistributions in binary form must reproduce the above copyright
notice, this list of conditions and the following disclaimer in the
documentation and/or other materials provided with the distribution.

- Neither the name of the Xiph.Org Foundation nor the names of its
contributors may be used to endorse or promote products derived from
this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
``AS IS'' AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
A PARTICULAR PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE FOUNDATION
OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```

## AndroidModLoader (AML) headers - see upstream
* The headers in `jni/mod` (`amlmod.h`, `iaml.h`, `logger.*`, `config.*` ...) come from AndroidModLoader by RusJJ: https://github.com/RusJJ/AndroidModLoader .
* The copy bundled here carries **no licence text of its own**. Check the licence in the upstream repository before you redistribute this project, and add that text here.

## stb_sprintf v1.10 - public domain / MIT (dual)
* `jni/mod/thirdparty/stb_sprintf.h` by Sean Barrett (RAD Game Tools). Used by the AML logger. Licence text: `jni/mod/thirdparty/STB_LICENSE` and the end of the header (choose MIT or public domain).

## Assets in `menu/` - please check before publishing
* `menu/lib/fAwesome6_solid.lua` and the PNG icons in `menu/HEXED/Icons/VoiceFX/` are generated from Font Awesome 6 Free (icons: CC BY 4.0, font files: SIL OFL 1.1, code: MIT; https://fontawesome.com/license/free). If you publish them, keep the attribution.
* **`menu/HEXED/Fonts/Arial_Bold.ttf` is Arial, a proprietary Monotype / Microsoft font, and is NOT covered by the GPL.** Redistributing it in a public repository may not be allowed. It is left in place because the menu loads it from there (as in the earlier releases); consider replacing it with an open font such as Liberation Sans Bold or Arimo (same metrics) and changing the file name in the menu.
