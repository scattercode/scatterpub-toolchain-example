# Tutorial: From Scans to a Reviewed Word Document

This tutorial walks you through the complete pipeline for this example project — from OCR'ing the sample scans to producing a clean, AI-reviewed Word document.

By the end you will have:
- A raw Markdown file assembled from the scans
- A cleaned-up version with OCR artefacts removed
- An HTML copy-edit review produced by Claude
- A Word document ready to share or import into Vellum

---

## Part 1 — Set up

### 1.1 Fork and clone the project

Fork this repository on GitHub (click **Fork** at the top right), then clone your fork:

```bash
git clone https://github.com/<your-username>/scatterpub-toolchain-example.git
cd scatterpub-toolchain-example
```

### 1.2 Pull in the toolchain

The scripts and skills live in a submodule. Initialise it:

```bash
git submodule update --init
```

### 1.3 Install system tools

These are installed once via [Homebrew](https://brew.sh). If you do not have Homebrew, install it first by following the instructions at https://brew.sh.

```bash
brew install poppler    # extracts text from PDFs
brew install pandoc     # converts between document formats
brew install tesseract  # OCR engine (fallback; the default engine is installed via Poetry below)
```

### 1.4 Install Python dependencies

The toolchain uses [Poetry](https://python-poetry.org) to manage its Python libraries. Install Poetry if you do not have it:

```bash
curl -sSL https://install.python-poetry.org | python3 -
```

Then install the toolchain's dependencies:

```bash
cd toolchain && poetry install && cd ..
```

This creates a self-contained virtual environment inside `toolchain/` and installs everything needed, including the default OCR engine (marker).

---

## Part 2 — OCR the scans

The sample scans are in `publishing/tell-me-bella/ocr/scans/clean/`. Run the OCR script to extract their text and assemble it into a single Markdown file:

```bash
poetry run --directory toolchain python toolchain/scripts/ocr-to-markdown.py \
  "publishing/tell-me-bella/ocr/scans/clean"
```

This will take a minute or two — the default OCR engine (marker) uses a machine-learning model and processes each page in turn. You will see progress as each file is processed.

When it finishes, you will find the result at:

```
publishing/tell-me-bella/ocr/tell-me-bella-raw.md
```

Open it and have a look. You will notice some issues typical of OCR output:
- Running headers (page number + title repeated at the top of each page)
- Occasional invisible characters
- Words split across lines with a hyphen

---

## Part 3 — Clean the raw output

The clean-up script removes the most common OCR artefacts automatically:

```bash
python3 toolchain/scripts/clean-ocr.py \
  "publishing/tell-me-bella/ocr/tell-me-bella-raw.md" \
  --join-hyphens
```

`--join-hyphens` joins words that were split across lines by the typesetter (e.g. `news-\npaper` becomes `news-paper`). The hyphen is kept so you can review each join and decide whether to remove it.

The cleaned file is written to:

```
publishing/tell-me-bella/ocr/tell-me-bella-clean.md
```

Compare it with the raw version — running headers will be gone, invisible characters removed, and line-break hyphens joined.

---

## Part 4 — AI copy-edit with Claude Code

Open [Claude Code](https://claude.ai/code) in this project folder. You will need two repositories open as a workspace:

1. **scatterpub-toolchain-example** (this project)
2. **scatterpub-toolchain** — clone it alongside if you have not already:
   ```bash
   git clone https://github.com/scattercode/scatterpub-toolchain.git
   ```
   Then add it to your Claude Code workspace.

With both repositories open, load the copy-editor skill:

```
/copyeditor
```

Claude will ask for the path to the Markdown file. Provide:

```
publishing/tell-me-bella/ocr/tell-me-bella-clean.md
```

Claude will read the file, work through it chapter by chapter, and write a self-contained HTML review to:

```
publishing/tell-me-bella/review/tell-me-bella-review.html
```

Open that file in any web browser. Each issue is shown as a colour-coded card:

| Colour | Category | Used for |
|---|---|---|
| Red | TYPO | Spelling errors, wrong or missing words |
| Orange | PUNCT | Quotation marks, dashes, ellipsis |
| Yellow | STYLE | British English, capitalisation, numbers |
| Blue | CONSISTENCY | The same word formatted differently in different places |
| Purple | QUERY | Ambiguous phrasing — flagged for a human to decide |

Work through the review and make any edits you agree with directly in `tell-me-bella-clean.md`.

---

## Part 5 — Generate a Word document

Once you are happy with the cleaned Markdown, convert it to a Word document:

```bash
python3 toolchain/scripts/md-to-docx.py \
  "publishing/tell-me-bella/ocr/tell-me-bella-clean.md"
```

The result is written to:

```
publishing/tell-me-bella/ocr/tell-me-bella-clean.docx
```

This Word document is ready to:
- **Share with a human editor** — they can read and annotate it in Word or Google Docs
- **Import into Vellum** — open Vellum, choose *Import Word Document*, and select the file; each chapter heading will be detected as a chapter break

---

## Next steps

This example covers five pages. For a full book project:

1. Prepare clean per-page PDFs for all pages and add them to `ocr/scans/clean/`
2. Re-run the OCR and clean-up scripts
3. Run the copy-editor on the full text
4. Generate the final Word document

See the [scatterpub-toolchain README](https://github.com/scattercode/scatterpub-toolchain) for instructions on setting up a new book project from scratch, and for documentation on all the available scripts.
