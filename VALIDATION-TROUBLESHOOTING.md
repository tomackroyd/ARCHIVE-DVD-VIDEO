# Extraction Validation Troubleshooting

This guide provides workarounds when the script's extraction validation fails, indicating that MakeMKV did not extract all DVD content.

## Understanding Validation Failures

### What Does Validation Failure Mean?

When you see this message:
```
⚠️  VALIDATION FAILED: Extracted MKVs are significantly shorter than source VOBs!
    This indicates incomplete extraction - some DVD content is missing.
    Possible causes: MakeMKV cell extraction issue, complex DVD structure
```

The script has detected that the total duration of your extracted MKV files is less than 98% of the total duration of the source VOB files on the ISO. This means **MakeMKV missed some content during extraction**.

### Example

**Expected**: DVD contains 1 hour 45 minutes (105 minutes) of content
**Extracted**: MKV files total only 23 minutes
**Result**: 82 minutes of content is missing (~78% lost)

### Why Does This Happen?

MakeMKV occasionally fails to extract all "cells" from DVD titles, particularly with:
- Complex DVD structures with multiple angles or programs
- DVDs with non-standard authoring
- Discs with certain copy protection schemes
- DVDs that use unusual cell ordering or branching

This is a **MakeMKV limitation**, not an issue with your disc or ISO.

---

## Workaround 1: Direct FFmpeg Extraction (Recommended)

This method bypasses MakeMKV by directly concatenating and remuxing VOB files from the ISO.

### Prerequisites

You need your ISO file. The script already created this in Option 1.

### Steps

1. **Mount the ISO** (macOS will usually auto-mount when you double-click it)
   ```bash
   open /path/to/CA0001234567.iso
   ```
   Or manually:
   ```bash
   hdiutil attach /path/to/CA0001234567.iso -mountpoint /tmp/dvd_mount
   ```

2. **Navigate to the VIDEO_TS folder**
   ```bash
   cd /Volumes/VOLUME_NAME/VIDEO_TS
   # or
   cd /tmp/dvd_mount/VIDEO_TS
   ```

3. **Identify the title set** you need (usually VTS_01)
   ```bash
   ls -lh VTS_*.VOB
   ```

   You'll see files like:
   - `VTS_01_0.VOB` - Menu/navigation
   - `VTS_01_1.VOB` - Main content part 1
   - `VTS_01_2.VOB` - Main content part 2
   - etc.

4. **Extract using ffmpeg concat**

   For a single title (e.g., VTS_01):
   ```bash
   ffmpeg -i "concat:VTS_01_1.VOB|VTS_01_2.VOB|VTS_01_3.VOB|VTS_01_4.VOB|VTS_01_5.VOB" \
     -map 0 -c copy \
     -f matroska ~/Desktop/CA0001234567-PM01.mkv
   ```

   **Important**: Only include numbered VOBs (VTS_01_1.VOB, not VTS_01_0.VOB which is menus)

5. **Unmount the ISO**
   ```bash
   hdiutil detach /Volumes/VOLUME_NAME
   # or
   hdiutil detach /tmp/dvd_mount
   ```

### Verification

Check the duration of your extracted MKV:
```bash
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 ~/Desktop/CA0001234567-PM01.mkv
```

Compare this to the validation report's "VOB total duration" to confirm completeness.

### Notes

- This method preserves **all streams** (video, audio, subtitles) without re-encoding
- The resulting MKV is lossless - identical quality to source
- You may need to repeat for multiple title sets (VTS_02, VTS_03, etc.)
- Some DVDs have multiple audio tracks or subtitle tracks - they're all preserved

---

## Workaround 2: HandBrake CLI

HandBrake is an alternative DVD extraction tool that may succeed where MakeMKV fails.

### Installation

```bash
brew install handbrake
```

### Steps

1. **List titles on the ISO**
   ```bash
   HandBrakeCLI --scan -i /path/to/CA0001234567.iso
   ```

   Look for output like:
   ```
   + title 1:
     + duration: 01:45:32
     + size: 720x480, pixel aspect: 32/27, display aspect: 1.33, 29.970 fps
   ```

2. **Extract a specific title**
   ```bash
   HandBrakeCLI -i /path/to/CA0001234567.iso \
     -t 1 \
     -o ~/Desktop/CA0001234567-PM01.mkv \
     --format av_mkv \
     -e copy:mpeg2 \
     -a 1,2,3,4 -E copy \
     --subtitle scan --subtitle-forced
   ```

3. **Extract all titles**
   ```bash
   for title in {1..5}; do
     HandBrakeCLI -i /path/to/CA0001234567.iso \
       -t $title \
       -o ~/Desktop/CA0001234567-PM0${title}.mkv \
       --format av_mkv \
       -e copy:mpeg2 \
       -a 1,2,3,4 -E copy \
       --subtitle scan
   done
   ```

### Verification

Use the validation check from Workaround 1 to verify extracted duration.

### Notes

- `copy:mpeg2` preserves original video without re-encoding
- `-a 1,2,3,4` includes first 4 audio tracks (adjust as needed)
- HandBrake may handle DVD navigation/branching differently than MakeMKV
- Some DVDs may still fail if protection is too complex

