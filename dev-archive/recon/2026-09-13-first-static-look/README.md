# First static look (2026-09-13)

Read from the installed Steam copy on the home PC, without launching the game. Every claim
below is `[inferred-static 2026-09-13]` unless tagged otherwise: it comes from reading file headers
and strings, not from running anything.

- **Install:** `DEATH STRANDING DIRECTORS CUT`, 75 GB.
- **Identity:** Death Stranding Director's Cut, Steam build, exe `ds.exe` (linked 2024-01-28).
- **Engine:** Decima, Guerrilla Games' engine `[reported]`. Havok and Scaleform strings are present in the exe `[inferred-static 2026-09-13]`.
- **Binary:** **64-bit** (PE32+), `ds.exe` 86.5 MB, linked 2024-01-28. Sections look ordinary (`.text` 62 MB), with no sign of a large protection blob `[inferred-static 2026-09-13]`.
- **Renderer:** Direct3D 12: `d3d12.dll` and `dxgi.dll` appear in the exe's strings, and `dxcompiler.dll`/`dxil.dll` ship beside it `[inferred-static 2026-09-13]`.
- **Protection:** No Denuvo string and no protection-shaped section found; Steam API present `[inferred-static 2026-09-13]`. Not tested live.
- **Other files:** Oodle compression (`oo2core_7_win64.dll`), Bink 2 video, and the DLSS and XeSS upscalers ship beside the exe. Data archives under `data\`, not yet looked at.

## Method

PE headers read with a short script: machine type, link timestamp, section names and sizes.
Then a case-insensitive search of each binary for renderer DLL names (`d3d9`, `d3d11`, `d3d12`,
`dxgi`, `vulkan-1`, `opengl32`), protection markers (`denuvo`, `securom`, `.bind`) and middleware
names. A string match shows a name is present in the file, not that the code path is used.

## Risks noted

- **Direct3D 12 is the hardest renderer family on this account**: no project here has done stereo on D3D12 yet.
- A 75 GB install on a modern engine: expect the frame to be built across many threads.
