---
name: android-ndk-rust
description: Compile native Rust code for Android targets (aarch64, x86_64) and link via JNI.
category: android
tags: [rust-android, ndk, jni-bindings, cargo-ndk, cmake]
---

# Android Ndk Rust

## When to Use
Use when compiling low-level components (VPN client, protocol parser) in Rust for use in Android composition layers.

## Prerequisites
- Android NDK installed.
- Rust toolchain with android targets (`aarch64-linux-android`).

## Workflow
1. Create a Cargo project with `crate-type = ["cdylib"]`.
2. Write JNI bindings using the `jni` crate in Rust.
3. Compile using `cargo ndk --target aarch64-linux-android --android-ndk <path> build --release`.
4. Load library in Kotlin via `System.loadLibrary("rust_core")`.

## Key Patterns
```rust
// lib.rs
use jni::JNIEnv;
use jni::objects::{JClass, JString};
use jni::sys::jstring;

#[no_mangle]
pub extern "system" fn Java_com_surfshield_core_RustBridge_startEngine(
    mut env: JNIEnv,
    _class: JClass,
    fd: jni::sys::jint
) -> jstring {
    let output = format!("Engine started on FD: {}", fd);
    env.new_string(output).unwrap().into_raw()
}
```

## Pitfalls
- **Mismatched architectures:** Ensure you build and package libraries for both `arm64-v8a` and `armeabi-v7a`.
- **String conversion panics:** Handle null pointer inputs carefully when decoding JStrings.

## Verification
- Verify generated `.so` sizes under `app/src/main/jniLibs/`.
- Call library function in JVM unit test to verify loading.
