# Miku//UI


A system-monitor HUD for Windows, drawn directly on your desktop background: CPU, RAM, GPU, drives, battery, clock, open apps and a live history graph, plus an ASCII-art Miku in the middle of the screen that you can turn off.

A single C file written with Win32 + GDI. No third-party libraries and no extra DLLs to install.

<img width="1919" height="1079" alt="Screenshot 2026-10-18 190803" src="https://github.com/user-attachments/assets/fdbf4b57-58ba-4419-a253-a00d9cfb2ad4" />

> **About this project:** most of the code was produced by "vibe coding" (written together with an AI). I use it daily on my own machine, but it has not been tested widely across different setups.

## Features

- **CPU:** CPU model name read from the machine, total load %.
- **RAM:** usage % and used / total capacity.
- **GPU:** name and load % for up to two GPUs (Intel / NVIDIA / AMD). Names are read from the driver, not hard-coded.
- **Drives:** used / free space for C: and D:.
- **System:** time, date, battery (charging / on battery / on AC), OS uptime.
- **Activity:** apps with open windows, plus a log of recent app open / close events.
- **History graph:** CPU, GPU and RAM over roughly the last 2 minutes.
- The overlay sits below all other windows, is click-through, never takes focus and does not appear on the taskbar.
- The Miku art can be toggled with a hotkey; the choice is remembered between runs.
- Only one instance runs at a time.

## Hotkeys

Work system-wide:

| Keys | Action |
| --- | --- |
| `Ctrl` + `Alt` + `H` | Show / hide the HUD cards (the Miku art stays) |
| `Ctrl` + `Alt` + `M` | Show / hide the Miku art |
| `Ctrl` + `Alt` + `U` | Toggle start with Windows |
| `Ctrl` + `Alt` + `Q` | Quit |

If another app has already taken one of these combinations, the program falls back to polling the keys every 40 ms (slightly more CPU use).

## Requirements

- Windows 10 or 11.
- **MSVC** compiler (Visual Studio or Build Tools). The code uses `#pragma comment(lib, ...)`, so MinGW is not supported.

## Build

Open **x64 Native Tools Command Prompt for VS** and run:

```bat
cl /O1 /MT /GS- /W3 Miku_UI.c /link /SUBSYSTEM:WINDOWS /OPT:REF /OPT:ICF
```

This produces `Miku_UI.exe`. Keep this name: the Windows startup entry and the process name are both `Miku_UI`.

## Usage

1. Run `Miku_UI.exe`. The overlay appears on the desktop right away.
2. By default the program **registers itself to start with Windows** (key `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`, value `Miku_UI`). Press `Ctrl+Alt+U` to turn it off; the choice is remembered for next time.
3. Quit with `Ctrl+Alt+Q`.

To uninstall: turn off autostart with `Ctrl+Alt+U`, quit the program and delete the `.exe`. To also clean the registry, delete the key `HKCU\Software\Miku_UI` (it stores your autostart and Miku preferences).

## Designed for low memory use

- No OpenGL, so no graphics driver is loaded into the process.
- The screen is redrawn once per second in 64-row bands through one small scratch surface, instead of keeping a full-screen back buffer.
- The Miku image is rendered once and its temporary memory is released immediately.
- Only the regions that actually change (cards, title, key hints, history graph) are repainted each second. The middle of the screen is never touched per tick.
- With Miku hidden (`Ctrl+Alt+M`), the art is not rendered at all: no temporary image is created and its data is never read. If you start the program with Miku already hidden, that cost is never paid. Hiding the HUD cards (`Ctrl+Alt+H`) as well leaves only the title and key hints to repaint.
- `dxgi.dll` is loaded only while enumerating GPUs, then unloaded.

<img width="1436" height="748" alt="Screenshot 2026-10-18 190710" src="https://github.com/user-attachments/assets/a6142081-8c9a-41cc-aabd-dbf1c0ae793b" />

Task Manager shows `Miku_UI.exe` uses only 0.1MB of RAM.

## Known limitations

- The drive card only shows **C:** and **D:**; a machine without a D: drive shows "DRIVE NOT AVAILABLE".
- The GPU card has two slots: Intel (or the first other vendor) and NVIDIA (or the next vendor). Machines with more than two GPUs only show two.
- The layout is in pixels and the app is not DPI-aware, so it may look slightly blurry on high display scaling.
- Draws on the primary monitor only.
- Hotkeys are fixed in the code.
- GPU load % comes from the Windows `GPU Engine` performance counters; the way it is calculated (busiest engine) can differ slightly from Task Manager.

## Code layout

Everything lives in `Miku_UI.c`:

- Embedded ASCII-art data (character table, color table, palette).
- GDI drawing layer: alpha-blended rectangles, anti-aliased lines, text, banded painting.
- Data collectors: CPU, RAM, drives, battery, GPU (PDH + DXGI), app list.
- Card drawing functions and the window loop (`wWinMain`, `WindowProc`).


## License

The code is released under the [MIT License](LICENSE). The license covers the code only; the embedded Miku artwork is not covered by it (see *Image note* above).
