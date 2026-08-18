# What's behind each number

The chart on the front page isn't a vibe. It now tracks **fifteen** areas — six were added on 2026-08-12 (DirectX 12, 32-bit games, controllers, shader stutter, cutscene video, older DirectX). Anti-cheat was briefly on the list and was removed on 2026-08-13: it turned out to depend on files only the anti-cheat vendor can publish, and they have not published them, so there is no engineering work to schedule. Adding them **lowered** the headline number, because they start at zero and the total is fixed at 100. Nothing got worse; we're counting more. Each bar has a fixed weight decided in
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
**98%**

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

**This number went down, from 99%, and that is the honest result of finding
something.** Windows lets a library reserve a private slot for per-thread data.
Two separate pieces of this system were handing out those slot numbers from their
own counters, neither aware of the other — so both could give out the same one.
That is exactly what happens when Hollow Knight loads the runtime its game code
is written in: it is handed a slot another component already owns, and reads the
wrong memory. What it finds there happens to look like "this thread is already
set up", so the real setup is skipped, and it stops a moment later insisting that
the thread it is running on does not exist.

That was proven rather than guessed — down to reading the exact instruction the
program did not take, and to a number in the diagnostic that could only have come
from the runtime's own file on disk.

**That is now fixed, and the number has come back up.** The repair went in and the
collision is gone — the game no longer reads the wrong memory, and the assertion
that had stopped it for dozens of milestones is absent from every run. Along the
way the earlier plan for "the second part" turned out not to exist: the real
system never used the mechanism it assumed, so reading the actual bytes replaced a
guess with a fact. With this cleared, Hollow Knight now runs all the way through
its engine startup — graphics device, physics, and input all come up — and stops
later, on a different thing (see *Sound*). The number is 98 rather than 100
because a few rarely-used facilities are still refused and the very last stretch of
startup isn't finished.

## Game windows and input
**90%**

A Windows program asks for a window and gets a real macOS one. It receives mouse
and keyboard events through the message loop games actually use, and can move,
resize, and repaint.

Not finished: mouse capture, the thing that keeps the cursor locked to the window
while you look around in a first-person game, and international text input.

## 2-D drawing for menus and HUDs
**80%**

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

The newest piece is the measuring a menu does before it draws: how wide a letter
is, how tall a line is, how much room a string needs. Four of those answers were
being given in screen pixels when the program had asked for its own units — which
is invisible in the ordinary case, where the two happen to be the same, and wrong
by exactly the scaling factor the moment a game scales its interface. Eleven more
questions of that kind, previously all refused, are now answered.

One of these deserves singling out, because the test for it had to be built
against a specific trap. When the driver refuses a request, the refusal is a
non-zero value — and non-zero is also how Windows says "yes, that worked". So a
program spacing out text got a confident yes and no spacing. The test therefore
checks that the ink on the screen actually moved, not that the call returned
something.

This is the widest surface in the whole project by raw count, which is why the
number is where it is even now.

## Drawing to the screen
**93%**

Getting a finished image onto a real Mac window.

The milestone here: **a GPU-rendered frame reaches a real macOS window** through
the same presentation path a game uses. That was verified from outside the
process — two frames drawn in different colours, both read back from the GPU's
own texture and confirmed to have swapped, and the macOS window server asked
separately whether the window was actually on screen.

And now the proof that matters most: **a real, modern commercial game presents
its own frames through this path.** Where every earlier measurement showed a
game acquiring and presenting zero swapchain images, the game now acquires and
presents hundreds — its menu is on screen, drawn by its own engine, with no
Rosetta anywhere in the process.

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
**98%**

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

And a real game has now drawn through it. A modern commercial title renders its
menu — thousands of draw calls per run across nearly two hundred render passes —
once six draw commands it relied on (indexed drawing chief among them) were
implemented. That was confirmed from a screenshot of the running game, not from a
return code.

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
**60%**

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

Two things landed since. Renaming a file now works, which sounds minor until you
know that it is how nearly every game saves: write the new save under a temporary
name, then rename it over the old one, so a crash mid-write cannot destroy a
save. Without it that last step silently failed.

And the settings store and the file system now report, at the end of a run,
exactly what a program asked for and did not get — naming the full thing
requested and which part of the program asked. Every other area here already did
that; these two were the last blind spots.

Reviewing that work turned up two faults in a path nothing tests: a relative
rename could reach one directory further than it was allowed to, and any filename
using characters outside the basic Latin set was being mangled.

This is still one of the least finished areas and that's deliberate. It's breadth
work, it's well understood, and it doesn't block the graphics milestones ahead of
it. No real installer has been run.

## DirectX 12
**35%**

The interface modern big-budget games use. It's on the
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

Since then the list of missing pieces was finished — it was a short, named list
rather than an open question — and the result is the milestone this section was
waiting for: **a DirectX 12 device now exists.** Every previous measurement,
including the ones taken after we'd patched the translation layer, came back with
no device at all. A game asking for one now gets one.

That is the gateway, not the picture. The device comes back at the older of the
two capability levels a game can ask for; the newer one still refuses. Nothing
has been drawn through it yet, and neither the screen path nor command execution
has been proven.

A wrinkle worth recording, because it nearly hid the result: four of the five
tests written to confirm this were checking for the *absence* of a function's
name in the program's output — but the system ends every run by listing the
functions it successfully used. So those tests failed precisely because the work
succeeded. They now check the one place a name means "refused".

