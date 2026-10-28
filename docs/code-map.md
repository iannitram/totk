# totk code map

- **Repo:** iannitram/totk (private mirror of MacNativeGames). **Kind:** Python build tool plus C, Objective-C and ARM64 assembly runtime, a C# shader translator, an iOS (XcodeGen) project: turns the user's own game dump into a native Apple silicon Mac app or iPhone/iPad IPA for *Tears of the Kingdom* (also Breath of the Wild, and a Bayonetta 3 experiment).
- **Upstream:** ly05010419/MacNativeGames (`upstream/main`). **Integration branch:** `main`.
- **Describes commit:** `3befc7e` (2026-10-27). This mirror was last merged with upstream on 2026-10-22; upstream's `main` is hundreds of commits further on, so newer upstream code is not described here.

The game is not emulated. Switch and Apple silicon are both ARM64, so the game's own code runs mostly as is: an offline rewriter patches the few things macOS forbids (platform register, `svc`, counter registers), a C loader maps it, and the project supplies the Switch OS services (HLE), an NVN graphics layer on Metal, audio, input and saves. Shaders are translated with SPIRV-Cross and a Ryujinx-derived translator. No game files or keys are in the repo; the app is built from the user's files on their own Mac.

## Layout

| Path | What it owns |
|---|---|
| `build_app.py` | The one entry script: game file plus keys in, `.app` or `.ipa` out (steps: ExeFS code, RomFS data, shader translation, packaging); recognises the game by title ID and checks each step against expected-hash files |
| `games/totk/` | The main game port: `runtime/` (C and Objective-C: loader, OS services, HLE stack and assembly gates, audio renderer and mixer, input and gamepad, movie host, game patches), `app/` (AppKit launcher `launcher.m`), `ios/` (XcodeGen `project.yml`, sources, settings bundle), `scripts/` (extraction, mod tooling, version-site maps, probes, dev helpers), `third_party/` (`d2r` NSO rewriter and verifiers, `d2r_gfx` NVN-on-Metal graphics and texture tools), `tools/` (shader translator front end `totk_xlate`), `prebuilt/macos-arm64/` (the built translator binary), `assets/` (expected-hash files, shader lists), `tests/`, `Makefile` (dependencies, probes, verification targets) |
| `games/botw/` | Breath of the Wild support (1.6.0 and 1.9.0): `assets/` (shader list) and `scripts/`; built through `build_app.py` with the `games/totk` tooling |
| `games/by3/` | Bayonetta 3 work in progress, with its own copy of runtime, third-party code, prebuilt tools, scripts, app and a `CHANGELOG.md` |
| `vendor/ryujinx/` | Pinned Ryujinx sources (Graphics.Shader, Common, Spv.Generator) patched for the shader translator |
| `tests/` | About 40 files: Python `unittest` modules (containers and updates, relocation, work folders, packaging, shader extraction and lists, mods, state report, and more) plus C and Objective-C probes and fixtures |
| `docs/` | `how-it-works.md`, `shader-list.md` and `shader-variants.md` (the shared shader list), `first-launch-shaders.md`, per-version port notes (`v1.2.1-port.md`, `v1.4.3-port.md`, `botw-v1.9.0-port.md`), `mods.md`, `audio-troubleshooting.md`, `performance-followups.md`, `pipeline-prediction-stalls.md`, `serial-history.md` |
| `.github/` | `workflows/tests.yml` (CI) and issue forms for shader-list contributions |
| `README.md`, `CONTRIBUTING.md`, `THIRD_PARTY_NOTICES.md`, `LICENSE` | English and Chinese readme, contribution guide, third-party license table, license |
| `ROADMAP.md`, `.gitignore`, `AGENTS.fork.md`, `CHANGELOG.fork.md`, `PIPELINE.md` | Ours: roadmap notes (mainly what a Linux or Windows target would need to replace), ignore guards for game files, rules for this mirror, change history, stage record |

## Where our changes live

Upstream owns everything. Against the merge-base with `upstream/main` the only differences are five files: `.gitignore`, `ROADMAP.md`, `AGENTS.fork.md`, `CHANGELOG.fork.md`, `PIPELINE.md` (73 added lines). Nothing in `games/`, `vendor/` or `tests/` is ours.

## Build and test

```sh
xcode-select --install
brew install opus libvpx python@3.14
python3.14 build_app.py <game.xci|game.nsp> -k <prod.keys>           # Mac app
python3.14 build_app.py <game> -k <prod.keys> --platform ios         # needs Xcode 26+, xcodegen, cmake, ninja
python -m unittest discover -s tests                                 # what CI runs
make -C games/totk help                                              # probe and verify targets
```

- CI (`.github/workflows/tests.yml`, job `test`): on every push and pull request, `macos-latest`, Python 3.14, `unittest discover`, then two C parsers (`tests/test_romfs_paths.c`, `tests/test_controller_map.c`) built with UBSan.
- The shared shader list is a data file that players extend through issues; the build reads it.

## Mac / PC / Linux

- macOS 26 or later on Apple silicon only (PIPELINE.md: platforms mac, ios). `build_app.py` refuses other hosts: Metal 4 and MetalFX graphics, AppKit and GameController, ad-hoc `codesign`, arm64 assembly, a Mach-O translator binary.
- Windows and Linux are not supported. `ROADMAP.md` lists what a port would have to replace (renderer, shader path, window and input, packaging, toolchain checks, arm64-specific code).
- A PC session can read and edit files; building the app needs a Mac (R04).
