<p align="center">
  <img src="docs/assets/ferrum-logo.png" alt="Ferrum — Fe with subscript x, element 26" width="260">
</p>

<h1 align="center">Ferrum</h1>
<p align="center"><strong>FEX on Apple Silicon.</strong><br>
A native macOS x86-64 translation layer — no Rosetta, no virtual machine, no Windows licence.</p>

---

## The problem

Apple is retiring **Rosetta**, the piece of macOS that lets Intel code run on Apple
Silicon. Almost every "Windows games on a Mac" setup quietly depends on it. When it
goes, those setups stop working.

Ferrum is the replacement path. It translates x86-64 Windows code to ARM64 directly,
sits underneath [Wine](https://www.winehq.org/) for the Windows API, and hands
graphics to [DXVK](https://github.com/doitsujin/dxvk) →
[MoltenVK](https://github.com/KhronosGroup/MoltenVK) → Metal. Your games keep
working, on hardware that never had an Intel chip in it.

## Why "Ferrum"?

**Ferrum** is Latin for [iron](https://en.wikipedia.org/wiki/Iron), whose symbol is
**Fe**. Add the subscript **x** and you get [**FEX**](https://github.com/FEX-Emu/FEX),
the translation layer underneath. In a chemical formula that subscript counts
**atoms**, so Feₓ reads as *an unspecified quantity of iron* — which is about right
for something that translates an unknown amount of somebody else's code.

Iron carries **26 protons** — and
[**Proton**](https://github.com/ValveSoftware/Proton) is the Wine-and-DXVK stack that
made the Steam Deck work, same job on a different architecture. Element **26**, built
in 2026, shipping in Bourbon 26.

This corner of software also runs on drinks — [Wine](https://www.winehq.org/),
[Whisky](https://getwhisky.app), Bourbon. Fer**rum** has one hiding in it too.

## Status

<!--progress-->
<p align="center">
  <img src="docs/assets/progress.svg" alt="Ferrum progress — 68% toward playable gameplay, 79% of the substrate" width="100%">
</p>

**≈68% toward playable gameplay · ≈79% of the substrate · 415 checks passing · milestone m162**<!--/progress-->

Real DXVK runs on this port and creates an **`ID3D11Device` at feature level 11_0**.
A **GPU-rendered frame reaches a real macOS window** through a genuine
`VK_KHR_swapchain`. Audio plays through CoreAudio. Translated code costs about
**3.2× native** on the CPU — comfortable headroom inside a 60 fps frame.

Not done: a full game has not been driven end to end yet.

Ferrum's source is closed. This repository carries **releases and documentation
only** — no source code. Development happens in a private repository.

## Bourbon 26

Ferrum is the engine. **Bourbon** is the app it ships in — a native macOS front end
that installs and drives the whole Windows-gaming stack so you don't have to.

- 🥃 **[Bourbon 26 on itch.io](https://mikethetech.itch.io/bourbon26)** — the app itself
- 📦 **[Bourbon-OSS](https://github.com/PyroSoftPro/Bourbon-OSS)** — open-source
  compatibility data, the DXVK patch, the Steam CEF shim, probes and community
  refresh tools

## Built on

| | |
|---|---|
| [FEX-Emu](https://github.com/FEX-Emu/FEX) | x86/x86-64 → ARM64 translation (MIT) |
| [Wine](https://www.winehq.org/) | the Windows API (LGPL) |
| [DXVK](https://github.com/doitsujin/dxvk) | Direct3D → Vulkan |
| [MoltenVK](https://github.com/KhronosGroup/MoltenVK) | Vulkan → Metal |

## Licensing

Ferrum is **proprietary, closed-source software**. The macOS port — the driver in
`Source/DarwinDriver/` and the work around it — is not open source, and no
open-source license is granted by its presence here. See
[`LICENSE`](LICENSE).

You can license it two ways:

- **As part of [Bourbon 26](https://mikethetech.itch.io/bourbon26)** — end users get
  Ferrum as an integrated component under Bourbon's own terms. Nothing extra to sign.
- **Commercially** — for use outside Bourbon, redistribution, embedding, or source
  access. Reach out at **contact@mikethetech.com**.

The code this is built on keeps its own licenses and always will: FEX-Emu stays
**MIT** for everyone (© 2019 Ryan Houdek and contributors),
Wine stays **LGPL** and is loaded at runtime rather than modified, DXVK is **zlib**
and MoltenVK is **Apache-2.0**. Upstream remains
[FEX-Emu/FEX](https://github.com/FEX-Emu/FEX) and is never pushed to.

---

<p align="center">
  <img src="docs/assets/ferrum-mark.png" alt="Ferrum mark" width="25%">
</p>
