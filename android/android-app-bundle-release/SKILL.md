---
name: android-app-bundle-release
description: 
category: android
tags: [android-app-bundle-release]
---

## When to Use
Use this skill when preparing release builds: signing, AAB generation, R8/ProGuard, Play Console publishing.

## Workflow
1. Create keystore and configure signing
2. Set build variants to release
3. Configure R8/ProGuard rules
4. Build AAB
5. Test with bundletool
6. Upload to Play Console

## Key Patterns
```kotlin
// build.gradle.kts (signing)
android {
    signingConfigs {
        create("release") {
            storeFile = file("keystore.jks")
            storePassword = System.getenv("STORE_PASSWORD")
            keyAlias = System.getenv("KEY_ALIAS")
            keyPassword = System.getenv("KEY_PASSWORD")
        }
    }
    buildTypes {
        release {
            isMinifyEnabled = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
            signingConfig = signingConfigs.getByName("release")
        }
    }
}
```

## Pitfalls
- **Keystore backup**: Losing keystore = impossible to update app
- **R8 stripping**: Test release builds thoroughly — R8 can remove needed classes
- **Version codes**: Must increment with each Play Store upload
- **Signing**: Use Play App Signing for automated signing

## Verification
- Build AAB: ./gradlew :app:bundleRelease
- Test with: bundletool install-apks
- Check mapping.txt for R8 obfuscation
- Upload to internal track first for testing