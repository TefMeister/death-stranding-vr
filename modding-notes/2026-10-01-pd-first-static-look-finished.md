# 2026-10-01 (`/pd`, dev PC): the first static look is finished

**The game was not launched.** Everything comes from `ds.exe` and the files beside it. Evidence (names only):
`dev-archive/recon/2026-10-01-first-static-look/selected-names.txt`. Dossier §2–4 filled in.

## What it found `[inferred-static 2026-10-01]` unless marked

- **Our file can probably get in as `dxgi.dll` or `d3d12.dll`:** neither is in the import table; the game loads
  both by name at run time.
- **The code is readable on disk** (no encryption: `.text` entropy 6.36, entry point in `.text`) `[measured]`, so
  static disassembly will work, unlike Witcher 2 and Heavy Rain.
- **Photo mode is a built-in free camera**, the cheapest way to move the camera for camera work.
- **A stereoscopic 3D setting is still in the PC build**: `Set/GetStereoscopic`, `Set3DScreenFactor`, and depth
  multipliers for the normal and the first-person view, among the user-settings functions. Probably the
  PlayStation 3D-TV option. Whether the PC renderer still draws two eyes from it is the most valuable unknown on this
  project `[hypothesis]`: it is answerable statically now, by following the code that reads it.
- Upscalers: DLSS, XeSS and FSR 2, the last built in. Audio is Wwise.
- A handful of command-line switches (`-enable_dred`, `-safe`, `-unlock_all_perks`, …); no developer console found.
- Settings live in a binary `profile` under `%LOCALAPPDATA%\KojimaProductions\DeathStrandingDC\<id>\`, so the window
  and the music are set from the in-game menu on the first launch.
