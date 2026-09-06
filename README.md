# OCR Solution for Unstructured Data in Tables in a PDF

> Extracting structured data from unstructured PDF tables — evolved from a **custom Computer Vision + OCR pipeline (v1)** into a **Vision-AI + Prompt Engineering approach (v2)**.

---

## Table of Contents
- [Project Overview](#project-overview)
- [Repository Structure](#repository-structure)
- [Version 1 — Computer Vision + OCR](#version-1--computer-vision--ocr)
- [Version 2 — Vision AI + Prompt Engineering](#version-2--vision-ai--prompt-engineering)
- [Version 1 vs Version 2](#version-1-vs-version-2)
- [Setup](#setup)
- [Usage](#usage)
- [Example Output](#example-output)
- [Known Limitations](#known-limitations)
- [Roadmap](#roadmap)
- [Technologies](#technologies)
- [Author](#author)
- [License](#license)

---

## Project Overview

Extracting structured information from PDF tables is hard when documents involve:

- Scanned or image-based tables
- Inconsistent layouts and merged cells
- Variable numbers of columns
- Technical/business specification tables with no fixed schema

This project explores two different solutions to that problem:

| | Approach |
|---|---|
| **v1** | Deterministic Computer Vision pipeline: detect gridlines → reconstruct table structure → crop cells → OCR each cell |
| **v2** | Vision-AI pipeline: detect and crop whole tables with Computer Vision, then hand the cropped image to a vision-language model guided by a strict extraction prompt |

v1 gives full explicit control over every step. v2 trades some of that control for the ability to generalize to new table layouts without writing new code for each one.

---

## Repository Structure

```
OCR-solution-for-unstructured-data-in-tables-in-a-pdf/
│
├── README.md
├── customized solution to extract data from tables in the pdf
│   in a structured form for business needs.ipynb      # Version 1
└── convert tables from pdf to csv file using prompt engineering
    to solve business problem.ipynb                     # Version 2
```

---

## Version 1 — Computer Vision + OCR

Table understanding happens entirely through explicit image-processing logic — no ML model interprets the table itself.

```
PDF → page images → preprocessing → detect horizontal/vertical lines
    → reconstruct table grid → detect & crop cells → remove border lines
    → OCR each cell → reassemble rows/columns → CSV
```

**Strengths:** deterministic, fully explainable, no dependency on an external AI model at inference time.
**Trade-off:** every new table layout (new merged-cell pattern, new column arrangement) may require new detection rules.

## Version 2 — Vision AI + Prompt Engineering

Table understanding happens inside a vision-capable AI model, guided by a prompt that defines strict extraction rules (attribute normalization, merge-safety rules, a zero-hallucination/strict-null policy, and an output contract requiring raw CSV).

```
PDF → page images → detect & crop table regions (Computer Vision)
    → send cropped table image(s) to a vision model with the extraction prompt
    → model returns normalized, structured CSV
```

**Strengths:** generalizes to new layouts without new code; handles merged cells and open-ended schemas better.
**Trade-off:** correctness depends on the model and the prompt; needs a null/hallucination policy to stay trustworthy, and needs accuracy spot-checks since there's no deterministic ground truth.

---

## Version 1 vs Version 2

| Aspect | Version 1 | Version 2 |
|---|---|---|
| Main technology | Computer Vision + OCR | Vision AI + Prompt Engineering |
| Table/cell detection | Explicitly programmed | AI-based understanding |
| Merged cells | Difficult to handle | Interpreted semantically |
| New table layouts | May require code changes | Prompt can often generalize |
| Control | High, explicit | Higher-level, model-dependent |
| Explainability | Every step traceable | Depends on the model's output |

---

## Setup

```bash
git clone https://github.com/YousifHisham-tech/OCR-solution-for-unstructured-data-in-tables-in-a-pdf
cd OCR-solution-for-unstructured-data-in-tables-in-a-pdf

pip install pandas numpy opencv-python pillow pdf2image tqdm google-genai
```

**External dependency:** [Poppler](https://poppler.freedesktop.org/) must be installed locally — `pdf2image` shells out to Poppler's `pdftoppm`/`pdfinfo` binaries. Point the notebook's `poppler_path` variable at its `bin/` directory.

**API key (Version 2 only):** the Gemini call requires an API key. Set it as an environment variable rather than hardcoding it in the notebook:

```bash
export GEMINI_API_KEY="your-key-here"
```

```python
import os
client = genai.Client(api_key=os.environ["GEMINI_API_KEY"])
```

---

## Usage

1. Open the notebook for the version you want to run.
2. Set the input/output folder variables (`pdf_folder`, output folder names) at the top.
3. Tune detection thresholds (`dpi`, `least_horizontal_and_vertical_length`, `thickness_thresold`) if tables in your documents aren't being detected reliably.
4. Run all cells. Each PDF gets its own output folder containing the rendered pages, cropped table images, and a final `output.csv`.

---

## Example Output

```csv
model,tv_inch_size,resolution,power_consumption_tv_on_w,net_weight_with_stand_kg
X55-A1,55,4K UHD,120,18.4
X65-A1,65,4K UHD,150,22.1
```

*(Illustrative — actual columns depend on the source document's schema.)*

---

## Known Limitations

Being upfront about the current state of both pipelines:

- **No automated error handling** around OCR/API calls yet — a single bad page or failed API call can interrupt a batch.
- **No accuracy validation loop** — Version 2's output has not yet been benchmarked against ground truth, so extraction accuracy is currently based on spot-checking, not measurement.
- **Version 2 sends all cropped tables from a PDF in a single model call** — large, table-heavy PDFs may need to be batched to stay within model context/image limits.
- **Version 1's cell/line detection thresholds are tuned per document style** — they may need retuning for documents with very different table styles or scan quality.

---

## Roadmap

The two versions are complementary, not competing — the long-term direction is a hybrid pipeline:

```
PDF → Document Analysis → Table Detection
    → [Computer Vision structure] + [Vision AI understanding]
    → Structure Validation → Extraction → Normalization
    → Quality Validation → CSV / JSON
```

Combining v1's deterministic structure detection with v2's semantic flexibility, with a validation layer that cross-checks the two.

---

## Technologies

**Version 1:** Python, OpenCV, NumPy, OCR (Tesseract), pandas, Jupyter
**Version 2:** Python, Vision-capable AI models (Gemini), Prompt Engineering, pandas, Jupyter

---

## Author

**Yousif Hisham**
AI Student, Faculty of Computers and Artificial Intelligence, Benha University
GitHub: [@YousifHisham-tech](https://github.com/YousifHisham-tech)

---

## License

This project is intended for educational and research purposes.
