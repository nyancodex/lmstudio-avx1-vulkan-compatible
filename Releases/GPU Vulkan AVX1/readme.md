# GPU Vulkan AVX1 — install guide

A drop-in Vulkan backend for LM Studio that works on AVX1-only CPUs. Extract, restart LM Studio, done.

Confirmed working on **LM Studio 0.4.14** (Ivy Bridge + Vulkan-capable GPU).

## What's in the zip

**`llama.cpp-win-x86_64-vulkan-avx1-main-549b9d8.zip`** *(22 MB)* — LM Studio's 2.16.0 Vulkan backend shell with the three core `ggml` DLLs (`ggml-base.dll`, `ggml-cpu.dll`, `ggml-vulkan.dll`) rebuilt from `llama.cpp main` and the manifest patched to advertise AVX1 instead of AVX2.

> An earlier "manifest-only patch" variant has been removed — it failed to load on at least one tester's machine, so this rebuilt version is the only one we ship now.

## Install

1. **Back up your backends folder.** Copy the whole thing somewhere safe:
   ```
   C:\Users\%USERNAME%\.lmstudio\extensions\backends\
   ```

2. **Extract the zip** into that folder. You should end up with a new subfolder:
   ```
   C:\Users\%USERNAME%\.lmstudio\extensions\backends\llama.cpp-win-x86_64-vulkan-avx1-main-549b9d8\
   ```

3. **Fully quit LM Studio** — right-click the system tray icon → *Quit*. Closing the window is not enough.

4. **Reopen LM Studio.** Pick *"Vulkan AVX1 (custom main 549b9d8)"* in the runtime/engine selector.

5. **Load your model.**

## If the model fails to load

LM Studio sometimes picks the wrong runtime when multiple Vulkan backends are installed alongside ours. To force it to use ours:

1. **Back up the whole `backends\` folder** somewhere safe (you did this in step 1 above — good).
2. **Delete everything inside** `C:\Users\%USERNAME%\.lmstudio\extensions\backends\` so it's empty.
3. **Extract our zip again** so it's the only backend present.
4. **Restart LM Studio.** It will now have no choice but to use our backend.

If it still fails after that, the model probably needs newer `llama.cpp` than what we shipped — build your own from [`Generate Backends/`](../../Generate%20Backends/) against the latest source.

## Other troubleshooting

- **Backend doesn't appear in the list** → wrong folder. The path is hidden by default on Windows (`.lmstudio` starts with a dot).
- **LM Studio crashes immediately on load** → the rebuild may be incompatible with your specific LM Studio version. Try restoring your backup and pinning LM Studio to 0.4.14.

## How this was built

LM Studio 2.16.0's stock Vulkan backend folder, with `ggml-base.dll`, `ggml-cpu.dll`, and `ggml-vulkan.dll` replaced by fresh builds from [`ggml-org/llama.cpp`](https://github.com/ggml-org/llama.cpp) commit `549b9d843`:

```
cmake -S . -B build_gpu_vulkan -G Ninja ^
      -DCMAKE_BUILD_TYPE=Release ^
      -DGGML_VULKAN=ON ^
      -DLLAMA_CURL=OFF ^
      -DGGML_NATIVE=OFF ^
      -DGGML_AVX=ON -DGGML_AVX2=OFF -DGGML_FMA=OFF ^
      -DGGML_F16C=ON -DGGML_AVX512=OFF
```

The manifest was patched: `"instruction_set_extensions": ["AVX2"]` → `"AVX"` so LM Studio stops gating it behind AVX2 detection.
