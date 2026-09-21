# Sonic Generations Android — Mali-G57 Bring-Up Report

Device: MediaTek Helio G99 (2x Cortex-A76 + 6x Cortex-A55), Mali-G57 MC2,
4 GB RAM. App built from this repo via GitHub Actions
(`assembleRelease`, ReXGlue SDK v0.10.0 + `android/patches/*`).
Game: Xbox 360 Sonic Generations (USA/Europe dump), title screen + Green Hill tested.

Status: **boots to title on Mali, renders correctly with the settings below,
~2.5–4 fps** (metronome-like frames, see §4).

Settings used (`settings.txt`, preset `performance`, guest 960x540):
`resolution_scale=1`, `bilinear`, `vsync=true`, `fps_cap=30`,
`readback_memexport=false`, `vulkan_readback_*=false`,
`texture_cache 192/384/16`, `async_shader_compilation=true`,
`vulkan_async_skip_incomplete_frames=true`, FIFO-only present,
`log_level=warning`, `vulkan_dynamic_rendering=true`,
`anisotropic_override=0`, `occlusion_query_enable=false`,
`native_2x_msaa=false`, `gamma_render_target_as_unorm16=false`,
`ignore_thread_affinities=false` (see §3), `REX_HEAL_DISCOVER=1`.

## 1. Boot gates removed (patches in this repo)

Mali-G57 lacks three Vulkan features the SDK treats as hard requirements.
Each aborted boot with `Unable to create graphics provider`:

1. `vertexPipelineStoresAndAtomics` (device selection,
   `src/ui/vulkan/vulkan_device.cpp`) **and** the duplicate check in the Xenos
   plugin (`src/graphics/vulkan/command_processor.cpp:932`,
   `... required for GPU emulation and D3D12 parity ...`).
2. `fillModeNonSolid` (same file). Fallback to solid fill exists in code.
3. (No gate for `fragmentStoresAndAtomics` on G57 — supported; relaxed anyway.)

Fix (`android/patches/rexglue-sdk-v0.10.0-mali-vertex-stores.patch`):
warn-and-continue instead of `return nullptr/false`. No-op on GPUs with the
features (they stay enabled). Verified by `git apply --check` against a local
pristine v0.10.0 clone, then against the full 9-patch CI sequence.

Known parity cost: Classic Sonic's hand renders glitched — character skinning
uses the vertex memexport path that no longer exists. Proper fix = emulate
vertex memexport via compute when the feature is absent.

## 2. Instrumentation added

`android/patches/rexglue-sdk-v0.10.0-fps-log.patch`: `IssueSwap` logs every 5 s
`present fps: X (N frames in M ms, swap body avg Y ms)` at `[gpu]` warning
level, so it shows with `log_level=warning`. RAII timer covers all returns.

## 3. Correctness findings (races, not perf)

- `ignore_thread_affinities=true` (the `android_main.cpp` default) →
  graphical glitches (missing logo, corrupt textures). `=false` → clean.
  Scheduling-dependent ordering issue in GPU emulation on big.LITTLE.
- Same glitch family with larger texture cache (`256/512`: black square,
  orange regions). Intermittent boot glitches (sometimes clean, sometimes
  not). All timing-dependent; default `192/384` + `affinities=false` is clean.
- Tolerant dispatcher (`REX_HEAL_DISCOVER=1`) logs 2 unknown guest functions
  per boot (`0x8310BEE0`, `0x8310CF08`), returns safely. Benign.

## 4. Performance measurements (all on title screen unless noted)

| run | fps | swap body avg |
|---|---|---|
| default | ~3.3–3.8 | ~80–130 ms (cold) → ~5 ms (warm) |
| `video_mode 640x360` | same (±0.2) | same |
| `vsync=false` / mailbox | same | same |
| cache 256/512 vs 192/384 | same | same |
| cutscene / Green Hill | ~2.5–2.9 | ~5–45 ms |

- `video_mode_*` only changes the output mode; internal scale floor is 1x
  (`draw_resolution_scale_*` is uint32, `> 1` = scaled). There is **no knob
  to render below native** — biggest missing lever for tile GPUs.
- Steady state: submit ~5 ms, present rate ~2.7 fps → **~370 ms/frame spent
  outside `IssueSwap` (guest thread)**. Pacing ruled out (vsync/mailbox no-op).
- AGI System Profiler: Mali fragment queue saturated solid; all 8 CPUs
  moderately busy; `kswapd0` active (3.55/4 GB used — memory pressure).
  Zoomed thread view: `Main XThread` + 3 `XThread*` solid Running (genuine
  continuous compute, not sleep-polling); 68 threads idle; vsync worker
  sleeps 1 ms/iter as coded. Touch UI + audio stay alive (decoupled threads).
- fps decays over a session (3.5–4 → 2.5–2.7): thermal throttling on top.

## 5. Ranked suspects for the ~370 ms

1. **Guest emulation throughput** (~365 ms, proven) — PPC→C++ + dispatcher
   on 2xA76. Needs profiler/recompiler-level work (simpleperf is blocked by
   this ROM: `perf_event_open` denied even for debuggable apps, even
   `task-clock`).
2. **DXT→uncompressed 4x fallback** (`missing format feature bits: 1001h`
   for all BC formats) — 4x texture memory + upload bandwidth on a
   bandwidth-starved tile GPU. ETC2 transcode (core Vulkan, Mali-native)
   instead of RGBA8 would cut it sharply.
3. Fragment saturation at native res with no sub-1x scale knob.
4. Memory pressure (4 GB device, 3.5 GB used, kswapd) + thermal decay.

## 6. Proposed upstream work

1. Sub-native internal render scale (e.g. 0.5x) for mobile GPUs.
2. BC→ETC2 transcode path for GPUs without BC filterable features.
3. Vertex-memexport fallback (compute) when `vertexPipelineStoresAndAtomics`
   is absent (fixes skinning glitches like ours).
4. Recompiler/dispatcher throughput (the 370 ms); PGO data welcome — we can
   measure any build on this device within a day.
5. Consider `ignore_thread_affinities=false` default on big.LITTLE (correctness).

Logs, AGI traces and `settings.txt` available on request. Happy to test any
build on this Mali-G57 within ~24 h.
