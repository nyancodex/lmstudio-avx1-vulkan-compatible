# LM Studio AVX1 Vulkan Compatible

A drop-in Vulkan backend for **LM Studio** that works on old **AVX1-only CPUs** (Ivy Bridge, Sandy Bridge, older AMD) where the official AVX2 backends refuse to load.

Confirmed loading **Gemma 4** and **Qwen 3.5** on Ivy Bridge under **LM Studio 0.4.14**.

## Quick install

1. Download the zip from [`Releases/GPU Vulkan AVX1/`](Releases/GPU%20Vulkan%20AVX1/).
2. Extract it into `C:\Users\%USERNAME%\.lmstudio\extensions\backends\`.
3. Fully quit LM Studio (system tray → Quit) and reopen it.
4. In the runtime/engine selector, pick the new backend, then load your model.

**If the model fails to load:** back up your `backends\` folder somewhere safe, then **delete everything inside it** and extract our zip again so it's the only backend present. LM Studio sometimes picks the wrong runtime when multiple Vulkan backends are installed.

See [`Releases/GPU Vulkan AVX1/readme.md`](Releases/GPU%20Vulkan%20AVX1/readme.md) for full install steps.

## Build your own

See [`Generate Backends/`](Generate%20Backends/) for build scripts. The Windows Vulkan script is already configured for AVX1 — just install Visual Studio 2022 Build Tools (with the C++ workload), CMake, Ninja, and the Vulkan SDK, then run the scripts from an *x64 Native Tools Command Prompt*.

## Credits

This is a fork of **[theIvanR/lmstudio-unlocked-backend](https://github.com/theIvanR/lmstudio-unlocked-backend)** by **theIvanR** — all build scripts, the original backend patches, and the foundational work are theirs. This fork adds an updated Vulkan+AVX1 backend built against current `llama.cpp` main, so newer models like Gemma 4 and Qwen 3.5 load correctly.

Underlying engine: **[ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp)** (MIT). The original AVX2-gated binaries belong to **[LM Studio](https://lmstudio.ai/)**.

## License & disclaimer

MIT, matching the upstream `llama.cpp` and the original repo. Unofficial / community-made; use at your own risk and back up your existing LM Studio backends folder before installing.
