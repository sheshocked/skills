---
name: test-driven-refactoring
description: Refactor legacy codebase architectures safely using mocking, unit tests, and CI boundaries checks.
category: engineering
tags: [refactoring, tdd, unit-testing, clean-code, legacy-code]
---

# Test Driven Refactoring

## When to Use
Use when modifying high-risk legacy code blocks that lack automated tests to avoid regressions and design-pattern breakage.

## Prerequisites
- Testing framework installed (`pytest`, `unittest`).

## Workflow
1. Write characterization unit tests capturing current output states.
2. Isolate external dependencies (network, filesystem, databases) using mock wrappers.
3. Perform target refactoring in small iterative steps.
4. Execute test boundaries checks after each change.

## Key Patterns

### Mock Refactoring Test Setup (test_refactor.py)
```python
import unittest
from unittest.mock import Mock, patch

# Target legacy class
class LegacyConfigFetcher:
    def fetch_endpoint(self, url: str):
        # Trigger real http request (hard to test!)
        import requests
        return requests.get(url).json()

# Refactored class with Dependency Injection
class ConfigFetcher:
    def __init__(self, http_client=None):
        self.client = http_client or __import__('requests')

    def fetch_endpoint(self, url: str):
        response = self.client.get(url)
        return response.json() if hasattr(response, 'json') else response

class TestConfigFetcher(unittest.TestCase):
    def test_fetch_endpoint_success(self):
        mock_client = Mock()
        mock_response = Mock()
        mock_response.json.return_value = {"status": "ok"}
        mock_client.get.return_value = mock_response

        fetcher = ConfigFetcher(http_client=mock_client)
        result = fetcher.fetch_endpoint("https://api.test.com")
        
        self.assertEqual(result["status"], "ok")
        mock_client.get.assert_called_once_with("https://api.test.com")

if __name__ == '__main__':
    unittest.main()
```

## Pitfalls
- **Refactoring without base tests:** Never touch structural logic before capturing base code behavior using unit tests.
- **Mocking internal details:** Mocking internals makes tests fragile. Mock only border integration interfaces.

## Verification
- Run tests: `pytest test_refactor.py` outputs all green checks.

