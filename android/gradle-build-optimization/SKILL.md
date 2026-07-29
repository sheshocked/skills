---
name: gradle-build-optimization
description: Speed up Android compilation using build caching, configuration caching, and dynamic module structures.
category: android
tags: [gradle, build-speed, kotlin-dsl, caching, devops]
---

# Gradle Build Optimization

## When to Use
Use when gradle builds take longer than 90 seconds, impacting local iteration speed.

## Prerequisites
- Gradle 8.0+.

## Workflow
1. Enable build cache and configuration cache in `gradle.properties`.
2. Configure JVM parameters for parallel execution.
3. Optimize project dependency declarations.

## Key Patterns
```properties
# gradle.properties
org.gradle.caching=true
org.gradle.configuration-cache=true
org.gradle.parallel=true
org.gradle.jvmargs=-Xmx4g -XX:+UseParallelGC -XX:MaxMetaspaceSize=1g
kotlin.incremental=true
android.enableJetifier=false
```

## Pitfalls
- **Configuration cache failures:** Custom build plugins using runtime objects block cache writing. Replace with serializable providers.
- **Excessive dependencies:** Unused libraries slow compilation. Use `dependencyGuard` to audit dependencies.

## Verification
- Measure speed with `./gradlew assembleDebug --profile --dry-run`.
- Verify caching success via `./gradlew build --scan` report.
