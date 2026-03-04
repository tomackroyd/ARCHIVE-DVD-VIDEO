# Sony DVD Recorder Authoring: Stub Chapters

## The Problem

DVDs produced on Sony consumer recorders (RDR-series, DVDirect, etc.) commonly trigger extraction validation failures — not because content is missing, but because the firmware over-reports disc duration.

Sony's firmware authors the disc using a fixed chapter grid sized to the maximum the device supports, regardless of how much content was actually recorded. Unused chapter slots are written into the IFO with nominal durations (typically 10 minutes each) but contain only empty or near-empty cells. The IFO structure is internally consistent, so MakeMKV faithfully reports the full declared duration — including the stub chapters.

**Example**: A 3h05m recording on a disc authored with a 4h30m chapter grid. MakeMKV reports 4:27:12 across 27 chapters. The final 7 chapters (~1h22m) are empty.

When the script compares VOB duration against MKV duration, the VOBs include those stub chapters, so the extracted MKV will appear to be significantly short. **Validation failure on these discs is a correct positive** — the script is accurately detecting that the MKV doesn't cover the full declared VOB duration. The disc is fine; the firmware simply lied about how long it is.

### Other Sony recorder quirks to note in accession records

- Volume label may display as `LOGICAL VOLUME IDENTIFIER` — a known firmware bug in several RDR models
- `lsdvd` may emit `libdvdread: Zero check failed` warnings — a non-compliant VMGI header, also a Sony firmware quirk
- Discs are recorded in VR mode and converted to Video mode on finalisation

---

## Identifying the Last Meaningful Chapter

Because the IFO reports uniform 10-minute durations for all chapters (real and stub alike), you cannot identify the content boundary from metadata alone. Visual inspection is required.

1. Open the ISO in VLC (File → Open File or drag onto VLC)
2. Navigate to the last few chapters using **Playback → Chapter**
3. Step backwards chapter by chapter until you find the last one containing real content
4. Note that chapter number

---

## Workaround: MakeMKV GUI with Chapter Range

The MakeMKV graphical application supports extracting a specific chapter range, which lets you exclude the stub chapters entirely.

1. Open MakeMKV
2. Go to **File → Open DVD files manually** and select the `.iso`
3. MakeMKV will display a disc information dialog, for example:
   ```
   Disc Information
   Label : LOGICAL VOLUME IDENTIFIER
   Titles count : 1
   Title information
   1: 1/1 - 27 chapter(s) 4:27:12 173 cell(s)
   ```
4. In the input field, enter a range string in the format `1:1-X`, where `X` is the last chapter containing real content — for example `1:1-19`
5. A file browser will appear — set your destination directory and enter the CA number in the Name field
6. MakeMKV will create a file named `CA0003341286-t00.mkv` (or similar)

Once you have the MKV, return to the preservation script and use **Option 4: Create access files only**, entering the path to the directory containing the MKV. The script will detect field order, apply deinterlacing, and generate the access MP4.

---

## Accession Note

When preserving a Sony recorder disc that required this workaround, record in the accession metadata:
- Disc authored on Sony consumer DVD recorder (firmware-generated stub chapters)
- Meaningful content ends at chapter N of N declared (e.g. chapter 19 of 27)
- MKV created via MakeMKV GUI chapter range `1:1-N`, excluding stub chapters
- Validation not applicable (VOB duration inflated by firmware padding)
