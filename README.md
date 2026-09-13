## About this fork (`asmlift-benchmark` branch)

This branch exists to make the [asmlift](https://github.com/macabeus/asmlift) decompiler
benchmark reproducible. It is the upstream
[mariopartyrd/marioparty4](https://github.com/mariopartyrd/marioparty4) tree at commit
[`147b165a`](https://github.com/mariopartyrd/marioparty4/commit/147b165a83187ac9e6cfdc3bf52f2e73437b1ffd)
("Backport mp5 macros", 2026-06-04), the exact commit the benchmark's functions are vendored
from, plus one integration commit touching exactly two files:

- `decomp.yaml`: new. It describes the `GMPE01_00` (USA Rev 0) build in the decomp.yaml format
  and adds a `tools.asmlift` block pointing asmlift at the project's symbol source
  (`tools.asmlift.elf`): `build/GMPE01_00/main.elf`, the DOL's ELF, which the ordinary build
  already links. No extra build step. That ELF covers the DOL only: the 99 REL modules are
  linked into separate `build/GMPE01_00/<module>/<module>.plf` files, which this key does not
  name.
- `README.md`: this section.
- Nothing else differs from upstream: no `src/` byte, no rename, no build fix. The
  `extern/musyx` submodule stays at upstream's pin.

The build needs exactly what upstream's [Dependencies](#dependencies) section lists: Python 3,
ninja and, on macOS or non-x86 Linux, wine. It also needs a Mario Party 4 USA Rev 0 (`GMPE01`,
revision 00) disc image. To reproduce the benchmark rows, build the project (all 100 SHA-1
lines must report OK), then follow the per-function scripts published in the benchmark report.
The symbol source is `build/GMPE01_00/main.elf` and needs no further command:

    git clone -b asmlift-benchmark https://github.com/macabeus/marioparty4
    cd marioparty4
    git submodule update --init --recursive
    cp <your disc image> orig/GMPE01_00/
    python3 configure.py
    ninja

Success is `100 files OK` from the `CHECK config/GMPE01_00/build.sha1` step, and the file
`build/GMPE01_00/ok`.

Build notes for macOS, where the compilers run under wine:

- A compile or link step can fail with `ShellExecuteEx failed: Internal error.`. That is wine
  failing to launch the tool, not a source or link error. Run `ninja` again; it resumes.
- On the first wine command, wine starts background services (`services.exe`,
  `winedevice.exe`, ...) that can inherit ninja's output pipe. ninja then waits forever a few
  steps from the end (for example at `[193/197]`), with its log no longer growing although the
  outputs exist. Stop only that `ninja` (Ctrl-C), leave the wine services alone, and run
  `ninja` again. It finishes the remaining steps.
- A single wine compile can also freeze at 0% CPU and never exit. The symptom is the same: the
  ninja log stops growing. Stop only that stuck compiler process (check that its working
  directory is this checkout), leave the wine services alone, and run `ninja` again. Anything
  that runs this build unattended needs a timeout on it.

None of these affects the output bytes; the SHA-1 check is the arbiter.

---

Mario Party 4  
[![Build Status]][actions] [![Progress]][progress site] [![DOL Progress]][progress site] [![RELs Progress]][progress site] [![Discord Badge]][discord]
=============

[Build Status]: https://github.com/mariopartyrd/marioparty4/actions/workflows/build.yml/badge.svg
[actions]: https://github.com/mariopartyrd/marioparty4/actions/workflows/build.yml
[Progress]: https://decomp.dev/mariopartyrd/marioparty4.svg?mode=shield&measure=code&label=Code&category=all
[DOL Progress]: https://decomp.dev/mariopartyrd/marioparty4.svg?mode=shield&measure=code&label=DOL&category=dol
[RELs Progress]: https://decomp.dev/mariopartyrd/marioparty4.svg?mode=shield&measure=code&label=RELs&category=modules
[progress site]: https://decomp.dev/mariopartyrd/marioparty4
[Discord Badge]: https://img.shields.io/discord/994839212618690590?color=%237289DA&logo=discord&logoColor=%23FFFFFF
[discord]: https://discord.gg/T4faGveujK

A work-in-progress decompilation of Mario Party 4. While the two USA versions are completely matching, most non-game-engine code is not documented at all. Work on the PAL and JP versions is currently stale, as it involves a lot of repetitive work, which could be done much easier later when there is special tooling for porting splits and symbols between versions.

There is **NO** working PC port yet, but it is in the making.

This repository does **not** contain any game assets or assembly whatsoever. An existing copy of the game is required.

Version Completion:

- `GMPE01_00`: Rev 0 (USA) ✅
- `GMPE01_01`: Rev 1 (USA) ✅
- `GMPP01_00`: Rev 0 (PAL) ❌
- `GMPP01_01`: Rev 1 (PAL) ❌
- `GMPP01_02`: Rev 2 (PAL) ❌
- `GMPJ01_00`: Rev 0 (JP)  ❌

Dependencies
============

Windows
--------

On Windows, it's **highly recommended** to use native tooling. WSL or msys2 are **not** required.  
When running under WSL, [objdiff](#diffing) is unable to get filesystem notifications for automatic rebuilds.

- Install [Python](https://www.python.org/downloads/) and add it to `%PATH%`.
  - Also available from the [Windows Store](https://apps.microsoft.com/store/detail/python-311/9NRWMJP3717K).
- Download [ninja](https://github.com/ninja-build/ninja/releases) and add it to `%PATH%`.
  - Quick install via pip: `pip install ninja`

macOS
------
- Install [ninja](https://github.com/ninja-build/ninja/wiki/Pre-built-Ninja-packages):
  ```
  brew install ninja
  ```
- Install [wine-crossover](https://github.com/Gcenx/homebrew-wine):
  ```
  brew install --cask --no-quarantine gcenx/wine/wine-crossover
  ```

After OS upgrades, if macOS complains about `Wine Crossover.app` being unverified, you can unquarantine it using:
```sh
sudo xattr -rd com.apple.quarantine '/Applications/Wine Crossover.app'
```

Linux
------
- Install [ninja](https://github.com/ninja-build/ninja/wiki/Pre-built-Ninja-packages).
- For non-x86(_64) platforms: Install wine from your package manager.
  - For x86(_64), [wibo](https://github.com/decompals/wibo), a minimal 32-bit Windows binary wrapper, will be automatically downloaded and used.

Building
========

- Clone the repository:
  ```
  git clone https://github.com/mariopartyrd/marioparty4.git
  ```

- Initialize and update submodules:

  ```sh
  git submodule update --init --recursive
  ```

- Copy your game's disc image to `orig/[GAMEID]`. The supported game IDs are listed above.
  - Supported formats: ISO (GCM), RVZ, WIA, WBFS, CISO, NFS, GCZ, TGC
  - After the initial build, the disc image can be deleted to save space.

- Configure:
  ```
  python configure.py
  ```

  To choose a version other than the USA Rev 0 one, add `--version [GAMEID]` to the command. 

- Build:
  ```
  ninja
  ```

Diffing
=======

Once the initial build succeeds, an `objdiff.json` should exist in the project root. 

Download the latest release from [encounter/objdiff](https://github.com/encounter/objdiff). Under project settings, set `Project directory`. The configuration should be loaded automatically. 

Select an object from the left sidebar to begin diffing. Changes to the project will rebuild automatically: changes to source files, headers, `configure.py`, `splits.txt` or `symbols.txt`.
