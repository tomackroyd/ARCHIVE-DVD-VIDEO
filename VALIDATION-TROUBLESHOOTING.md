# Cell Removal Troubleshooting

This guide explains what to do when the script halts due to MakeMKV removing cells from a title.

## What the Script Detected

When you see this message:

```
⚠️  HALTED: Cells X-Y were removed from title start
    MakeMKV has removed cells from the beginning of the title.
    The partial MKV has been deleted.
```

MakeMKV's automatic title detection heuristic identified cells at the start of the title as navigation or menu data and stripped them. The resulting MKV would be missing content from the beginning. The script has deleted the partial file and halted to prevent an incomplete preservation master being created silently.

### Why Does This Happen?

MakeMKV applies heuristics when analysing DVD title sets. On some discs — particularly non-standard or complex authoring — it incorrectly classifies content cells as menu cells and removes them. This is a MakeMKV limitation, not a disc or ISO problem.

---

## Resolution: MakeMKV GUI Manual Mode

The MakeMKV GUI provides an "Open DVD disc manually" option that bypasses the automatic heuristic and lets you specify the full chapter range explicitly. This is the most reliable fix.

### Steps

1. **Open MakeMKV** application

2. **Open the ISO**
   - File → Open Disk Image
   - Select the `.iso` file the script identified (path is shown in the halt message)

3. **Note the title duration shown** — it will be shorter than expected due to the removed cells

4. **Open the title manually**
   - Right-click the affected title, or look in the View/Tools menu for **Open DVD disc manually**
   - Enter the full chapter range, e.g. `1:1-19` (VTS 1, all chapters)
   - MakeMKV will re-analyse the title and include all cells

5. **Verify the duration** is now correct in the title list

6. **Set the output directory** to the path shown in the halt message (the script created it already)

7. **Click Make MKV** — name the output file using the preservation master convention:
   `IDENTIFIER-PM01.mkv` (e.g. `CA0001234567-PM01.mkv`)

8. **Return to the script and use Option 4** to create the access MP4 from your corrected MKV

---

## Alternative: Direct FFmpeg VOB Extraction

If MakeMKV GUI also fails, ffmpeg can concatenate the VOB files directly from the mounted ISO, bypassing MakeMKV entirely. This is lossless and includes all cells.

### Steps

1. **Mount the ISO**
   ```bash
   hdiutil attach /path/to/CA0001234567.iso -readonly -nobrowse
   ```

2. **Find the mount point** — look for the volume with a `VIDEO_TS` folder
   ```bash
   ls /Volumes/
   ```

3. **List the title VOBs** (exclude `_0.VOB` which is menus)
   ```bash
   ls -lh /Volumes/DISC_NAME/VIDEO_TS/VTS_01_*.VOB
   ```

4. **Concatenate with ffmpeg**
   ```bash
   ffmpeg -i "concat:/Volumes/DISC_NAME/VIDEO_TS/VTS_01_1.VOB|VTS_01_2.VOB|VTS_01_3.VOB" \
     -map 0 -c copy \
     -f matroska /path/to/output/CA0001234567-PM01.mkv
   ```
   Adjust the VOB list to match what is on your disc.

5. **Unmount the ISO**
   ```bash
   hdiutil detach /Volumes/DISC_NAME
   ```

6. **Use Option 4** in the script to create the access MP4

---

## After Resolving

Once you have a complete MKV in the output directory:

- Run the script and choose **Option 4**
- Enter the path to the output directory containing the MKV
- The script will detect field order, apply deinterlacing if needed, and create the access MP4

---

## Document the Workaround

Add a note to your archival metadata for the item indicating:
- That MakeMKV automatic extraction removed cells
- Which method was used to resolve it (GUI manual mode or ffmpeg)
- The MakeMKV version at the time (`makemkvcon --version`)
