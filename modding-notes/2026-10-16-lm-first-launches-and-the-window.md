# 2026-10-16 (`/lm`, dev PC): first launches, the window, and what the shader logger could not see

First live session on this game, on the dev PC (GTX 1660 SUPER). Everything is n=1 unless it says otherwise.

## The first-game checklist: done

1. **Runs as shipped:** started through Steam, reached the main menu (game version 1.004) in about a minute
   `[verified-live 2026-10-16]`. It opens fullscreen 1920x1080 by default.
2. **Runs with our file:** the `d3dcompiler_47.dll` logging proxy (staging `proxy-d3dcompiler/`, built from the
   game's own copy; 29 of 29 exports) loaded on two starts, and the game ran normally `[verified-live 2026-10-16, n=2]`.
3. **Windowed 1280x720:** the display settings live in a plain-text **`settings.cfg` beside `ds.exe`**. With the game
   closed: `window_mode 1`, `rendering_width 1280`, `rendering_height 720`, window rect (320,180)-(1600,900). The
   next start came up as a 1300x763 window with a 1280x720 picture area, desktop still 1920x1080
   `[verified-live 2026-10-16, n=1]`. Changing it in the menu also works live, but the first menu change was lost
   when the game was closed by force; the file is the reliable route. Backups outside the repo.

## Music

There is **no music-only volume**: Audio Settings has Master Volume, BB voice output and speaker optimisation only,
and `game_settings.cfg` has only `MasterVolumeSensitiviness`. Music was left on (the rule mutes music only, never the
effects or speech). Board row: look for a music setting or bus in the exe.

## Screen effects for VR

`settings.cfg` holds `postprocess_dof` and `postprocess_motionblur` (the menu's Depth of Field and Motion Blur). Film
grain is visible on the title screen and has no menu switch; chromatic aberration not seen. Flat play keeps them all.

## The shader logger cannot find the camera

`D3DReflect` was called for about ten small start-up compute shaders (`Uniforms`, `InUniformParams`,
`RasterizerVariables`, ...), none on the second start and none in gameplay `[measured 2026-10-16, n=2]`. The game
ships compiled shaders and reflects only a handful, so this route will not reveal the camera constants. The proxy
was taken out of the game folder and kept in `dev-archive/archive/2026-10-16-d3dcompiler-reflect-logger/`; the
original file is back (checked against the backup).

## Driving it

Menus take keyboard presses (hardware scancodes); mouse clicks are ignored. Continue loads the last save in about
2.5 minutes here: Episode 1, Sam carrying the president's body. W walks (the marker distance fell from 40 m to
33 m), mouse movement turns the camera. Closing the window (`taskkill` without `/F`) exits cleanly.
Detail: `ai-game-control-profiles/routes/death-stranding/KNOWN.md`.

## Recorded

`MEGA transfer\Videos\DEATH STRANDING DIRECTORS CUT\play_2026-10-16_19-24-51.mp4` (first start, the menus) and
`play_2026-10-16_19-34-18.mp4` (with our file, the second window setting). The third start, which reached gameplay,
was **not** recorded: the automatic recorder missed a relaunch again, the same gap seen with The Witcher 2 today.

## Installed now (dev PC)

Nothing of ours. Only `settings.cfg` is changed (windowed 1280x720).
