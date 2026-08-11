# Questions people actually ask

## Will my games work?

Not yet — not any of them, end to end. Ferrum can create a Direct3D 11 device and
put a rendered frame on a real Mac window, which are the two hardest structural
pieces, but no full game has been driven from launch to gameplay.

When that changes it'll be said plainly here, with what was tested and what wasn't.
A compatibility list that promises more than it delivers is worse than no list.

## Do I need a Windows licence?

No. Ferrum never runs Windows. The Windows interfaces are provided by
[Wine](https://www.winehq.org/), which reimplements them from scratch without using
any Microsoft code. You need the games you own and nothing else.

## Is this legal?

Yes. Wine has done this in the open for thirty years, and the approach — writing a
compatible implementation rather than copying one — is well-established. Ferrum
adds a translation layer on top of that; it doesn't change the legal shape of it.

## Isn't this what Rosetta does?

Rosetta translates Intel *Mac* apps to Apple Silicon. It does not know anything
about Windows. Most Windows-on-Mac setups today use Rosetta as one layer in a taller
stack — which is the problem, because Apple is retiring it.

Ferrum replaces that layer with one we control, so the stack keeps standing.

## How is this different from Whisky, CrossOver, or Game Porting Toolkit?

They're solving overlapping problems and they're good at it. The distinction is what
happens underneath: several existing setups lean on Rosetta to run the Intel parts.
Ferrum's reason for existing is to not need it.

Ferrum is also just the engine, not an app. The app is
[Bourbon](https://mikethetech.itch.io/bourbon26).

## How fast is it?

Translated code costs about **3.2× the CPU time** of code written for Apple Silicon
directly. That sounds worse than it is: a game only runs out of budget at 60 frames
per second if the same work already cost roughly 5 milliseconds per frame natively.
For most games there's comfortable headroom.

That figure was measured on workloads shaped like a real frame — lots of small
memory updates, lots of indirect calls — not on a benchmark chosen to flatter.

The graphics side hasn't been measured under real load yet, and it would be
dishonest to quote a number for it.

## When will it be ready?

No date. Ferrum is roughly **two-thirds of the way** to running a game, and the
remaining third contains the parts most likely to surprise us.

Rosetta's retirement sets the pace, not a marketing calendar.

## Will it be free? Can I see the source?

Ferrum ships as part of [Bourbon](https://mikethetech.itch.io/bourbon26) — if you
have Bourbon, you have Ferrum, with nothing extra to sign.

The source is closed. Commercial licensing, including source access, is available
separately — see the [licence](../LICENSE) or write to **contact@mikethetech.com**.

## What about anti-cheat?

Kernel-level anti-cheat does not work under any translation layer, on any platform,
and won't work here. That's a decision made by the anti-cheat vendors, not a
technical gap on our side.

## Which Macs?

Apple Silicon only — M-series. Intel Macs don't need it.

## Is any of this built on other people's work?

Enormously so, and all of it credited: [FEX-Emu](https://github.com/FEX-Emu/FEX) for
translation, [Wine](https://www.winehq.org/) for the Windows interfaces,
[DXVK](https://github.com/doitsujin/dxvk) and
[MoltenVK](https://github.com/KhronosGroup/MoltenVK) for graphics. They keep their
own licences, and the projects stay open for everyone. See the
[licence](../LICENSE).
