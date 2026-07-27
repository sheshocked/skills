---
name: local-llm-deployment
description: 
category: ai-tools
tags: [local-llm-deployment]
---

## When to Use
Run LLMs locally: Ollama, llama.cpp, vLLM, quantization, GPU sizing.

## Ollama Setup
```bash
# Install
curl -fsSL https://ollama.com/install.sh | sh

# Run model
ollama run llama3.1:8b

# API
curl http://localhost:11434/api/generate -d '{"model":"llama3.1:8b","prompt":"Hello"}'
```

## Quantization Guide
| Format | Size (7B) | Quality | Speed |
|---|---|---|---|
| FP16 | 14GB | Best | Slowest |
| Q8_0 | 7GB | Excellent | Fast |
| Q4_K_M | 4GB | Good | Fastest |
| Q2_K | 3GB | Reduced | Fastest |

## GPU Sizing
- **7B**: 8GB VRAM (Q4), 16GB (FP16)
- **13B**: 16GB VRAM (Q4), 32GB (FP16)
- **70B**: 48GB VRAM (Q4), 140GB (FP16)

## Pitfalls
- **Quantization loss**: Q4 loses ~2-5% accuracy on complex reasoning
- **Context length**: Local models may have shorter context windows
- **Memory**: RAM fallback is 10x slower than GPU
- **Batch serving**: vLLM is better than Ollama for high throughput

## Verification
- Benchmark tokens/second
- Test quality on domain-specific tasks
- Verify VRAM usage with nvidia-smi
- Check for memory leaks during long sessions