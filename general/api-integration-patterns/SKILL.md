---
name: api-integration-patterns
description: 
category: general
tags: [api-integration-patterns]
---

## When to Use
Integrate third-party APIs: webhooks, polling, SDK design, quota management.

## Webhook Handler
```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/webhook", methods=["POST"])
def webhook():
    payload = request.json
    event = payload.get("event")

    # Verify signature
    sig = request.headers.get("X-Signature")
    if not verify_signature(payload, sig):
        return "Invalid signature", 401

    # Process event
    if event == "payment.completed":
        handle_payment(payload["data"])
    elif event == "user.created":
        handle_user(payload["data"])

    return "OK", 200
```

## Polling with Backoff
```python
import time, requests

def poll_with_backoff(url, max_retries=5):
    for attempt in range(max_retries):
        resp = requests.get(url)
        if resp.status_code == 200:
            return resp.json()
        elif resp.status_code == 429:
            wait = 2 ** attempt
            time.sleep(wait)
        else:
            resp.raise_for_status()
    raise Exception("Max retries exceeded")
```

## Pitfalls
- **Rate limits**: Track and respect API quotas
- **Idempotency**: Use idempotency keys for POST requests
- **Error handling**: Retry on 5xx, don't retry on 4xx
- **Timeouts**: Always set connection and read timeouts

## Verification
- Test webhook with ngrok
- Verify signature validation
- Check error handling on API failures