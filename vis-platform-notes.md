# VIS platform notes — a checklist for the next disc

A running checklist for the **Tandy / Memorex Video Information System** (1992),
carried from one documentation pipeline to the next and added to by each.

This document started in an unusual position, and the position is worth stating
before anything else: **the machine was partly known and its discs were not.**
**Two** retail pressings have now been documented end to end, thirteen days
apart, and they disagree with each other about the thing this document is least
sure of. The first did not confirm what §1 expected; the second does. Both
boxes are in §1, and neither has been deleted to make room for the other.

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
| **[N of 5]** | Measured on retail pressings. **Five are in hand and two are documented**; the denominator moves as more are opened. Older `[N of 3]` and `[N of 4]` marks that a new disc did not exercise are left where they are and re-declared as such rather than silently renumbered. |
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

### **[2 of 5] The second retail pressing confirms every word of the section the first one contradicted**

[*Race the Clock*](https://github.com/vs-sr-dev/vis-racetheclock-doc), Tandy /
Mindplay, 1992, 3,625 files, 374,094,820 bytes, **cut thirteen days after
*Sherlock Holmes* by the same `Maketat` build**:

| | |
|---|---|
| NE binaries | **1** — `RTC.EXE`, 136,192 B, `e_lfanew` = 0x400, target OS 2, expects Windows 3.10 |
| `.INI` | **2** — `WIN.INI` and `SYSTEM.INI`, 765 bytes together |
| how it is launched | `SYSTEM.INI` says `shell=rtc.exe`. **The title is the Windows shell**, in text, on a retail disc, for the first time |
| how it puts pixels on screen | MCI and GDI. No `DISPDIB`, no `INT 10h`, no `0xA000` |
| how it reads input | **it imports `HC`, function `HCCONTROL`, by name** (§7) |
| import table | KERNEL, GDI, USER, WIN87EM, MMSYSTEM, **TVUI**, HC |
| what its vendor block names | **`minwin A:`** — Modular Windows, from the CD |
| files containing a VIS-aware token | **4 of 3,625** — `CONTROL.TAT`, `RTC.EXE`, both `.INI`, 137,430 B = **0.0367 %** |

**So the section is now one for and one against, and that is a better state
than either sample alone.** Two retail pressings, thirteen days apart, same
mastering tool build: one is a real-mode DOS product with a Tandy vendor block
on it and one is exactly the Modular Windows application this document
described before anybody opened anything.

The third of the four readings offered above — *"this is a DOS master with a
Tandy vendor block bolted on, and the finding is about ICOM's production, not
the console"* — is the one that survives best, and there is now a byte-level
reason to prefer it: **each disc's vendor block names its own launch path.**
*Sherlock*'s program list is `A:MOUSE.COM` then `SHI.EXE`, two DOS programs.
*Race the Clock*'s is `minwin A:`, the Modular Windows kernel on the CD drive.
The two discs are launched differently and each says so in the same field
(§2).

**[2 of 5] And a module this document has never named: `TVUI`.** Two ordinals
imported, and the data segment carries the window class name `tvBUTTON` in two
cases. It is not on the disc, so it is in ROM. On the reading its name
suggests it is Modular Windows' television-safe control library, and that is a
name and two ordinals and no more.

## 2. `CONTROL.TAT` — the vendor block, and the first thing to read [5 of 5]

Every VIS disc carries a root file called **`CONTROL.TAT`**. It is small — 473,
474 or **483** bytes on the four discs in hand — and it is the closest thing
this platform has to the CD-i pre-file-system region or the Amiga CD `.TM`
block: a vendor-supplied structure that travels with the mastering chain rather
than with the title.

**[5 of 5]** The first **84 bytes are byte-identical on all five discs**, from
five unrelated developers:

```
Copyright (c) 1992 Tandy Corporation. All Rights Reserved.    \0\0
                  \r\n

md5 ed9bfc904220e409f04c0772f1797ff7   (first 84 bytes)
```

The five discs are *Atlas of U.S. Presidents* (Applied Optical Media),
*Bible Lands, Bible Stories* (Context/InterMedia), *Fitness Partner*
(Computer Directions), *Sherlock Holmes: Consulting Detective, Vol. I*
(ICOM Simulations) and *Race the Clock* (Mindplay / Methods & Solutions).
Different studios, different subjects, different genres, same 84 bytes. **Hash the first 84 bytes of `CONTROL.TAT` on any new disc**;
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

**[5 of 5] The file names the tool that made it, and dates it.** Near the end of
every one of the five:

```
Atlas            minwin A:\    Maketat - Version is 1(12) 31-Aug-92
Bible            minwin A:     Maketat - Version is 1(12) 31-Aug-92
Fitness          minwin a:\    Maketat - Version is 1(13)  9-Oct-92
Sherlock         fdiv          Maketat - Version is 1(12) 31-Aug-92
Race the Clock   minwin A:     Maketat - Version is 1(12) 31-Aug-92
```

**And the left-hand column of that table has always been two different
fields.** See the box below before using it.

So `CONTROL.TAT` is produced by a Tandy tool called **Maketat**, it stamps its
own build version and date into every disc it makes, and **two builds of that
tool are visible across four discs** — three on `1(12) 31-Aug-92` and one on
`1(13) 9-Oct-92`. That is a mastering-chain fingerprint of exactly the kind
section 3 of the Amiga CD notes is built on: it dates the *pressing process*,
independently of anything the title claims about itself. Collect the `Maketat`
version on every disc; the set of versions and their date ranges is a
platform-level result that no single disc can produce.

**[2 of 5] CORRECTION, AND A RETRACTION OF A CORRECTION — there are two
fields here and this document put them in one column.**

This document originally read the left-hand column above as follows:

> ~~Note the third field too — `minwin A:\`, `minwin A:`, `minwin a:\`. It is
> the path the tool read from, it is inconsistent in case and in its trailing
> separator, and **it accounts for the one-byte length difference** between the
> Bible disc's 473 bytes and the other two's 474.~~

The first documented pressing then contradicted it, and this document carried
the contradiction as a `[1 of 4]` correction:

> ~~On *Sherlock Holmes: Consulting Detective* that field, at offset `0xB0`,
> reads **`fdiv`** … Whatever it holds, it is not "the path the tool read
> from". **Three samples were enough to describe a pattern and not enough to
> name a field.**~~

**Both are wrong, and the second disc says why.** *Race the Clock* has `fdiv`
at `0xB0` **too**, and `minwin A:` at `0x1A3` — which is the offset where
*Sherlock* has its **program list**, `A:MOUSE.COM \0 SHI.EXE \0`.

| offset | Sherlock | Race the Clock |
|---|---|---|
| `0x0B0` | `fdiv` | **`fdiv`** |
| `0x1A3` | `A:MOUSE.COM` `\0` `SHI.EXE` `\0` | **`minwin A:` `\0`** |

So `fdiv` at `0x0B0` is **[2 of 2]** on the discs whose offsets anybody has
recorded, and it was never the field this table tabulated. What the table
tabulated is the **program list at `0x1A3`**, and `minwin A:` is an entry in
it — Modular Windows, launched from the CD drive.

**[5 of 5] And the length falls straight out of it.** If the only thing that
varies is a NUL-terminated program list at `0x1A3`, the total length is a
constant plus its length:

```
disc            program list at 0x1A3       len  base   pred actual
Atlas           minwin A:\                   11   463    474    474  MATCH
Bible           minwin A:                    10   463    473    473  MATCH
Fitness         minwin a:\                   11   463    474    474  MATCH
Sherlock        A:MOUSE.COM|SHI.EXE          20   463    483    483  MATCH
Race the Clock  minwin A:                    10   463    473    473  MATCH
                                                                 5 of 5
```

**`CONTROL.TAT` = 463 + the length of its NUL-terminated program list**, on
every disc the platform has. The offsets agree independently: `Maketat` sits at
`0x1AE` on *Race the Clock* and `0x1B8` on *Sherlock*, a difference of ten,
which is 483 − 473. Two derivations, same answer.

That closes this section's standing question about the 483-byte length, and it
means the two wrong readings above failed for the same reason: **three of the
five program lists happen to begin with the word `minwin`**, which made a
launch list look like a path field.

The reason this was findable at all is that
[`controltat.py`](https://github.com/vs-sr-dev/vis-racetheclock-doc/blob/master/tools/controltat.py)
prints the offset a run was **found** at rather than the offset it was expected
at. The pre-briefing that prepared the second disc listed five printable runs
and omitted `fdiv` at `0x0B0`; the tool found six.

**What is still unknown is what `fdiv` means.** Four characters, 2 of 2, and
nobody has hexdumped byte `0xB0` of the other three pressings. **That is now
the cheapest check on the platform.**

**[5 of 5] The title field is at offset 0x54 and it is where leftovers live.**
The five read:

```
Atlas            Atlas of Presidents - By Applied Optical Media Corp. Ver. 1.0
Bible            Bible Prototype Demo - for Context/InterMedia
Fitness          FITNESS PARTNER V.90 BY Computer Directions (C)1992, 1993
Sherlock         Sherlock Holmes Volume I - ICOM Simulations
Race the Clock   Race the Clock(TM) by Mindplay, Tucson, Arizona, (C) 1992
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

> **[1 of 5] The second documented disc has four sixths of it, and a
> substitution the list did not anticipate.** *Race the Clock* has **six**
> extensions and no others:
>
> | ext | files | bytes | share |
> |---|---:|---:|---:|
> | `.AVI` | 1,208 | 358,400,376 | 95.8047 % |
> | `.DIB` | 2,405 | 15,043,790 | 4.0214 % |
> | `.WAV` | 8 | 513,224 | 0.1372 % |
> | `.EXE` (NE) | 1 | 136,192 | 0.0364 % |
> | `.INI` | 2 | 765 | 0.0002 % |
> | `.TAT` | 1 | 473 | 0.0001 % |
>
> The NE `.EXE`, the `.INI` and the `.DIB` and `.WAV` are all here. **No
> `.DLL`, no `.MID`, no `.HLP`, no `.TXT`, no `.FON`, no readme and no
> licence.** And where this list says `.FLI` and `.FLC`, the disc has
> **1,208 Video for Windows 1.0 `.AVI`** — genuine RIFF, BI_RLE8, 8 bpp, two
> streams each, 22,050 Hz PCM — and **zero FLI and zero FLC**.
>
> **This is the finding on that disc that is most likely to be
> over-claimed, so it is stated narrowly here too.** A Tandy-published retail
> title shipped 1,208 AVI files, registered the extension with MCI in its own
> `WIN.INI`, and tuned the player (`SkipFrames=1`, `AccurateSeek=1`). It also
> names, in `SYSTEM.INI`, **two non-standard AVI drivers at absolute paths
> that are not on the disc** — `\mci\coreavi\mciavi.cor` and
> `\mci\modavi\mciavi.mod`. So the disc establishes what somebody shipped
> and how they configured it. **It does not establish that the stock ROM
> plays them**, and the two drivers that would most plausibly explain it are
> somewhere else.
>
> Add `.AVI` to the expectation list, with that caveat attached.

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

**[1 of 5] And three fingerprints of one mastering house, all found by
accident on the second documented disc**, each checkable on any pressing in
seconds. *Race the Clock*'s primary descriptor names
`MERIDIAN_DATA_CD_PUBLISHER` in the data-preparer field, and it left:

* **an `MD20` record at LBA 15**, in the ISO 9660 system area, where the other
  fifteen sectors are zero. Magic `MD20` — Meridian Data 2.0 — and four
  both-endian 32-bit integers, the first of which is the LBA of the record
  below. The tag occurs **once** in the whole image;
* **a free-extent record at LBA 184,141**, sixteen non-zero bytes in a
  532-sector hole of zeroes: three both-endian integers reading 184,153 (the
  volume space size), 518 (a run length) and 183,623 (where the run starts) —
  and 183,623 + 518 is the record's own address;
* **a hundredths-of-a-second counter written into every file record's timezone
  byte.** The field is a signed 15-minute offset in ECMA-119 9.1.5, legal range
  −48..+52. On that disc it takes **exactly 100 values, every one of 0..99 and
  nothing above, on 3,625 of 3,625 file records**, while all 241 directory
  records hold 0. Twelve candidate identities tested; the best explains 39 of
  3,625. Every file record's seconds field is **even** — a FAT source volume —
  and the directory records' are odd 113 times.

Any ISO 9660 reader will mark those dates invalid and none will say what they
are. `tzfield.py` will.
[`tzfield.py`](https://github.com/vs-sr-dev/vis-racetheclock-doc/blob/master/tools/tzfield.py)

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

> **[2 of 5] Still zero observed on retail, and the second disc narrows it
> from the other side.** *Race the Clock* imports **no `DISPDIB`** and never
> calls `EnterDVA`; it goes through MCI and GDI, and its `SYSTEM.INI` names
> `display.drv=vga.drv`, the stock driver. So two documented pressings, two
> different ways of avoiding `DVA` entirely.
>
> **But its interface art is 640 × 400.** Three `.DIB` at 640 × 400 × 8, one
> at 480 × 300, and 2,400 tiles at 80 × 60 — every one `BM`, 8 bpp, BI_RGB,
> 256-entry palette, with all three size arithmetics closing on 2,405 of
> 2,405. A DIB's dimensions are not a display mode and `StretchDIBits` exists,
> so what this establishes is that **a retail title authored its full-screen
> art at twice the documented horizontal resolution**, and not what the screen
> did with it. **A column worth keeping: interface art resolution.**

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

> **[2 of 5] The three numbers turn up in a retail `SYSTEM.INI`, arrived at
> from the opposite direction.** *Race the Clock* carries:
>
> ```
> [drivers]              [vwavmidi]
> wave=vwavmidi.drv      channel=7
> midi=vwavmidi.drv      port=220
> timer=timer.drv        int=7
> ```
>
> **Port 220, DMA channel 7, IRQ 7** — the same three values this section
> found by disassembly and experiment, written down by a publisher in 1992.
>
> **And the same disc plays its sound through the API this section says does
> not work.** `[mci] WaveAudio=mciwave.drv`, `RTC.EXE` imports `MMSYSTEM`, and
> its string table holds four MCI command strings of the form
> `open \gamescrn\tckwin.wav type waveaudio alias congrat buffer 2 wait`.
>
> **The MMSYSTEM clause is qualified, not deleted.** The driver on the `wave=`
> line is `vwavmidi.drv`, which is not the stock Windows wave driver and is
> **not on the disc** — it comes from ROM. *"`waveOut` through MMSYSTEM does
> not work"* was measured against whatever driver the homebrew environment
> had; a retail title reaches the same hardware through a Tandy driver at the
> same three numbers. Which of the two is true of the console is not something
> a disc can settle.
>
> Digitised audio, both discs: *Sherlock Holmes* 143,276,490 B at 22,050 Hz;
> *Race the Clock* 53,972,192 B at 22,050 Hz, in 1,208 AVI audio streams plus
> eight `.WAV` plus a 515-byte `WAVE` resource inside the executable.
> **22,050 Hz on 2 of 2 discs and on every sample either of them carries.**

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
  > **[1 of 5] First retail confirmation.** *Race the Clock* imports seven
  > modules and makes 89 relocation fixups, of which **exactly one is by name:
  > `HC` → `HCCONTROL`.** The detector fired on the first retail import table
  > anybody has been able to read, which is also the first one that exists.
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
> **[1 of 5] And on the second documented disc the section's method applies
> in full, for the first time.** *Race the Clock*'s `RTC.EXE` gives up, from
> its own tables and without a disassembler: two segments (21,741 B of code,
> 6,830 B of auto-data — **20.98 % of the file**, the other 75.81 % being
> resources); seven imported modules; 89 relocation fixups; a module name
> `RTC`; a `.DEF` `DESCRIPTION` of `MindPlay Race The Clock`; six exported
> window procedures (`MAIN` `PLAY` `LEVEL` `TIMER` `AVI` `HELP`); 22 `BITMAP`
> and 11 `STRING` resources and one of a **custom type named `WAVE`**
> containing a 515-byte RIFF; and 59 strings including the whole user
> interface in two languages, where **the Spanish string id is always the
> English id plus ten, on 27 of 27 pairs**.
>
> The capability list this section promised, in one command:
> `KERNEL GDI USER WIN87EM MMSYSTEM TVUI HC`.
> [`ne.py`](https://github.com/vs-sr-dev/vis-racetheclock-doc/blob/master/tools/ne.py)
> reads all of it and takes `--validate`, `--imports`, `--strings` and
> `--dump`.
>
> **[1 of 5] The `WIN87EM` trap, from the other side.** This section says,
> `[authoring]`, that linking anything from `<math.h>` pulls in the
> floating-point runtime and requires `WIN87EM.DLL`, *"which the machine does
> not have"*. **A retail title imports `WIN87EM` ordinal 1 and does not ship
> the DLL** — that disc has no `.DLL` at all — and its data segment carries
> Microsoft C's `M6101: MATH` floating-point message table, so the mechanism
> described here is exactly what happened. Either the ROM carries `WIN87EM`
> or the note is over-stated. **The disc cannot say which, and this is the
> sharpest open question either documented pressing has produced.**
>
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

> **[2 of 5] And the second disc names the ceiling in its own headers, 1,208
> times.** Every one of *Race the Clock*'s AVI files declares
> `dwMaxBytesPerSec` = **153,600**, which is exactly 75 sectors a second times
> 2,048 — the sustained rate of a 1× Mode 1 drive, the number itself and not an
> approximation. Measured against the chunks that are really there, the mean
> payload rate is **70,578.6 B/s, 0.46×**, and the worst single file is 0.87×.
> Each clip is 80 × 60, about four seconds long, and about a quarter of a
> megabyte.
>
> **And the copy-versus-seek trade is the whole disc.** *Sherlock Holmes*
> shipped its interface art three times over — 4,913,992 B, **1.1024 %** — and
> this section called that a design signature. On *Race the Clock*:
>
> | | |
> |---|---|
> | files | 3,625 |
> | **distinct SHA-1** | **1,314** |
> | files sharing bytes with another file | 3,601 |
> | **all copies but one** | **300,346,116 B = 80.2861 % of the declared disc** |
>
> Sixty filmed clips, each written out **nine, ten or eleven times** — once
> into every one of the forty rounds that uses it — and forty drawn 480 × 300
> boards, each written out twice, once for each language. All forty rounds
> draw a **distinct** fifteen-clip set from the pool of sixty.
>
> **Expect intra-disc duplication and measure it** was the right instruction
> and the wrong order of magnitude. It is one twenty-eight-second hash pass and
> on this platform it can be four fifths of the object.

## 10. Baselines, so you can tell signal from noise

*Five discs are in hand and **two** are documented. Their rows are filled from
measurement; the other three still have only what §2 gave up.*

| Disc | Year | Studio | `Maketat` | TAT len | Title field | Files | Declared bytes | NE binaries | Imports `HC.DLL` | DVA mode | Digitised audio |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Atlas of U.S. Presidents | 1992 | Applied Optical Media | 1(12) 31-Aug-92 | 474 | `Atlas of Presidents … Ver. 1.0` | — | — | — | — | — | — |
| Bible Lands, Bible Stories | 1992 | Context/InterMedia | 1(12) 31-Aug-92 | 473 | `Bible Prototype Demo …` | — | — | — | — | — | — |
| Fitness Partner | 1992/93 | Computer Directions | 1(13) 9-Oct-92 | 474 | `FITNESS PARTNER V.90 …` | — | — | — | — | — | — |
| **[Sherlock Holmes: Consulting Detective, Vol. I](https://github.com/vs-sr-dev/vis-sherlockholmes-doc)** | **1992** | **ICOM Simulations** | **1(12) 31-Aug-92** | **483** | **`Sherlock Holmes Volume I - ICOM Simulations`** | **193** | **445,760,838** | **0** | **n/a — no import table** | **none; VGA mode 13h via `INT 10h`** | **yes, 143,276,490 B of 22,050 Hz PCM** |
| **[Race the Clock](https://github.com/vs-sr-dev/vis-racetheclock-doc)** | **1992** | **Mindplay / Methods & Solutions** | **1(12) 31-Aug-92** | **473** | **`Race the Clock(TM) by Mindplay, Tucson, Arizona, (C) 1992`** | **3,625** | **374,094,820** | **1** | **yes — `HC.HCCONTROL`, by name** | **none; MCI and GDI, no `DISPDIB`** | **yes, 53,972,192 B of 22,050 Hz PCM** |

Two columns this disc made me want, which is what the instruction above says
to do:

| Disc | Programs named in `CONTROL.TAT` | VIS-aware files, of all files |
|---|---|---|
| Sherlock Holmes | **`A:MOUSE.COM` then `SHI.EXE`** — two, both real-mode DOS | **1 of 193** (`CONTROL.TAT`), 483 B of 445,760,838 = **0.000108 %** |
| **Race the Clock** | **`minwin A:`** — one, the Modular Windows kernel on the CD | **4 of 3,625** (`CONTROL.TAT`, `RTC.EXE`, both `.INI`), 137,430 B = **0.0367 %** |

Both are cheap on any pressing and both separate this disc from what §1
expected, which is the test for keeping a column. **With a second row they
also separate the two discs from each other**, which is what a baseline table
is for and what a single row can never show.

Three more columns the second disc made me want:

| Disc | Program share | Intra-disc duplication | Interface art resolution |
|---|---|---|---|
| Sherlock Holmes | 119,618 B = **0.0268 %** | 7 files, 4,913,992 B, **1.1024 %** | two `.DIB`, 90 × 90 |
| **Race the Clock** | 136,192 B = **0.0364 %** | 3,601 files, 300,346,116 B, **80.2861 %** | three `.DIB` at **640 × 400**, one at 480 × 300 |

**Both discs put their program at three hundredths of one per cent of
themselves**, which is now the most stable number on the platform and the one
that says what a VIS title actually is: a large pile of media addressed by a
very small Windows or DOS program that leans on a runtime in ROM.

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

**[2 of 5] Two hash lists exist, and both crossed to zero.**
`vis-racetheclock-doc/notes/sha1-all.txt` — 3,625 records, 374,094,820 bytes,
**1,314 distinct hashes**. Crossed at file granularity against 77 repositories,
339 hash lists and 122,878 tokens: **0 hits.**

And the second list demonstrates this section's own thesis twice over. **The
unit of sharing was smaller than a file again, and this time it was inside one
object:** 1,314 distinct hashes over 3,625 files, 80.2861 % of the declared
bytes being copies, and 1,200 of 1,200 board tiles byte-identical between the
English and Spanish halves of the same disc while 0 of 600 video clips are.
A name-keyed comparison matches 1,800 of 1,800 paths and says nothing about
which of them are the same bytes; only the hash pass can tell you.

**And `CONTROL.TAT` still cannot be crossed.** Five discs carry it, two have
hash lists, and the two lists' files are 473 and 483 bytes and are **not**
identical — they differ in the title field, at `0x98`, at `0xC6` and in the
program list. The 84-byte block is the shared unit and no file-level list will
ever see it. What the fifth sample did produce is arithmetic: **473 + (program
list length − 10)** predicts all five total lengths exactly (§2).

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

*Two retail discs have now been documented. Each question below carries what
each did to it.*

1. **How many `Maketat` versions are there, and over what dates?**
   **[2 of 5] Converging harder.** Two builds across five discs —
   `1(12) 31-Aug-92` on Atlas, Bible, Sherlock and **Race the Clock**,
   `1(13) 9-Oct-92` on Fitness. Four of five on the older build, and the two
   documented pressings were cut 1992-11-11 and 1992-11-24 with a tool build
   a third disc had already superseded seven weeks earlier. Still costs
   nothing per disc.
2. **Does any commercial title use a display mode other than 320×200×8?**
   **[2 of 5] Still does not close, from a second direction.** *Race the
   Clock* imports no `DISPDIB`, never calls `EnterDVA`, names
   `display.drv=vga.drv` and goes through MCI and GDI — and its full-screen
   art is **640 × 400 × 8**. Two documented pressings, two different ways of
   not answering the question, and one of them authoring at twice the
   documented width. Nine documented, one driven, **still zero observed**.
   The `[1 of 4]` finding below stands unchanged.
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
   **[2 of 5] Closes further, and the second disc answers the half the first
   one left.** *Race the Clock* carries 53,972,192 bytes of 22,050 Hz PCM and
   its `SYSTEM.INI` configures the wave driver at **`port=220 channel=7
   int=7`** — this section's own three numbers, from a publisher. *How* it
   reaches the hardware is now visible too: through **MCI string commands to
   `mciwave.drv` via MMSYSTEM**, with `vwavmidi.drv` on the `wave=` line. Its
   executable contains **no `mov` immediate for `0x220`, `0x388` or
   `0x389`** in 28,571 bytes of segment, so it does not touch the ports
   itself. **A commercial title reaches the VIS DAC through the driver stack,
   and the driver it names is not the stock one and is not on the disc.**
   **[1 of 4] Closes halfway, and it was two questions.** A commercial VIS
   disc **does** carry digitised audio: 143,276,490 bytes of unsigned 8-bit
   PCM at 22,050 Hz interleaved inside its video, plus 8.5 MB more in a
   separate bank — **32.1420 % of the whole disc**. But *how it reaches the
   hardware* is unknown: that executable contains **no reference to port
   `0x220` and none to `0x388`/`0x389`**. Whether a commercial title drives
   *the VIS DAC* is still open, and it is a different question from whether
   one ships PCM.
4. **What do the twelve binary bytes in `CONTROL.TAT` do?**
   **[2 of 5] A fifth sample, still unexplained.** Sherlock:
   `2F 3B 37 05 76 77 DD 2A DD 7F E3 8D`. Race the Clock:
   `2F EB 40 27 BA 6B CC 3B CC 6E F2 9C`. The first byte is `2F` on both,
   which is one byte of agreement and is recorded rather than built on. The
   byte at `0xC6` is `0x00` and `0x40` respectively.
5. **How much of a VIS disc is the product, and how much is Windows?**
   **[2 of 5] The second answer is a different zero, and having two is what
   makes it a platform result.** On *Race the Clock* every Windows module is
   in ROM — `SYSTEM.INI` names twenty-six and **twenty-five are not on the
   disc**, the exception being the title itself on the `shell=` line. So
   again **0 % of the pressing is Windows code**, arrived at from the
   opposite architecture: disc one had no Windows because it was a DOS
   product, disc two has none because the runtime is in the console. The
   product itself is 0.0364 % program and the rest media.
   **[1 of 4] CLOSES, with an answer the question did not anticipate: none of
   it.** **0 % is Windows code.** The only Windows-format bytes on the whole
   pressing are two `.DIB` totalling 16,908 — **0.0038 %** — and they are
   indexed against a Windows system palette that is not on the disc and are
   not referenced by the DOS program that is. The question was framed
   expecting a ratio; on this disc the ratio is a zero and a footnote.
6. **Is there a shared component across publishers?**
   **[5 of 5] on the 84-byte block**, and **[5 of 5] on the length model**:
   `473 + (program list length − 10)` predicts every disc's total exactly
   (§2). Two hash lists now exist and both cross to zero; the file-level test
   still cannot see the thing that is actually shared.
   **[4 of 4] on the 84-byte block.** `MOUSE.COM` was the second candidate and
   it crosses to nothing in this collection (§12) — but no other VIS disc has
   a hash list yet, so **the test that matters has not been run.**

**New, from the first pressing:**

7. **Does any other disc's `CONTROL.TAT` carry the ATTENTION statement, and
   does any other name two programs?**
   **[2 of 5] Half closed.** *Race the Clock* carries the ATTENTION statement
   at `0x14C`, with the same `0xA0` non-breaking space between `System` and
   `Title` — **2 of 2 on the discs anybody has looked at**, which is starting
   to look like a platform-wide licensing stamp rather than one publisher's
   lawyer. And the length question **closes at 5 of 5** with the model in §2,
   which is a better answer than "a second program name": the total is
   `463 + list length` on every disc, however many programs are in the list.
8. **Does any of the other three pressings contain an NE binary?**
   **[2 of 5] The question is now the interesting one rather than the
   cheap one.** *Race the Clock* does, and it is the Modular Windows
   application §1 always described. So the platform ships both kinds and the
   remaining three pressings decide the ratio. Still one `find`.
9. **Is `.IMV` an ICOM format or a VIS one?** 157 files, 429,867,008 bytes,
   derived in full for the first time
   ([format](https://github.com/vs-sr-dev/vis-sherlockholmes-doc/blob/master/docs/04-imv-format.md)).
   Sector-aligned segments, 15 fps, 22,050 Hz interleaved PCM, 8-bit
   palettised. If a second VIS disc from a different publisher carries one,
   it is a platform component; if not, it is ICOM's. `imv.py --validate`
   answers it in a second.
   **[2 of 5] The second disc carries none.** *Race the Clock* has six
   extensions and `.IMV` is not among them. One disc against, three unopened.

**New, from the second pressing:**

10. **What is `TVUI`?** A module *Race the Clock* imports two ordinals from,
    with the window class `tvBUTTON` in its data segment, and which no
    homebrew project on this platform has ever named. It is not on the disc,
    so it is in ROM. A ROM dump with a resident-name table settles it in a
    minute; three unopened pressings would say whether every Windows VIS title
    uses it.
11. **Does the ROM carry `WIN87EM`?** A retail title imports it and does not
    ship it, and §8 says the machine does not have it. One of those three is
    wrong. **The sharpest open question either documented pressing has
    produced.**
12. **What is `fdiv` at `0x0B0` in `CONTROL.TAT`?** Four characters, on 2 of
    2 discs whose offsets anybody has recorded, and never the field §2's table
    was tabulating. Nobody has hexdumped byte `0xB0` of the other three
    pressings. **Now the cheapest check on the platform.**
13. **Does any other Meridian-mastered pressing carry the `MD20` record at
    LBA 15, and the hundredths counter in its file records' timezone byte?**
    (§4.) Five seconds each, and each one turns a fact about one disc into a
    fact about a mastering house.
14. **Does the stock ROM play AVI?** *Race the Clock* ships 1,208 Video for
    Windows files, registers the extension with MCI, tunes the player, and
    names two non-standard AVI drivers at `\mci\coreavi\mciavi.cor` and
    `\mci\modavi\mciavi.mod` that **are not on the disc**. Three years of
    homebrew concluded AVI was unsupported and that the platform's movies are
    `.FLI`. **A disc cannot settle this**; a ROM dump and a `find` for
    `mciavi.cor` can.
