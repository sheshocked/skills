---
name: image-generation
description: 
category: ai-tools
tags: [image-generation]
---

## When to Use
Work with image generation: Stable Diffusion, ControlNet, LoRA styles, upscaling, inpainting.

## Key Concepts
- **Text-to-Image**: Generate from prompt
- **img2img**: Transform existing image with prompt
- **Inpainting**: Edit specific regions
- **ControlNet**: Guide generation with structural inputs (edges, depth, pose)

## Prompt Structure
```
{quality tags}, {subject}, {style}, {lighting}, {composition}
Example: masterpiece, best quality, 1girl, studio ghibli style, soft lighting, centered
```

## Pitfalls
- **Prompt engineering**: More specific = better results
- **Negative prompts**: Always use to avoid common artifacts
- **CFG scale**: 7-12 for most cases; higher = more prompt adherence
- **Steps**: 20-30 for quality; diminishing returns above 50
- **LoRA stacking**: Too many LoRAs degrade quality

## Verification
- Generate 10+ samples per prompt
- Check resolution and detail quality
- Test prompt consistency across seeds
- Verify inpainting seams are invisible