# A Minecraft passthrough mod maps Death Stranding's camera on our exact build, and finds a first-person switch

**Found 2026-10-04 (`/gr`, estate sweep).** Sources: mihailinl's *minecraft-in-death-stranding* (MIT, created
2026-10-03/04), its `README.md` and `MODLOG.md`, read through the GitHub API; plus two Decima frameworks found in the
same search. Nothing run, no code copied.

## What it is

A mod that runs real Minecraft beside Death Stranding Director's Cut and draws its blocks into the game: Minecraft
renders from DS's camera and a ReShade add-on depth-tests the two frames together. To do that it had to read DS's
camera every frame, in process, which is exactly the first thing a VR mod needs.

**Same executable as ours:** its log describes `ds.exe` as 86 MB, PE32+, **linked 2024-01-28**, MSVC 14.16, no
Denuvo or packer, `d3d12`/`dxgi` loaded at run time, `WINHTTP.dll` imported directly `[reported 2026-10-04]`. Our
dossier records the same link date (§1). It names Steam build **13300582**.

## What it found `[reported 2026-10-04]`

- **The camera, through the game's own script exports** (strings → registration → thunk → body):
  `GetLocalDSPlayerEntity` (rva `0x2757a30`) = `*(*(mgr@0x7bc7568)+0x48)+0x70`; `GetCameraPosition` reads
  `*(player+0xd0)` (a camera controller without RTTI), then `+0x20` → a `Camera : WorldNode` with its
  `WorldTransform` at `+0x20` and `NearPlane/FarPlane/FieldOfView/ViewConeAspect` at `+0x6C/+0x70/+0x74/+0x78`,
  matching the type dump from **decima-workshop** (`dsdc_types.json`).
- **Conventions:** `RotMatrix` columns are right, forward, up (determinant +1); world **Z up**; `WorldPosition` is three
  **doubles**; depth is **reversed-Z**. Gameplay values: FOV **45.75° vertical**, near 0.2, far 4000, aspect 1.7778.
  Their check: markers pinned at Sam's feet stay put from opposite sides only with the vertical-FOV reading.
- **⭐ A built-in first-person camera:** `DSCameraInterface::SetEnableForceSubjectiveCameraMode(bool, float)`
  (rva `0x26e6130`) sets bit 7 of `[*(0x147be7998)+0x29c]` and a blend time at `+0x208`; they use it to switch DS to
  its own first-person view from a key.
- **A trap for camera writes:** a plain FOV write flickers, because the game recomputes the FOV on about 95% of frames;
  the fix needs the writer found (their hardware-watchpoint tool for it is built, not yet run).
- **Calling game functions from the ReShade present callback works** for synchronous engine queries, because those
  callbacks run on the game's worker threads, which carry the game's thread-local state (their physics ray casts).

## The two frameworks found alongside

- **Bridger** (suidvandiewereld; "all rights reserved", so read only): a native framework that exposes Death
  Stranding's runtime type information by name (9,867 types with layouts, 1,943 exported functions, every message
  handler, for example `CameraEntity::MsgEntityUpdate`). ⚠️ It supports **Steam appid 1190460, build 2022-08-04**,
  the original release rather than the Director's Cut, so its index does not fit our exe; the approach transfers.
- **Cauldron** (GPL-3.0): a Decima mod loader (Horizon, Death Stranding) with `pulse`, an RTTI and symbol dumper;
  and **eleuthia**, an IDA plugin for Decima RTTI. Tools for reading layouts by name on our build.

## Why it matters for this project

- It **skips most of the camera hunt**: a live-confirmed path to the camera node and its fields on our exact build,
  plus the axis conventions. The board's reflection-logger launch (ViewConstants) still matters for the renderer
  side; this is the gameplay-side camera.
- **The subjective-camera switch is the cheapest first-person route** for a VR mod: the game's own first-person
  mode, turned on from code.
- The FOV-flicker report says where to write: at or after the game's own writer, not from a frame callback.

## Next step

`[FLAT]` one read-only check: with a logger reading `GetLocalDSPlayerEntity` → `+0xd0` → `+0x20` → the `Camera`
fields each frame, confirm the FOV reads 45.75 and the position follows the camera; then call the subjective-camera
switch once and look. Everything above is theirs and `[reported]` until we measure it.

## Credits

mihailinl (minecraft-in-death-stranding, MIT); ShadelessFox (decima-workshop, whose type dump they matched);
suidvandiewereld (Bridger); the Cauldron authors (cauldron, eleuthia); Otis_Inf (closed-source DS camera tools, cited
by mihailinl as proof the camera is writable).
