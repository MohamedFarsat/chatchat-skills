---
id: nano-pdf
category: Content & Media
author: openclaw
name: Nano PDF
description: Edit PDFs with natural-language instructions using the nano-pdf CLI.
requires: [".pdf", "edit instructions"]
examples:
  - "On page 1 of deck.pdf, change the title to 'Q3 Results' and fix the typo in the subtitle."
  - "Apply this edit instruction to page 3 of report.pdf and output a new PDF with the changes."
---

# nano-pdf

Use `nano-pdf` to apply edits to a specific page in a PDF using a natural-language instruction.

## Quick start

```bash
nano-pdf edit deck.pdf 1 "Change the title to 'Q3 Results' and fix the typo in the subtitle"
```

Notes:

- Page numbers are 0-based or 1-based depending on the tool’s version/config; if the result looks off by one, retry with the other.
- Always sanity-check the output PDF before sending it out.
