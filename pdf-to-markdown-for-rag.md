---
title: "How to convert PDFs to clean Markdown for RAG and LLMs (with tables and without page headers)"
description: "Convert PDF reports, manuals and papers into structured Markdown and JSONL for ChatGPT, Claude or a vector database. Working Python, how to strip running headers and page numbers, and how to spot pages that need a human check."
permalink: /pdf-to-markdown-for-rag/
---

# How to convert PDFs to clean Markdown for RAG and LLMs

Plain text extraction loses the two things retrieval depends on: **headings** (so a chunk knows what section it belongs to) and **tables** (so numbers stay attached to their labels). It also keeps things you don't want, such as the running header and page number repeated on every page, which end up inside your chunks.

This page shows a pipeline that fixes all three. I tested it on NIST's 48-page AI Risk Management Framework: 8 seconds, 14 tables kept, and the running headers removed from every page.

## Step 1: Structured Markdown per page

[PyMuPDF4LLM](https://pymupdf.readthedocs.io/en/latest/pymupdf4llm/) detects headings from font sizes, rebuilds tables as Markdown tables, and can return one record per page:

```python
# pip install pymupdf4llm
import pymupdf4llm

pages = pymupdf4llm.to_markdown("report.pdf", page_chunks=True)
texts = [p["text"] for p in pages]   # one Markdown string per page
```

Licence note: PyMuPDF is AGPL. Using it to convert your own files is fine; shipping it inside a closed-source product needs a commercial licence.

## Step 2: Remove running headers, footers and page numbers

A header is a line that appears at the top or bottom of most pages. Compare only the first and last three non-blank lines of each page, mask the digits so "Page 3" and "Page 4" count as the same line, and drop anything that repeats on at least half the pages:

```python
import re
from collections import Counter

def key(line):
    return re.sub(r"\d+", "#", re.sub(r"[*_#>|`]", "", line)).strip().lower()

def edges(text, n=3):
    idx = [i for i, l in enumerate(text.splitlines()) if l.strip()]
    return set(idx[:n] + idx[-n:])

counts = Counter()
for t in texts:
    lines = t.splitlines()
    counts.update({key(lines[i]) for i in edges(t)} - {""})
repeated = {k for k, c in counts.items() if c >= max(3, len(texts) // 2)}

clean = []
for t in texts:
    lines = t.splitlines()
    drop = {i for i in edges(t) if key(lines[i]) in repeated}
    clean.append("\n".join(l for i, l in enumerate(lines) if i not in drop).strip())
```

Restricting the check to page edges matters: a phrase that repeats in the body (a product name, a table header) is left alone.

## Step 3: Chunk on headings, not on character counts

Split the document at every Markdown heading, then split long sections at paragraph breaks once they pass about 400 words. Keep the heading with every chunk so the retriever and the model both know the context. Never split inside a table.

Store one JSON line per chunk with the file name, the heading and the text. That file loads directly into any vector database.

## Step 4: Flag the pages a human should check

No converter is right on every page. Three cheap checks catch most problems:

- **Almost no text on a page** (under about 15 words): a figure, a cover, or a scan with no text layer.
- **Text that came from OCR**: fine for prose, risky for numbers. Check any figures you plan to rely on.
- **Replacement characters (`�`)**: the PDF's font has no text mapping, so the page needs OCR instead.

Write these to a short report and read it before you load the data.

## What converts well and what doesn't

| Converts well | Converts poorly |
|---|---|
| Reports, policies, contracts | Fillable forms |
| Manuals and documentation | Magazine layouts with many columns |
| Research papers | Handwriting |
| Catalogs with simple tables | Tables that span pages with merged cells |

For the hard cases, layout-model tools such as [Docling](https://github.com/docling-project/docling) and [Marker](https://github.com/datalab-to/marker) do better than rule-based parsing, at the cost of speed and a GPU for large jobs. Try your worst document first, not your best.

## Skip the build

- Only need the text of PDFs at public URLs? **[PDF Text Extractor](https://apify.com/signalfeed/pdf-text-extractor?fpr=kyk4x5)** returns one row per page with scanned-PDF detection, $0.50 per 1,000 pages. *(Built by the author of this guide.)*
- Want the full pipeline above run for you? I convert PDF sets into Markdown, per-page and per-chunk JSONL and a quality report, with a free 20-page sample first: **[PDFs to clean Markdown or JSON on Fiverr](https://www.fiverr.com/arabinowe/convert-your-pdfs-into-clean-markdown-or-json-for-rag-and-llms)**.

Related: [turn a whole website into data for ChatGPT and RAG](website-to-chatgpt-rag-data.md) · [extract text from a PDF URL](pdf-url-to-text.md) · [free PDF accessibility quick check (tags, alt text, language)](https://arabinowe.github.io/pdf-accessibility/)

[← Home](README.md)
