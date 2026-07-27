---
name: gradle-build-optimization
description: 
category: android
tags: [gradle-build-optimization]
---

## When to Use
Use this skill when optimizing Android build performance: configuration cache, KSP migration, modularization, build scans.

## Optimization Checklist
1. **Configuration cache**: Enable in gradle.properties
2. **KSP over kapt**: Migrate annotation processors
3. **Module splitting**: Smaller modules = faster incremental builds
4. **Gradle daemon**: Keep enabled and warm
5. **Parallel execution**: org.gradle.parallel=true
6. **Memory**: org.gradle.jvmargs=-Xmx4g

## Key Commands
```bash
# Enable configuration cache
org.gradle.configuration-cache=true

# Parallel builds
org.gradle.parallel=true

# Build scan (diagnose slow builds)
./gradlew assembleDebug --scan

# Profile build
./gradlew assembleDebug --profile

# Compare build times
./gradlew assembleDebug --rerun-tasks --scan
```

## KSP Migration
```kotlin
// Before (kapt)
kapt("com.google.dagger:hilt-android-compiler:2.48")

// After (ksp)
ksp("com.google.dagger:hilt-android-compiler:2.48")
```

## Pitfalls
- **Configuration cache breaks**: Some plugins don't support it — check compatibility
- **KSP not supported**: Some annotation processors lack KSP versions
- **Over-modularization**: Too many modules increases configuration time
- **Build cache**: Use local cache + remote cache for CI

## Verification
- Measure build time before/after optimizations
- Use build scan to identify slow tasks
- Check configuration cache hit rate
- Profile memory usage during builds