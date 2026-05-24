# GPU Vulkan AVX1 — LM Studio 2.16.0-era backends

Two drop-in **Vulkan + AVX1** backends for LM Studio, both **confirmed working on Ivy Bridge** with newer models (Gemma 4, Qwen 3.5, etc.) that the older 1.103.2 / 1.59.0 backends in this repo cannot load.

## Which one should I use?

| File | What it is | When to pick it |
|---|---|---|
| `llama.cpp-win-x86_64-vulkan-avx1-2.16.0-patched.zip` | LM Studio's stock 2.16.0 Vulkan backend (built on `llama.cpp` release **b9265**), with `backend-manifest.json` patched so the `instruction_set_extensions` field advertises `"AVX"` instead of `"AVX2"`. Binaries are unmodified. | **Default choice.** Closest to a stock LM Studio install, gets all of LM Studio's 2.16.0 features (MTP speculative decoding, Gemma 4 fixes, etc.), no custom binaries. |
| `llama.cpp-win-x86_64-vulkan-avx1-main-549b9d8.zip` | Same shell, but the three ggml DLLs (`ggml-base.dll`, `ggml-cpu.dll`, `ggml-vulkan.dll`) are rebuilt from `llama.cpp` **main @ 549b9d843** with explicit AVX1 flags (`-DGGML_AVX=ON -DGGML_AVX2=OFF -DGGML_FMA=OFF -DGGML_F16C=ON`). | If you want the newest llama.cpp commits (40+ ahead of b9265), or if the 2.16.0-patched build ever hits an `unknown architecture` for a brand-new model. |

Both backends were tested side-by-side on an **Ivy Bridge CPU + Vulkan-capable GPU**, running LM Studio 3.9.x, loading Gemma 4 and Qwen 3.5 GGUFs. Both load and generate text successfully.

## Why this works

LM Studio 2.16.0 ships a Vulkan backend that itself is built without AVX2-only critical paths — but its `backend-manifest.json` declares `instruction_set_extensions: ["AVX2"]`, which makes LM Studio refuse to expose it on AVX1-only CPUs. Patching that single field to `"AVX"` unlocks the backend without changing any code. (See discussion in the parent repo's root README.)

## Install

1. **Back up first.** Make a copy of `C:\Users\%USERNAME%\.lmstudio\extensions\backends\` somewhere safe.
2. Extract the zip into `C:\Users\%USERNAME%\.lmstudio\extensions\backends\` so you end up with one of:
   - `...\backends\llama.cpp-win-x86_64-vulkan-avx1-2.16.0-patched\`
   - `...\backends\llama.cpp-win-x86_64-vulkan-avx1-main-549b9d8\`
3. **Fully restart LM Studio** (close from the system tray, not just the window).
4. Open the runtime/engine selector; you should see *"Vulkan AVX1 (2.16.0 patched)"* or *"Vulkan AVX1 (custom main 549b9d8)"* in the list.
5. Pick it, then load your model.

## How the `-main-549b9d8` build was produced

- Source: `https://github.com/ggml-org/llama.cpp.git` at commit `549b9d843` (May 2026, ~40 commits ahead of tag `b9305`).
- Toolchain: Visual Studio 2022 Build Tools (MSVC 19.44, x64), CMake 4.3, Ninja 1.13, Vulkan SDK 1.4.350.0.
- Build invocation (from `x64 Native Tools Command Prompt for VS 2022`):
  ```cmd
  cmake -S . -B build_gpu_vulkan -G Ninja ^
        -DCMAKE_BUILD_TYPE=Release ^
        -DGGML_VULKAN=ON ^
        -DLLAMA_CURL=OFF ^
        -DGGML_NATIVE=OFF ^
        -DGGML_AVX=ON ^
        -DGGML_AVX2=OFF ^
        -DGGML_FMA=OFF ^
        -DGGML_F16C=ON ^
        -DGGML_AVX512=OFF
  cmake --build build_gpu_vulkan
  ```
- Only `ggml-base.dll`, `ggml-cpu.dll`, and `ggml-vulkan.dll` from the resulting `bin/` were dropped into a copy of LM Studio's stock 2.16.0 backend folder. `llama.dll`, `ggml_llamacpp.dll`, and the `.node` files were **not** replaced — those are LM Studio's own wrappers.

## Manifest patch (for reference)

The only change to `backend-manifest.json`:

```diff
   "cpu": {
     "architecture": "x86_64",
     "instruction_set_extensions": [
-      "AVX2"
+      "AVX"
     ]
   },
```

…plus a unique `"name"` field so LM Studio treats it as a separate entry from any official backend you have installed.

## Limitations / caveats

- **Flash Attention on Vulkan is still flaky** — same upstream `llama.cpp` issue noted in the parent repo's root README.
- **AVX1 + F16C only.** If your CPU lacks F16C (pre-Ivy Bridge / pre-Piledriver), this build will not run. Drop `-DGGML_F16C=ON` and rebuild.
- **Not an official LM Studio release.** Use at your own risk. Keep the backup from step 1.
