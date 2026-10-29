> **Download:** get the Windows ZIP from the [Releases page](https://github.com/InexperiencedPotato/PhotoCull/releases/latest). PhotoCull's own source code is not published in this repository (see [LICENSE.txt](LICENSE.txt)); sections below about running from source or building describe the private development tree. Third-party notices: [THIRD_PARTY_NOTICES.txt](THIRD_PARTY_NOTICES.txt) and [LICENSES](LICENSES).

# PhotoCull 0.1.0 (development build)

A local, offline Windows desktop tool for culling photos: browse a folder, mark photos Keep or Reject,
find exact and visually similar duplicates, get blur/exposure hints, and copy the Keep photos elsewhere.
Everything runs on your computer. There is no network access, account, telemetry or AI.

**Status:** runs from source, or as an unsigned Windows ZIP build (see [Windows build](#windows-build)). There is no installer. See [Known limitations](#known-limitations).

## Support PhotoCull

PhotoCull is free and every feature stays free. If it saves you time, you can optionally
[support it on Ko-fi](https://ko-fi.com/inexperiencedpotato). Donations are never required, and the app itself
contains no donation prompts, links or network access.

## Run from source

Requires Windows and Python 3.12 (`requires-python >=3.12,<3.13`).

```
python -m venv .venv
.venv\Scripts\pip install -e .[dev]
.venv\Scripts\python -m photocull
```

Tests: `.venv\Scripts\python -m pytest -q`

## Windows build

`powershell -ExecutionPolicy Bypass -File packaging\build.ps1` builds `dist\PhotoCull-0.1.0-win64.zip` in a fresh
`build\venv` (PyInstaller 6.22.3, `--onedir --windowed`, every package version pinned by
`packaging\constraints.txt`), runs the tests and a headless frozen self-test first. For a release candidate, build
into new folders so nothing existing is touched:
`packaging\build.ps1 -Out release-candidates\0.1.0-rc1 -Work release-candidates\0.1.0-rc1\build`.
The script refuses to overwrite an existing build or ZIP, and fails if a licence file named in
`THIRD_PARTY_NOTICES.txt` is missing from the ZIP.

Installing: unzip the whole ZIP to any folder (for example `%LOCALAPPDATA%\Programs\PhotoCull`) and run `PhotoCull\PhotoCull.exe`. Keep the `_internal` folder next to it. No Python is needed, but
Microsoft's current Visual C++ Redistributable (x64) must be installed; PhotoCull will not start without it. Get it
from https://aka.ms/vc14/vc_redist.x64.exe (Windows Update does not always provide it). If it is missing, Windows or
the launcher fails before PhotoCull can show its own message (typically "Failed to load Python DLL ...
python312.dll"); install the redistributable and start again. The ZIP contains no Microsoft runtime files and
PhotoCull copies none: the build changes one import-name string in each of NumPy's two C++ extension modules so they
use your installed `msvcp140.dll` (exact change: `LICENSES\NumPy-import-patch.txt`, `packaging\unmangle_msvcp.py`).
PhotoCull writes only to `%LOCALAPPDATA%\PhotoCull` and to the folder you export to, never to its own folder.

First launch: the exe is **unsigned**, so for a downloaded ZIP Windows SmartScreen may show "Windows protected your
PC" ("More info" -> "Run anyway") and antivirus products may scan or flag it. Check the ZIP's SHA-256 against the
published value before running it. `PhotoCull\PhotoCull-selftest.exe` runs the headless checks on synthetic images
in a temporary folder. Licence texts are in `LICENSES\` and `THIRD_PARTY_NOTICES.txt` next to the `PhotoCull`
folder.

## Appearance: dark mode only

PhotoCull is **dark-mode only**. It always uses its own dark design, whatever the Windows light/dark setting is,
and there is no light theme or theme switch. The colours, spacing and type sizes are defined in one place
(`src/photocull/ui/theme.py`). The Windows folder-selection dialogs are drawn by Windows, not by PhotoCull, and
follow the Windows setting.

## Using it

Open a folder (Ctrl+O); subfolders are included. Supported types: .jpg, .jpeg, .png, .webp, .bmp, .tif, .tiff (no RAW).

| Action | What it does |
|---|---|
| Arrow keys, **K** / **X** / **U** | Move; mark Keep / Reject / Undecided |
| **Find Duplicates** | Groups byte-identical files (SHA-256). Badge `D<n>`. |
| **Find Similar** + threshold box | Groups visually similar photos (64-bit difference hash). Badge `S<n>`. |
| **Check Blur/Exposure** | Adds heuristic warnings: `blur?`, `dark?`, `bright?`. |
| **Show** filter | All, Keep, Reject, Undecided, Duplicates, Similar, Possibly blurry, Exposure warnings. An active filter is highlighted, with a **Clear filter** button and an "N of M photos" count |
| **Contact Sheet…** | Makes a printable PDF or 300-dpi PNG contact sheet of Keep photos, all photos or the current view (see below) |
| **Export Kept…** | Copies Keep photos into a folder you choose |
| **Cancel** | Stops the running scan, analysis, contact sheet or export (shown only while one is running) |

### Contact sheets

**Contact Sheet…** opens a dialog with a live preview of page 1 (page arrows below it). Choose the photos (Keep,
All photos, or the Current view, i.e. what the Show filter displays), A4 or Letter, portrait or landscape,
2–8 columns, narrow/normal/wide margins, an optional title and date, and whether to print page numbers, file
names and Keep/Reject labels. **Create…** asks for a destination folder, then writes either one multi-page
**PDF** (photos embedded at 200 dpi) or one **PNG per page** (300 dpi).

- Photos are placed in natural file-path order (IMG_2 before IMG_10), whatever order the grid shows. Each is
  fitted whole inside its square, never cropped. Colours are plain sRGB; this is not a colour-proofing tool.
- The preview uses the small cached thumbnails; your originals are opened (read-only) only while the sheet
  is created. A photo that cannot be read gets a grey "Unreadable" cell, is listed under Errors, and the sheet
  is still produced.
- Same destination rules as Export Kept: an existing folder outside the source folder, nothing overwritten
  (`name_1.pdf`), no partial files (each file is written under a temporary name and renamed when complete;
  Cancel leaves no PDF, and for PNG only whole pages that were already finished).
- The same settings and the same photos give byte-identical files on the same PC. The PDF's creation time and
  random IDs are replaced by constants; sheet text uses the bundled Noto Sans (SIL OFL 1.1; a subset is embedded in PDFs), so sheets look the same on every PC.

Duplicate, similar and quality results are suggestions. They never change a decision, never reorder or
reject anything, and are not stored as decisions.

## Your photos are not modified

- Source photos are only ever opened for reading. PhotoCull never deletes, moves, renames or edits them.
- The only things it writes outside its own data folder are **Contact Sheet** files and **Export Kept** copies,
  both into a folder you pick, never into the source folder. **Export Kept** copies (`copy2`: contents and
  timestamps) into a folder you pick:
  - the destination must exist and must not be the source folder or inside it (checked on resolved paths);
  - existing files are never overwritten; a name clash becomes `name_1.ext`, `name_2.ext`, …;
  - one failed file does not stop the rest, and failures are listed in the summary.
- Automated tests check that the SHA-256, size and modification time of every source file are unchanged
  after scanning, thumbnails, previews, all analyses, export, and after cancelling each of them.

## Where PhotoCull keeps its own data

All under `%LOCALAPPDATA%\PhotoCull\`:

| Path | Contents |
|---|---|
| `photocull.db` (+ `-wal`, `-shm`) | Keep/Reject decisions and hash caches (SQLite) |
| `thumbs\` | Cached 192 px thumbnails (regenerated if missing) |
| `logs\photocull.log` (+ `.1`–`.3`) | Rotating log, 512 KB each: errors, per-file problems, crash details |

A decision is shown only while the file's size and modification time still match what was recorded.
To reset everything, close the app and delete that folder; photos are not affected. Deleting `thumbs\`
alone is safe.

**Damaged database:** if `photocull.db` is corrupt, it is renamed to `photocull.db.corrupt-<date-time>`
(with its `-wal` data) and a new empty database is created; you are told where the old one is. Nothing is
deleted or overwritten, but earlier decisions will not be shown until you restore that file by hand.
If the database is merely locked, read-only or inaccessible, it is left untouched and PhotoCull explains
why it cannot start. A failed decision save is reported in the Errors list and the decision is not applied.

## What the analyses can and cannot tell you

- **Exact duplicates** are byte-identical files. Cached hashes are re-verified before a group is trusted.
- **Similar** is a heuristic: a 64-bit difference hash, grouped so that *every* pair in a group is within
  the threshold (default 6 of 64 bits). It ignores rotation unless stored in EXIF, can group smooth
  gradients, skips almost uniform images, and group membership can depend on file order.
- **Blur/Exposure** are heuristic warnings, not quality judgments. Blur is a Laplacian-variance score on an
  image reduced to at most 800 px, with a 4×4 tile layout (default warning below 100). Exposure flags
  images where at least 70% of pixels are at or below brightness 40, or at or above 215. Expect false
  warnings on fog, sky, minimalist or studio shots, silhouettes, night scenes, snow and deliberate low/high
  key photos; noise can hide real blur. **The thresholds were chosen on synthetic images and have not
  been validated on real photographs.** The check is repeated each time you run it (results are not stored).

## Known limitations

- The ZIP build is unsigned, has no installer or icon, and has not been tested on a freshly installed (clean)
  Windows machine; it was tested on the build machine with a stripped PATH and environment.
- Tested on Windows only. GUI tests ran on Qt's offscreen platform with simulated input; the layout was also
  rendered and inspected on the native Windows platform at 1200x750, 1600x1000 and 900x600 (one display scale,
  one machine). Other DPI settings and multiple monitors have not been verified.
- Real-photo accuracy of similar/blur/exposure results is unverified.
- Measured only with synthetic images, up to 10,000. Larger sets (for example 50,000) are untested and
  not supported. Analyses run one file at a time; a first Find Similar on 10,000 12-megapixel photos is
  estimated at around 5 minutes, and Check Blur/Exposure at around 13 minutes (estimates, not measured).
- Paths longer than about 260 characters may fail to read on Windows unless long paths are enabled; such
  files appear in the Errors list.
- The thumbnail cache is never pruned and grows with the number of photos viewed (`thumbs\` can be deleted).
- There is no guard against running two copies of PhotoCull on the same data folder; this is untested.
- Old database rows are not cleaned up when photos are moved or renamed.
- No RAW, face recognition or AI ranking.

## Licensing of dependencies

PySide6-Essentials 6.12.0 (LGPL-3.0 / GPL / commercial), Pillow 13.0.0, numpy 2.5.4 and
opencv-python-headless 5.0.0.93. Notices for the ZIP build are in `THIRD_PARTY_NOTICES.txt` and `packaging\licenses\`;
open licensing points are listed there. `packaging\fetch_qt_attributions.py` regenerates the Qt third-party
licence texts when the Qt version changes. PhotoCull itself is free to use and share under the terms in `LICENSE.txt` (copyright InexperiencedPotato).
