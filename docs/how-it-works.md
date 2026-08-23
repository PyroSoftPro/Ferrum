# How Ferrum works

*Plain English. No code, no internals — just the shape of the thing.*

## The problem in one paragraph

A Windows game is a box of instructions written for an Intel processor, asking
Windows for things: give me a window, play this sound, draw these triangles. Your
Mac has neither an Intel processor nor Windows. Every one of those assumptions has
to be answered by something.

## Four translations, not one

People say "emulator" and picture one program pretending to be a computer. What's
actually happening is four different translations stacked on top of each other, and
they're independent problems.

**1. The instructions.** The game's code speaks x86-64, the language Intel chips
understand. Apple Silicon speaks ARM64. Ferrum reads the game's instructions and
writes equivalent ARM64 instructions as it goes — not ahead of time, but in the
moment, keeping the translations around so a loop isn't retranslated a thousand
times. This is the part Ferrum inherits from FEX and takes to macOS.

**2. The requests to Windows.** When the game asks to open a file or create a
window, something has to answer as Windows would. That's [Wine](https://www.winehq.org/),
a 30-year-old open-source project that reimplements the Windows interfaces from
scratch. No Microsoft code, no Windows licence, no piracy — Wine is the reason any
of this is legal and the reason it works at all.

**3. The graphics.** Games talk several generations of DirectX, OpenGL or Vulkan;
Macs speak Metal. Ferrum routes each API through the best maintained path for that
workload. That can mean a direct Metal translator, or open layers such as
[DXVK](https://github.com/doitsujin/dxvk), WineD3D,
[VKD3D](https://gitlab.winehq.org/wine/vkd3d) and
[MoltenVK](https://github.com/KhronosGroup/MoltenVK). An authorized optional
component can be managed by Ferrum too, but it cannot replace the offline open
baseline. Ferrum's job is to carry every conversation across the boundary intact
and select the route without making the user assemble it.

**4. The window itself.** A game expects a Windows window it can draw into. It gets
a real macOS window instead, and never notices.

## Why this is hard

The hard part isn't any single translation. It's that a game assumes all four are
consistent with each other, at speed, forever.

When a game asks the clock what time it is — thousands of times a second, to decide
how far to move a character — the answer has to be right, in the right units, and
cheap. When it hands over a block of memory to draw from, the graphics card has to
see exactly the bytes the game wrote. When it asks whether the machine supports a
graphics feature, the honest answer matters, because it will make different choices
based on what you say.

A translation layer doesn't fail loudly. It fails by being *subtly wrong* in a way
the game happily builds on until something looks broken three steps later.

## How we know it works

Every capability Ferrum claims has a test that fails if the capability goes away.
Not "the program didn't crash" — a test that reads the numbers the game itself
reports and checks they're right.

That distinction is the whole discipline. Software that isn't implemented yet tends
to return something that *looks* like success: a zero that could be "no error" or
could be uninitialised memory, a "yes" that's really a shrug. Several times during
Ferrum's development a feature appeared to work and didn't, and was caught only
because a test insisted on the actual measured value rather than the absence of a
crash.

There are currently **1,403** such checks. They all pass.

## What Ferrum is not

- **Not a virtual machine.** Nothing boots Windows. There's no second operating
  system running, and no gigabytes of disk set aside for one.
- **Not Rosetta.** Rosetta is Apple's own translator, and it's being retired.
  Ferrum exists so that its retirement isn't the end of Windows gaming on a Mac.
- **Not a Windows licence.** You don't need one, and you never will.
- **Not finished.** See [the roadmap](roadmap.md) for an honest account.

## Where it runs

Apple Silicon Macs — M-series chips. Ferrum is built for ARM64 macOS specifically
and does not run on Intel Macs, which don't need it anyway.
