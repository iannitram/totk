# Roadmap

Purpose: a Ryujinx-based, Mac-native game project (target game: Tears of the Kingdom), mirrored privately for study and porting work.

Status: not started. This repository is a private mirror of its upstream; nothing here is published.

## Phases

1. Phase 1: placeholder
2. Phase 2: placeholder
3. Phase 3: placeholder

## Assets

Game assets and keys are never in this repository. They live in a private folder outside the repo (and outside every clone). `.gitignore` guards the usual game file types so none can be added by accident.

## Known issues / to do

- Mac-only today: needs Linux and Windows targets.
  - Host checks: `build_app.py` refuses anything but macOS 26 or later on Apple silicon (arm64), because the graphics use Metal 4.
  - Build toolchain: Xcode command line tools (`xcrun clang`, `clang++`, `codesign`, `xcode-select`, `mdfind`, `lsregister`, `getconf DARWIN_USER_CACHE_DIR`), Homebrew packages (opus, libvpx, python 3.14), `make` with `CC=clang` for hactool, and `-arch arm64` flags. iOS builds also need Xcode 26+, xcodegen, CMake and Ninja.
  - Graphics: Metal and MetalFX (upscaling, temporal anti-aliasing, frame interpolation), QuartzCore, CoreGraphics, CoreVideo, ImageIO, Vision. Shaders are translated with SPIRV-Cross to Metal source (`*.metal`), compiled ahead of time with Xcode's Metal toolchain into `.metallib` files, or warmed into macOS's Metal cache (`games/totk/tools/warm_metal_cache.m`).
  - App shell and input: AppKit/Cocoa launcher (`games/totk/app/launcher.m`), a macOS `.app` bundle with ad-hoc `codesign`, GameController and CoreHaptics (pads), IOKit, CoreText, UniformTypeIdentifiers, AudioToolbox (audio).
  - CPU side: the game code is rewritten for Apple silicon (arm64 assembly such as `runtime/gate.S`, Mach-style symbols), and the prebuilt shader translator `totk_xlate` is a macOS arm64 Mach-O binary.
  - A Linux or Windows target would need to replace: the renderer (Vulkan or D3D12 instead of Metal, with an upscaler such as FSR or DLSS instead of MetalFX), the shader path (SPIR-V consumed directly or via another cross compiler, no `.metallib`), the window, input, haptics and audio layer (SDL or native APIs instead of AppKit, GameController, CoreHaptics, AudioToolbox), the packaging steps (no `.app`, `codesign` or Metal cache warming), the host and toolchain checks, the translator build for each OS (`totk_xlate` from `vendor/ryujinx` via `publish_native.sh`), and the arm64-specific code rewrite and assembly (x86-64 hosts need their own).
