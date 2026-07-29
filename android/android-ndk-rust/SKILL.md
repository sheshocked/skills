---
name: android-ndk-rust
description: Compile low-level Rust dynamic libraries for Android architectures and bind using JNI.
category: android
tags: [rust, ndk, jni, dynamic-library, arm64-v8a, cargo-ndk]
---

# Android Ndk Rust

## When to Use
Use when compiling high-performance, low-level components (like packet processors, protocol engines) in Rust and linking them into Android applications via JNI.

## Prerequisites
- Rust targets installed: `rustup target add aarch64-linux-android`.
- Android NDK configured on system.
- `cargo-ndk` installed.

## Workflow
1. Configure `Cargo.toml` with `crate-type = ["cdylib"]`.
2. Write JNI bindings using the `jni` crate.
3. Build the library using cargo ndk.
4. Import compiled `.so` files under the project's `jniLibs` directory.

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
```

### JNI Rust Implementation (src/lib.rs)
```rust
use jni::JNIEnv;
use jni::objects::{JClass};
use jni::sys::{jint};
use std::os::unix::io::FromRawFd;
use std::os::unix::prelude::RawFd;

#[no_mangle]
pub unsafe extern "system" fn Java_com_surfshield_vpn_VpnCoreService_startNativeCore(
    mut env: JNIEnv,
    _class: JClass,
    tun_fd: jint,
) {
    let raw_fd: RawFd = tun_fd as RawFd;
    
    // Spawn async background processing loop in Tokio runtime
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
                // Perform encryption/routing logic here
                let mut response = vec![0u8; size];
                response.copy_from_slice(&buffer[..size]);
                let _ = file.write_all(&response);
            }
            _ => break,
        }
    }
}
```

### Compilation Command
```bash
cargo ndk -t arm64-v8a -o ./jniLibs build --release
```

## Pitfalls
- **JNI Panics:** Unhandled Rust panics crashing the JNI bridge trigger immediate JVM aborts. Compile with `panic = 'abort'` to ensure clean exits.
- **Architecture mismatches:** Ensure you ship libraries under both `arm64-v8a` and `armeabi-v7a` folders inside the package.

## Verification
- Verify file architecture: `file ./jniLibs/arm64-v8a/libsurfshield_core.so` output should match ELF 64-bit LSB shared object.

