# VIS platform notes — a checklist for the next disc

A running checklist for the **Tandy / Memorex Video Information System** (1992),
carried from one documentation pipeline to the next and added to by each.

This document starts in an unusual position, and the position is worth stating
before anything else: **the machine is partly known and its discs are not.**

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
| **[N of 3]** | Measured on retail pressings. Three are in hand; the denominator moves as more are opened. |
| **[authoring]** | Proven by writing code that runs on the machine and observing what it does. Real measurement, wrong direction — it says what the console does, not what a published disc contains. **To date the "machine" has been MAME's VIS driver, not silicon.** The console itself is genuinely scarce. |
| **[unverified]** | From the Tandy SDK documents, and not measured at all. A hypothesis with a section number. |

**A mark is never promoted because nothing contradicted it.** An `[authoring]`
finding does not become `[3 of 3]` because it seems like it ought to hold on a
retail disc — it becomes that when three retail discs say so.

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

## 2. `CONTROL.TAT` — the vendor block, and the first thing to read [3 of 3]

Every VIS disc carries a root file called **`CONTROL.TAT`**. It is small — 473 or
474 bytes on the three discs in hand — and it is the closest thing this platform
has to the CD-i pre-file-system region or the Amiga CD `.TM` block: a
vendor-supplied structure that travels with the mastering chain rather than with
the title.

**[3 of 3]** The first **84 bytes are byte-identical on all three discs**, from
three unrelated developers:

```
Copyright (c) 1992 Tandy Corporation. All Rights Reserved.    \0\0
                  \r\n

md5 ed9bfc904220e409f04c0772f1797ff7   (first 84 bytes)
```

The three discs are *Atlas of U.S. Presidents* (Applied Optical Media),
*Bible Lands, Bible Stories* (Context/InterMedia) and *Fitness Partner*
(Computer Directions). Different studios, different subjects, same 84 bytes.
**Hash the first 84 bytes of `CONTROL.TAT` on any new disc**; it is a two-second
check and it is the platform's cheapest identity test.

**[3 of 3] The file names the tool that made it, and dates it.** Near the end of
every one of the three:

```
Atlas      minwin A:\    Maketat - Version is 1(12) 31-Aug-92
Bible      minwin A:     Maketat - Version is 1(12) 31-Aug-92
Fitness    minwin a:\    Maketat - Version is 1(13)  9-Oct-92
```

So `CONTROL.TAT` is produced by a Tandy tool called **Maketat**, it stamps its
own build version and date into every disc it makes, and **two builds of that
tool are already visible across three discs**. That is a mastering-chain
fingerprint of exactly the kind section 3 of the Amiga CD notes is built on: it
dates the *pressing process*, independently of anything the title claims about
itself. Collect the `Maketat` version on every disc; the set of versions and
their date ranges is a platform-level result that no single disc can produce.

Note the third field too — `minwin A:\`, `minwin A:`, `minwin a:\`. It is the
path the tool read from, it is inconsistent in case and in its trailing
separator, and **it accounts for the one-byte length difference** between the
Bible disc's 473 bytes and the other two's 474. A structure whose length depends
on how somebody typed a drive letter is not a fixed-size struct, whatever else it
is.

**[3 of 3] The title field is at offset 0x54 and it is where leftovers live.**
The three read:

```
Atlas      Atlas of Presidents - By Applied Optical Media Corp. Ver. 1.0
Bible      Bible Prototype Demo - for Context/InterMedia
Fitness    FITNESS PARTNER V.90 BY Computer Directions (C)1992, 1993
```

**A retail pressing of *Bible Lands, Bible Stories* describes itself as a
prototype demo**, and a retail pressing of *Fitness Partner* calls itself
version .90. Neither is what the box says. This field is written once by whoever
ran Maketat and evidently never reviewed, which makes it the single most
promising leftover on a VIS disc and the reason to read it before anything else.

**[authoring]** The block can be **cloned from a retail disc onto a homebrew one
without recalculating anything**. The twelve binary bytes it carries have never
been shown to be a checksum or a signature: a copied `CONTROL.TAT` boots. That is
a fact about the platform's protection, or its absence, and it is stated here as
an observation rather than as a claim that nothing validates it — nobody has
tried to make it fail.

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
Windows help and resource files. The extensions mean what they mean on a PC of
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

## 10. Baselines, so you can tell signal from noise

*Three discs are in hand and none is documented yet. The `CONTROL.TAT` columns
are filled from §2 because they are measured; everything else waits for a
pipeline.*

| Disc | Year | Studio | `Maketat` | TAT len | Title field | Capacity used | Files | NE binaries | Imports `HC.DLL` | DVA mode | Digitised audio |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Atlas of U.S. Presidents | 1992 | Applied Optical Media | 1(12) 31-Aug-92 | 474 | `Atlas of Presidents … Ver. 1.0` | — | — | — | — | — | — |
| Bible Lands, Bible Stories | 1992 | Context/InterMedia | 1(12) 31-Aug-92 | 473 | `Bible Prototype Demo …` | — | — | — | — | — | — |
| Fitness Partner | 1992/93 | Computer Directions | 1(13) 9-Oct-92 | 474 | `FITNESS PARTNER V.90 …` | — | — | — | — | — | — |

Add a column when a disc makes you want one; delete any that never separates two
titles.

## 11. Order of work — the disc half is proposed, the machine half is not

Steps marked **[authoring]** are established practice from three homebrew
projects. The rest is a proposal and the first documented disc has standing to
rewrite it.

1. **Track layout and dump form**, before the image.
2. **`CONTROL.TAT` first.** Hash the leading 84 bytes against
   `ed9bfc904220e409f04c0772f1797ff7`; read the title field; record the `Maketat`
   version and date; note the total length. Five minutes, and it produces two
   comparable numbers plus a leftover candidate before anything else is opened.
3. **ISO 9660**: descriptor fields, then the listing.
4. **Census**: all-zero files, both date sources, sector map.
5. **`.INI` files before executables.**
6. **NE headers and import tables, as a table**, for every binary. Capability
   list for free.
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

**[3 of 3]** And this platform already has its cross-disc result, before its
first pipeline: the leading 84 bytes of `CONTROL.TAT`, identical on three
unrelated discs. That block belongs to no title, was written by a Tandy tool, and
is the VIS's answer to the CD-i bumper and the Amiga CD `.TM` block. **It is
already the cheapest identity check on the platform, and it was free.**

The lesson the CD-i family paid four discs for transfers directly: **the unit of
sharing may be smaller than a file.** On a platform whose products are Windows
applications, the obvious candidates are shared DLLs, shared resource chunks and
a runtime redistributed by more than one publisher — none of which a file-level
hash list will see if the surrounding file differs by a byte.

---

## Open questions

1. **How many `Maketat` versions are there, and over what dates?** Two builds are
   visible across three discs, six weeks apart. This is a mastering-chain
   fingerprint and it costs nothing per disc.
2. **Does any commercial title use a display mode other than 320×200×8?** Nine
   are documented; one has been driven; none has been observed on a retail disc.
3. **Does any commercial title use the DAC?** The hardware is there — this
   document was wrong about that for a while — and nobody has looked.
4. **What do the twelve binary bytes in `CONTROL.TAT` do?** They survive being
   copied between discs, so they are not a signature over the content. That is
   not the same as their being inert.
5. **How much of a VIS disc is the product, and how much is Windows?** With the
   runtime in ROM, a title can be almost nothing. Measure the ratio.
6. **Is there a shared component across publishers?** A runtime DLL, an
   authoring toolkit, a player. On a platform where three of three discs already
   share 84 bytes, this is the likeliest place for the next one.
