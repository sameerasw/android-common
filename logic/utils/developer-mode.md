# Developer Mode Logic

A hidden diagnostic and power-user state that enables advanced configuration, debugging tools, and internal state resets.

## 1. Activation Pattern

Developer Mode is activated via a **Long-Press** on the developer's avatar in the [About Section](file:///Users/sameerasandakelum/GIT/jetpack-common/ui/components/about.md).

### Flow
1.  User performs a long-press (e.g., 500ms+).
2.  App triggers a Haptic feedback.
3.  The `MainViewModel` toggles the `isDeveloperModeEnabled` state.
4.  A `Toast` confirms the status change.

## 2. State Management

The state is persistent across app restarts via `SettingsRepository`.

```kotlin
class MainViewModel : ViewModel() {
    val isDeveloperModeEnabled = mutableStateOf(false)

    fun setDeveloperModeEnabled(enabled: Boolean) {
        isDeveloperModeEnabled.value = enabled
        settingsRepository.putBoolean(KEY_DEVELOPER_MODE_ENABLED, enabled)
    }
}
```

## 3. Developer Options Section

When `isDeveloperModeEnabled` is true, an additional section appears in the Settings UI (usually at the very bottom).

### Features included:
- **Export/Import Config**: Allows users to back up or restore their entire app configuration as a JSON file.
- **Onboarding Reset**: Clears the "onboarding completed" flag to test the first-run experience.
- **Update Note Reset**: Resets the counter for the "What's New" sheet to force it to reappear.
- **Deep Diagnostics**: (Optional) Toggles for internal logging or experimental features.

## 4. UI Conditional Rendering

```kotlin
if (isDeveloperModeEnabled) {
    Text("Developer Options", ...)
    RoundedCardContainer {
        Row {
            Button(onClick = { exportConfig() }) { Text("Export Config") }
            Button(onClick = { importConfig() }) { Text("Import Config") }
        }
        // ...
    }
}
```

## 5. Security & Safety
Developer options are generally safe but hidden to avoid confusing casual users. Resets (like Onboarding) should always be preceded by a confirmation dialog or a haptic warning.
