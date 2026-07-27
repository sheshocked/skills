---
name: llm-fine-tuning
description: 
category: ai-tools
tags: [llm-fine-tuning]
---

## When to Use
Fine-tune LLMs with LoRA/QLoRA: dataset curation, training setup, eval harnesses, avoiding catastrophic forgetting.

## LoRA Setup (PEFT)
```python
from peft import LoraConfig, get_peft_model
from transformers import AutoModelForCausalLM

model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-3-8B")

lora_config = LoraConfig(
    r=16, lora_alpha=32,
    target_modules=["q_proj", "v_proj", "k_proj", "o_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM"
)

model = get_peft_model(model, lora_config)
# Trainable params: ~0.1% of total
```

## Dataset Format
```json
{"messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "What is X?"},
    {"role": "assistant", "content": "X is..."}
]}
```

## Pitfalls
- **Data quality > quantity**: 1000 high-quality examples beat 100k noisy ones
- **Catastrophic forgetting**: Keep base model capabilities with replay
- **Overfitting**: Monitor eval loss, stop when it plateaus
- **Rank (r)**: Start with 8-16, increase only if needed

## Verification
- Compare base vs fine-tuned on held-out set
- Check for regression on general tasks
- Test edge cases and adversarial inputs