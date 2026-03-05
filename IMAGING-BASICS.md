# DVD Imaging — Basics Guide

This guide walks through creating an ISO disk image from a DVD using Terminal on a Mac. No technical experience is required. Once imaging is complete you can refer to the README and VALIDATION-TROUBLESHOOTING files for the next steps.

---

## What you will need

- A Mac with the imaging software set up
- An external DVD drive connected to the Mac
- The DVD you want to image
- The imaging folder (where ISO files will be saved, e.g. `DVD-IMAGING` on the Desktop) open in Finder
- The `ARCHIVE-DVD-VIDEO.zsh` script file visible in Finder (it lives in a separate folder)
- An admin password for the Mac, or TouchID configured

---

## Step 1 — Open Terminal

Terminal is an app on every Mac that lets you type instructions directly to the computer. It is similar in concept to Command Prompt on Windows, but works differently.

1. Press **Command + Space** to open Spotlight Search
2. Type `Terminal`
3. Press **Return**

A window opens with a line ending in `%` — this means Terminal is ready for input.

---

## Step 2 — Navigate to the imaging folder

Terminal always works inside a particular folder. ISO files will be created wherever Terminal is pointed when the script runs, so you need to point it at your imaging folder first. Terminal may already be set up to automatically work inside the correct folder, called the "working directory". In case it is not, the simplest way to set the working directory on a Mac is to drag the folder in:

1. In Terminal, type `cd ` — that is the letter **c**, the letter **d**, then a **space**. Do not press Return yet.
2. In Finder, locate the imaging folder (e.g. `DVD-IMAGING` on the Desktop)
3. Click the imaging folder and **drag it into the Terminal window** and let go
4. The folder's full path will appear automatically after `cd `
5. Now press **Return**

The prompt will change to show the folder name, confirming you are in the right place.

> **Tip:** If you make a mistake on a line, press **Control + C** to cancel it and start again.

> **Tip:** Pressing the **up arrow key** recalls the last command you typed. This saves retyping if you need to run the script again.

---

## Step 3 — Insert the DVD

Insert the DVD into the external drive and wait around 10 seconds for the Mac to recognise it. A disc icon may appear on the Desktop — this is normal and can be ignored.

---

## Step 4 — Run the script

The script file lives in a different folder from your imaging folder, so you need to point Terminal at it directly. Use the same drag technique as Step 2, but drag the script file rather than a folder.

1. In Terminal, type `zsh ` — the letters **z**, **s**, **h**, then a **space**. Do not press Return yet.
2. In Finder, locate the `ARCHIVE-DVD-VIDEO.zsh` script file
3. Click the script file and **drag it into the Terminal window** and let go
4. The script's full path will appear automatically after `zsh `
5. Now press **Return**

A menu will appear. Type `1` and press **Return** to start imaging.

---

## Step 5 — Identify the DVD drive

The script lists the external disks currently connected to the Mac. The output looks something like this:

```
/dev/disk3 (external, physical):
   #:        TYPE NAME              SIZE       IDENTIFIER
   0:                               *4.7 GB    disk3
```

Look for the entry marked `external, physical` with a size around 4.7 GB (single-layer DVD) or 8.5 GB (dual-layer DVD). That is your DVD drive.

When prompted, type just the disk identifier — the part that reads `disk3`, `disk4`, or similar — and press **Return**. Do not include the `/dev/` part.

> **If more than one external disk is listed:** The DVD drive will be the one with the correct disc size. A USB hard drive will show a much larger size (e.g. 1 TB).

---

## Step 6 — Enter the output filename

The script will ask for a filename for the ISO. Enter the identifier for this disc — for example, the accession number or barcode — and press **Return**:

```
CA0001234567
```

Use only letters, numbers, and hyphens. No spaces.

---

## Step 7 — Enter your password

The script needs admin access to read the disc directly. A password prompt will appear:

```
Password:
```

- **If TouchID is set up:** place your finger on the sensor
- **Otherwise:** type your admin password and press **Return**

> **Note:** Nothing will appear on screen as you type your password. This is normal on a Mac — just type it and press Return.

---

## Step 8 — Wait for imaging to complete

The script will display ddrescue progress. A healthy disc typically takes 5-15 minutes. A damaged disc may take considerably longer. This process will also depend on how fast your optical disk drive is.

The progress display looks like this:

```
pct rescued:   45.23%,  read errors:    0,  remaining time:      8m
```

- `pct rescued` — how much of the disc has been copied so far
- `read errors` — number of unreadable sectors found (zero is ideal)
- `remaining time` — estimated time to completion

**Do not close Terminal or put the Mac to sleep during imaging.** The script prevents automatic sleep, but closing the lid on a laptop running on battery may still interrupt it.

**If you need to stop early:** Press **Control + C** and wait a few seconds for ddrescue to stop cleanly. Progress is saved automatically — the next run will resume from where it left off.

---

## Step 9 — Exit the script

When imaging finishes the script returns to the main menu. Type `5` and press **Return** to exit.

The ISO file will be in the imaging folder, named after the identifier you entered (e.g. `CA0001234567.iso`). A log file will also be saved there.

The disc can now be ejected.

---

## Common problems

| What you see | What to do |
|---|---|
| `command not found` or `no such file` after Step 4 | The script file was not dragged in correctly — repeat Step 4 |
| No external disk listed in Step 5 | Eject and reinsert the disc, wait 10 seconds, press Control+C and run the script again |
| Password prompt does not respond | Type your password and press Return — nothing will appear as you type |
| Imaging stops with an error | Note any message on screen, then refer to the README |
| Progress shows many read errors | The disc is damaged — let it run to completion and refer to VALIDATION-TROUBLESHOOTING |
