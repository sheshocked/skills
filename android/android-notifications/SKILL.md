---
name: android-notifications
description: 
category: android
tags: [android-notifications]
---

## When to Use
Use this skill when implementing notifications: channels, styles, actions, FCM push, and notification permissions on Android 13+.

## Core Concepts
- **NotificationChannel**: Required on Android 8+ for every notification
- **NotificationCompat.Builder**: Backward-compatible notification creation
- **FCM**: Firebase Cloud Messaging for push notifications
- **Permission**: POST_NOTIFICATIONS runtime permission on Android 13+

## Workflow
1. Create NotificationChannel(s) on app start
2. Request POST_NOTIFICATIONS permission on Android 13+
3. Build notifications with appropriate style and priority
4. Handle notification taps with PendingIntent

## Key Patterns
```kotlin
// Channel creation (on app start)
val channel = NotificationChannel(
    "messages", "Messages",
    NotificationManager.IMPORTANCE_DEFAULT
).apply {
    description = "Direct messages"
}
notificationManager.createNotificationChannel(channel)

// Build notification
val notification = NotificationCompat.Builder(context, "messages")
    .setSmallIcon(R.drawable.ic_message)
    .setContentTitle("New message")
    .setContentText("Hey, how are you?")
    .setPriority(NotificationCompat.PRIORITY_DEFAULT)
    .setContentIntent(pendingIntent)
    .setAutoCancel(true)
    .build()

// FCM Handler
class MessagingService : FirebaseMessagingService() {
    override fun onMessageReceived(message: RemoteMessage) {
        message.notification?.let { showNotification(it) }
    }
}
```

## Pitfalls
- **Android 13+ permission**: Must request POST_NOTIFICATIONS at runtime
- **Channel importance**: Cannot change after creation — must delete and recreate
- **Dead letter**: Notifications without channel silently fail on Android 8+
- **FCM token**: Must refresh and send to server on each app install

## Verification
- Test on Android 8+ for channel requirement
- Test on Android 13+ for permission flow
- Verify notification taps open correct Activity
- Check FCM token refresh logic