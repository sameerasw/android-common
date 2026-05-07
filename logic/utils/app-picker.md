# App Picker & Selection Pattern

A comprehensive implementation for listing, filtering, and selecting installed applications (system and user) with high performance and premium UI.

## 1. Data Models

The system uses two primary models: one for the UI representation (`NotificationApp`) and one for persistence (`AppSelection`).

```kotlin
data class NotificationApp(
    val packageName: String,
    val appName: String,
    val isEnabled: Boolean,
    val icon: ImageBitmap,
    val isSystemApp: Boolean,
    val lastUpdated: Long
)

data class AppSelection(
    val packageName: String,
    val isEnabled: Boolean
)
```

## 2. The App Utility (`AppUtil`)

The `AppUtil` object handles the heavy lifting of interacting with the `PackageManager`.

### Key Features
- **Icon Caching & Scaling**: App icons are scaled down (e.g., 64x64) and cached in memory to prevent IPC overhead and high memory usage when scrolling through hundreds of apps.
- **System App Detection**: Accurate detection using `ApplicationInfo.FLAG_SYSTEM`.
- **Merge Logic**: Merges the live installed apps list with user-saved preferences.
- **Brand Color Extraction**: Uses the `Palette` API to extract vibrant colors from app icons for themed UI elements.

```kotlin
suspend fun getInstalledApps(context: Context): List<NotificationApp> = withContext(Dispatchers.IO) {
    val pm = context.packageManager
    pm.getInstalledApplications(0).map { appInfo ->
        NotificationApp(
            packageName = appInfo.packageName,
            appName = pm.getApplicationLabel(appInfo).toString(),
            icon = getLowQualityIcon(context, appInfo.packageName).asImageBitmap(),
            isSystemApp = (appInfo.flags and ApplicationInfo.FLAG_SYSTEM) != 0,
            // ...
        )
    }
}
```

## 3. UI Variants

### Multi-Selection Sheet (`AppSelectionSheet`)
Used when the user needs to enable/disable multiple apps (e.g., for notification lighting).

**Features:**
- **Search**: Real-time filtering by app name.
- **System Toggle**: Ability to show/hide system apps.
- **Invert Selection**: A quick action to flip all visible checkboxes.
- **Sorting**: Enabled apps are pinned to the top, followed by alphabetical order.

### Single Selection Sheet (`SingleAppSelectionSheet`)
Used when a single app needs to be picked for a specific action (e.g., "Open this app on click").

## 4. Performance Optimizations

### 1. IO Offloading
Always load apps inside a `LaunchedEffect` using `Dispatchers.IO` to keep the UI responsive.

```kotlin
LaunchedEffect(Unit) {
    withContext(Dispatchers.IO) {
        val allApps = AppUtil.getInstalledApps(context)
        selectedApps = AppUtil.mergeWithSavedApps(allApps, savedSelections)
    }
}
```

### 2. Lazy Loading
The `LazyColumn` uses the `packageName` as a key to ensure efficient list updates and animations.

```kotlin
LazyColumn {
    items(filteredApps, key = { it.packageName }) { app ->
        AppToggleItem(...)
    }
}
```

### 3. Icon Management
Scaling icons to a fixed `ICON_SIZE` (e.g., 64px) is critical. Large, unscaled Bitmaps will quickly lead to `OutOfMemoryError` or severe scroll stuttering.

## 5. Implementation Tips
- **Haptics**: Apply virtual key haptics on every toggle and sheet action.
- **Visuals**: Use a `LoadingIndicator` while the apps are being fetched from the system.
- **Edge-to-Edge**: Ensure the sheet has a `fillMaxHeight(0.9f)` or similar to prevent it from covering the entire screen while allowing plenty of space for scrolling.
