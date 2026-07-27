---
name: game-audio-design
description: - Interactive audio systems with spatial sound
category: threed
tags: [game-audio-design]
---

## When to Use
- Creating sound effects for games (UI, environment, combat)
- Interactive audio systems with spatial sound
- Music layering and adaptive soundtracks
- Audio middleware integration (Wwise, FMOD)
- Voice chat and voice-over integration

## Core Concepts
- FMOD / Wwise: middleware for dynamic audio with parameter-driven behavior
- Spatial audio: 3D positioned sounds with HRTF for VR
- Audio occlusion: muffled sound behind walls
- Reverb zones: different acoustic spaces (cave, room, outdoor)
- Audio priorities: manage simultaneous sound count
- Compressed audio formats: Vorbis, Opus, ADPCM (quality vs memory)
- Sound banks: loading strategy for memory management

## Workflow
```
# FMOD Studio integration
1. Install FMOD Studio + Unity/Unreal integration
2. Create FMOD project with event categories
3. Define parameters: distance, speed, health, intensity
4. Set up parameter-driven sound variations
5. Implement in game code: set parameter values per frame
6. Test in different acoustic environments
7. Profile audio memory and CPU usage
```

## Key Patterns
```csharp
// Unity FMOD integration
using FMODUnity;
using FMOD.Studio;

public class AudioManager : MonoBehaviour
{
    [SerializeField] private EventReference footstepEvent;
    private EventInstance footstepInstance;

    void Start()
    {
        footstepInstance = RuntimeManager.CreateInstance(footstepEvent);
    }

    public void PlayFootstep(float surfaceType)
    {
        // Set surface parameter (0=concrete, 1=grass, 2=wood)
        footstepInstance.setParameterByName("Surface", surfaceType);
        footstepInstance.start();
    }

    // 3D spatial audio
    public void PlayGunshot(Vector3 position)
    {
        RuntimeManager.PlayOneShot("event:/Weapons/Gunshot", position);
    }

    void Update()
    {
        // Update listener position
        var attr = RuntimeUtils.To3DAttributes(transform.position);
        footstepInstance.set3DAttributes(attr);
    }

    void OnDestroy()
    {
        footstepInstance.release();
    }
}
```

Audio parameter system:
```python
# FMOD-style parameter-driven audio (conceptual)
# Event: /Environment/Ambience
# Parameters: TimeOfDay (0-24), Weather (0-1), Danger (0-1)

# Time of day layers
# 0-6:   Night crickets, owl hoots, wind
# 6-12:  Birds, distant traffic, rustling leaves
# 12-18: Afternoon insects, wind gusts, activity
# 18-24: Evening frogs, crickets, distant thunder

# Weather layers
# 0:     No weather sounds
# 0-0.5: Light rain (intensity = weather * 2)
# 0.5-1: Heavy rain + thunder (random triggers)

# Danger layers
# 0:     Ambient music only
# 0-0.5: Tension underscore fades in
# 0.5-1: Combat music, heart rate bass
```

## Pitfalls
- Too many simultaneous sounds: implement priority system (max 32-64 voices)
- Streaming large audio files causes hitches — preload with sound banks
- Compression: Vorbis for music (smaller), PCM/ADPCM for SFX (fast decode)
- 3D audio on mobile: HRTF limited — use stereo panning fallback
- Voice-over: lip sync needs phoneme timing data — use subtitle system as backup
- Audio memory: compressed audio still loads into RAM — monitor peak usage

## Verification
- Profile audio CPU usage: < 5% of frame budget
- Check voice count per frame in FMOD profiler
- Test on lowest-spec device (integrated audio, Bluetooth latency)
- Verify spatial audio: walk around 3D sound sources, check panning
- Load test: verify streaming audio doesn't cause hitches during gameplay