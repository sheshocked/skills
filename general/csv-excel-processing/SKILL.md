---
name: csv-excel-processing
description: 
category: general
tags: [csv-excel-processing]
---

## When to Use
Process CSV/Excel files: batch processing, streaming, format conversion, data validation.

## Python Processing
```python
import pandas as pd

# Read large CSV in chunks
chunks = pd.read_csv("large.csv", chunksize=10000)
for chunk in chunks:
    process(chunk)

# Excel with openpyxl
df = pd.read_excel("data.xlsx", sheet_name="Sheet1")
df.to_excel("output.xlsx", index=False, engine="openpyxl")

# CSV validation
def validate_csv(path, required_cols):
    df = pd.read_csv(path)
    missing = set(required_cols) - set(df.columns)
    if missing:
        raise ValueError(f"Missing columns: {missing}")
```

## Command Line
```bash
# CSV tools
csvcut -c 1,3 data.csv          # Select columns
csvgrep -c name -m "test" data.csv  # Filter rows
csvstat data.csv                  # Statistics

# Excel
xlsx2csv data.xlsx > data.csv
```

## Pitfalls
- **Encoding**: Use utf-8-sig for Excel exports
- **Large files**: Use chunking or polars for >1GB
- **Date parsing**: Specify date_format explicitly
- **Quoting**: Handle commas in values

## Verification
- Check data types after loading
- Verify row/column counts
- Test with edge cases (empty cells, special chars)