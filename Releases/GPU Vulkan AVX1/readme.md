# GPU Vulkan AVX1 — install guide

Two drop-in Vulkan backends for LM Studio that work on AVX1-only CPUs. Pick one, extract, restart LM Studio.

## Which zip?

| Zip | Pick this if… |
|---|---|
| **`llama.cpp-win-x86_64-vulkan-avx1-2.16.0-patched.zip`** *(18 MB, recommended)* | You just want LM Studio to work on your old CPU. This is LM Studio's official 2.16.0 Vulkan backend with one tiny manifest tweak so it stops blocking AVX1 hardware. Everything else is unmodified. |
| **`llama.cpp-win-x86_64-vulkan-avx1-main-549b9d8.zip`** *(22 MB)* | The 2.16.0 build is too old for some brand-new model you're trying to load. This one uses freshly built `ggml` DLLs from the latest `llama.cpp main` commit. |

If unsure, start with the first one.

## Install

1. **Back up your existing backends folder** (just copy it somewhere safe):
   ```
   C:\Users\%USERNAME%\.lmstudio\extensions\backends\
   ```

2. **Extract the zip** into that same folder. You should end up with a new subfolder like:
   ```
   C:\Users\%USERNAME%\.lmstudio\extensions\backends\llama.cpp-win-x86_64-vulkan-avx1-2.16.0-patched\
   ```

3. **Fully quit LM Studio** (right-click the system tray icon → *Quit*, not just close the window).

4. **Reopen LM Studio.** Go to runtime/engine settings, pick the new backend (it will be labeled *"Vulkan AVX1 (2.16.0 patched)"* or *"Vulkan AVX1 (custom main 549b9d8)"*).

5. **Load your model.** It should work.

## If something goes wrong

- **Backend doesn't appear in the list** → you extracted into the wrong folder. Confirm the path above (`.lmstudio` is hidden by default on Windows).
- **LM Studio crashes when loading a model** → switch to the other zip and try again.
- **Model still won't load** with a message about "unknown architecture" → the model needs llama.cpp newer than what's in either zip. Build your own from `Generate Backends/` against the latest `llama.cpp` source.

## How these were built

- **`-2.16.0-patched`**: copy of LM Studio 2.16.0's stock Vulkan backend, with `backend-manifest.json` patched: `"instruction_set_extensions": ["AVX2"]` → `"AVX"`. No binaries changed.
- **`-main-549b9d8`**: same shell, but `ggml-base.dll`, `ggml-cpu.dll`, and `ggml-vulkan.dll` rebuilt from `ggml-org/llama.cpp` commit `549b9d843` with:
  ```
  cmake -S . -B build_gpu_vulkan -G Ninja ^
        -DCMAKE_BUILD_TYPE=Release ^
        -DGGML_VULKAN=ON ^
        -DLLAMA_CURL=OFF ^
        -DGGML_NATIVE=OFF ^
        -DGGML_AVX=ON -DGGML_AVX2=OFF -DGGML_FMA=OFF ^
        -DGGML_F16C=ON -DGGML_AVX512=OFF
  ```
