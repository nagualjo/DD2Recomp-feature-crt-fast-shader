## Experimental CRT Fast Shader Fork

This is an experimental, unofficial fork of [DD2Recomp](https://github.com/gabbycat4/DD2Recomp).

The goal of this fork is to explore the possibility of adding lightweight CRT shader support, with a particular focus on **CRT Fast**, while preserving the original project's performance, visual quality and gameplay experience.

This fork is also a personal learning project focused on understanding the rendering pipeline and exploring graphics programming.

**Current status:** Experimental development. CRT shader support is a work in progress and is not yet implemented.

All credit for the original project goes to [gabbycat4](https://github.com/gabbycat4).

# Double Dragon II Recompiled

A native PC port of **Double Dragon II: The Revenge** (NES, USA Rev 1), built
with [NESRecomp](https://github.com/mstan/nesrecomp). The game's own 6502 code
is recompiled to C and runs natively, cycle-accurate, at the speed of the
real console, with additions the NES could never do:

- **Smooth scrolling.** The camera glides instead of stepping 2 pixels on the
  game's uneven 2,2,3,1 rhythm, horizontally and on ladders.
- **Character glide.** Billy, Jimmy and every enemy glide between the game's
  30 Hz positions instead of hopping. Never ahead of the game, at most 1/30 s
  behind; punches and poses happen exactly when the game shows them.
- **120 / 180 / 240 pictures a second.** In-between pictures move the camera
  and characters a fraction of a pixel each, for high-refresh and FreeSync /
  G-Sync monitors.
- **No sprite flicker.** Every sprite is drawn every frame, in one fixed
  order (the NES can only draw 8 per line).
- **Steady HUD line** and an **edge crop** that hides the NES's scrolling
  garbage at the screen edges.
- An in-game **settings menu**, save states and rewind, two players with two
  controllers.

Everything above changes only the picture; the game itself runs exactly as
on a NES. Each feature can be switched off in the menu.

**You need your own ROM.** This repository contains no game data. You must
supply a dump of *Double Dragon II - The Revenge (USA) (Rev 1)*; the program
checks it is the right one before it runs.

## Download

Get the latest version from the
[Releases page](https://github.com/gabbycat4/DD2Recomp/releases/latest):

- **Windows:** `DD2Recomp-Windows-x64.zip`. Extract it, double-click
  `dd2recomp.exe`.
- **Linux:** `DD2Recomp-x86_64.AppImage`. Mark it executable (right-click,
  Properties, Permissions, or `chmod +x`), then double-click it.

The first time, the game asks for your ROM and checks it. No building needed.

## Recommended settings

The game starts with these already on, except two you set yourself:
**Pictures per second** starts at 60, because only high-refresh monitors
benefit from more, and the game opens in a window (press **F11** for
fullscreen). In game, press **Escape** (or the right shoulder button) for the
menu:

| Menu | Setting | Recommended |
|---|---|---|
| Smooth scrolling | Camera smoothing | **Scenery only** |
| Smooth scrolling | Character glide | **On** |
| Smooth scrolling | Pictures per second | **180** on a 240 Hz FreeSync/G-Sync monitor; your monitor's rate or lower otherwise; **60 (off)** on a 60 Hz monitor |
| Picture | No sprite flicker | **On** (always on while Character glide is on) |
| Picture | Steady HUD line | **On** |
| Picture | Edge crop | **Standard** |
| Graphics | VSync | **Off** with FreeSync/G-Sync; **On** without |
| Graphics | Integer scaling / Linear filter | **On / Off** |
| Display | Fullscreen | On (**F11** toggles) |

Settings are saved in `config.ini` beside the program.

## Building

Only needed if you want to change the code. You need Git, CMake 3.20+, a C/C++ compiler, Python 3.11+ and SDL2.

**Linux** (Arch/CachyOS: `sudo pacman -S git cmake ninja sdl2 python`;
Debian/Ubuntu: `sudo apt install git cmake ninja-build build-essential libsdl2-dev python3`):

```sh
git clone --recursive https://github.com/gabbycat4/DD2Recomp.git
cd DD2Recomp
cp /path/to/your/rom.nes dd2.nes
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build
./build/dd2recomp dd2.nes
```

`--recursive` also downloads the engine and the menu (they are submodules).
If you cloned without it, run `git submodule update --init --recursive`.

The first configure takes a while: it builds the recompiler and translates
the game.

**Optional, for 100% recompiled code:** a little of the game's code is copied
into RAM at run time and runs on the built-in interpreter unless it is
captured. `tools/capture.sh` records it (see the comments in the script);
the game plays correctly either way.

## Controls

Keyboard and controllers are rebindable in the launcher's Controls page. A
second controller plays as player 2 (pick a 2-player mode on the title
screen).

| | |
|---|---|
| Escape / right shoulder | Menu |
| F7 | Save states |
| F8 | Rewind |
| F11 | Fullscreen |
| Tab | Fast forward |
| F12 | Screenshot |

## Command line

Every menu setting also has an option for one run (it does not change
`config.ini`): `--smooth 0|1|2`, `--char-smooth 0|1`, `--present-rate N`,
`--no-sprite-limit 0|1`, `--hud-split-fix 0|1`, `--overscan T,B,L,R`.
Developer logs: `--scroll-log FILE`, `--oam-log FILE`.

## How it works

The NESRecomp engine runs the recompiled game cycle by cycle. This project
adds a presentation layer (`src/dd2_extras.c`, `src/dd2_smooth.c`) through
the engine's `HOST_EXTRAS` interface:

- the camera's real per-line scroll is read from the PPU, its speed is
  tracked over whole cycles of the game's rhythm, and the playfield is
  redrawn from the background alone at the smoothed position;
- DD2 writes its sprite list forwards and backwards on alternate frames (that
  is how it flickers); the list is read in one fixed order, split into
  characters (each is written top row to bottom) and matched frame to frame;
- with a picture rate above 60, the window asks for extra pictures between
  machine frames and they are drawn at 4x resolution, so a step can be a
  quarter of a NES pixel.

`notes.txt` has the timing measurements that started it.

## Credits and licences

- [NESRecomp](https://github.com/mstan/nesrecomp) and
  [recomp-ui](https://github.com/mstan/recomp-ui) by Matthew Stanley; their
  own licences apply (see the `nesrecomp` and `recomp-ui` folders). The engine
  submodule points at a fork with two small window additions (in-between
  pictures, display-sized windows, a VSync menu row).
- This project's code is licensed under the PolyForm Noncommercial License
  1.0.0 (`LICENSE`): free to use, share and change; not to be sold.
- *Double Dragon II: The Revenge* is © Technōs Japan / Arc System Works. This
  project is not affiliated with them and includes none of the game.
