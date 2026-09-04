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
| **[N of 5]** | Measured on retail pressings. **Five are in hand and two are documented**; the denominator moves as more are opened. |
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
| 1 | What the machine is — an 80286 running **Modular Windows in ROM**, so a title is a **Win16 NE application**, not a boot image; the two hard limits (`tlaunch` ignores `AUTOEXEC`, `LoadLibrary` raises "insert main disc") — **and a retail pressing that contradicts all of it, stated as a contradiction** |
| 2 | **`CONTROL.TAT`** — the vendor block, **84 bytes byte-identical across three unrelated discs**, naming the tool that wrote it (**`Maketat`**), its build version and its date, with the title field where the leftovers live |
| 3 | ISO 9660, and reading a disc that is really an uninstalled Windows install tree |
| 4 | Censusing the disc — including **two independent dates per pressing**, which no other platform here offers |
| 5 | The display path: nine documented `DVA` modes, one proven, **zero measured on retail**; GDI against DispDib; and the flag that was read wrong for a session |
| 6 | Audio: OPL3 at `0x388`/`0x389`, **the DAC this document said did not exist**, and the PIT that runs at **≈596 kHz** so every constant borrowed from PC arithmetic is off by two |
| 7 | The hand controller — virtual keys `0x70`–`0x79`, no `WM_KEYUP`, and why **`HC.DLL` in the import table detects a title that reads the controller** |
| 8 | The executables: NE headers, and an **import table that is a capability list, for free** — plus **what to do when the binary turns out to be MZ**, which is what the first documented disc shipped |
| 9 | Under 1 MB of DOS arena — the constraint every commercial title was also written under, and what it predicts about their discs |
| 10 | Baselines — **five rows, two of them filled from measurement**, and five columns the two documented discs made me want |
| 11 | Order of work — the machine half established, the disc half proposed |
| 12 | The hash lists — **two exist and both crossed to zero**, and the second one found four fifths of its own disc duplicated inside itself |

## Discs it is drawn from

