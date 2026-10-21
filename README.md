# melonDS 1.1 — Keep music at normal speed while fast-forwarding

An unofficial build of [melonDS](https://github.com/melonDS-emu/melonDS) **1.1** with one change.
It is not part of melonDS and is not endorsed by the melonDS team.
The original melonDS README follows further below.

(한국어 설명은 아래에 있습니다.)

## Download

**Releases** (right side of this page) → `melonDS.exe` (Windows x64, single file).
Put it in a folder of its own, or replace the `melonDS.exe` of an official 1.1 download —
your `melonDS.toml` settings keep working.

## What it does

In stock melonDS, fast-forward speeds up the whole console, so music and sound effects
play faster and higher. With this change the game runs fast (or slow, in slow motion) while
music and sound effects keep their **normal speed and pitch**.

DS games play music on the ARM7: a periodic alarm driven by the ARM7 hardware timers
steps the sound sequencer about every 5.2 ms, and the SPU plays the notes. While
fast-forward or slow motion is active, only the ARM7 timers and the SPU run at 1/N of
emulated time, where N is how much faster than real time the emulation actually runs.
The sound hardware stays on real time; everything else runs at the chosen speed.
No ROM changes are needed.

- Option: **Config → Audio settings → "Keep music at normal speed when fast-forwarding /
  in slow motion"** (on by default). Turned off, melonDS behaves exactly as before.
- Speed: **Config → Interface settings → Fast-forward FPS** (e.g. 240 = 4×).
- Hotkeys: "Fast forward" works while held; "Toggle fast forward" switches on and off.

Limits:
- Where a game waits for a sound to finish (for example a cry), that wait takes real
  time, so those moments are not sped up.
- If your PC cannot reach the fast-forward target, the speed actually reached is followed.
- DSi-mode DSP audio is not covered.
- While the option is active, all four ARM7 timers run slower. This fits games built on
  Nintendo's standard SDK sound system; a game whose ARM7 uses its timers for other
  timing could misbehave while fast-forwarding. If that happens, turn the option off —
  melonDS then behaves exactly as before.

## For ROM hacks and homebrew: speed hint

Because sounds keep real time while fast-forwarding, a game that waits for a fanfare or
a cry to end still waits for its full length. A program can ask how fast it is being run
and skip such waits:

- A 32-bit read of **`0x04FFFA40`** (ARM9) returns `0x5350` ("SP") in the upper 16 bits and
  the current speed in 8.8 fixed point in the lower 16 bits (`0x0100` = normal speed,
  `0x0400` = 4×) while "Keep music at normal speed" is in effect.
- The address is unused on hardware; real hardware and other emulators read `0` there,
  so a program that checks for `0x5350` behaves as before everywhere else.

## Source

This branch is the melonDS **1.1** tag plus two commits (the music change and the speed
hint). The same changes as a patch file:
[`keep-music-speed.patch`](keep-music-speed.patch) — apply it to a 1.1 checkout with
`git am`, `git apply` or `patch -p1`, then build as described in `BUILD.md`.
The Windows exe in Releases is built by this repository's GitHub Actions workflow —
melonDS 1.1's own Windows workflow with one fix: it clears MSYS2's `PKG_CONFIG_PATH` so
that the static link uses vcpkg's libraries (MSYS2's current libarchive no longer links
statically), and it keeps the vcpkg cache even when a build fails.

Files changed: `src/NDS.cpp`, `src/NDS.h`, `src/SPU.cpp`, `src/SPU.h`, and
`src/frontend/qt_sdl/` `AudioSettingsDialog.cpp/.h/.ui`, `Config.cpp`, `EmuInstance.h`,
`EmuInstanceAudio.cpp`, `EmuThread.cpp`, `Window.cpp`.

## License

melonDS is licensed under the GNU General Public License, version 3 or later (`LICENSE`).
This change modifies melonDS 1.1 (October 2026) and is distributed under the same license,
**GPL-3.0-or-later**. The copyright notices in the melonDS source are unchanged.
No warranty.

## 한국어

**melonDS 1.1에 기능 하나를 더한 비공식 빌드**입니다. melonDS 팀이 만든 것이 아닙니다.

원래 melonDS는 배속하면 기기 전체가 빨라져 음악·효과음도 빠르고 높아집니다. 이 빌드에서는
게임은 빠르게(슬로모션이면 느리게) 돌고, **음악·효과음은 원래 빠르기와 음높이**로 나옵니다.

