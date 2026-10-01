# 2026-10-01 (`/pd`, dev PC): the camera's constant structure, read from the exe

**The game was not launched, and nothing here has been run.** Read from `ds.exe` on disk; the code is not encrypted.

## How it was found

1. Decima describes its shader constant structures with its own type system: 150 `SRT_RTTI_*` names in the exe,
   one of them `SRT_RTTI_ViewConstants`.
2. The description is filled at start-up, so on disk it is empty, but the member names sit next to it in `.rdata`.
3. Searching the code for references to those names found one registration function, `0x141823d60`, that registers
   each member with three arguments: its name (`rdx`), its byte offset (`r9d`) and a type code (a byte on the stack).
   Decoding every call gives the whole table (`dev-archive/recon/2026-10-01-view-constants-layout/view-constants.txt`).

## Why the reading is trustworthy

- **The type codes match the offset steps exactly**: every `0x1b` member is 0x40 bytes after the previous one
  (a 4×4 matrix), the one `0x1a` member takes 0x30 (3×4), `0x08` members take 0x10 (a vector of four), single values
  take 4. Nothing was assumed about the codes; they fell out of the offsets.
- **It is `ViewConstants`, not a sibling:** the neighbouring registration function (`0x141823cf0`) registers
  `OffscreenParams` (`PrevOffscreenVPMatrix`, …), and the type names in `.rdata` run `SRT_RTTI_OffscreenParams`, its
  members, then `SRT_RTTI_ViewConstants`: the same order twice.

## The layout `[inferred-static 2026-10-01]`

`View` +0x000 · `Proj` +0x040 · `ViewProj` +0x080 · `InvView` +0x0C0 · `OldViewProj` +0x100 · three depth-reconstruct
matrices +0x140/+0x180/+0x1C0 · **`ViewToObserverView` +0x200 (3×4)** · `Viewport` +0x230 · … · **`ViewPos` +0x270** ·
**`FloatingOrigin` +0x290** · **`DepthDirection` +0x29C** · … `UseKJPVolumetricFog` +0x33C.

## What it means

- The camera reaches the shaders as **one shared structure per view**, the shape this estate handles best (one
  place to edit per eye, not every object's own matrix).
- **Several members must move together** for an eye: `View`, `ViewProj`, `InvView`, the depth-reconstruct matrices
  and `ViewPos`. The engine draws relative to a `FloatingOrigin`, so positions are probably camera-relative.
- **`ViewToObserverView`** is a ready-made transform from the drawn view to a separate "observer" view. If the
  renderer applies it, it may be a head-pose slot like Prototype's `View+0x2c` `[hypothesis]`.

## Not established

- Which register slot the structure binds to, matrix row/column order, handedness.
- Which CPU function fills it each frame (the natural hook point): the next static step.
- What reads `ViewToObserverView`.

## Later the same session: looking for the per-frame filler (not found, and why)

Searched the whole `.text` for code that **stores** to several `ViewConstants` offsets (`+0x230`, `+0x240`, `+0x270`,
`+0x290`, `+0x29C`) through one non-stack pointer within a short span. Four candidates came out
(`0x142b7485a`, `0x142be09f6`, `0x142c22557`, `0x143471dda`); all four are **initialisers** — three zero a large
object field by field from a zero register, one copies stored values one field at a time (a loader) — not a
per-frame fill `[inferred-static 2026-10-01]`. A rip-relative scan for the member descriptors gave only chance
byte matches. Most likely the fill goes through Decima's descriptor system at run time (values set by registered
offset, or a whole struct copied), which a fixed-offset search cannot see `[hypothesis]`.

**Cheaper routes, for the next step:**
- the game ships `d3dcompiler_47.dll` and `dxcompiler.dll` **beside the exe**, and imports `D3DReflect`; a logging
  stand-in for the compiler library could report, live, which register slot `ViewConstants` binds to
  `[hypothesis]` (⚠️ if the shaders are DXIL, the reflection goes through `dxcompiler.dll`, not `D3DReflect`);
- or, after the first launch, read the camera block live from our `dxgi.dll`/`d3d12.dll` proxy.

## Later still: the reflection logger is built (not installed)

`staging/death-stranding-vr/proxy-d3dcompiler/`: a 64-bit `d3dcompiler_47.dll` that forwards all 29 exports of the
game's own copy (renamed `d3dcompiler_47_real.dll`) and wraps `D3DReflect`, logging to `dsvr_reflect_log.txt` each
constant buffer's name, size and register slot, and every `ViewConstants` member with its offset. No-game test
(`test/reflect_selftest.c`): a shader with a `ViewConstants` buffer compiled through the forwarder and reflected
through the wrapper logs `register b3` and all four members with the right offsets; a DXIL-style blob fails with the
real error and is logged. **8 of 8 checks pass** `[verified-numerically 2026-10-01, n=8]`.

Not installed: the game has not had its first plain launch yet, and installing means renaming one of its files.
