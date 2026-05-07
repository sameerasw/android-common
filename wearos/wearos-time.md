# WearOS Custom Time View Pattern

Enhancing the standard system `TimeText` with real-time device status information (Phone Battery, Device Name, etc.) while maintaining a clean, non-intrusive UI.

## 1. Extending `TimeText`

Instead of creating a custom clock from scratch, we extend the system `TimeText` and utilize its `startLinearContent` and `endLinearContent` slots.

### UI Hierarchy
- **Start Content**: Device Name + Mobile Icon.
- **Center**: Standard System Time.
- **End Content**: Battery Icon + Phone Percentage.

## 2. Scroll-Aware Visibility

To prevent the time view from cluttering the screen when the user is scrolling through a list, we use a `derivedStateOf` logic based on the `ScalingLazyListState`.

```kotlin
val showDetails by remember(scrollState) {
    derivedStateOf {
        // Only show details at the very top of the list
        scrollState == null || (scrollState.centerItemIndex <= 1 && scrollState.centerItemScrollOffset <= 500)
    }
}
```

## 3. Data Synchronization

The view observes `SharedPreferences` changes to update the status in real-time as data arrives from the [WearOS Communication Bridge](file:///Users/sameerasandakelum/GIT/jetpack-common/logic/utils/wearos.md).

```kotlin
DisposableEffect(Unit) {
    val listener = OnSharedPreferenceChangeListener { p, key ->
        when (key) {
            "phone_battery_level" -> batteryLevel = p.getInt(key, -1)
            // ...
        }
    }
    prefs.registerOnSharedPreferenceChangeListener(listener)
    onDispose { prefs.unregisterOnSharedPreferenceChangeListener(listener) }
}
```

## 4. Curved Support

For circular screens, content must be provided in both `linear` and `curved` formats to ensure it follows the screen edge correctly.

```kotlin
TimeText(
    startLinearContent = { DeviceNameRow() },
    startCurvedContent = { 
        curvedText(deviceName)
        curvedComposable { MobileIcon() }
    }
)
```

## 5. Implementation Tips
- **Color Consistency**: Use a light accent color (derived from the [Theming System](file:///Users/sameerasandakelum/GIT/jetpack-common/ui/theme.md)) to ensure the status info is readable but secondary to the main time.
- **Adaptive Icons**: Change the battery icon based on charging state (`bolt` icon) and level (`alert` icon for low battery).
- **Separator logic**: Use `TimeTextDefaults.TextSeparator` to maintain system-standard spacing between the clock and custom content.
