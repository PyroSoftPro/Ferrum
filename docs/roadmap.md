# Roadmap

An honest account of what stands up and what doesn't. Percentages are a judgement,
not a measurement — the weights behind them are fixed in advance so that a number
moving means work happened, not that the mood changed.

## Direction update — August 2026

The product target is broad Windows x86/x64 application and game compatibility
on Apple Silicon in one self-managing Ferrum package. That changes the shape of
the remaining work:

- **Native host, translated guest.** Wine host services stay native ARM64. FEX
  translates the Windows x86/x64 code at the guest boundary instead of carrying
  an entire translated host stack.
- **Pinned release baseline, separate next-generation lane.** The first product
  runtime stays on the exact WineCX 10 ABI already exercised by Ferrum's tests.
  Wine 11, ARM64EC, its newer WoW64 design and page-size improvements advance in
  a separate lane until they pass the same gates.
- **A graphics router, not one mandatory backend.** Direct3D 10/11 gets a direct
  Metal performance lane. DXVK, WineD3D, MoltenVK and VKD3D provide maintained
  open compatibility paths for Vulkan and Direct3D generations. D3DMetal can be
  a Ferrum-managed option when authorized, never a silently assumed dependency.
- **Offline base, managed enhancements.** A fresh package works from its bundled
  open baseline. Verified optional components can be downloaded, cached,
  activated atomically, rolled back, and reused offline without asking the user
  to install development tools or edit paths.
- **Hard runtime isolation.** Ferrum owns its runtime, prefixes, component store,
  caches and user data. It does not inspect or reuse Bourbon, Homebrew, MacPorts,
  another Wine installation, or arbitrary user symlinks.

These are architecture decisions, not newly completed compatibility claims.
Promotion still requires exact functional proof on the local development Mac
and the remote M2, followed by release-package verification.

### Current implementation order

1. Finish real process creation, waits, handle duplication and cross-process
   object semantics without races or host-path escapes.
2. Build the pinned Wine runtime, complete x64/i386 PE trees, Wineboot, NLS,
   fonts, registry hives, immutable base prefixes and transactional per-title
   copies.
3. Land the graphics router and measure direct-Metal, Vulkan and D3D12 routes
   separately, including cold and warm JIT/shader-cache behavior.
4. Run class-balanced Windows-versus-Ferrum compatibility proofs for installers,
   COM, .NET/Mono, media, networking, input, DirectX generations and Vulkan.
5. Sign, notarize and reproduce the complete package on both Macs, including
   fail-closed component download, rollback and offline tests.

Kernel drivers, vendor anti-cheat and protected DRM remain vendor-controlled
boundaries. They will not be represented as solved without authorization and
physical evidence.

## Working

**Windows programs run.** Real Windows executables load and run: the loader, the
Microsoft C runtime, exception handling, threads, synchronisation. Programs start,
do work, and exit cleanly.

**Graphics reach the GPU.** Real DXVK — the same translation layer the Steam Deck
uses — runs on Ferrum, finds your GPU by name, allocates memory on it, and creates
a **Direct3D 11 device at feature level 11_0**, the level modern games ask for.

**A frame reaches the screen.** A GPU-rendered image arrives in a real macOS window
through the same presentation path a game uses. Verified from outside the process,
by asking the window server what it can see.

**Sound plays.** Windows audio arrives at your speakers through CoreAudio.

**Menus and HUDs draw.** The 2-D drawing games use for interface elements — text,
shapes, patterns, blits — largely works.

**Speed is known.** About 3.2× native CPU cost, with the dominant remaining cost
identified and understood.

## Not working yet

**No game has been run end to end.** This is the honest headline. The pieces exist;
they have not yet been driven by a real game from launch to gameplay.

**General multiprocess Windows behavior isn't complete.** A supervised guest can
launch another guest in development builds, but handles, inherited objects,
services, IPC and full wineserver-compatible semantics still need promotion gates.

**The product runtime isn't self-contained yet.** Prefix materialization and
isolation foundations exist, but the complete pinned Wine payload, Wineboot data,
component store and signed package are still being assembled.

**Installers and save files are early.** Enough to run things; not enough to trust
with a library.

**Graphics performance is unmeasured.** CPU translation cost is known. What the GPU
path costs under real load is not, and no number will be quoted until it is.

## What comes next

The implementation sequence is listed under
[Current implementation order](#current-implementation-order). The next public
claim will follow whichever of those capabilities passes independent local and
remote-M2 verification first; research decisions alone do not move the progress
bars.

## How progress is judged

Nothing counts as working because it compiled or because a program didn't crash.
Each capability has a test that reads the numbers the software itself reports, and
fails if the capability is removed. There are **1,403** of those, and they all pass.

Where a number in this document went *down*, it went down because something was
found to be less finished than believed. That's information worth publishing, and
it will keep being published that way.
