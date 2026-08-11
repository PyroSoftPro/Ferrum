# What's behind each number

The chart on the front page isn't a vibe. Each bar has a fixed weight decided in
advance, and only the completion figure moves — so a bar going up means work
happened, not that we got optimistic. This page says what that work was.

Percentages are still a judgement. What isn't a judgement is the checks: every
capability below has automated tests that fail if the capability disappears.
There are **437** of them and they all pass.

One rule shapes everything here. Software that isn't finished tends to return
something that *looks* like success — a zero that could mean "no error" or could
be uninitialised memory, a "yes" that's really a shrug. So nothing counts as done
because a program didn't crash. It counts when a test reads the actual number the
software reports and insists it's the right one. Several things on this page
looked finished and weren't, and were caught exactly that way.

---

## Windows programs run
**99%**

The foundation: a real Windows program file gets loaded into memory, its
dependencies resolved, and its code executed.

Everything a normal program assumes now works — the Microsoft C runtime, error
handling, threads, locks, and the machinery that lets one library call into
another. Programs also *stop* properly, which sounds trivial and isn't: for a
while a program that finished simply spun forever instead of exiting, which made
every failure look like a hang.

Loading additional libraries at runtime — the thing every game does to reach
graphics and audio — works, and the loaded code runs.

Not 100% because a couple of rarely-used facilities are still refused. Nothing
seen so far needs them.

## Game windows and input
**90%**

A Windows program asks for a window and gets a real macOS one. It receives mouse
and keyboard events through the message loop games actually use, and can move,
resize, and repaint.

Not finished: mouse capture, the thing that keeps the cursor locked to the window
while you look around in a first-person game, and international text input.

## 2-D drawing for menus and HUDs
**64%**

The flat drawing games use for interface elements — health bars, inventory
screens, subtitles, loading screens — rather than the 3-D world.

Done: copying and stretching images, transparency masks, pattern fills, text,
lines, rectangles, ellipses, rounded rectangles, arcs, multi-colour and
image-based brushes, and the rules for how a drawing operation combines with what
is already on screen. Also the coordinate systems: a program can set up its own
scale and rotation and have the results come out right, including the fiddly case
where two transformations combine and the order matters.

This is the widest surface in the whole project by raw count, which is why the
number is where it is. Roughly a quarter of it is covered, but the specific
operations games actually present frames with are the ones that are done.

## Drawing to the screen
**88%**

Getting a finished image onto a real Mac window.

The milestone here: **a GPU-rendered frame reaches a real macOS window** through
the same presentation path a game uses. That was verified from outside the
process — two frames drawn in different colours, both read back from the GPU's
own texture and confirmed to have swapped, and the macOS window server asked
separately whether the window was actually on screen.

This number went *down* once. A merge revealed that a whole class of drawing
surface was being written to and then silently never displayed. Everything looked
green; nothing was visible. Fixing that, and then connecting a real GPU rendering
path, is what brought it back up and past where it started.

Not finished: resizing, multiple windows, and syncing to your display's refresh
rate.

## The Vulkan graphics bridge
**92%**

Vulkan is the modern graphics interface DXVK speaks. Apple doesn't support it
directly; MoltenVK translates it to Metal. This bar is the plumbing that carries
those conversations across the boundary between the Windows side and the Mac side
without anything being lost or quietly corrupted.

Real DXVK — not a test harness — drives this bridge. It finds your GPU by name
("Apple M5"), reads its real capabilities, creates a graphics device and queues,
allocates hundreds of megabytes of GPU memory, and builds the objects it needs to
record work. Memory allocation, images, buffers, render passes, samplers and
command buffers all work, with every object it creates accounted for on the way
back out.

A specific proof point: the capabilities DXVK reads back are *mixed* — some
supported, some not. If the bridge were subtly broken it would have returned all
zeros, which is indistinguishable from "your GPU supports nothing" and would have
looked plausible for weeks.

## DirectX 11
**68%**

The interface games actually draw with. This bar sat at zero for a long time.

**Real DXVK now creates a Direct3D 11 device at feature level 11_0** — the level
modern games ask for. Verified by using the device rather than by checking a
pointer isn't null: creating a real buffer of a requested size and getting that
exact size back, confirming the device reports itself healthy, and confirming the
error message that had appeared on every single run for months is now gone.

Not finished, and this is the honest gap: **nothing has been drawn yet**, and the
rendering device is not yet joined to the on-screen window from the bar above.
Those are two separate achievements that have to meet. Making them meet is the
next milestone that matters.

## Speed
**82%**

Translated code costs about **3.2× the CPU time** of code compiled for Apple
Silicon directly, measured on work shaped like a real game frame — many small
memory updates and indirect calls — not on a benchmark picked to flatter.

In practice that means a game only runs out of CPU budget at 60 frames per second
if its rendering thread already cost around 5 milliseconds per frame natively.
Most games have room.

Three things that were expected to be expensive were measured and turned out not
to be. One thing is expensive: keeping memory updates in the strict order Intel
chips guarantee and ARM chips don't. That single requirement accounts for about
**60% on top** of the 3.2×, and it's the largest remaining opportunity — Apple
Silicon has hardware that can do it directly, which is what Rosetta uses.

Graphics performance under real load is **not measured**, and no number will be
quoted for it until it is.

## Sound
**55%**

Windows audio reaches your speakers through CoreAudio. A program opens an audio
device, describes the format it wants, and plays — and the output was checked
against the exact signal level it should have produced, not just "sound came out".

Not finished: microphone input, MIDI, and the older audio interfaces some
long-lived games still use.

## Installers and saves
**38%**

The unglamorous surface: starting other programs, the Windows registry, and file
system breadth. Games need this to install, to find their settings, and to write
saves.

Recent work closed every outstanding gap that had been blocking real programs,
including the clock games read thousands of times a second to decide how far to
move a character. That one is a good illustration of the rule at the top of this
page: it *reported success* while handing back meaningless numbers, because the
result of the failed call was never actually checked.

Also fixed: the naming system that lets two parts of a program agree they're
talking about the same thing. Without it, two components asking for the same
shared resource would each get their own — while everything visible reported that
it worked.

This is the least finished area and it's deliberate. It's breadth work, it's well
understood, and it doesn't block the graphics milestones ahead of it.

---

## How the two headline numbers differ

**Toward playable gameplay** weights each area by how much a real game depends on
it, then applies a discount for the fact that no frame has been drawn by a game
yet. It's the honest answer to "can I play something".

**Of the substrate** measures only the foundation — programs running, windows,
2-D drawing, speed, and system breadth — leaving out the 3-D graphics chain. It's
the answer to "is the ground solid".

The substrate is further along than gameplay. That's expected: the foundation had
to come first.

## When a number goes down

It has, and it will again. Twice a bar dropped because something believed
finished turned out to be quietly broken — once when drawing surfaces were being
filled and never shown, once when a whole area was re-scored honestly after a
closer look.

A drop isn't a setback being confessed, it's the measurement working. A progress
chart that only ever goes up isn't measuring anything.
