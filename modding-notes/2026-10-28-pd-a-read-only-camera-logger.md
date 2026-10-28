# 2026-10-28 (dev PC, `/pd`, NO LAUNCH): a read-only camera logger, built and self-tested

The game was not launched, and nothing here has been run in it.

## What was done

- **The player getter read from the exe** `[inferred-static 2026-10-28]`: `GetLocalDSPlayerEntity` (`ds.exe+0x2757a30`)
  loads a manager from the global `ds.exe+0x7bc7568` (zero on disk, filled at run time), calls a lookup at
  `0x1423c5060` with the descriptor at `ds.exe+0x4b1acb8`, and returns `[result+0x70]`. The lookup's fast path, taken
  when the descriptor reads `a == 1, b == 2, ptr == 0`, is a plain `[mgr + index*8 + 0x48]`; on disk the descriptor
  holds index 0, a 1, b 2, ptr 0. That is the `*(*(mgr)+0x48)+0x70` the public mod reports, reached independently
  from the code, so the first link of the path is corroborated, not only reported.
- **The logger** lives in our `d3dcompiler_47.dll` stand-in, the file already proven to load in this game
  (2026-10-16), replacing its disproved D3DReflect wrapper (moved to `archive/`). Source:
  `staging/death-stranding-vr/proxy-d3dcompiler/src/camera_log.c`. When the dll loads inside `ds.exe` it starts one
  thread that waits 5 s, checks the getter's first bytes are the ones read statically (else logs "a different build"
  and stops), then four times a second walks manager -> player -> `+0xd0` controller -> `+0x20` camera and logs
  position (three doubles at `+0x20`), the 3x3 floats assumed at `+0x38`, and near/far/FOV/aspect at `+0x6C..+0x78`;
  every 10 s a raw hex dump of `camera+0x20..0x7F` so the assumed layout can be checked against real bytes.
  Every read goes through `ReadProcessMemory`: a bad pointer is logged as the step where the chain stopped, never a
  crash. It writes nothing in the game and calls no game code.
- Built with all 29 exports forwarded (`verify_exports` MATCH) `[compile-verified 2026-10-28]`; the no-game self-test
  `test/camera_selftest.c` passes 9/9 (full walk, each failure step, unmapped pointers, logger stays off outside
  `ds.exe`) `[verified-numerically 2026-10-28, n=1]`. Build sha256 `6a93123229fc45f3...`.

## What is NOT established

- That the chain holds in the running game: all of it past the getter is `[reported]` until the log shows it.
- The rotation's place (`+0x38`) is a guess from the position being 24 bytes; the raw dump settles it.
- If the log says the chain stops at step 3, the descriptor is not on its fast path at run time and the logger
  needs the slow path (a list walk with a type check that calls game code); that would be a design change, not a knob.

## The one test for the next live start (`[FLAT]`)

1. Game closed. In the game folder rename `d3dcompiler_47.dll` to `d3dcompiler_47_real.dll` (keep a copy), copy
   `build/d3dcompiler_47.dll` in.
2. Start, Continue, walk forward, turn on the spot left and right, stand still. Quit.
3. Read `dsvr_camera_log.txt`: a `chain` line with four addresses, then `cam` rows. FOV about 0.80 radians
   (45.75 degrees) means the lens offsets are right; position changing while walking and steady while standing means
   the camera node is right; the rotation columns changing on a turn and keeping length 1 means `+0x38` is right.
   `stops at step N` names the link that is wrong.
4. Take ours out again if it fails (rename the real one back), and archive the log either way.
