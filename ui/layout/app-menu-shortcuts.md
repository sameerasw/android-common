# App Menu Shortcuts Pattern

Implementing dynamic, context-aware shortcuts (App Long-Press Menu) to provide quick access to key features or background actions.

## 1. Context-Aware Dynamic Shortcuts

Unlike static shortcuts defined in XML, dynamic shortcuts can be added, updated, or removed programmatically based on the app's state (e.g., whether a device is connected).

### Implementation (`ShortcutUtil`)
Use `ShortcutManagerCompat` to maintain a consistent API across Android versions.

```kotlin
fun refreshShortcuts(context: Context, isConnected: Boolean) {
    val shortcuts = mutableListOf<ShortcutInfoCompat>()

    if (isConnected) {
        // Add "Lock Mac" and "Disconnect"
        shortcuts.add(createLockShortcut(context))
        shortcuts.add(createDisconnectShortcut(context))
    } else {
        // Add "Scan QR" and "Reconnect"
        shortcuts.add(createScanShortcut(context))
        shortcuts.add(createReconnectShortcut(context))
    }

    ShortcutManagerCompat.setDynamicShortcuts(context, shortcuts)
}
```

## 2. Transparent Proxy Activity Pattern

For actions that don't require a full UI (like "Lock Mac"), use a **Transparent Proxy Activity**. This activity performs the action, shows a minimal feedback pill, and finishes itself immediately.

### Characteristics:
- **Transparent Theme**: `Theme.Translucent` or `Theme.Transparent`.
- **Fast Execution**: Finishes within ~1-2 seconds.
- **Minimalist Feedback**: A centered or bottom-anchored "Pill" showing status (Success/Error).

```kotlin
class ProxyActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Show status pill...
        handleAction(intent.action)
    }
}
```

## 3. Deep-Linking to Screens

Shortcuts can also be used to jump directly to specific tabs or screens in the main app.

```kotlin
val intent = Intent(context, MainActivity::class.java).apply {
    action = ACTION_OPEN_REMOTE
    flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TOP
}
```

In `MainActivity`:
```kotlin
val initialPage = if (intent?.action == ACTION_OPEN_REMOTE) 1 else 0
AirSyncMainScreen(initialPage = initialPage)
```

## 4. Manifest Configuration

Ensure the proxy activity is excluded from recents and has no history to keep the multitasking view clean.

```xml
<activity
    android:name=".ProxyActivity"
    android:theme="@style/Theme.Transparent"
    android:exported="true"
    android:excludeFromRecents="true"
    android:noHistory="true"
    android:taskAffinity="" />
```

## 5. Implementation Tips
- **Haptic Confirmation**: Trigger a haptic "tick" when the shortcut activity starts.
- **Icon Consistency**: Use the same icons for shortcuts as used in the in-app UI to build mental models.
- **Max Shortcuts**: Android allows a maximum of 4 dynamic shortcuts at a time; prioritize the most frequent actions.
