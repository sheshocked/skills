---
name: ndk-lldb-debugging
description: Debug native C++/Rust crashes, parse tombstones using ndk-stack, and bind LLDB via adb.
category: android
tags: [lldb, debugging, ndk-stack, tombstones, crash-reports, android-ndk]
---

# Ndk Lldb Debugging

## When to Use
Use when JNI code or native library binaries crash the application (causing segfaults) to inspect stack traces and locate crash coordinates.

## Prerequisites
- Android Studio with NDK and LLDB tools configured.
- Root or debuggable APK build.

## Workflow
1. Compile target libraries with debug symbols (`--release` with debug info or debug mode).
2. Fetch the crash report (tombstone) from the device via `adb bugreport` or `/data/tombstones/`.
3. Resolve addresses using `ndk-stack`.
4. Optionally, attach LLDB to the running process via adb.

## Key Patterns

### Address Resolution using ndk-stack
When a native crash occurs, Android prints backtrace addresses. Resolve them using:
```bash
# Parse crash stack trace from logcat output using ndk-stack
adb logcat -d | ndk-stack -sym /path/to/app/build/intermediates/merged_native_libs/debug/out/lib/
```

### Attaching LLDB manually via adb
```bash
# Push lldb-server to device
adb push $ANDROID_NDK/prebuilt/android-arm64/lldb/bin/lldb-server /data/local/tmp/
adb shell "chmod 777 /data/local/tmp/lldb-server"

# Run server on port 5039
adb shell "/data/local/tmp/lldb-server gdbserver *:5039 --attach <process_pid>"
# Forward local port
adb forward tcp:5039 tcp:5039
```

## Pitfalls
- **Missing debug symbols:** Running ndk-stack against stripped release binaries yields unresolved hex address lines. Ensure target libs match build symbol parameters.
- **Permissions limits:** Accessing tombstones on Android 10+ requires debuggable flags inside the manifest.

## Verification
- Verify ndk-stack prints files name and crash line numbers instead of hex coordinates.

