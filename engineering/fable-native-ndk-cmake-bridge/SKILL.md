---
name: fable-native-ndk-cmake-bridge
description: Fable 5 methodology for debugging NDK compilation logs, CMake configurations, and Go-based native libraries.
category: engineering
tags: [fable-5, ndk, cmake, compilation, go]
---
# Fable 5 Native NDK & CMake Bridge

Use this skill when resolving complex native builds (such as AmneziaWG C/Go libraries in Android), parsing compiler output, or debugging link errors.

## Execution Steps
1. **Log Parsing:** Pass full compiler stderr logs to Fable 5. Instruct the model to group errors by compiler phase (Preprocessor, Compilation, Linker).
2. **CMake Variable Tracking:** Verify CMake caches and NDK toolchain pathways (`ndkVersion`, `compileSdkVersion`) to resolve architecture discrepancies.
3. **Go-Cgo Boundary Analysis:** Debug memory leaks or pointer safety warnings occurring on the boundary between Kotlin JNI and Go runtime.
