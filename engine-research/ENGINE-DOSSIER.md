# Engine Dossier — Death Stranding Director's Cut (Decima)

> One consolidated, living reference for this game's engine, filled in as the
> `PLAYBOOK.md` phases are worked. Chronological blow-by-blow belongs in the
> `dev-archive/` and `modding-notes/` folders; this file is the *distilled current
> truth*. Update it whenever a fact changes; correct false leads in place.

**Status:** M0, first static look (2026-09-13); the game has not been launched yet. · **VR-readiness verdict:** TBD. Nothing seen so far rules it out.

## 1. Identity
- Game / build / version: Death Stranding Director's Cut, Steam build, exe `ds.exe` (linked 2024-01-28).
- Platform & store; unofficial port? (extra fragility/legal notes): Steam (PC). Official release, not a fan port.
- Legitimacy: owned copy confirmed.

## 2. Engine lineage
- Family / base engine and how it was modified: Decima, Guerrilla Games' engine `[reported]`. Havok and Scaleform strings are present in the exe `[inferred-static 2026-09-13]`.
- Middleware (animation, audio, physics, megatexture, CUDA, etc.): Havok (physics), **Wwise** (audio; `AK::SoundEngine`, including `MuteBackgroundMusic`), Scaleform-era UI names, Oodle, Bink 2; upscalers **DLSS** (`nvngx_dlss.dll`), **XeSS** (`libxess.dll`, `igxess.dll`, `XeFX*.dll`) and **FSR 2** (`AMD_FSR_2_0`, built in) `[inferred-static 2026-10-01]`.
- Distinctive file formats / build tags / symbol naming: Oodle compression (`oo2core_7_win64.dll`), Bink 2 video, and the DLSS and XeSS upscalers ship beside the exe. Data archives under `data\`, not yet looked at.

## 3. Binary & memory
- 32/64-bit, size, module base, ASLR behaviour (stable base? relocations?): **64-bit** (PE32+), `ds.exe` 86.5 MB, linked 2024-01-28. Sections look ordinary (`.text` 62 MB), with no sign of a large protection blob `[inferred-static 2026-09-13]`.
- Renderer API (D3D11/12, DXGI, GL, Vulkan) with evidence: Direct3D 12: `d3d12.dll` and `dxgi.dll` appear in the exe's strings, and `dxcompiler.dll`/`dxil.dll` ship beside it `[inferred-static 2026-09-13]`.
- **(2026-10-01) Graphics libraries are loaded at run time:** the import table has no `d3d12.dll` or `dxgi.dll` (both appear only as strings), so a proxy named either, beside the exe, is the likely foothold `[inferred-static 2026-10-01]`. `D3DReflect` IS imported from `d3dcompiler_47.dll`: the game reads shader reflection itself, at run time. `.text` entropy 6.36, entry point in `.text`: the code is NOT encrypted on disk, so static disassembly works `[measured 2026-10-01]`. ASLR on.
- Developer console / cvar system present? how opened?: no console string beyond the Win32 console API (`AllocConsole`, `EnableConsoleLogging`) `[inferred-static 2026-10-01]`. Command-line switches found as plain strings: `-enable_dred` (D3D12 crash breadcrumbs), `-safe`, `-job_thread_affinity`, `-job_thread_adjust_smt`, `-disable_initial_highlight`, `-unlock_all_perks`, `-unlock_hack_perks` `[inferred-static 2026-10-01]`; what each does is untested.
- **A built-in free camera: PHOTO MODE** (`DSPhotoMode`, `DSPhotoModeCameraCollisionComponent`, menu data sources) `[inferred-static 2026-10-01]` — the cheapest route to a free camera for camera RE.
- **⭐ A stereoscopic 3D SETTING exists**: `SetStereoscopic` / `GetStereoscopic`, `Set3DScreenFactor`, `SetStereoscopicDepthMultiplier`, **`SetStereoscopicFPDepthMultiplier`** (a separate first-person depth), listed among the user-settings functions beside gamma, volumes and photo mode; also a `StereoDepth` property on camera entities `[inferred-static 2026-10-01]`. Probably the PlayStation 3D-TV option. **❌ STUBBED ON PC** `[inferred-static 2026-10-01]`: in the script-binding table (entries {name, signature, function, flags} at `0x144e3a2e8`…`0x144e3a3a8`) every stereo setter's wrapper ends in a jump to `0x1418e9b10`, which is a bare `ret`, and `GetStereoscopic` calls `0x141920dc0`, which is `xor al,al; ret` (always off). So no two-eye path is reachable through this setting; see §11.

## 4. DRM / anti-debug & injection foothold
- DRM (CEG/Denuvo/GOG/none); launch-time-debugger behaviour: No Denuvo string and no protection-shaped section found; Steam API present `[inferred-static 2026-09-13]`; `steam_api64.dll` is not a static import (loaded at run time); `.text` is plain code (entropy 6.36) `[measured 2026-10-01]`. Not tested live.
- **Settings and saves:** gameplay settings in `%LOCALAPPDATA%\KojimaProductions\DeathStrandingDC\<steam id>\profile\game_settings.cfg` (plain text; audio has only `MasterVolumeSensitiviness`, no music-only volume), `profile.dat` is scrambled. **Display settings are a plain-text `settings.cfg` beside `ds.exe`** `[verified-live 2026-10-16, n=1]`: `window_mode 1`, `rendering_width/height`, `window_left/top/right/bottom`; set to 1280x720 at (320,180) with the game closed, the next start came up windowed 1280x720 with the desktop untouched. The in-menu Confirm wrote this file, but the first menu change did not survive a forced close. **VR screen-effect switches in the same file:** `postprocess_dof`, `postprocess_motionblur` (menu: Depth of Field, Motion Blur) `[verified-live 2026-10-16, read from the file]`; film grain (visible on the title screen) and any chromatic aberration have no menu switch yet. Flat play keeps them on.
- Attach workflow that works: not yet tested.
- Injection vector that works: **a `d3dcompiler_47.dll` proxy beside `ds.exe` loads** (29/29 exports forwarded to the renamed original) `[verified-live 2026-10-16, n=2 launches]`. Its `D3DReflect` wrapper saw only ~10 small start-up compute shaders (`Uniforms`, `RasterizerVariables`, ...), none on the second start: the game ships compiled shaders, so **reflection logging cannot find the camera constants** `[measured 2026-10-16, n=2]`. Taken out again; kept in `dev-archive/archive/2026-10-16-d3dcompiler-reflect-logger/`.

## 5. Threading & frame structure
- Immediate context only, or deferred contexts + command lists?:
- Which thread(s) do what; render-thread name(s):
- One-frame walkthrough (record → replay → present):

## 6. Camera & projection delivery (the crucial section)
- How the world transform reaches the GPU (shared VP buffer / per-draw MVP /
  other), with **shader-reflection / disassembly evidence**: **a shared per-view constant structure,
  `ViewConstants`**, one of Decima's described shader resource tables (`SRT_RTTI_ViewConstants`; siblings
  `GlobalConstants`, `OffscreenParams`, `LightConstants`, `MaterialConstants`, …). Its members are registered at
  start-up by `0x141823d60`, one call per member (name, byte offset, type code), so the layout is readable from the
  exe with no game running `[inferred-static 2026-10-01]`. Identified as `ViewConstants` by two orderings that
  agree: the code registers `OffscreenParams` (`0x141823cf0`) then this table, and the type-name strings sit in the
  same order in `.rdata`.
- Exact constant-buffer slot, parameter name(s), byte offset(s), layout,
  handedness, row/column convention: **`ViewConstants` layout** `[inferred-static 2026-10-01]` (type codes read
  from the offset steps: `0x1b` float4x4, `0x1a` float3x4, `0x08` float4, `0x13` float3, `0x05` float, `0x15` uint):
  `View` +0x000 · `Proj` +0x040 · `ViewProj` +0x080 · `InvView` +0x0C0 · `OldViewProj` +0x100 ·
  `DepthReconstructMatrix` +0x140 · `HalfRes…` +0x180 · `QuarterRes…` +0x1C0 · **`ViewToObserverView` +0x200
  (3×4)** · `Viewport` +0x230 · `WPOSScaleOffset` +0x240 · `MVScaleBias` +0x250 · `PixelSize` +0x260 · **`ViewPos`
  +0x270** · `HPOSReconstructScaleOffset` +0x280 · **`FloatingOrigin` +0x290** (camera-relative world) ·
  **`DepthDirection` +0x29C** (reverse-Z sign) · force-field probes +0x2A0…0x320 · `LodDistanceMul` +0x330 · … up to
  `UseKJPVolumetricFog` +0x33C. Full table: `dev-archive/recon/2026-10-01-view-constants-layout/view-constants.txt`.
  Slot number, row/column order and handedness are NOT known yet (need the filling code or a capture).
  ⚠️ For a per-eye edit, `View`, `ViewProj`, `InvView`, the depth-reconstruct matrices and `ViewPos` must move
  together; `ViewToObserverView` (a transform from the rendered view to a separate "observer" view) is a candidate
  head-pose slot, like Prototype's View+0x2c `[hypothesis]`.
- **A public mod maps the camera on our exact build** (from `/gr` 2026-10-04; topic
  `external-research/topics/2026-10-04-minecraft-in-ds-maps-the-camera-on-our-exact-build.md`). mihailinl's
  *minecraft-in-death-stranding* (MIT) targets `ds.exe` linked 2024-01-28, the same link date as §1 `[reported]`.
  Camera path: `GetLocalDSPlayerEntity` (rva `0x2757a30`) → player `+0xd0` (camera controller) → `+0x20` →
  `Camera : WorldNode`; `WorldTransform` at `+0x20`; near/far/FOV/aspect at `+0x6C/+0x70/+0x74/+0x78`; `RotMatrix`
  columns right/forward/up, Z up, `WorldPosition` three doubles, reversed-Z depth, FOV 45.75° vertical `[reported]`.
  **Built-in first-person switch:** `DSCameraInterface::SetEnableForceSubjectiveCameraMode(bool, float)` at rva
  `0x26e6130` (bit 7 of `[*(0x147be7998)+0x29c]`, blend time at `+0x208`) `[reported]`.
  **First link corroborated from the code** `[inferred-static 2026-10-28]`: the getter loads the manager from
  `ds.exe+0x7bc7568`, looks up with the descriptor at `ds.exe+0x4b1acb8` (on disk index 0, fast path `[mgr+0x48]`),
  returns `[+0x70]`. A read-only logger of it is built (`staging/death-stranding-vr/proxy-d3dcompiler/src/camera_log.c`,
  note `modding-notes/2026-10-28-pd-a-read-only-camera-logger.md`), not yet run. FOV writes flicker unless made
  at the game's own FOV writer (recomputed in ~95% of frames) `[reported]`. Our own read-only logger has to confirm all
  of it before it is trusted. The Bridger framework found beside it targets the ORIGINAL release (appid 1190460), not ours.
- Where projection `P` / FOV comes from:
- The per-eye override maths (`K_eye = …`):

## 7. Constant-buffer fill mechanism
- Map/DISCARD ring / UpdateSubresource / D3D11.1 offset / **persistent map +
  memcpy** (trap):
- Can source contents be read cheaply (captured CPU pointer) or need staging
  read-back?:
- The chosen override patch point and why:

## 8. Pass inventory (by render target)
- Main scene (res/formats):
- Shadow passes (depth-only sizes):
- Post / AA chain (SMAA/TAA/motion vectors; downscale sizes):
- UI / HUD (how it's kept separate):

## 9. cvar / console cheat sheet
| command / cvar | effect | use |
|---|---|---|
| | | |

## 10. Autonomous harness recipe (this game)
- Launch to a known scene (commands used): `steam://rungameid/1850570` (about 60 s to the menu on the dev PC; the very first start prepares shaders for longer). Menu: keyboard only (hardware-scancode presses; mouse clicks ignored); Continue loads the last save (about 2.5 min on the dev PC), into Episode 1 carrying the president's body. W walks, mouse movement turns the camera. Closing: a plain `taskkill` (window close) exits cleanly. Detail: `ai-game-control-profiles/routes/death-stranding/KNOWN.md`. `[verified-live 2026-10-16, n=1]`
- In-process input / camera drive method that worked:
- Frame-capture method; where images land:

## 11. Dead ends & false leads (save future time)
- **The stereoscopic 3D setting** (`SetStereoscopic`, `Set3DScreenFactor`, the depth multipliers): stubbed in the PC build — setters do nothing, the getter always says off `[inferred-static 2026-10-01]`. Not a route to two eyes.
- none yet.

## 12. Open risks toward the North Star
- **Direct3D 12 is the hardest renderer family on this account**: no project here has done stereo on D3D12 yet.
- A 75 GB install on a modern engine: expect the frame to be built across many threads.
