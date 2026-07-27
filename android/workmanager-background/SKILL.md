---
name: workmanager-background
description: 
category: android
tags: [workmanager-background]
---

## When to Use
Use this skill when implementing reliable background work in Android: periodic sync, one-time tasks, constraints, chains, and expedited work.

## Core Concepts
- **Worker**: DoWork() runs once; return Result.success/failure/retry
- **CoroutineWorker**: Coroutine version with suspend doWork()
- **PeriodicWorkRequest**: Runs repeatedly with minimum interval (15 min)
- **OneTimeWorkRequest**: Runs once with optional delay
- **Constraints**: Network, battery, charging, storage requirements
- **WorkRequest.Builder**: Configure backoff, tags, chaining

## Workflow
1. Create Worker class extending CoroutineWorker
2. Build WorkRequest with constraints
3. Enqueue via WorkManager
4. Observe work status with LiveData or Flow

## Key Patterns
```kotlin
// Worker
class SyncWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    override suspend fun doWork(): Result {
        return try {
            repository.sync()
            Result.success()
        } catch (e: Exception) {
            if (runAttemptCount < 3) Result.retry() else Result.failure()
        }
    }
}

// Enqueue with constraints
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.CONNECTED)
    .setRequiresBatteryNotLow(true)
    .build()

val syncWork = OneTimeWorkRequestBuilder<SyncWorker>()
    .setConstraints(constraints)
    .setBackoffCriteria(BackoffPolicy.EXPONENTIAL, 10, TimeUnit.SECONDS)
    .addTag("sync")
    .build()

WorkManager.getInstance(context).enqueueUniqueWork(
    "sync", ExistingWorkPolicy.REPLACE, syncWork
)

// Observe status
WorkManager.getInstance(context).getWorkInfosByTagLiveData("sync")
    .observe(this) { works ->
        works?.firstOrNull()?.let { workInfo ->
            when (workInfo.state) {
                WorkInfo.State.SUCCEEDED -> showSuccess()
                WorkInfo.State.FAILED -> showError()
                WorkInfo.State.RUNNING -> showProgress()
                else -> {}
            }
        }
    }
```

## Pitfalls
- **Minimum interval**: Periodic work minimum is 15 minutes
- **Expedited work**: Use setExpedited() for near-immediate execution
- **Chaining**: Use .then() to sequence tasks
- **Tags**: Use unique tags for cancellation and querying
- **Test**: Use TestWorkerBuilder or TestListenableWorkerBuilder

## Verification
- Use WorkManager Testing APIs
- Verify constraints are respected
- Test retry logic with custom WorkerFactory