---
name: android-security-keystore
description: 
category: android
tags: [android-security-keystore]
---

## When to Use
Use this skill when implementing Android security: Keystore encryption, biometric authentication, certificate pinning, secure storage.

## Core Concepts
- **Android Keystore**: Hardware-backed key storage
- **EncryptedSharedPreferences**: Encrypted key-value storage
- **BiometricPrompt**: Biometric authentication (fingerprint, face, iris)
- **Network Security Config**: Certificate pinning, cleartext rules

## Workflow
1. Generate keys in Android Keystore
2. Use EncryptedSharedPreferences for sensitive data
3. Implement BiometricPrompt for auth
4. Configure network security for pinning

## Key Patterns
```kotlin
// Keystore key generation
val keyGenerator = KeyGenerator.getInstance(KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore")
keyGenerator.init(KeyGenParameterSpec.Builder("my_key",
    KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT)
    .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
    .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
    .build())
val secretKey = keyGenerator.generateKey()

// EncryptedSharedPreferences
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()
val prefs = EncryptedSharedPreferences.create(context, "secret_prefs", masterKey,
    PrefKeyEncryptionScheme.AES256_SIV, PrefValueEncryptionScheme.AES256_GCM)

// BiometricPrompt
val promptInfo = BiometricPrompt.PromptInfo.Builder()
    .setTitle("Verify identity")
    .setAllowedAuthenticators(BiometricManager.Authenticators.BIOMETRIC_STRONG)
    .build()
biometricPrompt.authenticate(promptInfo)

// Certificate pinning (network_security_config.xml)
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">api.example.com</domain>
        <pin-set>
            <pin digest="SHA-256">base64_hash=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

## Pitfalls
- **Keystore on emulator**: Some KeyStore features unavailable — test on real device
- **Biometric fallback**: Always provide device credential fallback
- **Certificate pinning**: Update pins before expiry; use backup pins
- **Encrypted prefs**: Don't store large data — use EncryptedFile instead

## Verification
- Test on physical device for KeyStore
- Verify biometric prompt appears correctly
- Test certificate pinning with Charles proxy
- Check EncryptedSharedPreferences encrypts at rest