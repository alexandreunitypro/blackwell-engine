# Blackwell Engine

The Wine engine of **MetalHorizon**, the macOS launcher for the Xbox / Microsoft Store edition of Forza Horizon 6 on
Apple silicon. MetalHorizon downloads the files of this repository's releases and installs them by itself: players never
need to come here.

**Nothing from CrossOver.app is used.** No game file is modified, and the game's protection, licence check and
anti-cheat are not bypassed: every fix lives in the compatibility layer.

## What a release contains

| File | What it is | Licence |
|---|---|---|
| `blackwell-engine-<version>.tar.xz` | Wine 11 built from CrossOver 26.3's published sources and the patch series in [`engine/`](engine/), with its own x86_64 GnuTLS, GMP, FreeType and MoltenVK, and Highball's DirectX 12 shim | LGPL-2.1-or-later (Wine, GnuTLS, GMP, Nettle), FreeType licence, Apache-2.0 (MoltenVK), GPL-3.0 (shim) |
| `d3dmetal-<version>-redist.tar.xz` | The redistributable part of Apple's Game Porting Toolkit (`redist/lib`: D3DMetal.framework and the Wine d3d12/dxgi halves), **unmodified**, with Apple's `License.rtf` | Apple's licence (see below) |
| `crossover-sources-26.3.0.tar.gz`, `gmp-…`, `nettle-…`, `gnutls-…`, `freetype-…` | The exact upstream sources the engine is built from, for the LGPL and GPL | as upstream |

**D3DMetal** is © Apple Inc. Apple's Game Porting Toolkit licence (sections 2A(iii) and 2C) allows the Framework and the
Redistributables to be distributed separately, for non-commercial purposes, with their notices. MetalHorizon is free; it
shows Apple's licence and installs D3DMetal only if the user accepts it. It then copies Apple's `d3d12.dll` and changes
the copy's export name on the user's Mac (as Highball does) so the shim can load it; no modified Apple file is
distributed here.

## How it was made to run without CrossOver

- In Apple's toolkit, `wine/x86_64-unix/*.so` (`d3d12.so`, `dxgi.so`, `d3d12core.so`…) are **links** to one library,
  `external/libd3dshared.dylib`. Copies make dyld load separate images and only one of them receives Wine's dispatch
  table, which looked like "the toolkit's d3d12 does not work with this Wine" (a null call in `WineMonitor::Init`).
  Kept as links, Apple's own halves work.
- GnuTLS (with Nettle built in, needing only GMP), GMP, FreeType and MoltenVK are built here for x86_64
  ([`engine/build-deps.sh`](engine/build-deps.sh)) and shipped in the engine's `lib/` with `@loader_path` names.

## Build it

On an Apple silicon Mac with Rosetta, Xcode Command Line Tools, Python 3, `brew install bison pkgconf`:

```sh
bash engine/build.sh                 # sources pinned in engine/inputs.json, patches in engine/series
bash engine/package.sh 1.0           # adds the shim (tools/tsshim), refuses any Apple or CrossOver file
bash engine/package-d3dmetal.sh "Evaluation environment for Windows games 4.0 beta 3.dmg"
```

[`engine/README.md`](engine/README.md) (French) has the details: what each patch carries, how the series is checked against
the tree it came from, and what is left to do.

## Credits

[Highball](https://github.com/gauthierpiarrette/highball) (Gauthier Piarrette, GPL-3.0): the DirectX 12 shim and Wine
patches 0017–0025. [Ferestre](https://github.com/icex/ferestre) (icex): Wine patches for the Xbox edition.
[xodus](https://github.com/xodus-gaming/xodus): the Xbox Gaming Runtime (`xgameruntime`, LGPL-2.1-or-later). Wine, and
CodeWeavers for publishing the CrossOver sources. Apple for D3DMetal.

Unofficial. Not affiliated with or endorsed by Microsoft, Xbox, Playground Games, Apple or CodeWeavers. "CrossOver" is a
trademark of CodeWeavers; this is not CrossOver.

The patches to Wine are LGPL-2.1-or-later ([LICENSE-LGPL-2.1](LICENSE-LGPL-2.1)); the shim in `tools/tsshim` and
`tools/pe_tools.py` are GPL-3.0 ([LICENSE-GPL-3.0](LICENSE-GPL-3.0)).
