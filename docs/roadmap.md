# Roadmap

An honest account of what stands up and what doesn't. Percentages are a judgement,
not a measurement — the weights behind them are fixed in advance so that a number
moving means work happened, not that the mood changed.

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

**The device and the window aren't joined.** Direct3D can create a device. A frame
can reach the screen. Connecting those two paths so a game's own rendering lands in
its own window is the next milestone that matters.

**Installers and save files are early.** Enough to run things; not enough to trust
with a library.

**Graphics performance is unmeasured.** CPU translation cost is known. What the GPU
path costs under real load is not, and no number will be quoted until it is.

## What comes next

1. Join the rendering device to the on-screen window
2. Drive a real game far enough to see its first frame
3. Measure what that actually costs
4. Widen compatibility from "a game" to "games"

## How progress is judged

Nothing counts as working because it compiled or because a program didn't crash.
Each capability has a test that reads the numbers the software itself reports, and
fails if the capability is removed. There are **415** of those, and they all pass.

Where a number in this document went *down*, it went down because something was
found to be less finished than believed. That's information worth publishing, and
it will keep being published that way.
