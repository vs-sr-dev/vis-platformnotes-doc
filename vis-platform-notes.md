# VIS platform notes — a checklist for the next disc

A running checklist for the **Tandy / Memorex Video Information System** (1992),
carried from one documentation pipeline to the next and added to by each.

This document started in an unusual position, and the position is worth stating
before anything else: **the machine was partly known and its discs were not.**
One retail pressing has now been documented end to end, and it did not confirm
what this document expected — see the box in §1.

Three years of homebrew work on this platform — a Win16 OPL3 synthesizer, a
media browser, and a native *Wolfenstein 3D* port — produced a large number of
real measurements about the *console*: its timers, its audio hardware, its
display path, its input, and a long list of things that do not work the way the
documentation says. None of that came from opening a retail disc. It came from
writing code, running it, and watching it fail.

So this document uses **three marks**, not one, and the difference between them
is the whole point:

| Mark | Means |
|---|---|
| **[N of 4]** | Measured on retail pressings. **Four are in hand and one is documented**; the denominator moves as more are opened. Older `[N of 3]` marks that this disc did not exercise are left at 3 and re-declared as such rather than silently renumbered. |
| **[authoring]** | Proven by writing code that runs on the machine and observing what it does. Real measurement, wrong direction — it says what the console does, not what a published disc contains. **To date the "machine" has been MAME's VIS driver, not silicon.** The console itself is genuinely scarce. |
| **[unverified]** | From the Tandy SDK documents, and not measured at all. A hypothesis with a section number. |

**A mark is never promoted because nothing contradicted it.** An `[authoring]`
finding does not become `[3 of 3]` because it seems like it ought to hold on a
retail disc — it becomes that when retail discs say so.

**And it is not demoted for the reverse reason either.** The first documented
pressing contradicts §1 and §8 outright, and neither section was rewritten:
the contradiction is stated *as* a contradiction, in place, with the original
text intact. An `[authoring]` finding about what the console *does* is not
falsified by what a publisher *shipped*, and one retail disc is entitled to add
a column, not to delete a section.

The `[authoring]` marks have already earned their own category twice, by being
wrong in ways only running code could reveal. Both corrections are in this
document, in place, with the wrong version still visible: see §6 on the DAC that
was documented here as absent, and §5 on a display flag that was read as
`NOWAIT` for a whole session and was something else entirely.

---

## 1. What this machine is, and why its discs look like nothing else here

**[unverified]** except where marked. An Intel **80286 at 12 MHz**, a Yamaha
**YMF262 (OPL3)**, a **1× Mitsumi CD-ROM**, output to a television, and under
1 MB of usable DOS arena.

And then the part that makes the platform unlike every other optical console in
this collection: **it runs Windows.** Specifically **Modular Windows**, a
cut-down Windows 3.1 for consumer appliances, in ROM. A VIS title is not a boot
image and not a flat executable — it is a **Win16 NE application**, launched by a
shell, using GDI to put pixels on a television.

Three consequences that shape the whole checklist:

- **The file system is ordinary ISO 9660**, and ordinary tools open it. The
  interesting structure is not in the volume layout, it is in the executables and
  in what the Windows runtime does to them.
- **The executables are NE binaries**, so the entire Win16 toolset applies:
  segment tables, import and export tables by name, resource directories. A VIS
  disc gives up its imports for free, which is more than any other platform here
  does.
- **A large part of what a title "is" lives in ROM, not on the disc.** The
  runtime, the shell and the system DLLs are in the console. A disc can therefore
  be very small and still be a whole product, and the disc alone never tells the
  whole story.

**[authoring]** Two hard limits found the expensive way, both worth knowing
before reading any executable:

- **The `tlaunch` ROM shell ignores `AUTOEXEC` content, and `WinExec` on a DOS
  application is not supported.** There is no real-mode DOS path on this machine.
  Everything goes through the Modular Windows `shell=` line. Any analysis that
  assumes a DOS program can run here is analysing a machine that does not exist.
- **`LoadLibrary` at run time raises an "insert main disc" dialog.** ROM DLLs
  must be statically imported. This is visible in the import table, so it is also
  a thing to *read* rather than only a thing to avoid.

### **[1 of 4] A retail pressing that contradicts all of the above, stated as a contradiction**

