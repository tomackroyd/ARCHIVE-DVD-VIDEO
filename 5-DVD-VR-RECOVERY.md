# DVD-VR Recovery on macOS

Guide to recovering content from DVD-RW discs recorded in DVD-VR (Video Recording) mode on macOS, using ddrescue and FFmpeg.

This guide was written by Claude following a Claude chat to troubleshoot DVD-VR discs that were unable to be processed by the main [ARCHIVE-DVD-VIDEO](https://github.com/tomackroyd/ARCHIVE-DVD-VIDEO/blob/main/ARCHIVE-DVD-VIDEO.zsh) shell script. Beware of possible errors!

## Background

### What is DVD-VR?

DVD-VR (DVD Video Recording) is a recording format used by consumer DVD recorders (Sony, Panasonic, Pioneer, etc.). Unlike DVD-Video, which uses a fixed VOB/IFO structure, DVD-VR uses a continuously updated Recording Management Data (RMD) layer on top of a UDF 2.0 filesystem. This allows real-time recording and editing without finalisation.

Key characteristics:

- Uses UDF Revision 2.0
- Does not require finalisation for playback (unlike DVD-Video mode)
- Video is recorded as a Video Recording Object (VRO) — a continuous MPEG-2 program stream
- Title and chapter boundaries are defined in the RMD, not embedded in the stream
- Compatible DVD recorders read the RMD natively; most computers do not

### Why macOS and Windows Cannot Read These Discs

Although macOS and Windows support UDF 2.0 in general, they do not implement the DVD-VR application layer that sits on top of it. The operating system sees the UDF volume but cannot interpret the RMD or VRO structures. The disc typically presents as blank, unformatted, or unreadable.

This is not a hardware problem, a finalisation problem, or a disc defect. It is a software compatibility problem.

Consumer DVD recorders that recorded the disc will play it back correctly because they implement the full DVD-VR specification natively.

### Why There Is No Finalise Option on the Sony Recorder

If a disc is in VR mode and no finalise option appears in the recorder menu, this is expected behaviour — not a symptom of a problem. DVD-VR discs do not require finalisation in the same way DVD-Video discs do. The absence of a finalise option is diagnostic evidence that the disc is in VR mode.

---

## Symptoms

| Symptom | Likely Cause |
|---|---|
| Disc presents as blank or unformatted on Mac/Windows | Normal DVD-VR behaviour — OS lacks VR parser |
| `drutil status` reports Sessions: 0, Tracks: 0, Space Used: 0 | Drive firmware cannot interpret VR management structures |
| `hdiutil attach` fails with "no mountable file systems" | Expected — macOS cannot mount VR filesystem |
| HandBrake reports "not a DVD" | HandBrake's libdvdread requires DVD-Video structure |
| MakeMKV cannot read disc or ISO | MakeMKV requires DVD-Video structure |
| Disc plays on original recorder but not on computer | Confirms VR mode — recorder has native VR support |

---

## Recovery Workflow

### Requirements

- macOS with Homebrew
- FFmpeg: `brew install ffmpeg`
- ddrescue: `brew install ddrescue`
- External optical drive connected directly (not via hub)

### Step 1 — Confirm the disc device

With the disc inserted:

```zsh
diskutil list
```

Look for an entry with no recognised partition table, typically showing the disc capacity (e.g., 1.5 GB). Note the device identifier (e.g., `disk4`).

```zsh
drutil status
```

This confirms disc type, capacity, and whether the drive sees content. On DVD-VR discs, Sessions and Tracks will typically report as 0 and Space Used as 0 — this is normal and does not indicate a blank disc.

### Step 2 — Extract directly with FFmpeg

FFmpeg can read the raw MPEG-2 program stream directly from the disc device, bypassing the VR filesystem layer entirely:

```zsh
ffmpeg -f mpeg -i /dev/disk4 -c copy IDENTIFIER_extracted.mkv
```

Replace `/dev/disk4` with your actual device identifier and `IDENTIFIER` with your disc identifier. The `-c copy` flag performs a lossless remux — no re-encoding occurs.

This is the recommended primary workflow. Testing has confirmed that extracting directly from the disc device produces output identical to extracting from a ddrescue ISO image.

### Step 3 (Optional) — Create a ddrescue ISO first

If you prefer a block-level archival image before extraction:

```zsh
sudo ddrescue -n /dev/disk4 IDENTIFIER.iso IDENTIFIER.log
```

Then extract from the ISO:

```zsh
ffmpeg -f mpeg -i IDENTIFIER.iso -c copy IDENTIFIER_extracted.mkv
```

See the [ddrescue behaviour section](#ddrescue-behaviour-on-dvd-vr-discs) below before deciding whether this step is necessary.

### Step 4 — Verify the output

```zsh
mediainfo IDENTIFIER_extracted.mkv
```

Expected output for a standard Sony HSP recording:

| Field | Expected value |
|---|---|
| Format | Matroska |
| Video codec | MPEG Video, Version 2, Main@Main |
| Resolution | 720 × 576 |
| Frame rate | 25.000 FPS |
| Standard | PAL |
| Scan type | Interlaced |
| Audio codec | AC-3 (Dolby Digital) |
| Audio channels | 6 (5.1 surround) or 2 (stereo) |
| Audio bit rate | 448 kb/s (5.1) or 192/256 kb/s (stereo) |

---

## ddrescue Behaviour on DVD-VR Discs

### Read errors in unwritten disc regions

When running ddrescue on DVD-VR discs, significant read errors are expected and normal. Testing on multiple Sony DVD-RW discs recorded in HSP mode produced the following consistent pattern:

- The first pass (`-n`) rescues the recorded content successfully
- ddrescue reports a block of unrecoverable data, typically 40–360 MB
- Retry passes (`-r3`) fail to recover any additional data from this block, accumulating thousands of read errors
- Despite these errors, the extracted MKV file contains the complete recorded content

**The explanation:** DVD-VR discs do not pre-allocate the full disc surface. Unwritten regions — space beyond the last recording — produce read errors because there is no valid data signal at those locations. The drive reports these regions as bad sectors. They are not damaged; they are simply blank disc surface.

The correlation between the size of the unrecoverable block and the `Space Free` value reported by `drutil status` confirms this interpretation.

**Practical implication:** The `-r3` retry pass is unnecessary for these discs. The first pass (`-n`) is sufficient to capture all recorded content. The unrecovered block will always remain unrecovered because it contains no data.

**Recommended ddrescue command for DVD-VR:**

```zsh
sudo ddrescue -n /dev/disk4 IDENTIFIER.iso IDENTIFIER.log
```

Do not run retry passes on these discs — they will fail and waste significant time without recovering any additional content.

### Why the ISO approach may still be useful

- Provides a block-level archival record of the disc exactly as it existed
- Allows extraction to be repeated without the physical disc
- Useful if the physical disc is marginal or at risk of further degradation
- The ISO can be stored as an archival object even if its VR filesystem cannot be mounted

---

## Limitations of FFmpeg Extraction

### Titles are merged into a single stream

FFmpeg extracts the raw MPEG-2 program stream without reference to the VR title map (RMD). All titles on the disc are joined into a single continuous MKV file. Title and chapter boundaries are not preserved.

To recover individual titles, play the extracted MKV in VLC and note the timecodes where content transitions occur. Then split using FFmpeg:

```zsh
ffmpeg -i IDENTIFIER_extracted.mkv -ss 00:00:00 -to 00:12:00 -c copy title1.mkv
ffmpeg -i IDENTIFIER_extracted.mkv -ss 00:12:00 -to 00:24:00 -c copy title2.mkv
ffmpeg -i IDENTIFIER_extracted.mkv -ss 00:24:00 -to 00:36:27 -c copy title3.mkv
```

The `-c copy` flag is lossless — these operations simply reposition the container boundaries.

### Timestamp warnings

During extraction, FFmpeg will produce a large number of messages similar to:

```
[vost#0:0/copy @ ...] Non-monotonic DTS; previous: 45936, current: 45912; changing to 45936.
```

This is expected. The VR-mode stream has occasional out-of-order packet timestamps due to the way Sony recorders write the stream across recording sessions. FFmpeg corrects these automatically. The output file will have monotonic timestamps. These messages do not indicate data loss or corruption.

---

## macOS Software Compatibility

| Tool | DVD-VR support |
|---|---|
| VLC | Partial — plays content but title/chapter navigation is greyed out |
| FFmpeg | Yes — raw stream extraction works reliably |
| HandBrake / HandBrakeCLI | No — requires DVD-Video structure |
| MakeMKV | No — requires DVD-Video structure |
| hdiutil | No — cannot mount VR filesystem |
| macOS Finder | No |

No macOS tool currently provides full DVD-VR title enumeration and discrete title extraction. FFmpeg raw stream extraction is the practical state of the art on macOS for this format.

On Windows, IsoBuster provides better VR-mode filesystem navigation and may be able to enumerate titles from healthy discs.

---

## Optical Drive Compatibility

Testing was conducted with:

- **LG BH16NS55** (Blu-ray burner) — produced read errors consistent with the unwritten space issue described above; otherwise functional
- **LG GP60NB50** (DVD-only slim portable) — same behaviour; no improvement over the Blu-ray drive for this format

The read errors on unwritten disc regions appear to be a characteristic of the disc format rather than a drive-specific issue. Both drives successfully imaged the recorded content in the first ddrescue pass.

If a disc appears genuinely damaged (rather than simply having unwritten space), trying multiple drives may help, as laser calibration differences can affect readability of marginal media.

---

## Relation to the Main DVD-Video Workflow

This DVD-VR workflow is separate from and does not replace the main `ARCHIVE-DVD-VIDEO.zsh` script, which is designed for DVD-Video discs with standard VOB/IFO structure. DVD-VR discs cannot be processed by that script because:

- MakeMKV cannot read VR-mode discs or ISOs
- There is no VIDEO_TS structure or IFO navigation layer
- The VR filesystem is not recognised by libdvdread

Use this guide for discs originating from consumer DVD recorders. Use the main script for pressed or authored DVD-Video discs.
