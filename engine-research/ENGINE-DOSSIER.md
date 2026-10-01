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
- **Settings and saves:** `%LOCALAPPDATA%\KojimaProductions\DeathStrandingDC\<steam id>\profile` (binary) plus save slots; no ini found, so window mode and music are set in the in-game menu `[inferred-static 2026-10-01]`.
- Attach workflow that works: not yet tested.
- Injection vector that works (proxy DLL name / injector / framework): not yet tested.

## 5. Threading & frame structure
- Immediate context only, or deferred contexts + command lists?:
- Which thread(s) do what; render-thread name(s):
- One-frame walkthrough (record → replay → present):

## 6. Camera & projection delivery (the crucial section)
- How the world transform reaches the GPU (shared VP buffer / per-draw MVP /
  other), with **shader-reflection / disassembly evidence**:
- Exact constant-buffer slot, parameter name(s), byte offset(s), layout,
  handedness, row/column convention:
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
- Launch to a known scene (commands used):
- In-process input / camera drive method that worked:
- Frame-capture method; where images land:

## 11. Dead ends & false leads (save future time)
- **The stereoscopic 3D setting** (`SetStereoscopic`, `Set3DScreenFactor`, the depth multipliers): stubbed in the PC build — setters do nothing, the getter always says off `[inferred-static 2026-10-01]`. Not a route to two eyes.
- none yet.

## 12. Open risks toward the North Star
- **Direct3D 12 is the hardest renderer family on this account**: no project here has done stereo on D3D12 yet.
- A 75 GB install on a modern engine: expect the frame to be built across many threads.
