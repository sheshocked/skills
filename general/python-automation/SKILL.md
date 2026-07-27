---
name: python-automation
description: 
category: general
tags: [python-automation]
---

## When to Use
Automate tasks with Python: file processing, API calls, web scraping, scheduling.

## Patterns
```python
import os, sys, json, subprocess, shutil
from pathlib import Path
from datetime import datetime

# File operations
def batch_rename(directory, pattern, replacement):
    for path in Path(directory).glob(pattern):
        new_name = path.name.replace(pattern, replacement)
        path.rename(path.parent / new_name)

# API automation
import requests

def fetch_all_pages(base_url, params=None):
    results = []
    page = 1
    while True:
        resp = requests.get(f"{base_url}?page={page}", params=params)
        data = resp.json()
        if not data["results"]:
            break
        results.extend(data["results"])
        page += 1
    return results

# Scheduling
import schedule

def job():
    print("Running daily backup...")

schedule.every().day.at("03:00").do(job)

while True:
    schedule.run_pending()
    time.sleep(60)
```

## CLI with argparse
```python
import argparse

parser = argparse.ArgumentParser(description="Process files")
parser.add_argument("input", help="Input file")
parser.add_argument("-o", "--output", default="output.txt")
parser.add_argument("-v", "--verbose", action="store_true")
args = parser.parse_args()
```

## Pitfalls
- **Hardcoded paths**: Use os.path.expanduser or pathlib
- **No error handling**: Wrap calls in try/except
- **Blocking**: Use asyncio for concurrent operations
- **Dependencies**: Document requirements.txt

## Verification
- Test with dry-run mode
- Verify file permissions
- Check error handling on invalid input
- Test with large files