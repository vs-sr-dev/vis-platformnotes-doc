# vis-platformnotes-doc

**The canonical Tandy / Memorex VIS platform checklist**, carried from one
documentation pipeline to the next and added to by each.

→ **[vis-platform-notes.md](vis-platform-notes.md)**

Each VIS title I document will produce two things: a repository about that disc,
and whatever it taught me about the *format*. The second kind of finding does not
belong to any one title, and keeping a copy of it in every pipeline is a recipe
for three copies that disagree. This repository is the single copy. Pipelines
link here rather than fork it.

## Where this one starts, which is not where the others did

**The machine is partly known and its discs are not.** That is an unusual place
to open a platform checklist from, and it is why this document uses three marks
rather than one.

Three years of homebrew on this console — an [OPL3
synthesizer](https://github.com/vs-sr-dev/vis-synth), a [media
browser](https://github.com/vs-sr-dev/vis-fileviewer) and a native
[*Wolfenstein 3D* port](https://github.com/vs-sr-dev/vis-wolf3d) — produced a
great many real measurements about the *console*: its timers, its audio
hardware, its display path, its input, and a long list of things that do not work
the way the SDK says. None of it came from opening a retail disc. It came from
writing code, running it, and watching it fail.

| Mark | Means |
|---|---|
| **[N of 3]** | Measured on retail pressings. Three are in hand; the denominator moves as more are opened. |
| **[authoring]** | Proven by writing code that runs on the machine. Real measurement, wrong direction — it says what the console does, not what a published disc contains. To date the "machine" has been MAME's VIS driver, not silicon; the console itself is genuinely scarce. |
| **[unverified]** | From the Tandy SDK documents, and not measured at all. |

**A mark is never promoted because nothing contradicted it.** An `[authoring]`
finding does not become `[3 of 3]` because it ought to hold on a retail disc.

The three-mark scheme is not bureaucracy. The `[authoring]` category has already
been wrong twice in ways only running code could reveal, and both corrections are
in the document in place, with the wrong version still visible: a **PCM DAC this
document recorded as absent** and that turns out to sit at `0x220`–`0x22f` behind
raw DMA channel 7, and a **display flag read as `NOWAIT` for a whole session**
that was `STRETCH 2X` the entire time.

## What it covers

| Section | |
|---|---|
| 1 | What the machine is — an 80286 running **Modular Windows in ROM**, so a title is a **Win16 NE application**, not a boot image; and the two hard limits (`tlaunch` ignores `AUTOEXEC`, `LoadLibrary` raises "insert main disc") |
| 2 | **`CONTROL.TAT`** — the vendor block, **84 bytes byte-identical across three unrelated discs**, naming the tool that wrote it (**`Maketat`**), its build version and its date, with the title field where the leftovers live |
| 3 | ISO 9660, and reading a disc that is really an uninstalled Windows install tree |
| 4 | Censusing the disc — including **two independent dates per pressing**, which no other platform here offers |
| 5 | The display path: nine documented `DVA` modes, one proven, **zero measured on retail**; GDI against DispDib; and the flag that was read wrong for a session |
| 6 | Audio: OPL3 at `0x388`/`0x389`, **the DAC this document said did not exist**, and the PIT that runs at **≈596 kHz** so every constant borrowed from PC arithmetic is off by two |
| 7 | The hand controller — virtual keys `0x70`–`0x79`, no `WM_KEYUP`, and why **`HC.DLL` in the import table detects a title that reads the controller** |
| 8 | The executables: NE headers, and an **import table that is a capability list, for free** |
| 9 | Under 1 MB of DOS arena — the constraint every commercial title was also written under, and what it predicts about their discs |
| 10 | Baselines — three rows, `CONTROL.TAT` columns filled because they are measured, everything else waiting |
| 11 | Order of work — the machine half established, the disc half proposed |
| 12 | The hash lists, and the cross-disc result this platform already has |

## Discs it is drawn from

**None documented yet.** Three retail pressings are in hand and have given up
their `CONTROL.TAT` (section 2) and nothing else. The full write-up for each will
live in the family index:
**[vis-gamelist-doc](https://github.com/vs-sr-dev/vis-gamelist-doc)**.

| Disc | Year | `Maketat` | What is known |
|---|---|---|---|
| Atlas of U.S. Presidents | 1992 | 1(12) 31-Aug-92 | Applied Optical Media. Title field claims `Ver. 1.0` |
| Bible Lands, Bible Stories | 1992 | 1(12) 31-Aug-92 | Context/InterMedia. **A retail pressing whose title field calls it a prototype demo** |
| Fitness Partner | 1992/93 | 1(13) 9-Oct-92 | Computer Directions. Title field says **`V.90`** — a pre-1.0 version number, shipped |

## The result this repository already has

The leading **84 bytes of `CONTROL.TAT` are byte-identical on all three discs**:

```
Copyright (c) 1992 Tandy Corporation. All Rights Reserved.
md5 ed9bfc904220e409f04c0772f1797ff7   (first 84 bytes)
```

Three studios, three subjects, one block — written by Tandy's mastering tool
rather than by anybody who made a title. This is the VIS's answer to the CD-i
publisher's bumper and the Amiga CD `.TM` block, it is the platform's cheapest
identity check, and it cost one afternoon of comparing three small files.

The same file also stamps **which build of `Maketat` pressed the disc, and
when** — and two builds six weeks apart are already visible across three discs.
That is a mastering-chain fingerprint of exactly the kind section 3 of the Amiga
CD notes is built on: it dates the pressing process independently of whatever the
title claims about itself.

And it has already produced a leftover, in the field nobody reviews: **the retail
*Bible Lands* pressing describes itself as `Bible Prototype Demo`**, and the
retail *Fitness Partner* calls itself version `.90`.

## Contributing from a pipeline

When a title turns up something about the *format* rather than the title:

1. Add it to the relevant section here rather than to the title's repository.
2. **Respect the three marks.** `[unverified]` → `[authoring]` needs code that
   ran; `[authoring]` → `[N of 3]` needs retail discs. The two are not
   interchangeable and the whole value of this document is that it says which is
   which.
3. **Move the denominator.** When the fourth disc arrives, every `[3 of 3]`
   becomes `[3 of 4]` or `[4 of 4]` — re-derive the ones your disc exercises and
   re-declare the rest in place with a note. Two sessions of postponing a mark
   turn it into furniture.
4. If a finding contradicts what is written, **correct the text and say so in
   place.** This document already has two such corrections and both are more
   useful with the history attached than without.
5. Update the baseline table in section 10 and the order of work in section 11,
   and publish `notes/sha1-all.txt` for your disc (section 12).

State what is measured and what is inferred, and keep the measurements in the
document. Half of what makes a disc interesting is the list of things that are
measurably odd and not yet explained.
