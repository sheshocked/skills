---
name: ai-art-direction
description: 
category: creative
tags: [ai-art-direction]
---

## When to Use
Direct AI image generation (Midjourney, DALL-E, Stable Diffusion, Flux) for brand assets, illustrations, marketing visuals, concept art, or product mockups. Use when creating visual content at scale or exploring creative directions rapidly.

## Core Concepts
- **Prompt engineering for images**: Structure prompts as: Subject + Style + Composition + Lighting + Color + Medium + Quality modifiers.
- **Style references**: Upload reference images to guide aesthetic. Midjourney's --sref, DALL-E's image input.
- **Consistency across generations**: Seed locking, style references, character references, and detailed descriptions maintain visual consistency.
- **Inpainting/outpainting**: Edit specific regions of generated images without regenerating the whole thing.
- **Upscaling**: Most generators output 1024x1024. Use Topaz Gigapixel, Real-ESRGAN, or built-in upscalers for production resolution.
- **AI art is a starting point**: Generate, then refine in Photoshop/Figma. AI creates the base; you create the final.

## Workflow
1. **Define the visual brief**: What is this image for? What style? What mood? What must it communicate?
2. **Create reference board**: Collect 5-10 images that capture the desired aesthetic. These become style references.
3. **Write base prompts**: Start broad, then iterate. Change one variable at a time to understand its effect.
4. **Generate variations**: Create 20-50 variations. Don't settle for the first good one.
5. **Select and refine**: Choose top 3-5. Use inpainting to fix specific issues. Upscale for production use.
6. **Post-process**: Color correction, text overlay, cropping, compositing in Figma/Photoshop.
7. **Document prompts**: Save successful prompts in a library for future consistency.

## Key Patterns
```markdown
# Prompt Structure Formula
[Subject] + [Action/Pose] + [Environment] + [Style] + [Lighting] + [Color] + [Medium] + [Quality]

# Examples:
Product photo:
"Minimalist white sneaker on marble surface, soft directional lighting, 
pastel color palette, commercial product photography, 8K, clean background"

Brand illustration:
"Team of diverse professionals collaborating in modern office, 
flat illustration style, Miro-style, vibrant brand colors (blue, coral, white), 
clean lines, no gradients, vector aesthetic"

Concept art:
"Abandoned space station interior, overgrown with bioluminescent plants, 
cinematic wide shot, volumetric lighting, teal and orange color grading, 
concept art for AAA game, ultra detailed"

# Style Reference Keywords
- Photography: "shot on Canon EOS R5, 85mm f/1.4, shallow depth of field"
- Illustration: "flat design, vector illustration, editorial style, New Yorker magazine"
- 3D render: "Unreal Engine 5, octane render, photorealistic, ray tracing"
- Retro: "1970s Kodachrome film, grain, warm tones, vintage advertisement"
- Minimalist: "Bauhaus, geometric, limited color palette, clean typography"

# Consistency Techniques
1. Save the seed value for variations you like
2. Use style reference images (Midjourney --sref)
3. Maintain exact subject descriptions across generations
4. Use the same lighting and color descriptors
5. Create a "prompt library" document for each brand/project
```

## Pitfalls
- **AI artifacts**: Extra fingers, weird text, impossible geometry. Always check before using.
- **Copyright concerns**: AI-generated art from specific artist styles may have legal implications. Check current regulations.
- **Inconsistency**: Each generation is different. Use style references + seeds for brand consistency.
- **Over-reliance**: AI generates starting points, not finished work. Post-processing is essential.
- **Ignoring resolution**: AI outputs are often low-res. Upscale for print or large-format use.
- **Prompt pollution**: Adding too many modifiers confuses the model. Be specific but concise.

## Verification
- Image meets the brief: style, mood, subject all align with requirements
- Technical quality: resolution sufficient for intended use, no obvious artifacts
- Brand consistency: matches existing visual identity (colors, style, tone)
- Legal review: no trademark/copyright concerns for intended commercial use
- Multiple generations reviewed: selected best option from 20+ variations
- Post-processed: color corrected, cropped, and formatted for distribution