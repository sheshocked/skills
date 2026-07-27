---
name: android-ndk-native
description: 
category: android
tags: [android-ndk-native]
---

## When to Use
Use this skill when integrating native C/C++/Go code in Android: CMake builds, JNI bridges, native debugging, ABI splits.

## Core Concepts
- **NDK**: Native Development Kit for C/C++ compilation
- **CMake**: Build system for native code
- **JNI**: Java Native Interface for Kotlin↔C/C++ interop
- **ABI splits**: Separate APKs per architecture (arm64-v8a, armeabi-v7a)

## Workflow
1. Place native code in src/main/cpp/
2. Configure CMakeLists.txt
3. Add CMake to build.gradle
4. Write JNI functions in C/C++
5. Declare native functions in Kotlin
6. Build and test

## Key Patterns
```kotlin
// Kotlin side
class NativeBridge {
    companion object {
        init { System.loadLibrary("native-lib") }
    }
    external fun processPacket(data: ByteArray): ByteArray
    external fun calculateHash(input: String): String
}
```

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.22)
project(native-lib)

add_library(native-lib SHARED native-lib.cpp)

find_library(log-lib log)
target_link_libraries(native-lib ${log-lib})
```

```cpp
// native-lib.cpp
#include <jni.h>
#include <android/log.h>

extern "C" JNIEXPORT jbyteArray JNICALL
Java_com_example_NativeBridge_processPacket(
    JNIEnv *env, jobject thiz, jbyteArray data) {
    jbyte* bytes = env->GetByteArrayElements(data, nullptr);
    jsize length = env->GetArrayLength(data);
    // Process packet
    env->ReleaseByteArrayElements(data, bytes, 0);
    return data;
}
```

## Pitfalls
- **Memory leaks**: Always ReleaseByteArrayElements after use
- **Thread safety**: JNI calls are not thread-safe by default
- **ABI compatibility**: Test on all target architectures
- **Debugging**: Use ndk-gdb or LLDB for native debugging

## Verification
- Test native code with JUnit calling JNI methods
- Use addr2line for crash symbolication
- Verify ABI splits in APK analyzer
- Profile native code with simpleperf