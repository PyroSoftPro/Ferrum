# What's behind each number

The chart on the front page isn't a vibe. It now tracks **sixteen** areas — seven were added on 2026-08-12 (DirectX 12, 32-bit games, controllers, shader stutter, cutscene video, older DirectX, and anti-cheat for titles whose publishers already support Linux). Adding them **lowered** the headline number, because they start at zero and the total is fixed at 100. Nothing got worse; we're counting more. Each bar has a fixed weight decided in
advance, and only the completion figure moves — so a bar going up means work
happened, not that we got optimistic. This page says what that work was.

Percentages are still a judgement. What isn't a judgement is the checks: every
capability below has automated tests that fail if the capability disappears.
There are **616** of them and they all pass.

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
**74%**

The flat drawing games use for interface elements — health bars, inventory
screens, subtitles, loading screens — rather than the 3-D world.

Done: copying and stretching images, transparency masks, pattern fills, text,
lines, rectangles, ellipses, rounded rectangles, arcs, multi-colour and
image-based brushes, and the rules for how a drawing operation combines with what
is already on screen. Also the coordinate systems: a program can set up its own
scale and rotation and have the results come out right, including the fiddly case
where two transformations combine and the order matters.

Since added: shapes a program builds up piece by piece before drawing them in one
go, and the whole business of asking for a font by name and measuring how wide
text will be — half of the drawing calls a real program uses are now covered,
up from four in ten.

That work turned up six defects of exactly the kind this page's rule is about.
All six reported success while drawing nothing, so the picture was right whenever
anyone looked at the *finished* result and wrong at every moment in between — the
kind of fault that shows up as flicker rather than as an error. A seventh handed
back an error code where a font was expected, and it passed every check anyone
had thought to write, because the check was "did we get something back".

This is the widest surface in the whole project by raw count, which is why the
number is where it is even now.

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
**97%**

The interface games actually draw with. This bar sat at zero for a long time.

**Real DXVK now creates a Direct3D 11 device at feature level 11_0** — the level
modern games ask for. Verified by using the device rather than by checking a
pointer isn't null: creating a real buffer of a requested size and getting that
exact size back, confirming the device reports itself healthy, and confirming the
error message that had appeared on every single run for months is now gone.

**They have since met.** The rendering device is joined to a real Mac window, a
frame is presented, and geometry shaded by a real compiler comes out the far end
— checked pixel by pixel against what DirectX was asked to draw, not by eye and
not by "it didn't crash". Arbitrary content works, so this is a path rather than
one hard-coded picture.

Not finished: compute shaders and the wider set of pixel formats. The remaining
next milestone that matters.

## Speed
**90%**

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
**78%**

Windows audio reaches your speakers through CoreAudio. A program opens an audio
device, describes the format it wants, and plays — and the output was checked
against the exact signal level it should have produced, not just "sound came out".

Microphone input now works too, against real audio hardware, along with listing
the audio devices on the machine and agreeing a format with them. That last part
caught a genuine defect: a call meant to hand out a unique identifier returned
the *same* one every time, which quietly turned every recording client in the
program into a playback client.

Not finished: MIDI, exclusive-mode output, and the older audio interfaces some
long-lived games still use.

## Installers and saves
**52%**

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

## DirectX 12
**12%**

The interface modern big-budget games use. Nothing here works yet; it's on the
list because it's where the industry is, and because the commercial Mac
alternative's Apple-Silicon build doesn't have it either. Getting there means
routing DirectX 12 through the same Vulkan bridge that already carries DirectX 11
— using VKD3D, the translation layer Steam Deck uses for DX12 games.

We said we'd test that risk early and say so plainly. We did, and the answer is
better than expected: **DirectX 12 now runs far enough through our bridge to ask
the graphics driver for what it needs, and it is held up by exactly one missing
feature** — not the several that a translation layer like ours would normally be
expected to fail. The demanding capabilities everyone assumes are the problem
turned out to be present.

Two limits are permanent and worth stating: Apple's Metal has no equivalent for a
couple of older DirectX 12 features, so those specific effects will never have a
floor here. Everything else is work, not a wall.

## 32-bit games
**40%**

Older Windows games — roughly anything before 2015, plus a great many indies —
are 32-bit. Apple removed 32-bit support from macOS entirely, and Rosetta never
translated 32-bit code, so **those games are currently impossible on a modern Mac
by any route at all**.

Our translator can run 32-bit code natively. Nobody else can offer this, which is
why it's on the list.

