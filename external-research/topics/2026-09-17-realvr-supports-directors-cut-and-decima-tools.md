# Luke Ross's RealVR already runs Death Stranding Director's Cut; Decima Workshop and camera tools exist

**Status:** 🆕 new · **Priority:** high — it changes what this project would add.

## What is public

- **Luke Ross's RealVR mod supports Death Stranding Director's Cut**, using **alternate-eye rendering**
  (one eye per frame; the profile defaults to 120 fps AER) `[reported]`. Helix Mod's page (July 2025)
  documents using it with VRto3D or WibbleWobbleVR for 3D displays, and lists the known issues: UI depth,
  screen effects (water, dirt), halos around foreground objects, and some background flicker from
  occlusion culling `[reported]`. The page describes it as free, with optional Patreon support
  `[reported]`.
- **It targets the Director's Cut specifically** `[reported]`. Which edition this project owns is a
  first thing to confirm.
- **Otis_Inf's photomode camera** supports Death Stranding and the Director's Cut (free camera, camera
  paths) `[reported]`.
- **DSDC Camera Tuner** (Nexus mod 111) changes gameplay FOV and camera distance externally, without
  replacing game files `[reported]` — its approach shows the camera values are reachable in memory.
- **Decima Workshop** is the public editor for Decima-engine games `[reported]`.

## Why it matters here

1. **Alternate-eye VR already exists** for this game. What it does not do is synchronous stereo
   (both eyes rendered each frame), which is exactly where AER's artefacts come from. A synchronous
   route is the gap this project could fill.
2. The camera tools prove the camera is reachable from outside, which shortens the static search.

## Next step

Confirm the owned edition (standard or Director's Cut); read the camera tools' public notes for where
the camera lives.

## Sources

- Helix Mod, "Death Stranding Director's Cut [RealVR + VRto3D/WWVR]" — <https://helixmod.blogspot.com/2025/07/death-stranding-directors-cut-realvr.html>
- Otis_Inf Photomode Mods, Death Stranding — <https://opm.fransbouma.com/Cameras/deathstranding.htm>
- DSDC Camera Tuner — <https://www.nexusmods.com/deathstranding/mods/111>
