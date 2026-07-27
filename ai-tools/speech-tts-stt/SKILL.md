---
name: speech-tts-stt
description: 
category: ai-tools
tags: [speech-tts-stt]
---

## When to Use
Build speech pipelines: Whisper STT, TTS engines, voice cloning, streaming transcription.

## Whisper STT
```python
import whisper

model = whisper.load_model("medium")
result = model.transcribe("audio.mp3", language="en")
print(result["text"])

# Streaming with faster-whisper
from faster_whisper import WhisperModel
model = WhisperModel("medium", device="cuda", compute_type="float16")
segments, info = model.transcribe("audio.mp3", beam_size=5)
for segment in segments:
    print(f"[{segment.start:.2f}s -> {segment.end:.2f}s] {segment.text}")
```

## TTS Options
| Engine | Quality | Speed | Cost |
|---|---|---|---|
| Edge TTS | Good | Fast | Free |
| Piper | Good | Fast | Free (local) |
| OpenAI TTS | Excellent | Medium | Paid |
| ElevenLabs | Excellent | Slow | Paid |

## Pitfalls
- **Whisper accuracy**: Medium model is good balance; large is better but slower
- **Audio quality**: Noise, music, multiple speakers reduce accuracy
- **Language detection**: Specify language if known for better accuracy
- **TTS prosody**: Most TTS lacks emotional prosody

## Verification
- WER (Word Error Rate) for STT accuracy
- MOS (Mean Opinion Score) for TTS quality
- Test with various audio conditions
- Measure latency for real-time applications