Two limits are permanent and worth stating: Apple's Metal has no equivalent for a
couple of older DirectX 12 features, so those specific effects will never have a
floor here. Everything else is work, not a wall.

## 32-bit games
**70%**

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

**A real 32-bit program now loads and runs.** It gets placed in that relocated
region, its internal addresses are corrected for where it actually landed, and
it runs to a known answer. Five separate places were found where the translator
handed back an address in the *host's* numbering instead of the program's —
two of them with no symptom the program itself could ever have noticed, because
the lower half of the two numbers happens to look identical.

Also fixed: the 64-bit atomic compare-and-swap that older 32-bit code uses for
thread safety. It had been left deliberately failing rather than guessing.

That obstacle is now solved. A 32-bit program can call into Windows' own
libraries. The problem was real and specific: the calling convention needs to
know how many bytes of arguments to discard, and that number is nowhere in the
file — the name a program asks for has been stripped of it. So it is declared in
a table, and because a table is exactly the sort of thing that can be quietly
wrong, every entry is checked three independent ways: against the compiler
toolchain's own libraries, against the stack the running program actually leaves
behind, and against whether the answers come out right.

Not finished: only six functions are declared, so a real game runs into an
unknown-name refusal almost immediately. Several pieces a Windows program expects
to exist are still absent, and no real 32-bit game has run.

## Controllers
**80%**

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

**DirectInput is in.** That is the older of the two ways Windows games talk to a
pad, and most games from before about 2010 use nothing else — they previously had
no route to a controller at all.

Two faults were caught on the way, both of the "looks like success" kind this
page is about. A pad that had just been unplugged reported *gone* through one
call and *still here* through another. And one axis was being read from a slot
nothing ever writes: in a range where the middle means centred, it read as a
stick pushed fully to one side rather than at rest.

Rumble also stopped correctly for the first time — a game that lost focus used to
leave the pad buzzing on the desk indefinitely.

⚠️ **Stated plainly: none of this has been tested on a physical controller**,
because there isn't one on the development machine. Everything is driven by a
simulated pad, which exercises every layer except the few lines that read the
real hardware. That is why this sits at 80 and not higher.

## Shader stutter
**55%**

The first time a game shows you a new effect, the graphics translation has to
build a shader for it — and the game hitches while that happens. It's the single
most-complained-about artifact of this kind of translation.

The fix is to remember that work between runs, so the second launch is smooth and
eventually so is the first. Nothing about it is glamorous; players notice it
immediately.

**A shader compiled once is no longer recompiled on the next run.** Measured:
the same work drops from 18.7 ms to 4.3 ms on a second launch, and with the
saving switched off it stays flat — which is what proves the saving is what did
it, rather than the machine simply being warmed up.

An honest complication worth stating, because it changes what to expect: **macOS
already keeps a shader cache of its own.** The first measurement here looked like
a 120x improvement, and a control run with our saving completely disabled still
showed a large speedup — that part was Apple's, not ours. What we add sits on top
of it.

The saving now also covers the other kind of shader — the sort games use for
physics, lighting and post-processing rather than for drawing shapes. That turned
out to matter for a reason beyond speed: those shaders were not being built at
all. A game asking for one got nothing back, or got something that ran nothing,
and neither reported a problem. Repeat runs now do about a fifth of the work,
which closely matches what the drawing shaders already showed.

Not finished: this does not help the *first* run, which is where most of the
stutter a player notices comes from. That half belongs to Apple's cache. One
timing claim in this area still has no way to reproduce it, and is marked
unverified rather than quoted.

## Cutscene video
**70%**

Games play video — intros, cutscenes, background footage — through a Windows
media system that used to be entirely missing here. When a game hit one, it hung
or closed, which looks exactly like a crash and isn't.

A program can now open a video file and get **real decoded frames** back. Each
frame is checked to be genuinely different from the last, because a decoder that
hands back the same buffer over and over, or a black one, would otherwise pass a
test that only asks "did I get a frame?". That check also caught a colour error
that was subtle enough to look right: the picture was recognisable, and the
greens were wrong.

That last gap is now closed: the decoder is connected to the route a game
actually asks through, and a program reads frames the ordinary way rather than
getting a refusal. Getting there meant finding a second closed door — even once
the system agreed it could handle the file, it still handed the work to a
component this Mac cannot run, unless one specific setting says otherwise.

Setting that one value turned out to cost every program on the machine a little
work, including ones that never play video at all. A test that counts exactly
how much work a simple program does noticed, which is what that test is for. It
is now done only when a video is actually opened.

Not finished: only one video format route is wired, and no game has yet played a
cutscene from start to finish.

## Older DirectX versions
**35%**

DirectX 9 and earlier, for the back catalogue. Cheaper to reach than DirectX 12,
because the translation layer we already use ships a DirectX 9 implementation —
it mostly needs connecting to the bridge that's already built.

**A DirectX 9 program now puts a frame on screen.** The colours it asked for are
read back from the display and checked value by value.

Two things stood in the way, and neither was what you'd guess. DirectX 9 asks the
system to list *monitors* rather than graphics cards, and that particular question
had never been answered here — so the program was told there were no display
devices at all and gave up. And the graphics layer was asking Apple's driver for
a capability Metal simply does not have, which made every attempt to open a device
fail. It now asks only for what the machine reports.

Not finished: nothing is *drawn* yet — the frame is cleared and presented, with no
shapes or textures. DirectX 8 and DirectDraw are untouched.

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
