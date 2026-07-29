---
name: android-ndk-rust
description: Build low-level proxy libraries in Rust, compile for arm64 Android targets, and bind using JNI.
category: android
tags: [rust, ndk, jni, dynamic-library, arm64, compile, gradle]
---

# Android NDK Rust compilation Masterclass

## When to Use
Use when you want to write a high-performance network engine, packet parser, or encryption module in Rust and integrate it into your Android application (like Aethery/SurfShield).

## Prerequisites
- Rust toolchain (`rustup target add aarch64-linux-android`).
- Android NDK installed (`$ANDROID_HOME/ndk/<version>`).
- `cargo-ndk` helper utility installed (`cargo install cargo-ndk`).

## Workflow
1. Configure `Cargo.toml` to build a dynamic system library (`cdylib`).
2. Write safety wrappers mapping JVM types using the Rust `jni` crate.
3. Run compilation targeting target platforms.
4. Copy outputs to Android dynamic library directories (`jniLibs`).

## Key Patterns

### Cargo Configuration (Cargo.toml)
```toml
[package]
name = "surfshield_core"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[dependencies]
jni = { version = "0.21.1", default-features = false }
tokio = { version = "1.30", features = ["full"] }
log = "0.4"
```

### Rust JNI Bridge (src/lib.rs)
```rust
use jni::JNIEnv;
use jni::objects::{JClass, JString};
use jni::sys::{jint, jstring};
use std::os::unix::io::FromRawFd;
use std::os::unix::prelude::RawFd;

#[no_mangle]
pub unsafe extern "system" fn Java_com_surfshield_vpn_VpnCoreService_startNativeCore(
    mut env: JNIEnv,
    _class: JClass,
    tun_fd: jint,
) {
    let raw_fd: RawFd = tun_fd as RawFd;
    
    // Spawn network task asynchronously in Tokio runtime
    std::thread::spawn(move || {
        let rt = tokio::runtime::Runtime::new().unwrap();
        rt.block_on(async {
            run_packet_loop(raw_fd).await;
        });
    });
}

async fn run_packet_loop(fd: RawFd) {
    use std::fs::File;
    use std::io::{Read, Write};

    let mut file = unsafe { File::from_raw_fd(fd) };
    let mut buffer = [0u8; 1500];

    loop {
        match file.read(&mut buffer) {
            Ok(size) if size > 0 => {
                // Process packet payloads here
                let mut response = vec![0u8; size];
                response.copy_from_slice(&buffer[..size]);
                
                // Write back to interface
                let _ = file.write_all(&response);
            }
            Ok(_) => break,
            Err(_) => break,
        }
    }
}
```

### Compilation Command
```bash
# Compile library for Arm64 platform using cargo-ndk
cargo ndk -t arm64-v8a -o ./jniLibs build --release
```

## Pitfalls
- **Panic overheads:** Rust panics across JNI boundary trigger undefined behaviors or JVM segfaults. Always wrap native functions inside `catch_unwind` closures or compile with `panic = 'abort'`.
- **NDK mismatch error:** Ensure the NDK version specified in cargo configuration matches the NDK target version inside gradle builds.

## Verification
- Verify generated `.so` format using file utility: `file ./jniLibs/arm64-v8a/libsurfshield_core.so` should display ELF 64-bit dynamic library.
- Deploy APK and confirm JNI call registers.
