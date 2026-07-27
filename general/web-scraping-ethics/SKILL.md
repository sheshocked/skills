---
name: web-scraping-ethics
description: 
category: general
tags: [web-scraping-ethics]
---

## When to Use
Scrape web data ethically: respect robots.txt, rate limiting, headless browsers, parsing.

## Ethical Rules
1. Check robots.txt first
2. Rate limit requests (1-2/sec)
3. Identify with User-Agent
4. Don't scrape behind authentication
5. Respect ToS

## BeautifulSoup
```python
import requests
from bs4 import BeautifulSoup

resp = requests.get(url, headers={"User-Agent": "MyBot/1.0"})
soup = BeautifulSoup(resp.text, "html.parser")

# Extract data
titles = [h2.text for h2 in soup.find_all("h2")]
links = [a["href"] for a in soup.find_all("a", href=True)]
```

## Playwright (dynamic pages)
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("https://example.com")
    page.wait_for_selector(".content")
    data = page.query_selector_all(".item")
```

## Pitfalls
- **Rate limiting**: Add delays between requests
- **Dynamic content**: Use headless browser for JS-rendered pages
- **Legal**: Check ToS before scraping
- **Storage**: Cache results to avoid re-scraping

## Verification
- Verify data accuracy against source
- Check no IP blocks
- Test with different user agents