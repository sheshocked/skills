---
name: ocr-document-ai
description: 
category: ai-tools
tags: [ocr-document-ai]
---

## When to Use
Extract structured data from PDFs and images: OCR, layout analysis, table extraction.

## Tools
- **Tesseract**: Open-source OCR for simple documents
- **PaddleOCR**: Multi-language OCR with layout detection
- **DocTR**: Deep learning-based document understanding
- **LlamaParse**: LLM-powered document parsing

## Key Pipeline
```python
# PaddleOCR example
from paddleocr import PaddleOCR
ocr = PaddleOCR(use_angle_cls=True, lang='en')
result = ocr.ocr('document.png', cls=True)
for line in result[0]:
    print(f"{line[1][0]} (confidence: {line[1][1]:.2f})")
```

## Pitfalls
- **Image quality**: DPI < 300 reduces accuracy significantly
- **Handwriting**: Most OCR tools struggle with handwriting
- **Tables**: Use specialized table extraction models
- **Multi-column**: Layout analysis needed to read in correct order

## Verification
- Measure character accuracy vs ground truth
- Test with various document types
- Verify table structure preservation