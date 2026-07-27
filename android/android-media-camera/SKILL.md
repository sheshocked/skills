---
name: android-media-camera
description: 
category: android
tags: [android-media-camera]
---

## When to Use
Use this skill when integrating CameraX, ExoPlayer/Media3, audio recording, or media playback.

## Key Patterns
```kotlin
// CameraX preview
val preview = Preview.Builder().build()
val cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA

val cameraProvider = ProcessCameraProvider.getInstance(context).await()
cameraProvider.unbindAll()
cameraProvider.bindToLifecycle(this, cameraSelector, preview)

// ExoPlayer/Media3
val player = ExoPlayer.Builder(context).build()
player.setMediaItem(MediaItem.fromUri(uri))
player.prepare()
player.play()

// Audio recording
val recorder = AudioRecord.Builder()
    .setAudioSource(MediaRecorder.AudioSource.MIC)
    .setOutputFormat(MediaRecorder.OutputFormat.MPEG_4)
    .build()
```

## Pitfalls
- **Camera permission**: Runtime permission required
- **Audio focus**: Request and handle audio focus properly
- **Lifecycle**: Release player/recorder in onPause/onDestroy

## Verification
- Test camera preview on physical device
- Verify media playback across format types
- Check audio recording quality and permissions