**Two documented, three in hand.** The other three have given up their
`CONTROL.TAT` (section 2) and nothing else. The full write-up for each lives in
the family index:
**[vis-gamelist-doc](https://github.com/vs-sr-dev/vis-gamelist-doc)**.

| Disc | Year | `Maketat` | What is known |
|---|---|---|---|
| **[Sherlock Holmes: Consulting Detective, Vol. I](https://github.com/vs-sr-dev/vis-sherlockholmes-doc)** | **1992** | **1(12) 31-Aug-92** | **ICOM Simulations. Documented end to end. 193 files, 445,760,838 bytes, and it contains no Windows executable at all — see the box in section 1** |
| **[Race the Clock](https://github.com/vs-sr-dev/vis-racetheclock-doc)** | **1992** | **1(12) 31-Aug-92** | **Mindplay / Methods & Solutions. Documented end to end. 3,625 files, 374,094,820 bytes, and it is the Win16 NE application section 1 always said a VIS title would be — cut thirteen days after the disc above, by the same tool build** |
| Atlas of U.S. Presidents | 1992 | 1(12) 31-Aug-92 | Applied Optical Media. Title field claims `Ver. 1.0` |
| Bible Lands, Bible Stories | 1992 | 1(12) 31-Aug-92 | Context/InterMedia. **A retail pressing whose title field calls it a prototype demo** |
| Fitness Partner | 1992/93 | 1(13) 9-Oct-92 | Computer Directions. Title field says **`V.90`** — a pre-1.0 version number, shipped |

### And what the second disc did to this document

**It confirmed section 1 where the first disc contradicted it, and neither box
was deleted.** A note with one sample for and one against is in better shape
than a note with one sample, and the section says so instead of picking a
winner.

| | |
|---|---|
| §1, *a VIS title is a Win16 NE application* | **[2 of 5] confirmed.** `NE` at 0x400, Windows 3.10, `shell=rtc.exe` in the disc's own `SYSTEM.INI` |
| §2, the third field | **the correction is withdrawn and replaced.** This document's table put **two different fields in one column**: `fdiv` at `0x0B0` is on both documented discs, and what was tabulated is the **program list at `0x1A3`**. And the total length is **`463 + list length` on 5 of 5** |
| §2, the 84-byte block | **`[4 of 4]` → `[5 of 5]`** |
| §2, the ATTENTION statement | **2 of 2**, same `0xA0` non-breaking space |
| §7, *`HC.DLL` in the imports detects a controller-reading title* | **first retail confirmation.** 89 relocation fixups, exactly one by name: `HC` → `HCCONTROL` |
| §8, the NE method | **applies in full for the first time** — segments, imports, resources, strings, a `.DEF` description and six exported window procedures |
| §8, the `WIN87EM` trap | **a retail title imports it and does not ship it.** Either the ROM has it or the note is over-stated |
| §6, the DAC | **`port=220 channel=7 int=7` in a retail `SYSTEM.INI`** — this document's own three numbers, from the other direction |
| §9, the 1× budget | **`dwMaxBytesPerSec` = 153,600 on 1,208 of 1,208 files**, and **80.2861 % of the disc is copies of itself** |
| §3, the expectation list | **four sixths confirmed**, and **1,208 `.AVI`** where the list says `.FLI` and `.FLC` |
| new | a ROM module called **`TVUI`**, and three fingerprints of the **Meridian Data** mastering chain (§4) |

### And what the first disc did to this document

**It broke section 1 and section 8, and neither was rewritten.** The
contradiction is stated *as* a contradiction, in place, with the original text
intact, because an `[authoring]` finding about what the console *does* is not
falsified by what a publisher *shipped*. What the disc earned is a `[1 of 4]`
column, a fourth baseline row, and a named test for the next pressing. Those
`[1 of 4]` marks are left as they are; the second disc did not exercise most
of them.

| | |
|---|---|
| the 84-byte identity block | **`[3 of 3]` → `[4 of 4]`.** Four studios, four subjects, one block |
| the third field, read as `minwin <drive>` | ~~**corrected.** On the fourth disc it reads **`fdiv`** — no path, no drive letter. Three samples described a pattern and did not name a field~~ **— and that correction is itself withdrawn by the fifth disc, above** |
| the title field's "leftovers" pattern | **one disc weaker.** Three of four are now ordinary |
| §12 Q5, *how much of a disc is Windows?* | **closed: none of it.** 0 % Windows code; 0.0038 % Windows-format bytes |
| §5 Q2, *which `DVA` mode?* | **does not close.** That title never calls `EnterDVA`; it sets VGA mode 13h through the BIOS |
| §6 Q3, *does a title use the DAC?* | **half.** It ships 32.1420 % of itself as 22,050 Hz PCM — and names neither `0x220` nor `0x388` |
| §9, *expect the disc to be shaped by the memory limit* | **confirmed on every clause**, and it is this document's best prediction |
| two things §2 had never seen | an **authorisation statement**, and a **two-program launch list** — the strongest reading yet of what `CONTROL.TAT` is *for* |

## The result this repository already has

The leading **84 bytes of `CONTROL.TAT` are byte-identical on all five discs**:

```
Copyright (c) 1992 Tandy Corporation. All Rights Reserved.
md5 ed9bfc904220e409f04c0772f1797ff7   (first 84 bytes)
```

Five studios, five subjects, one block — written by Tandy's mastering tool
rather than by anybody who made a title, and it has now survived two discs from
outside the reference-title corner of the library. This is the VIS's answer to the CD-i
publisher's bumper and the Amiga CD `.TM` block, it is the platform's cheapest
identity check, and it cost one afternoon of comparing three small files.

**And the fifth disc closed the length question.** Every one of the five files
is `463 + the length of its NUL-terminated program list at 0x1A3`, exactly —
473, 473, 474, 474 and 483 — which replaces both the drive-letter explanation
this document carried and the correction the fourth disc offered. See section 2.

The same file also stamps **which build of `Maketat` pressed the disc, and
when** — and two builds six weeks apart are visible across five discs, four of
them sharing `1(12) 31-Aug-92`.
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
