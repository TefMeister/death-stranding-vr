# A public mod maps the camera on our exact `ds.exe`, and names a built-in first-person switch

From: `/gr`, 2026-10-04. Topic: `external-research/topics/2026-10-04-minecraft-in-ds-maps-the-camera-on-our-exact-build.md`.

- mihailinl's *minecraft-in-death-stranding* (MIT) works on `ds.exe` **linked 2024-01-28**, the same link date as
  dossier §1 `[reported 2026-10-04]`.
- Camera path `[reported]`: `GetLocalDSPlayerEntity` (rva `0x2757a30`) → player `+0xd0` (camera controller) →
  `+0x20` → `Camera : WorldNode`; `WorldTransform` `+0x20`; near/far/FOV/aspect at `+0x6C/+0x70/+0x74/+0x78`.
  `RotMatrix` columns right/forward/up, Z up, `WorldPosition` three doubles, reversed-Z depth, FOV 45.75° vertical.
- **First-person switch** `[reported]`: `DSCameraInterface::SetEnableForceSubjectiveCameraMode(bool, float)` at rva
  `0x26e6130` (bit 7 of `[*(0x147be7998)+0x29c]`, blend time at `+0x208`).
- **FOV writes flicker** unless made at the game's own FOV writer (recomputed ~95% of frames) `[reported]`.
- Suggested dossier additions: these under §4 (camera), tagged `[reported]` until our own read-only logger confirms
  them; a `[FLAT]` row for that logger plus one call to the first-person switch. Note that the Bridger framework found
  alongside targets the ORIGINAL release (appid 1190460, 2022 build), not ours.
