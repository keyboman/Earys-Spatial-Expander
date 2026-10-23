# Build for macOS (Apple Silicon)

This fork adds an official macOS (Apple Silicon / arm64) build for Eary's Spatial Expander.

The upstream project only ships a Windows x64 VST3. This repo provides:

- `CMakeLists.txt` — CMake build script (JUCE 8.0.13, VST3 only)
- Pre-built release asset: `SpatialExpander_macOS_arm64_v1.0.0.zip` (see Releases)

## Prerequisites

- macOS with Apple Silicon (M1/M2/M3/M4)
- Xcode Command Line Tools (`xcode-select --install`)
- CMake 3.22+ and Ninja (e.g. `brew install cmake ninja`)
- JUCE 8.0.13 SDK (https://github.com/juce-framework/JUCE/releases/tag/8.0.13)

## Build

```bash
# point JUCE_DIR at your JUCE 8.0.13 checkout
export JUCE_DIR=/path/to/JUCE

cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build
```

Output: `build/SpatialExpander_artefacts/Release/VST3/SpatialExpander.vst3`

## Install

```bash
# system-wide (all users, requires admin password)
sudo cp -R build/SpatialExpander_artefacts/Release/VST3/SpatialExpander.vst3 /Library/Audio/Plug-Ins/VST3/

# or current user only
mkdir -p ~/Library/Audio/Plug-Ins/VST3
cp -R build/SpatialExpander_artefacts/Release/VST3/SpatialExpander.vst3 ~/Library/Audio/Plug-Ins/VST3/
```

Rescan plugins in your DAW (Element / Reaper / Cubase...).

### Note for downloaded builds

If the DAW reports "damaged" / Gatekeeper blocks a downloaded VST3, clear the quarantine flag:

```bash
xattr -dr com.apple.quarantine /path/to/SpatialExpander.vst3
```

## Build notes

- Platform-sensitive scan is clean: no Windows-only headers, no x86 SIMD, DSP uses JUCE cross-platform `juce::dsp::FFT` (auto uses Apple vDSP on arm64).
- `JUCE_VST3_CAN_REPLACE_VST2=0` matches the upstream `.jucer` setting.
- Intel Macs: not supported by this build; compile with `-DCMAKE_OSX_ARCHITECTURES=x86_64` if needed.
