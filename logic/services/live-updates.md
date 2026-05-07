# Live Update Notifications

A modern pattern for implementing real-time, high-priority notifications using Android 15 (SDK 35) and Android 16 (SDK 36) features like `ProgressStyle` and "Promoted Ongoing" status.

## 1. Core Concept

Live Update notifications are designed for ongoing background tasks that require the user's immediate awareness (e.g., Flashlight brightness, Location tracking ETA, or Caffeinate timer). They occupy a prioritized space at the top of the notification shade and can display status chips in the status bar.

## 2. Android 15+ Features (SDK 35)

### Promoted Ongoing Status
This flag ensures the notification stays in the "Promoted" area of the system UI, making it much harder to miss.

```kotlin
val builder = Notification.Builder(context, CHANNEL_ID)
    .setOngoing(true)
    .setCategory(Notification.CATEGORY_SERVICE)

// Reflection/Direct call for promoted status
val extras = Bundle()
extras.putBoolean("android.requestPromotedOngoing", true)
builder.addExtras(extras)

// Direct method call if available (SDK 35+)
builder.javaClass.getMethod("setRequestPromotedOngoing", Boolean::class.javaPrimitiveType)
    .invoke(builder, true)
```

### Critical Text (Status Bar Chips)
Allows showing a short string (e.g., "80%" or "2.5km") in the system status bar and lock screen.

```kotlin
extras.putString("android.shortCriticalText", "80%")
builder.javaClass.getMethod("setShortCriticalText", CharSequence::class.java)
    .invoke(builder, "80%")
```

## 3. Android 16+ Features (SDK 36)

### `ProgressStyle`
A new, rich notification style specifically for progress tracking.

```kotlin
val progressStyle = Notification.ProgressStyle()
    .setStyledByProgress(true) // Highlights the progress bar
    .setProgress(currentLevel)
    .setProgressTrackerIcon(
        Icon.createWithResource(context, R.drawable.active_icon)
    )

// Add colored segments
progressStyle.addProgressSegment(
    Notification.ProgressStyle.Segment(maxLevel).setColor(Color.YELLOW)
)

builder.style = progressStyle
```

## 4. Implementation Strategy

### The Update Loop
For features like Location Tracking, use a Coroutine loop to refresh the notification. For hardware states (Flashlight), use callbacks.

```kotlin
serviceScope.launch {
    while (isActive) {
        val progress = calculateProgress()
        updateNotification(progress)
        delay(1000) // Adaptive delay based on proximity or battery
    }
}
```

### Reflection-Based Fallback
Since these APIs are extremely new (often Alpha/Beta), always use reflection or `Bundle` extras to maintain compatibility across different build environments.

## 5. Usage Examples
- **[Flashlight & Brightness](../utils/flashlight.md)**: Uses `ProgressStyle` for brightness level and `shortCriticalText` for percentage.
- **Location Tracker**: Uses `ProgressStyle` for distance and `shortCriticalText` for ETA.
- **Caffeinate**: Uses `shortCriticalText` to show remaining "awake" time.

## 6. Implementation Tips
- **Minimize Alerts**: Always use `setOnlyAlertOnce(true)` to prevent the device from vibrating or making sound on every progress update.
- **Colorized Backgrounds**: Avoid `setColorized(true)` unless it's a media-style notification, as it can be too distracting for simple progress trackers.
- **Icon Tinting**: Tint the `ProgressTrackerIcon` to match the system accent for a native feel.