- 받기: 오른쪽 **Releases** → `melonDS.exe`. 새 폴더에 두거나 공식 1.1의 `melonDS.exe`와 바꿔 넣으면 됩니다(설정 파일 그대로 사용).
- 켜고 끄기: **Config → Audio settings → "Keep music at normal speed…"**(기본 켬). 끄면 원래 melonDS와 같습니다.
- 배율: **Config → Interface settings → Fast-forward FPS**(240 = 4배).
- 단축키: "Fast forward"는 누르고 있는 동안, "Toggle fast forward"는 한 번 눌러 켜고 끕니다.
- 배속 중에는 ARM7 타이머 넷이 함께 느려집니다. 닌텐도 표준 SDK 사운드를 쓰는 게임에 맞춘 방식이라, ARM7 타이머를 다른 용도로 쓰는 게임은 배속 중 이상할 수 있습니다. 그때는 옵션을 끄면 원래 melonDS와 같습니다.
- 롬핵·홈브루용: 배속 중 `0x04FFFA40`을 32비트로 읽으면 위 16비트 `0x5350`, 아래 16비트에 현재 배속(8.8 고정소수, `0x0100` = 1배)이 나옵니다. 팡파르·울음소리 기다림을 배속 중에 건너뛰는 데 씁니다. 실기와 다른 에뮬레이터에서는 0이 나옵니다.
- 소스: 이 브랜치 = melonDS 1.1 + 커밋 둘(음악 유지, 배속 신호). 같은 변경을 패치 파일 `keep-music-speed.patch`로도 둡니다.
- 라이선스: melonDS와 같은 **GPL-3.0-or-later**. 보증 없음.

---

<p align="center"><img src="https://raw.githubusercontent.com/melonDS-emu/melonDS/master/res/icon/melon_128x128.png"></p>
<h2 align="center"><b>melonDS</b></h2>
<p align="center">
<a href="http://melonds.kuribo64.net/" alt="melonDS website"><img src="https://img.shields.io/badge/website-melonds.kuribo64.net-%2331352e.svg"></a>
<a href="http://melonds.kuribo64.net/downloads.php" alt="Release: 1.1"><img src="https://img.shields.io/badge/release-1.1-%235c913b.svg"></a>
<a href="https://www.gnu.org/licenses/gpl-3.0" alt="License: GPLv3"><img src="https://img.shields.io/badge/License-GPL%20v3-%23ff554d.svg"></a>
<a href="https://kiwiirc.com/client/irc.badnik.net/?nick=IRC-Source_?#melonds" alt="IRC channel: #melonds"><img src="https://img.shields.io/badge/IRC%20chat-%23melonds-%23dd2e44.svg"></a>
<a href="https://discord.gg/pAMAtExcqV" alt="Discord"><img src="https://img.shields.io/badge/Discord-Kuribo64-7289da?logo=discord&logoColor=white"></a>
<br>
<a href="https://github.com/melonDS-emu/melonDS/actions/workflows/build-windows.yml?query=event%3Apush"><img src="https://github.com/melonDS-emu/melonDS/actions/workflows/build-windows.yml/badge.svg" /></a>
<a href="https://github.com/melonDS-emu/melonDS/actions/workflows/build-ubuntu.yml?query=event%3Apush"><img src="https://github.com/melonDS-emu/melonDS/actions/workflows/build-ubuntu.yml/badge.svg" /></a>
<a href="https://github.com/melonDS-emu/melonDS/actions/workflows/build-macos.yml?query=event%3Apush"><img src="https://github.com/melonDS-emu/melonDS/actions/workflows/build-macos.yml/badge.svg" /></a>
<a href="https://github.com/melonDS-emu/melonDS/actions/workflows/build-bsd.yml?query=event%3Apush"><img src="https://github.com/melonDS-emu/melonDS/actions/workflows/build-bsd.yml/badge.svg" /></a>
</p>
DS emulator, sorta

The goal is to do things right and fast, akin to blargSNES (but hopefully better). But also to, you know, have a fun challenge :)
<hr>

## How to use

Firmware boot (not direct boot) requires a BIOS/firmware dump from an original DS or DS Lite.
DS firmwares dumped from a DSi or 3DS aren't bootable and only contain configuration data, thus they are only suitable when booting games directly.

### Possible firmware sizes

 * 128KB: DSi/3DS DS-mode firmware (reduced size due to lacking bootcode)
 * 256KB: regular DS firmware
 * 512KB: iQue DS firmware

DS BIOS dumps from a DSi or 3DS can be used with no compatibility issues. DSi BIOS dumps (in DSi mode) are not compatible. Or maybe they are. I don't know.

As for the rest, the interface should be pretty straightforward. If you have a question, don't hesitate to ask, though!

## How to build
See [BUILD.md](./BUILD.md) for build instructions.

## TODO LIST

 * better DSi emulation
 * better OpenGL rendering
 * netplay
 * the impossible quest of pixel-perfect 3D graphics
 * support for rendering screens to separate windows
 * emulating some fancy addons
 * other non-core shit (debugger, graphics viewers, etc)

### TODO LIST FOR LATER (low priority)

 * big-endian compatibility (Wii, etc)
 * LCD refresh time (used by some games for blending effects)
 * any feature you can eventually ask for that isn't outright stupid

## Credits

 * Martin for GBAtek, a good piece of documentation
 * Cydrak for the extra 3D GPU research
 * limittox for the icon
 * All of you comrades who have been testing melonDS, reporting issues, suggesting shit, etc

## Licenses

[![GNU GPLv3 Image](https://www.gnu.org/graphics/gplv3-127x51.png)](http://www.gnu.org/licenses/gpl-3.0.en.html)

melonDS is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

### External
* Images used in the Input Config Dialog - see `src/frontend/qt_sdl/InputConfig/resources/LICENSE.md`
