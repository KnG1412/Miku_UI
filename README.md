# KnG1412//UI

A system-monitor HUD for Windows, drawn directly on your desktop background: CPU, RAM, GPU, drives, battery, clock, open apps and a live history graph, plus an ASCII-art Miku in the middle of the screen.

A single C file written with Win32 + GDI. No third-party libraries and no extra DLLs to install.

<img width="1919" height="1079" alt="screenshot-ui" src="https://github.com/user-attachments/assets/923be738-7312-4a44-99c0-f5d118f1f99f" />


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
- Only one instance runs at a time.

## Hotkeys

Work system-wide:

| Keys | Action |
| --- | --- |
| `Ctrl` + `Alt` + `H` | Show / hide the HUD cards (the Miku art stays) |
| `Ctrl` + `Alt` + `U` | Toggle start with Windows |
| `Ctrl` + `Alt` + `Q` | Quit |

If another app has already taken one of these combinations, the program falls back to polling the keys every 40 ms (slightly more CPU use).

## Requirements

- Windows 10 or 11.
- **MSVC** compiler (Visual Studio or Build Tools). The code uses `#pragma comment(lib, ...)`, so MinGW is not supported.

## Build

Open **x64 Native Tools Command Prompt for VS** and run:

```bat
cl /O1 /MT /GS- /W3 KnG1412_UI.c /link /SUBSYSTEM:WINDOWS /OPT:REF /OPT:ICF
```

This produces `KnG1412_UI.exe`. Keep this name: the Windows startup entry and the process name are both `KnG1412_UI`.

## Usage

1. Run `KnG1412_UI.exe`. The overlay appears on the desktop right away.
2. By default the program **registers itself to start with Windows** (key `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`, value `KnG1412_UI`). Press `Ctrl+Alt+U` to turn it off; the choice is remembered for next time.
3. Quit with `Ctrl+Alt+Q`.

To uninstall: turn off autostart with `Ctrl+Alt+U`, quit the program and delete the `.exe`. To also clean the registry, delete the key `HKCU\Software\KnG1412_UI`.

## Designed for low memory use

- No OpenGL, so no graphics driver is loaded into the process.
- The screen is redrawn once per second in 64-row bands through one small scratch surface, instead of keeping a full-screen back buffer.
- The Miku image is rendered once and its temporary memory is released immediately.
- `dxgi.dll` is loaded only while enumerating GPUs, then unloaded.
- Runs at below-normal priority with EcoQoS enabled, and uses a timer with tolerance so Windows can coalesce wake-ups.

<img width="1386" height="752" alt="screenshot-memory" src="https://github.com/user-attachments/assets/18688f28-7640-4dab-9846-ce0bbd4489ef" />

<img width="1297" height="742" alt="Screenshot 2026-10-18 121642" src="https://github.com/user-attachments/assets/86a21e0a-6bbe-4d02-a3bd-c729070ac33e" />


- Task Manager (Details tab) shows `KnG1412_UI.exe` using approximately 200 KB of RAM, with a commit size of about 6.5 MB.

## Known limitations

- The drive card only shows **C:** and **D:**; a machine without a D: drive shows "DRIVE NOT AVAILABLE".
- The GPU card has two slots: Intel (or the first other vendor) and NVIDIA (or the next vendor). Machines with more than two GPUs only show two.
- The layout is in pixels and the app is not DPI-aware, so it may look slightly blurry on high display scaling.
- Draws on the primary monitor only.
- Hotkeys are fixed in the code.
- GPU load % comes from the Windows `GPU Engine` performance counters; the way it is calculated (busiest engine) can differ slightly from Task Manager.

## Code layout

Everything lives in `KnG1412_UI.c`:

- Embedded ASCII-art data (character table, color table, palette).
- GDI drawing layer: alpha-blended rectangles, anti-aliased lines, text, banded painting.
- Data collectors: CPU, RAM, drives, battery, GPU (PDH + DXGI), app list.
- Card drawing functions and the window loop (`wWinMain`, `WindowProc`).

## Image note

The ASCII art in the middle depicts the character Hatsune Miku, which belongs to Crypton Future Media. If you redistribute this or use it commercially, check their usage terms and replace the art if needed.

## License

The code is released under the [MIT License](LICENSE). The license covers the code only; the embedded Miku artwork is not covered by it (see *Image note* above).