32-bit code now executes in genuine 32-bit mode. The obstacle after that was
unusual: macOS reserves the whole first 4 GB of memory for itself, and refuses to
start any program built to give that up. For a 64-bit game that collides with a
single page and is easy to work around. For a 32-bit game, that reserved region
is its *entire* address space — there is nothing to move aside.

So the address space itself is now relocated: the game gets its own full-size
region elsewhere in memory, and every address is adjusted on the way through.
The adjustment is free — the processor does it as part of the instruction that
reads memory — and it also fixes something the old approach got quietly wrong at
the very top of the range.

**That is now connected end to end, and the wall is gone.** A 32-bit program can
do arithmetic on its own pointers — stepping through an array, reaching a field
inside a structure, the most ordinary thing code does — and read what it points
at. Until now that was the exact operation that failed, because a 32-bit
addition cannot carry the extra bits a large address needs.

The test insists on the pointer itself, not just the values fetched, since the
right values with a wrong pointer would mean the reads happened to work. And it
is checked both ways: with the relocation switched off, the old failure is still
precisely where it was, so the result measures the fix rather than a test that
got easier.

Two faults were found on the way, both of the kind this page's rule is about.
Some memory instructions take their address in a form that cannot carry the
adjustment, and they had been overlooked because they *also* use a form that
can. And the part that decides whether a piece of memory holds runnable code was
asking about the wrong address, so every page looked unrunnable and programs
stopped after a single instruction.

Not finished: the loader does not yet place a real 32-bit program into that
region, so no 32-bit game has run end to end. That is loading work now, not an
address-space question.

## Controllers
**70%**

Gamepad support: the Windows controller interfaces (XInput and DirectInput)
connected to macOS's own controller framework, so a pad you've paired with your
Mac just works.

Not a headline feature — a requirement. A large share of games are unplayable
without one.

**A game can now read your gamepad**: sticks, triggers and the button mask all
arrive, and the counter a game uses to tell "nothing changed" from "I read a
stale value" moves correctly with the state. It works under all three names
Windows has shipped this interface as, which matters more than it sounds —
a game asking for the older name would otherwise fail to start at all.

This one needed care because of the rule at the top of this page: reporting
"success" alongside an all-zero controller state is indistinguishable from a pad
that is simply sitting still, so the tests insist on values that actually move.

Not finished: rumble and force feedback, the older DirectInput interface, and
breadth around pads connecting and disconnecting mid-game.

## Shader stutter
**0%** — just added

The first time a game shows you a new effect, the graphics translation has to
build a shader for it — and the game hitches while that happens. It's the single
most-complained-about artifact of this kind of translation.

The fix is to remember that work between runs, so the second launch is smooth and
eventually so is the first. Nothing about it is glamorous; players notice it
immediately.

## Cutscene video
**35%**

Games play video — intros, cutscenes, background footage — through a Windows
media system that used to be entirely missing here. When a game hit one, it hung
or closed, which looks exactly like a crash and isn't.

A program can now open a video file and get **real decoded frames** back. Each
frame is checked to be genuinely different from the last, because a decoder that
hands back the same buffer over and over, or a black one, would otherwise pass a
test that only asks "did I get a frame?". That check also caught a colour error
that was subtle enough to look right: the picture was recognisable, and the
greens were wrong.

Not finished: the decoder isn't yet connected to the route a game actually asks
through, so a game still gets a clear refusal rather than a picture.

## Older DirectX versions
**0%** — just added

DirectX 9 and earlier, for the back catalogue. Cheaper to reach than DirectX 12,
because the translation layer we already use ships a DirectX 9 implementation —
it mostly needs connecting to the bridge that's already built.

## Anti-cheat
**0%** — just added, deliberately narrow

Some multiplayer games refuse to run unless their anti-cheat is satisfied. There
are two kinds, and only one is reachable.

**Kernel-level anti-cheat cannot work** — not here, not under any translation
layer, on any platform. It needs to load a driver into the operating system
itself, which is exactly what a translation layer doesn't provide. We won't
pretend otherwise.

**The other kind can.** Easy Anti-Cheat and BattlEye both shipped a version that
runs without a kernel driver, because Steam Deck needed it — and publishers opt
in per game. Our rule is simple and fixed: **we support the games whose
publishers already turned that on, and we don't chase the ones who haven't.**
Pursuing a title whose publisher hasn't opted in would mean trying to defeat an
anti-cheat, which we won't do.

One honest caveat: that opt-in shipped as Linux software, and there's no Mac
equivalent today. Whether it can be made to work here is an open question we'll
answer with evidence, not optimism — and if the answer is no, we'll say so.

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