The first retail VIS disc anybody here has opened —
[*Sherlock Holmes: Consulting Detective, Vol. I*](https://github.com/vs-sr-dev/vis-sherlockholmes-doc),
ICOM Simulations, 1992, 193 files, 445,760,838 bytes — **contains no Windows
executable at all.** Not a stub, not a loader, none:

| | |
|---|---|
| NE, LE, LX or PE binaries | **0** |
| `.DLL`, `.INI`, `.HLP`, `.FON` | **0** |
| import tables | **0**, because there is no NE header for one to live in |
| the program it ships | `SHI.EXE`, 119,618 bytes, **real-mode MZ**, Borland C++ 1991 |
| what its vendor block names, in order | `A:MOUSE.COM`, then `SHI.EXE` — **two DOS programs** |
| how it puts pixels on screen | `INT 10h AH=0 AL=13h`, then writes segment `A000h` directly, stride 320 |
| how it reads input | `INT 33h`, a Microsoft DOS mouse, range 0–639 × 0–199 |
| files containing any VIS-aware token | **1 of 193** — `CONTROL.TAT`, 483 bytes, **0.000108 %** |

**This section is not rewritten and the `[authoring]` marks above are not
demoted.** This document's own rule is that a mark is never promoted because
nothing contradicted it, and the reverse holds: **an `[authoring]` finding
about what the console *does* is not falsified by what a publisher *shipped*.**
The two halves have never been measured by the same instrument — everything
above the line came from running code, everything in this box came from
reading files, and neither can settle the other.

What *can* be said is that **the wording is broader than the experiment behind
it.** *"`WinExec` on a DOS application is not supported"* is a specific API
failing that was observed. *"There is no real-mode DOS path on this machine"*
is a much larger claim that the same experiment does not establish. **The note
may be over-stated rather than wrong**, and keeping those apart is what the
three marks are for.

Four readings survive the measurements, and three of them are boring:
the console runs DOS and the note is over-stated; the note is right and this
disc does not boot as it stands; **this is a DOS master with a Tandy vendor
block bolted on and the finding is about ICOM's production, not the console**;
or some mixture. The third is the cheapest and it is the most likely. **None
of them is settled**, and the checks that would settle each are listed in
[that repository's chapter 11](https://github.com/vs-sr-dev/vis-sherlockholmes-doc/blob/master/docs/11-the-question.md).

**The cheapest next step for whoever holds the other three pressings: does any
of them contain an NE binary?** That is one `find` and it turns "this pressing
has no Windows program" into a claim about the platform, or kills it.

## 2. `CONTROL.TAT` — the vendor block, and the first thing to read [4 of 4]

Every VIS disc carries a root file called **`CONTROL.TAT`**. It is small — 473,
474 or **483** bytes on the four discs in hand — and it is the closest thing
this platform has to the CD-i pre-file-system region or the Amiga CD `.TM`
block: a vendor-supplied structure that travels with the mastering chain rather
than with the title.

**[4 of 4]** The first **84 bytes are byte-identical on all four discs**, from
four unrelated developers:

```
Copyright (c) 1992 Tandy Corporation. All Rights Reserved.    \0\0
                  \r\n

md5 ed9bfc904220e409f04c0772f1797ff7   (first 84 bytes)
```

The four discs are *Atlas of U.S. Presidents* (Applied Optical Media),
*Bible Lands, Bible Stories* (Context/InterMedia), *Fitness Partner*
(Computer Directions) and *Sherlock Holmes: Consulting Detective, Vol. I*
(ICOM Simulations). Different studios, different subjects, different genres,
same 84 bytes. **Hash the first 84 bytes of `CONTROL.TAT` on any new disc**;
it is a two-second check and it is the platform's cheapest identity test, and
it has now survived its first challenge from outside the reference-title
corner of the library.

**[1 of 4] There is a tool for this now.**
[`controltat.py`](https://github.com/vs-sr-dev/vis-sherlockholmes-doc/blob/master/tools/controltat.py)
turns this section's four paragraphs of prose into a table: the 84-byte md5,
the title field, the `Maketat` string, the third field, the twelve binary
bytes, any program list and the total length, each printed with the offset it
was **found** at rather than the offset it was expected at. It takes
`--strict` and fails loudly when the identity block does not match, which is
the whole point of having a cheap identity test. It belongs to this document's
orbit rather than to any one pipeline; copy it into the next one.

**[4 of 4] The file names the tool that made it, and dates it.** Near the end of
every one of the four:

```
Atlas      minwin A:\    Maketat - Version is 1(12) 31-Aug-92
Bible      minwin A:     Maketat - Version is 1(12) 31-Aug-92
Fitness    minwin a:\    Maketat - Version is 1(13)  9-Oct-92
Sherlock   fdiv          Maketat - Version is 1(12) 31-Aug-92
```

So `CONTROL.TAT` is produced by a Tandy tool called **Maketat**, it stamps its
own build version and date into every disc it makes, and **two builds of that
tool are visible across four discs** — three on `1(12) 31-Aug-92` and one on
`1(13) 9-Oct-92`. That is a mastering-chain fingerprint of exactly the kind
section 3 of the Amiga CD notes is built on: it dates the *pressing process*,
independently of anything the title claims about itself. Collect the `Maketat`
version on every disc; the set of versions and their date ranges is a
platform-level result that no single disc can produce.

**[1 of 4] CORRECTION — the third field is not a path.** This document
previously read that field as follows, and the fourth disc does not support it:

> ~~Note the third field too — `minwin A:\`, `minwin A:`, `minwin a:\`. It is
> the path the tool read from, it is inconsistent in case and in its trailing
> separator, and **it accounts for the one-byte length difference** between the
> Bible disc's 473 bytes and the other two's 474.~~

On *Sherlock Holmes: Consulting Detective* that field, at offset `0xB0`, reads
**`fdiv`**. Four characters, no drive letter, no separator, no `minwin`.
Whatever it holds, it is not "the path the tool read from" — there is no path
in it. Either it holds something else that happened to look like a source path
on the first three discs, or it is free text three authoring houses filled in
one way and a fourth filled in another. **Three samples were enough to
describe a pattern and not enough to name a field.** The wrong reading is left
above on purpose.

And the length explanation does not survive either: `483 − 474 = 9`, and one
byte of drive-letter typing cannot account for nine. The candidate that can is
in the next paragraph.

**[4 of 4] The title field is at offset 0x54 and it is where leftovers live.**
The four read:

```
Atlas      Atlas of Presidents - By Applied Optical Media Corp. Ver. 1.0
Bible      Bible Prototype Demo - for Context/InterMedia
Fitness    FITNESS PARTNER V.90 BY Computer Directions (C)1992, 1993
Sherlock   Sherlock Holmes Volume I - ICOM Simulations
```

**A retail pressing of *Bible Lands, Bible Stories* describes itself as a
prototype demo**, and a retail pressing of *Fitness Partner* calls itself
version .90. Neither is what the box says. This field is written once by whoever
ran Maketat and evidently never reviewed, which makes it the single most
promising leftover on a VIS disc and the reason to read it before anything else.

**[1 of 4] And the fourth disc is the boring direction, which is a result.**
*Sherlock Holmes* reads as an accurate, unembarrassing title with the studio's
name after it. **Three of four are now ordinary**, and the pattern this
section identified is one disc weaker than it looked. Keep reading the field
first; stop expecting it to embarrass somebody.

**[1 of 4] TWO THINGS IN THIS FILE THIS SECTION DID NOT KNOW ABOUT.**

**A program list, and it is the strongest candidate yet for what this file is
*for*.** At `0x1A3` on the *Sherlock* disc:

```
0x1A3   A:MOUSE.COM \0 SHI.EXE \0
```

Two executable names, in order, the first carrying `A:` — the drive letter
this machine gives the CD. **Both are real-mode DOS programs** (§8, §11 and
`vis-sherlockholmes-doc`). Not a copyright notice that happens to have a title
in it: **the disc's launch list**, standing where a PC would have an
`AUTOEXEC.BAT`.

It is also the best available account of the 483-byte length. If the other
three discs name **one** program where this names two, their `Maketat` string
lands eight or nine bytes earlier and their files end that much sooner.
**Checking that is five minutes on discs somebody already holds**, and it
would replace the drive-letter explanation this section has been carrying.

**An authorisation statement**, at `0x14C`, which §2 has never recorded:

```
[ ATTENTION: This is an Authorized Video Information System<A0>Title.
  END OF STATEMENT ]
```

In English, in square brackets, with a formal terminator, and with a `0xA0`
non-breaking space between `System` and `Title`. This section lists the run at
`0x1aa–0x1d3` as *"holding the tool string"* and nothing else; it holds this
too. **Whether the other three discs carry it is unknown and is a five-minute
check** that would make it `[4 of 4]` — a platform-wide licensing stamp — or
`[1 of 4]`, one publisher's lawyer. Those are very different findings.

**[1 of 4]** The twelve binary bytes at `0x98` on the fourth disc:
`2F 3B 37 05 76 77 DD 2A DD 7F E3 8D`. Still unexplained, still different from
every other sample. The byte at `0xC6` reads `0x00`.

**[authoring]** The block can be **cloned from a retail disc onto a homebrew one
without recalculating anything**. The twelve binary bytes it carries have never
been shown to be a checksum or a signature: a copied `CONTROL.TAT` boots. That is
a fact about the platform's protection, or its absence, and it is stated here as
an observation rather than as a claim that nothing validates it — nobody has
tried to make it fail.

**And a hypothesis about that experiment, worth exactly one sentence.** If the
block is a launch list (above), then cloning one onto a homebrew disc booted a
disc whose vendor block named a program list belonging to a *different* disc —
and it worked anyway. That would say something quite specific about how much
of the block the ROM shell actually reads. It is a reinterpretation of an
existing `[authoring]` result, not a new measurement, and it is an open
question rather than a finding.

**Open questions on this file.** What the bytes at 0x98–0xa3 are, which differ on
all three; what the byte at 0xc6 selects, which differs between Atlas and the
other two; and what the run at 0x1aa–0x1d3 holds beyond the tool string. All
three regions are small, they vary per disc, and three samples is not enough.

## 3. ISO 9660, and reading a Windows product rather than a disc

**[unverified]** Ordinary ISO 9660. Read the volume descriptor's own fields and
record them — volume identifier, publisher, data preparer, application
identifier, creation and modification timestamps — because on the neighbouring
platforms those fields have repeatedly named a person or dated a build.

Then read the disc as what it is: **a Windows application's install tree that was
never installed.** Expect `.EXE` and `.DLL` in NE format, `.INI` files,
`.BMP`/`.DIB` images, `.FLI` and `.FLC` animations, `.WAV` and `.MID`, and
Windows help and resource files.

> **[1 of 4] The first documented disc has none of that.** Nine extensions,
> and the Windows-shaped ones are two `.DIB` totalling 16,908 bytes — 0.0038 %
> — whose pixels index a system palette the disc does not carry. No `.DLL`, no
> `.INI`, no `.HLP`, no `.WAV`, no `.MID`, no `.FLI`, no `.FLC`, no `.TXT`, no
> readme and no licence. **96.4344 % of it is one undocumented streaming
> format** and the rest is one undocumented container. Expect the extensions
> this paragraph lists, and **measure whether you got them**, because on the
> only pressing anybody has opened the answer was no. The extensions mean what they mean on a PC of
the same year, which makes this the *least* exotic file layout in this
collection and moves the difficulty somewhere else entirely — into the runtime.

## 4. Census the disc before reading any of it

Unchanged from the other checklists, and the passes pay for themselves here too:

- **All-zero and near-zero files**, a byte histogram per file.
- **Timestamps**, bucketed by day, checked against the volume descriptor and
  against the `Maketat` date in `CONTROL.TAT` (§2). **Two independent dates for
  the same pressing is a luxury** the other platforms do not have — use it.
- **Sector map**: attribute every sector to a file, to the file system, or to
  nothing, and hexdump whatever belongs to nothing.

**[unverified]** Add the platform-specific pass: **read the `.INI` files first.**
A Modular Windows title is configured in text, that text names modules, drivers
and start-up behaviour, and it is the fastest route into how the product is put
together.

## 5. The display path, and the flag that was read wrong for a session

**[unverified]** The SDK documents a set of display modes reached through
`EnterDVA`:

```
DVA_MODE_320x200x8         8-bit palettised
DVA_MODE_555_320x200       16-bit direct colour
DVA_MODE_565_320x200
DVA_MODE_555_320x400       the 400-line variants
DVA_MODE_565_320x400
DVA_MODE_YUV8_320x200      YUV modes
DVA_MODE_Y1JV16_320x200
DVA_MODE_Y1JV16_320x400
DVA_MODE_Y1JV8_320x400
```

**[authoring]** Only `DVA_MODE_320x200x8` has been driven end to end. **What a
retail disc actually uses is unmeasured, and it is one of the better questions on
the platform** — nine modes are documented, several of them direct-colour or YUV
on a machine with under a megabyte, and it would be genuinely informative to
learn that commercial titles used one of them.

**[authoring]** The two ways to get pixels onto the screen, and what each costs:

- **GDI**, via `StretchDIBits`. `SetDIBits` fails; `StretchDIBits` works. The DIB
  must be **bottom-up**. Palette realization is required in 8 bpp. The driver
  section is `[tvvga]`, lowercase, and the case matters.
- **DispDib**, the direct `0xA000` path, which is roughly **twice as fast** —
  measured as 7–8 up to 14–18 frames per second on the same application.

**[authoring] The correction.** The DispDib `wFlag` values were derived by
disassembly, and one earlier reading was wrong and stood for a whole session:

```
BEGIN   0x8000
END     0x4000
0x0100  read as NOWAIT for a session — it was STRETCH 2X all along
```

The wrong reading is left here on purpose. It was not corrected by better
reasoning about the documentation; it was corrected by disassembling
`DisplayDibCommon` and then testing five variants against a running machine.

**[authoring]** Reaching the `0xA000` selector needs a non-obvious idiom:
`(WORD)((DWORD)(LPVOID)&_A000H)`. The C source declares `_A000H` with one
underscore and the Watcom compiler decorates it to `__A000H`. Wrong
interpretations hang the machine silently rather than failing loudly, which is
the worst possible failure mode and the reason it is written down.

## 6. Audio, and the DAC this document said did not exist

**[authoring]** **OPL3 at ports `0x388` / `0x389`**, by direct port I/O. Nine to
eighteen FM channels, and it is the obvious sound path.

**[authoring] The correction, and the more interesting one.** These notes
previously recorded that the VIS *had no DAC*, and that digitised speech would
have to fall back to buzzy FM approximations. **That was wrong.** The machine has
a **PCM DAC at `0x220`–`0x22f`**, and it is driven by **raw DMA on channel 7**.

Getting there falsified the obvious approach on the way: the `waveOut` route
through MMSYSTEM was tried and **does not work**; the raw DMA route does. Both
halves of that are worth carrying, because "the high-level API exists and does
not work" is a thing you can only learn by trying it.

So: **a retail VIS disc may well contain digitised audio, and the platform can
play it.** Whether any of them do is unmeasured.

**[authoring]** Two timing traps, and they are the same trap twice:

- **`timeGetTime` returns at half rate and `timeBeginPeriod` is not honoured.**
  Do not use MMSYSTEM for audio timing on this machine.
- **The PIT runs at ≈596 kHz, not the 1.193 MHz a PC person expects.** The
  empirical constant is **852 cycles per tick** where the theoretical figure says
  1704. Reading the latch is non-disruptive, so PIT-direct timing is the reliable
  path.

That second one is the most portable finding in this document: **any timing
constant on this machine derived from PC arithmetic is off by a factor of two.**
If a retail title's audio runs at the wrong speed under emulation, this is the
first thing to check.

## 7. Input — the hand controller

**[authoring]** The controller arrives as **`WM_KEYDOWN` with virtual-key codes
in the range `0x70`–`0x79`**, which is to say it reuses the standard Windows
`VK_F1`–`VK_F10` range. So an executable that handles the hand controller looks,
to a naïve reader, like an executable that handles function keys. Do not read it
as a keyboard.

Three quirks, all of which cost real time:

- **`HC.DLL` must be statically imported for `WM_KEYDOWN` routing to work at
  all** — not merely for polling it. This is visible in an executable's import
  table, which makes **the presence of `HC.DLL` in the imports a reliable
  detector of a title that reads the controller**.
- **`hcGetCursorPos` must be polled**; the cursor needs suppressing in three
  separate places to stay hidden.
- **There is no `WM_KEYUP` and no auto-repeat.** Anything that needs held-button
  state uses `GetAsyncKeyState`. A title with continuous movement must be doing
  this, so it is another thing to look for rather than to wonder about.

## 8. The executables

**[unverified]** NE format, and therefore readable with ordinary tools: segment
table, entry table, imported and exported names, resource directory. Dump all of
it before disassembling anything.

> **[1 of 4] On the one retail disc that has been opened, none of this
> section's method applies.** *Sherlock Holmes: Consulting Detective* ships a
> **real-mode MZ** executable — no NE header, no segment table, no entry
> table, no resource directory, and **no import table**, so "the import table
> is a capability list and it is free" has nothing to run on. `HC.DLL`,
> `DISPDIB` and `MMSYSTEM` cannot appear in a table that does not exist.
>
> `[unverified]` is the mark that is supposed to be cheap to overturn, and one
> disc overturns it **for one disc**. The section is left standing because
> three pressings have not been looked at, and because the SDK documents it
> came from describe what Tandy told developers to build. **Do the `find` for
> an NE binary on the next disc before doing anything else in this section.**
>
> **What to do instead when a disc turns out to be MZ**, from the pipeline
> that had to: read the DOS header and check that `(e_cp − 1) × 512 + e_cblp`
> equals the file length — if it does, nothing is appended; walk the
> relocation table and confirm every entry lands inside the load image, which
> dates the linker and sizes the model; then grep the *bytes* for `CD 10`,
> `CD 33`, `CD 21` and for `B8 00 A0`, and read every hit by hand. On that
> disc there were four `INT 10h` sites in 119,618 bytes and they answered the
> display question outright.
> [`mz.py`](https://github.com/vs-sr-dev/vis-sherlockholmes-doc/blob/master/tools/mz.py)
> and
> [`vistokens.py`](https://github.com/vs-sr-dev/vis-sherlockholmes-doc/blob/master/tools/vistokens.py)
> do both, and the second one prints file names and offsets rather than
> counts, because **on a disc that is 96 % compressed video a three-byte
> constant hits hundreds of times by arithmetic** — 230 chance hits on
> `mov ax,0A000h`, in an executable that contains three.

The greps that pay:

1. **Imports, as a table.** `HC.DLL` (§7), `DISPDIB` (§5), `MMSYSTEM` (§6), and
   whatever else. On this platform the import table is a capability list, and it
   is free.
2. **Toolchain fingerprints.** Which compiler and which SDK. Commercial VIS
   titles were built by small studios on a short-lived platform, and the spread
   of toolchains is likely to be narrow and informative.
3. **Filenames, both ways.** Every path-like string in every binary against the
   directory listing, and back. Cut content on one side, generated names on the
   other.
4. **Resources.** Dialogs, strings, bitmaps, menus. Windows resources are
   structured, named and easy to enumerate, which makes a VIS disc unusually
   generous with its own text.

**[authoring]** One trap that will look like a corrupt file and is not: **linking
anything from `<math.h>` pulls in the floating-point runtime and requires
`WIN87EM.DLL` at load time**, which the machine does not have, producing an
"Error loading EXE" loop. If a retail title uses trigonometry, it either ships
that DLL or uses lookup tables — and which one it chose is worth a sentence.

## 9. Memory, and why a VIS title is shaped the way it is

**[authoring]** The DOS arena is **under 1 MB and it is genuinely tight**.
`GlobalDosAlloc` of a 38 KB buffer can simply fail in a large application, and
the workaround is a size ladder plus a sub-window that avoids crossing a 128 KB
boundary.

This is not a footnote about homebrew: it is the constraint every commercial VIS
title was also written under, and it predicts what those discs will look like —
small working sets, streaming from a 1× drive, and assets sized to fit rather
than to look good. **Expect the disc to be organised around a memory limit, and
measure whether it is.**

> **[1 of 4] It is, and the prediction is the best one this document has
> made.** On *Sherlock Holmes: Consulting Detective* the video is
> **160 × 100** on 155 of 157 clips — a quarter-screen window on a machine
> whose only documented modes are 320 wide — and the stream runs at **65,978
> bytes a second, 43 % of a 1× drive's rate**, leaving the rest for seeks. Its
> streaming format re-states its own header on a sector boundary **3,879
> times** so that a player can start anywhere without having read what came
> before, and it spends **0.4302 %** of its bytes doing it. And the interface
> art bank ships **three times, byte for byte identical** — 4.9 MB of
> deliberate duplication, because on a 1× mechanism a copy is cheaper than a
> seek.
>
> Small working sets, streaming from a 1× drive, assets sized to fit. **Every
> clause of the paragraph above, on the first disc that could test it.**

## 10. Baselines, so you can tell signal from noise

*Four discs are in hand and **one** is documented. Its row is filled from
measurement; the other three still have only what §2 gave up.*

| Disc | Year | Studio | `Maketat` | TAT len | Title field | Files | Declared bytes | NE binaries | Imports `HC.DLL` | DVA mode | Digitised audio |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Atlas of U.S. Presidents | 1992 | Applied Optical Media | 1(12) 31-Aug-92 | 474 | `Atlas of Presidents … Ver. 1.0` | — | — | — | — | — | — |
| Bible Lands, Bible Stories | 1992 | Context/InterMedia | 1(12) 31-Aug-92 | 473 | `Bible Prototype Demo …` | — | — | — | — | — | — |
| Fitness Partner | 1992/93 | Computer Directions | 1(13) 9-Oct-92 | 474 | `FITNESS PARTNER V.90 …` | — | — | — | — | — | — |
| **[Sherlock Holmes: Consulting Detective, Vol. I](https://github.com/vs-sr-dev/vis-sherlockholmes-doc)** | **1992** | **ICOM Simulations** | **1(12) 31-Aug-92** | **483** | **`Sherlock Holmes Volume I - ICOM Simulations`** | **193** | **445,760,838** | **0** | **n/a — no import table** | **none; VGA mode 13h via `INT 10h`** | **yes, 143,276,490 B of 22,050 Hz PCM** |

Two columns this disc made me want, which is what the instruction above says
to do:

| Disc | Programs named in `CONTROL.TAT` | VIS-aware files, of all files |
|---|---|---|
| Sherlock Holmes | **`A:MOUSE.COM` then `SHI.EXE`** — two, both real-mode DOS | **1 of 193** (`CONTROL.TAT`), 483 B of 445,760,838 = **0.000108 %** |

Both are cheap on any pressing and both separate this disc from what §1
expected, which is the test for keeping a column.

Add a column when a disc makes you want one; delete any that never separates two
titles.

## 11. Order of work — the disc half is proposed, the machine half is not

Steps marked **[authoring]** are established practice from three homebrew
projects. The rest is a proposal and the first documented disc has standing to
rewrite it.

**[1 of 4] The first documented disc exercised this list and it survived, with
one branch added at step 6.**

1. **Track layout and dump form**, before the image.
2. **`CONTROL.TAT` first.** Hash the leading 84 bytes against
   `ed9bfc904220e409f04c0772f1797ff7`; read the title field; record the `Maketat`
   version and date; note the total length. Five minutes, and it produces two
   comparable numbers plus a leftover candidate before anything else is opened.
3. **ISO 9660**: descriptor fields, then the listing.
4. **Census**: all-zero files, both date sources, sector map.
5. **`.INI` files before executables.** *(On the first documented disc there
   are none at all, and the absence is itself a measurement — record it.)*
6. **NE headers and import tables, as a table**, for every binary. Capability
   list for free. **If the binary turns out to be MZ, take the branch in §8**:
   page arithmetic against the file length, walk the relocation table, then
   grep the bytes for `CD 10`, `CD 33` and `B8 00 A0` and read every hit by
   hand.
7. **Resources**, enumerated and dumped.
8. **Strings, both ways.**
9. **Which display mode does it actually use?** (§5) — nine documented, one
   proven, zero measured on retail.
10. **Is there digitised audio, and how is it played?** (§6)
11. **Publish `notes/sha1-all.txt`** (§12).
12. **Fill in one row of §10**, and re-mark in this document every claim the disc
    exercised — respecting the difference between `[authoring]` and `[N of 3]`.

**[authoring]** For anything that needs running rather than reading: the working
environment is Open Watcom V2 for Win16, `pycdlib` for images, and **MAME's VIS
driver**, including a locally patched build where the stock driver lacks a
device. That is how every `[authoring]` mark here was obtained, and it is also
the honest limit on all of them.

## 12. The hash lists

**[unverified]** Every disc repository publishes `notes/sha1-all.txt`: one record
per file, with path, size and SHA-1.

**[4 of 4]** And this platform already has its cross-disc result, before its
first pipeline: the leading 84 bytes of `CONTROL.TAT`, identical on four
unrelated discs. That block belongs to no title, was written by a Tandy tool, and
is the VIS's answer to the CD-i bumper and the Amiga CD `.TM` block. **It is
already the cheapest identity check on the platform, and it was free.**

The lesson the CD-i family paid four discs for transfers directly: **the unit of
sharing may be smaller than a file.** On a platform whose products are Windows
applications, the obvious candidates are shared DLLs, shared resource chunks and
a runtime redistributed by more than one publisher — none of which a file-level
hash list will see if the surrounding file differs by a byte.

**[1 of 4] The first hash list exists, and it crossed to zero.**
`vis-sherlockholmes-doc/notes/sha1-all.txt` — 193 records, sizes summing to
445,760,838, **186 distinct hashes**. Crossed at file granularity against 75
repositories, 324 hash lists and 54,901 tokens: **0 hits.**

Two things worth carrying from that:

* **`MOUSE.COM` was the best cross-hit candidate this collection has ever had**
  — Microsoft's own DOS mouse driver, 56,448 bytes, shipped on a game disc —
  and it missed on every DOS-era object in the library. Ordinary DOS games did
  not ship the driver; they expected `AUTOEXEC.BAT` to have loaded it.
  **A VIS owner has no `AUTOEXEC.BAT`**, which is why this disc carries it and
  why its presence is a platform fact rather than a packaging one;
* **the 84-byte block is the case the file-level list cannot see**, exactly as
  this section predicts, and there is still **no second VIS disc in the
  collection to cross it against.** Publishing the md5 is the whole mitigation:
  `ed9bfc904220e409f04c0772f1797ff7`, two seconds on the next pressing.

Also useful: the same list found **7 duplicate files inside one disc** —
4,913,992 bytes, 1.1024 % — because a 1× drive makes shipping the interface
art three times cheaper than seeking back to one copy. Expect intra-disc
duplication on this platform and measure it; it is a design signature, not
waste.

---

## Open questions

*One retail disc has now been documented. Each question below carries what it
did to it.*

1. **How many `Maketat` versions are there, and over what dates?**
   **[1 of 4] Converging.** Two builds across four discs — `1(12) 31-Aug-92`
   on Atlas, Bible and Sherlock, `1(13) 9-Oct-92` on Fitness. Six weeks apart.
   Still costs nothing per disc.
2. **Does any commercial title use a display mode other than 320×200×8?**
   **[1 of 4] Does NOT close, and that is the finding.** *Sherlock Holmes*
   sets **VGA mode 13h through `INT 10h`** and never calls `EnterDVA` at all,
   so it says nothing about what a `DVA`-using title does. Nine documented,
   one driven, **still zero observed on retail**. Recording that honestly is
   worth more than logging 320×200×8 as an answer to a question about `DVA`.
   *(It does call `INT 10h` with `AH=50h, AL=01, DL=00` immediately after
   setting the mode. That is not a standard IBM VGA BIOS function and nobody
   knows what it is. **It is the single most machine-specific instruction
   found on a retail VIS disc so far** and identifying it needs a VIS BIOS.)*
3. **Does any commercial title use the DAC?**
   **[1 of 4] Closes halfway, and it was two questions.** A commercial VIS
   disc **does** carry digitised audio: 143,276,490 bytes of unsigned 8-bit
   PCM at 22,050 Hz interleaved inside its video, plus 8.5 MB more in a
   separate bank — **32.1420 % of the whole disc**. But *how it reaches the
   hardware* is unknown: that executable contains **no reference to port
   `0x220` and none to `0x388`/`0x389`**. Whether a commercial title drives
   *the VIS DAC* is still open, and it is a different question from whether
   one ships PCM.
4. **What do the twelve binary bytes in `CONTROL.TAT` do?**
   **[1 of 4] A fourth sample, still unexplained.** `2F 3B 37 05 76 77 DD 2A
   DD 7F E3 8D`.
5. **How much of a VIS disc is the product, and how much is Windows?**
   **[1 of 4] CLOSES, with an answer the question did not anticipate: none of
   it.** **0 % is Windows code.** The only Windows-format bytes on the whole
   pressing are two `.DIB` totalling 16,908 — **0.0038 %** — and they are
   indexed against a Windows system palette that is not on the disc and are
   not referenced by the DOS program that is. The question was framed
   expecting a ratio; on this disc the ratio is a zero and a footnote.
6. **Is there a shared component across publishers?**
   **[4 of 4] on the 84-byte block.** `MOUSE.COM` was the second candidate and
   it crosses to nothing in this collection (§12) — but no other VIS disc has
   a hash list yet, so **the test that matters has not been run.**

**New, from the first pressing:**

7. **Does any other disc's `CONTROL.TAT` carry the ATTENTION statement, and
   does any other name two programs?** Five minutes on discs somebody already
   holds. It decides whether the authorisation string is a platform-wide
   licensing stamp or one publisher's lawyer, and it decides whether the
   483-byte length is explained by a second program name.
8. **Does any of the other three pressings contain an NE binary?** One `find`.
   It turns "a retail VIS disc exists with no Windows executable" into a claim
   about the platform, or kills it. **This is the cheapest high-value check on
   the list.**
9. **Is `.IMV` an ICOM format or a VIS one?** 157 files, 429,867,008 bytes,
   derived in full for the first time
   ([format](https://github.com/vs-sr-dev/vis-sherlockholmes-doc/blob/master/docs/04-imv-format.md)).
   Sector-aligned segments, 15 fps, 22,050 Hz interleaved PCM, 8-bit
   palettised. If a second VIS disc from a different publisher carries one,
   it is a platform component; if not, it is ICOM's. `imv.py --validate`
   answers it in a second.
