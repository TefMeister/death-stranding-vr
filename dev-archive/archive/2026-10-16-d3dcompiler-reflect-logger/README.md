# d3dcompiler_47 reflection logger (taken out 2026-10-16)

Built from `staging/death-stranding-vr/proxy-d3dcompiler/` (sha256 `2bc5fbbe1eda8a84...`). It loaded in the game on
two starts and forwarded all 29 exports, but `D3DReflect` is only called for about ten small start-up shaders and none
of them carry the camera, so it was taken out of the game folder. The two logs from those starts are kept beside it.
