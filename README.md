# fdup

`fdup` is a small Python utility for finding duplicate files across one or more directories. It uses a three-pass detection method to minimize disk reads:

1. Group files by size
2. Compute a partial SHA-256 hash of the first 64 KiB
3. Compute a full SHA-256 hash for exact duplicate detection

Confirmed duplicate candidates are also verified byte-for-byte using `filecmp`.

## Features

- Detect duplicate files by content
- Filter duplicates by filename behavior
- Exclude directories from scanning
- Filter by file extension and file size
- Export results to CSV or JSON
- Interactive duplicate deletion with dry-run support
- Optional hardlink handling and symlink traversal

## Requirements

- Python 3.11 or newer
- No third-party dependencies

---

## Installation and setup

### Linux and macOS

The file is named `fdup` without an extension. Make it executable once, then run it directly:

```bash
chmod +x fdup
./fdup /photos
```

To run it from anywhere without `./`, move it to a directory on your PATH:

```bash
mv fdup ~/.local/bin/
fdup /photos
```

### Windows

Python does not automatically recognize files without a `.py` extension on Windows.
You have two options:

**Option A – Rename to `fdup.py` and run with Python:**
```cmd
python fdup.py /photos
```
If Python is installed with default settings, `.py` files are associated with the interpreter automatically.

**Option B – Keep the name `fdup` and create a wrapper `fdup.bat`:**

Create `fdup.bat` in the same folder with this content:
```bat
@echo off
python "%~dp0fdup" %*
```
Then run:
```cmd
fdup /photos
```
This lets you use the same name as on Linux/macOS.

### Global access (run from anywhere)

Windows does not have a default user binary directory like Linux, so the best practice is to create your own:

1. Create a dedicated folder for your CLI tools, e.g. `C:\bin`
2. Move both `fdup` and `fdup.bat` into that folder
3. Add `C:\bin` to your `PATH`:
   - Search for **Environment Variables** in the Start Menu
   - Click **Environment Variables...**
   - Under **User variables**, select **Path** and click **Edit...**
   - Click **New** and type `C:\bin`
   - Click **OK** and restart your terminal

Now you can run `fdup /photos` from any directory.

---

## Usage

```bash
fdup <directory> [<directory> ...] [options]
```

### Common options

- `--all` — show all duplicate groups
- `--diff-names` — show only duplicate groups whose filenames differ (default)
- `--same-names` — show only duplicate groups whose filenames are identical
- `--case-sensitive` — compare filenames with case sensitivity
- `--exclude` / `-e` — skip named directories during scan
- `--follow-symlinks` — traverse symlinks while scanning
- `--keep-hardlinks` — treat hardlinks as duplicates instead of skipping them
- `--min-size` / `--max-size` — filter files by size in bytes
- `--ext` — include only files matching specified extensions
- `--csv` — export results to a CSV file
- `--json` — export results to a JSON file
- `--delete-interactive` — interactively choose one copy to keep and delete the rest
- `--dry-run` — simulate interactive deletion without removing files

---

## Examples

Basic scan using the default `--diff-names` mode:

```bash
fdup /photos /backup
```

Scan two directories and show all duplicate groups:

```bash
fdup /photos /backup --all
```

Show only groups whose copies share the same filename:

```bash
fdup /photos /backup --same-names
```

Exclude common repository and virtualenv folders:

```bash
fdup /projects --exclude venv .venv .git
```

Search only JPEG and PNG files larger than 1 MB:

```bash
fdup /photos --ext jpg png --min-size 1048576
```

Show only files smaller than 10 KB:

```bash
fdup /docs --max-size 10240
```

Export results to CSV and JSON simultaneously:

```bash
fdup /photos --csv report.csv --json report.json
```

Preview what would be deleted without touching anything:

```bash
fdup /photos --dry-run
```

Interactively choose which duplicate copy to keep:

```bash
fdup /photos --delete-interactive
```

Preview the interactive deletion choices without deleting files:

```bash
fdup /photos --delete-interactive --dry-run
```

Treat hardlinks as duplicates instead of skipping them:

```bash
fdup /data --keep-hardlinks
```

Traverse symbolic links during scan:

```bash
fdup /data --follow-symlinks
```

Force case-sensitive filename comparison:

```bash
fdup /data --diff-names --case-sensitive
```

---

## Notes

- The default filtering mode is `--diff-names`, which shows groups only when filenames differ.
- Use `--same-names` to find files that have identical names but live in different locations.
- `--dry-run` is useful for safely previewing `--delete-interactive` before committing to deletions.
- Symlinks are skipped by default. Use `--follow-symlinks` to traverse them.
- Hardlinks pointing to the same inode are skipped by default. Use `--keep-hardlinks` to treat them as duplicates.
