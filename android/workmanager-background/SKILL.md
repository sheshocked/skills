---
name: workmanager-background
description: Schedule persistent background tasks with network and device constraints using Android WorkManager.
category: android
tags: [workmanager, background-task, scheduler, kotlin]
---

# Workmanager Background

## When to Use
Use for non-immediate, guaranteed tasks (uploading connection diagnostics, syncing proxy configurations, pruning local logs).

## Prerequisites
- WorkManager dependency.

## Workflow
1. Create a class extending `CoroutineWorker`.
2. Set execution constraints (network state, battery levels).
3. Enqueue the work using `WorkManager.getInstance()`.

## Key Patterns
```kotlin
class ConfigSyncWorker(context: Context, params: WorkerParameters) :
    CoroutineWorker(context, params) {
    override suspend fun doWork(): Result {
        val success = syncConfigWithServer()
        return if (success) Result.success() else Result.retry()
    }
}

val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.CONNECTED)
    .setRequiresBatteryNotLow(true)
    .build()

val syncRequest = PeriodicWorkRequestBuilder<ConfigSyncWorker>(1, TimeUnit.HOURS)
    .setConstraints(constraints)
    .build()

WorkManager.getInstance(context).enqueueUniquePeriodicWork(
    "config_sync", ExistingPeriodicWorkPolicy.KEEP, syncRequest
)
```

## Pitfalls
- **Background limitations:** OEM power managers (Samsung/Xiaomi) kill workers aggressively. Handle failures gracefully or request battery exemption.
- **Short sync intervals:** Minimum periodic interval is 15 minutes; setting shorter defaults to 15.

## Verification
- Inspect scheduled jobs with `./gradlew workmanager-inspection`.
- Trigger work via adb command `adb shell cmd jobscheduler run -f <package> <id>`.
