---
name: firebase-integration
description: 
category: android
tags: [firebase-integration]
---

## When to Use
Use this skill when integrating Firebase services: FCM push, Remote Config, Analytics, Crashlytics, App Distribution.

## Services
- **FCM**: Push notifications (topic-based, device-targeted)
- **Remote Config**: Server-driven feature flags without app update
- **Analytics**: Event tracking, user properties, funnels
- **Crashlytics**: Crash reporting with symbolication
- **App Distribution**: Beta testing distribution

## Workflow
1. Add Firebase to project (google-services.json)
2. Initialize each service in Application class
3. Set up analytics event tracking
4. Configure Remote Config defaults
5. Handle FCM messages

## Key Patterns
```kotlin
// Remote Config
val config = FirebaseRemoteConfig.getInstance()
config.setDefaultsAsync(mapOf("max_retries" to 3))
config.fetchAndActivate().addOnCompleteListener {
    val maxRetries = config.getLong("max_retries")
}

// Analytics
FirebaseAnalytics.getInstance(context).logEvent(
    FirebaseAnalytics.Event.SELECT_CONTENT,
    bundleOf("item_id" to itemId, "item_name" to itemName)
)

// Crashlytics
Crashlytics.getInstance().setCustomKey("user_id", userId)
Crashlytics.getInstance().recordException(e)
```

## Pitfalls
- **Remote Config caching**: Always call fetchAndActivate, not just activateFromDefaults
- **Analytics privacy**: Respect user consent; use analytics.setAnalyticsCollectionEnabled(false)
- **Crashlytics sizing**: Don't log huge strings — truncate to 1024 chars
- **FCM topics**: Use topics for broadcast, tokens for targeted

## Verification
- Test Remote Config in Firebase Console
- Verify analytics events in DebugView
- Check Crashlytics for non-fatal exceptions
- Test FCM with Firebase Console notification composer