---
name: multimodal-pipelines
description: 
category: ai-tools
tags: [multimodal-pipelines]
---

## When to Use
Build multimodal AI: vision+text pipelines, image understanding, document vision.

## Key Patterns
```python
# Vision + Text
from openai import OpenAI
client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "Describe this image"},
            {"type": "image_url", "image_url": {"url": image_url}}
        ]
    }]
)
```

## Use Cases
- Image captioning and description
- Document understanding (charts, tables)
- Visual question answering
- Code screenshots to code

## Pitfalls
- **Image size**: Large images increase token cost
- **Resolution**: Some models downsample images
- **Context**: Vision models may miss fine details

## Verification
- Test with various image types
- Verify text extraction accuracy
- Measure response quality