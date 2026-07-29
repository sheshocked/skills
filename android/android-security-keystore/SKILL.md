---
name: android-security-keystore
description: Encrypt user credentials and private configs using Android Hardware-Backed Keystore API.
category: android
tags: [keystore, encryption, biometrics, aes-gcm, security]
---

# Android Security Keystore

## When to Use
Use when storing high-risk configs (VPN private keys, tokens) securely on the device.

## Prerequisites
- Android SDK 23+ (Marshmallow).

## Workflow
1. Initialize `KeyStore` instance.
2. Generate AES key using `KeyGenParameterSpec` configured with StrongBox/Hardware backing.
3. Encrypt data with AES-GCM and store initialization vector (IV) alongside cipher text.

## Key Patterns
```kotlin
import android.security.keystore.KeyGenParameterSpec
import android.security.keystore.KeyProperties
import java.security.KeyStore
import javax.crypto.Cipher
import javax.crypto.KeyGenerator

fun generateSecretKey(alias: String) {
    val keyGenerator = KeyGenerator.getInstance(
        KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore"
    )
    val spec = KeyGenParameterSpec.Builder(
        alias,
        KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
    )
        .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
        .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
        .setKeySize(256)
        .build()
    keyGenerator.init(spec)
    keyGenerator.generateKey()
}
```

## Pitfalls
- **Key invalidation:** Using `setUserAuthenticationRequired(true)` invalidates keys when users change device locks. Configure fallback flows.
- **No StrongBox support:** Some hardware lacks StrongBox; fallback to TEE-backed generation.

## Verification
- Test file decryption on root/unrooted hardware to confirm file safety.
- Verify cryptographic operations throw standard keys exceptions on lock changes.
