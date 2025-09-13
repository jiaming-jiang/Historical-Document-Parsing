# Brevet OCR & Metadata Extraction — Handoff Guide

A concise guide to run, understand, and extend the notebook-driven pipeline that OCRs **tome PDFs**, detects **BREVET** header blocks with bounding boxes, extracts **per‑patent metadata**, and **matches** results to a reference database.

---

## 1) Project in one page
**Context**: You have scanned “tome” PDFs (multi‑page). We need reliable OCR, better accuracy via B/W preprocessing, detection of the **BREVET** header block per page (with coordinates), extraction of patent metadata (number/date/page), and fuzzy matching against a master database.

**Goal**: Produce structured tables (CSV/XLSX) per tome with: page‑level OCR text, per‑block bounding boxes, extracted metadata, and links/matches to the reference DB.

**What exists (root notebooks)**
- **OCR.ipynb** — PDF→images→OCR text (baseline)
- **BW.ipynb** — single B/W preprocessing pipeline + OCR
- **BW_test.ipynb** — compare multiple B/W variants (diagnostic)
- **brevet_bbox_detection.ipynb** — find *BREVET* lines + compute bounding boxes
- **Detection.ipynb** — regex/heuristic parsing of OCR text → per‑patent records
- **new_pipeline.ipynb** — integrated flow (OCR DF → text → metadata → bbox)
- **Fuzzymatch.ipynb** — join extracted records to `patent_database.xlsx`
- **OCR-EVAL.ipynb** — quick quality metrics to choose the best B/W approach

---

## 2) Quick setup
- **Python** 3.10, **Tesseract** installed (ensure it’s on PATH)
- Install deps: `pip install -r requirements.txt`
- Recommended folders (optional):
  - `data/raw/` (PDFs), `data/pages/` (page_###.png), `data/ocr/` (text), `data/tables/` (XLSX)

---

## 3) Tutorial — end‑to‑end in ~6 steps
> Prefer **new_pipeline.ipynb** for a single, integrated run. If you want the clearer, modular path, follow the steps below.

1) **Render & baseline OCR** — *OCR.ipynb*
- **In**: `tome_XX.pdf`
- **Out**: `page_###.png`, `extracted_text_XX.txt`
- Notes: set DPI, `lang` (e.g., `fra`), PSM/OEM if needed.

2) **Pick the best B/W preprocessing (optional but recommended)** — *BW_test.ipynb* → *BW.ipynb*
- **BW_test** compares 2–4 variants side‑by‑side on sample pages.
- **BW** runs the chosen variant over the tome.
- **Out**: preprocessed `page_###.png` and `extracted_text_XX_BW.txt`.

3) **Detect BREVET blocks** — *brevet_bbox_detection.ipynb*
- **In**: page images (baseline or B/W)
- **Out**: table with `(page_id, x0, y0, x1, y1, score)` per detected block.

4) **Extract metadata** — *Detection.ipynb* **or** *new_pipeline.ipynb*
- **In**: OCR text (`extracted_text_*.txt`)
- **Out**: `ocr_extracted_patents_XX*.xlsx` (records with patent number/date/page, etc.)
- Tip: `new_pipeline.ipynb` can also enrich blocks with their coordinates.

5) **Fuzzy match to the reference DB** — *Fuzzymatch.ipynb*
- **In**: `ocr_extracted_patents_*.xlsx`, `patent_database.xlsx`
- **Out**: `matched_patents.xlsx` / `matched_patents_with_ID.xlsx`
- Notes: cleans text, normalizes dates, applies fuzzy match (consider `rapidfuzz`).

6) **Evaluate OCR quality (optional)** — *OCR-EVAL.ipynb*
- **In**: `extracted_text_*.txt`
- **Out**: quick metrics (entropy, dictionary hit rate) to justify your B/W choice.

---

## 4) Inputs/Outputs at a glance
- **Inputs**: `tome_XX.pdf`, optional `patent_database.xlsx` (columns: id/title/date/...)
- **Intermediates**: `page_###.png`, `extracted_text_XX[_BW].txt`
- **Detections**: `brevet_blocks_XX.csv` (or DF) with page + bbox + score
- **Metadata**: `ocr_extracted_patents_XX*.xlsx`
- **Matches**: `matched_patents*.xlsx`

---

## 5) What to do next
- Standardize parameters (DPI, PSM/OEM, threshold method) in **new_pipeline.ipynb**.
- Move shared helpers into a small `src/` module; import in notebooks.
- Add a simple CLI for headless runs; persist intermediate OCR DFs (parquet).
- Tighten parsers/regexes and date normalization; constrain fuzzy candidates by tome/page.
- Consider `rapidfuzz` for speed and add a few unit tests for the parsers.

---

## 6) Troubleshooting (quick)
- **Tesseract not found** → set `pytesseract.pytesseract.tesseract_cmd` to the executable path.
- **Poor OCR** → increase DPI; try adaptive thresholding + denoise; tweak `psm`.
- **BBoxes look off** → confirm coordinate system and that OCR is run with `image_to_data`.
- **Bad matches** → normalize accents/punctuation; lowercase consistently; limit candidates.