---

## Workaround 3: Try MakeMKV with Different Settings

While MakeMKV has limited configuration options, you can try adjusting the minimum title length.

### Steps

1. **Mount your ISO** (if not already mounted)
   ```bash
   open /path/to/CA0001234567.iso
   ```

2. **Try with shorter minimum length** (default in script is 5 seconds)
   ```bash
   makemkvcon --minlength=1 mkv iso:/path/to/CA0001234567.iso all ~/Desktop/output_dir/
   ```

3. **Or extract specific titles**

   First, list titles:
   ```bash
   makemkvcon -r info iso:/path/to/CA0001234567.iso
   ```

   Then extract specific title:
   ```bash
   makemkvcon mkv iso:/path/to/CA0001234567.iso 0 ~/Desktop/output_dir/
   ```
   (Replace `0` with the title number you want)

### Notes

- This rarely solves cell extraction issues but worth trying
- May produce many small title fragments
- Check duration of each extracted file to identify the main content

---

## Workaround 4: Use MakeMKV GUI

The graphical interface sometimes provides more control and visibility into extraction issues.

### Steps

1. **Open MakeMKV application**

2. **Open your ISO**
   - File → Open Disk Image
   - Navigate to your `.iso` file

3. **Review the title list**
   - MakeMKV shows all detected titles with durations
   - Look for titles that match your expected content length
   - Check for warnings or errors in the log panel

4. **Select titles manually**
   - Uncheck titles you don't need (menus, trailers, etc.)
   - Check that duration matches expectations

5. **Set output directory** and click "Make MKV"

6. **Review log output** for errors or warnings

### Notes

- GUI may reveal specific errors not visible in CLI
- Some DVDs show multiple "versions" of the same content - compare durations
- Log panel at bottom shows detailed extraction progress

---

## Workaround 5: Report to MakeMKV Developers

If none of the above work, the issue may require MakeMKV developer attention.

### Steps

1. **Create a backup of the problematic ISO**
   - Keep a copy for testing

2. **Visit the MakeMKV forum**
   - https://forum.makemkv.com/

3. **Search for similar issues**
   - Your DVD title + "cells" or "incomplete"

4. **Post in the "MakeMKV for Mac" section**
   - Include:
     - DVD title/product information
     - Validation results (VOB duration vs MKV duration)
     - MakeMKV version (`makemkvcon --version`)
     - Any error messages from logs

5. **Provide dump file if requested**
   - Developers may ask for a small diagnostic file
   - This helps them fix the issue in future versions

### Notes

- MakeMKV developers are responsive to bug reports
- Your report helps improve the tool for everyone
- Meanwhile, use Workarounds 1 or 2 to complete your archival work

---

## After Using a Workaround

Once you've successfully extracted complete MKV files using one of the workarounds above:

### 1. Generate Access Files

Return to the script and use **Option 4**:
```bash
cd /path/to/script/directory
zsh ARCHIVE-DVD-VIDEO.zsh
# Choose Option 4
# Enter path to directory containing your MKV files
```

The script will:
- Detect field order automatically
- Apply appropriate deinterlacing
- Generate H.264 MP4 access files

### 2. Verify Output

**Check MKV duration:**
```bash
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 /path/to/file.mkv
```

**Check MP4 was created:**
```bash
ls -lh /path/to/output/directory/*.mp4
```

**Play files** in VLC or IINA to verify quality

### 3. Document the Issue

Add a note to your archival metadata indicating:
- Which extraction method was used
- That MakeMKV failed validation
- The workaround that succeeded

This helps if you need to regenerate files later.

---

## Prevention: When to Skip MakeMKV

If you consistently encounter validation failures with certain types of DVDs:

### Consider going straight to ffmpeg/HandBrake for:

- Educational DVDs (often complex branching)
- Training/corporate DVDs (non-standard authoring)
- Foreign DVDs (different regional standards)
- DVDs with known extraction issues (check forums)

### MakeMKV works best with:

- Mainstream commercial DVDs
- Standard Hollywood releases
- Simple disc structures
- Single main title + extras

---

## Need Help?

If these workarounds don't solve your issue:

1. **Check the main README.md** for general troubleshooting
2. **Visit MakeMKV forums** for DVD-specific issues
3. **Check FFmpeg documentation** for advanced options
4. **Consider professional archival services** for particularly valuable/rare content

---

## Summary of Methods

| Method | Pros | Cons | Success Rate |
|--------|------|------|--------------|
| **FFmpeg Direct** | Lossless, fast, complete control | Requires manual VOB selection | High |
| **HandBrake CLI** | Good DVD handling, many options | May still fail on complex discs | Medium-High |
| **MakeMKV Settings** | Easy, familiar interface | Limited control over cell extraction | Low |
| **MakeMKV GUI** | Visual feedback, manual selection | Same cell extraction limitations | Medium |
| **Report Bug** | Helps future users, may get fix | Doesn't solve immediate problem | N/A |

**Recommended order**: Try FFmpeg Direct first (fastest, most reliable), then HandBrake if needed.
