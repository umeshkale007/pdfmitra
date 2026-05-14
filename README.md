# pdfmitra — पीडीएफ मित्र

A [Claude Code](https://docs.claude.com/en/docs/claude-code) slash command for **lossless PDF operations**: merge, split, extract pages/text/tables, watermark, rotate, OCR.

> **मित्र** (*mitra*) — Sanskrit/Hindi for "friend". Your PDF friend in the terminal.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Plugin-orange)](https://docs.claude.com/en/docs/claude-code)

---

## What it does

Type `/pdfmitra <operation> <args>` in any Claude Code session and the assistant runs the right tool, verifies the output, and reports back. Designed for **zero quality loss** — pypdf object-level operations and qpdf where available, never re-encoding streams or images unless you explicitly ask.

| Operation | Syntax | Example |
|---|---|---|
| **merge** | `merge <folder> [output.pdf]` | `/pdfmitra merge ~/Documents/Chapters` |
| **split** | `split <file.pdf> <output_dir>` | `/pdfmitra split book.pdf ./pages/` |
| **extract-pages** | `extract-pages <file> <range> <out>` | `/pdfmitra extract-pages book.pdf "1-5,8,10-12" excerpt.pdf` |
| **extract-text** | `extract-text <file> [out.txt]` | `/pdfmitra extract-text report.pdf` |
| **extract-tables** | `extract-tables <file> <out.xlsx>` | `/pdfmitra extract-tables filings.pdf tables.xlsx` |
| **watermark** | `watermark <file> <wm.pdf> <out>` | `/pdfmitra watermark draft.pdf confidential.pdf out.pdf` |
| **rotate** | `rotate <file> <deg> <out>` | `/pdfmitra rotate scan.pdf 90 fixed.pdf` |
| **ocr** | `ocr <scanned.pdf> [out.txt]` | `/pdfmitra ocr scan.pdf` |

## Why lossless matters

Many "online PDF merger" tools re-encode every page through a JPEG pipeline, dropping image quality, breaking embedded fonts, and losing hyperlink annotations. `pdfmitra` uses **pypdf's object-level append** (`PdfWriter.append()`), which copies page objects byte-for-byte — text, fonts, images, bookmarks, and links are preserved.

For a 932-page, 16-file academic textbook merge: source total **19.47 MB**, output **19.60 MB** (slightly *larger*, confirming no compression).

---

## Install

### Option A — Claude Code plugin (recommended)

In any Claude Code session:

```
/plugin install github.com/umeshkale007/pdfmitra
```

The `/pdfmitra` slash command becomes available immediately and travels with your Claude Code config.

### Option B — Standalone slash command

If you'd rather not install a full plugin, drop the command file directly:

```bash
mkdir -p ~/.claude/commands
curl -fsSL https://raw.githubusercontent.com/umeshkale007/pdfmitra/main/pdfmitra.md \
  -o ~/.claude/commands/pdfmitra.md
```

That's it — restart your Claude Code session and `/pdfmitra` is live.

### Option C — Project-local

To make `/pdfmitra` available only in one repo (and commit it for your team):

```bash
mkdir -p .claude/commands
curl -fsSL https://raw.githubusercontent.com/umeshkale007/pdfmitra/main/pdfmitra.md \
  -o .claude/commands/pdfmitra.md
git add .claude/commands/pdfmitra.md
```

---

## Requirements

Auto-installed via `pip3 install --user` on first use:

- **pypdf** — merge, split, extract pages, rotate, watermark
- **pdfplumber** — text extraction with layout, table extraction
- **openpyxl** — write extracted tables to `.xlsx`

Optional (only needed for `ocr`):

- **pytesseract**, **pdf2image**, **Pillow** — Python OCR bindings
- **tesseract** binary — `brew install tesseract` (macOS) or `apt install tesseract-ocr` (Linux)
- **poppler** — `brew install poppler` / `apt install poppler-utils`

The slash command checks for missing libraries and installs them on demand. For OCR, the assistant asks before running `brew install`.

---

## Built-in discipline

The slash command's operating doctrine (see [commands/pdfmitra.md](commands/pdfmitra.md)) enforces:

1. **Lossless by default** — object-level pypdf operations, never re-encoding.
2. **Verify after writing** — every output is re-opened and page count / content is confirmed before reporting success.
3. **Natural sort** — files named `01_, 02_, 10_` merge in numeric order, not lexicographic.
4. **Skip non-PDFs** — ignores `.DS_Store`, `Thumbs.db`, anything not ending in `.pdf`.
5. **Preserve input** — never modifies source files; always writes to a new output path.
6. **Path handling** — expands `~`, resolves iCloud Drive paths, quotes paths with spaces.
7. **Refuse to overwrite** — asks before clobbering an existing output file.
8. **Surface failures honestly** — page count mismatch, missing tables, empty OCR text are warnings, never silent passes.

---

## Examples

### Merge a folder of chapters

```
/pdfmitra merge ~/Documents/ICAI_Material
```

Output:
```
Operation:    merge
Input:        ~/Documents/ICAI_Material (16 files, 932 pages)
Output:       ~/Documents/ICAI_Material/ICAI_Material_MERGED.pdf (19.60 MB)
Verification: 932/932 pages — OK
```

### Pull tables from a financial filing into Excel

```
/pdfmitra extract-tables 10-K.pdf q4_tables.xlsx
```

Each detected table lands on its own sheet (`p3_t1`, `p7_t2`, …) in the workbook.

### OCR a scanned contract

```
/pdfmitra ocr scanned_contract.pdf
```

Pages are rendered at 300 DPI and passed through Tesseract; output is written to `scanned_contract.txt` next to the source.

---

## How `/pdfmitra` differs from the bundled `pdf` skill

Claude Code's bundled `anthropic-skills:pdf` skill is a *reference guide* — the assistant reads it and decides how to implement each task. `pdfmitra` is a **direct slash command**: one invocation, fixed contract, verified output, terse report. Use the bundled skill for exploratory PDF work; use `/pdfmitra` when you know exactly what you want done.

---

## Contributing

Issues and PRs welcome. The slash command lives in [`commands/pdfmitra.md`](commands/pdfmitra.md); the top-level [`pdfmitra.md`](pdfmitra.md) is a copy for direct-download installs (keep both in sync).

When adding a new operation:

1. Add the row to the **Operations supported** table in `commands/pdfmitra.md`.
2. Add a reference implementation snippet.
3. Add an example in this README.
4. Sync the two `pdfmitra.md` copies.

---

## License

[MIT](LICENSE) © Umesh Kale. Use it however you like.
