# Contraluz

> Hold your documents up to the light.

**Contraluz** is an open source desktop app that inspects documents (PDF, DOCX, DOC, ODT, RTF) for hidden content designed to manipulate LLMs, such as invisible text, misleading metadata and concealed layers. It then produces a version that contains **only what a human reader can actually see**.

The name comes from the Portuguese *contraluz*: holding a sheet of paper up against the light to reveal what is hidden in it.

> **Status:** early development. Nothing is usable yet.

## Why

Documents are increasingly read by AI before (or instead of) people. Attackers have started hiding instructions inside them, for example white text on a white background or a 1pt font, so that a human sees one thing and the model reads another. In Brazil this has already been reported in court filings aimed at steering AI-assisted judicial decisions ([Fantástico/G1, Nov 2026](https://g1.globo.com/fantastico/noticia/2026/11/01/comandos-ocultos-ia.ghtml)).

Contraluz gives lawyers, courts and anyone else who feeds documents to an LLM a simple way to check a file and use only its visible content.

## Principles

1. **100% local and offline.** Documents never leave your machine. The app makes no network requests.
2. **Never execute document content.** No macros, JavaScript or OLE objects are ever run.
3. **Explainable findings.** Every finding reports its type, location, the hidden text and the evidence (for example, text color vs. background color).
4. **Low false-positive rate.** Alerts must be trustworthy enough for legal work.
5. **Visible content first.** The main output is the visible-only text; the report complements it.

## What it detects (planned)

| Technique | PDF | DOCX/ODT | DOC/RTF |
|---|:-:|:-:|:-:|
| Text colored like the background (white on white) | ✓ | ✓ | ✓ |
| Tiny or zero-scale fonts | ✓ | ✓ | ✓ |
| Text outside the visible page area | ✓ | | |
| Text covered by images or shapes | ✓ | ✓ | ✓ |
| Invisible render mode / zero opacity | ✓ | | |
| Hidden text property, comments, tracked changes | | ✓ | ✓ |
| Suspicious metadata (title, keywords, XMP, custom properties) | ✓ | ✓ | ✓ |
| Annotations, form fields, hidden optional content layers | ✓ | | |
| Embedded files and OLE objects | ✓ | ✓ | ✓ |
| Image alt text | ✓ | ✓ | |
| Invisible Unicode (zero-width, tag characters, bidi controls) | ✓ | ✓ | ✓ |
| Tampered text mapping (what you see ≠ what is extracted) | ✓ | | |

## How it works

Contraluz compares **what a human sees** with **what a machine extracts**:

- **Deterministic checks (primary):** for every extracted character, inspect its color, size, render mode and position, and compare its color with the rendered pixels behind it.
- **OCR cross-check (secondary):** render each page, run OCR and diff the result against the extracted text to catch what the rules miss.
- **Format-specific rules** for metadata, hidden properties, layers, attachments and invisible Unicode.

## Tech stack

- **Rust** (edition 2024)
- **[GPUI](https://github.com/zed-industries/zed/tree/main/crates/gpui)** for the native, GPU-accelerated desktop UI (no webview)
- **PDFium** via [`pdfium-render`](https://github.com/ajrcarey/pdfium-render) for PDF rendering and per-character text attributes
- [`lopdf`](https://github.com/J-F-Liu/lopdf) for low-level PDF structure
- [`zip`](https://github.com/zip-rs/zip2) + [`quick-xml`](https://github.com/tafia/quick-xml) for DOCX/ODT
- [`just`](https://github.com/casey/just) as the task runner

## Planned layout

```
crates/core     # parsers, detectors, findings model, sanitization (no UI)
crates/cli      # command-line interface over the core
apps/desktop    # GPUI desktop app
fixtures/       # one test document per detection rule
```

## Contributing

The project is just getting started. Issues and ideas are welcome, especially **real-world samples of hidden-content techniques** (please strip any personal or confidential data first). Every detection rule must ship with a fixture that reproduces it.

## License

Licensed under the [Apache License, Version 2.0](LICENSE).
