# Welcome & Onboarding System

A multi-step, animated onboarding experience designed to guide users through initial setup, preferences, and feature education.

<img width="356" height="792" alt="screen-20260507-162920-1778151549679" src="https://github.com/user-attachments/assets/190e0f3c-b566-4563-8831-1544be1d6234" />

## 1. Onboarding Flow

The system uses an `OnboardingStep` enum to manage the state of the wizard. Transitions between steps are animated using `AnimatedContent` with horizontal slide-and-fade effects.

```kotlin
enum class OnboardingStep {
    WELCOME,
    ACKNOWLEDGEMENT,
    PREFERENCES,
    FEATURE_INTRODUCTION,
    WHATS_NEW
}
```

### Steps Overview
1.  **Welcome**: App branding and primary language selection.
2.  **Acknowledgement**: Critical disclaimers regarding system modifications (Shizuku/Root) and crash reporting toggles.
3.  **Preferences**: Theme and basic app configuration.
4.  **Feature Introduction**: High-level overview using educational GIFs and [Help/Guides](../components/sheets/help-guide-sheet.md).
5.  **Finalize**: Marks the onboarding as complete and transitions to the main app.

## 2. Release Notes Screen

A specialized screen shown only after an app update. It fetches and renders the latest [GitHub Release Notes](../../logic/services/updates.md).

### Logo Easter Egg
Using `pointerInput` and `detectDragGestures`, users can manually spin the logo.
- **Haptic Notches**: Small "micro-haptics" are triggered every 2 degrees during rotation.
- **Rick Roll**: If the user spins the logo 10 full rotations (3600°), it triggers an Easter egg that opens a specific YouTube link.

```kotlin
detectDragGestures(
    onDrag = { change, _ ->
        // Calculate rotation based on drag angle
        currentRotation += delta
        if (abs(currentRotation) >= 3600f) triggerEasterEgg()
        HapticUtil.performMicroHaptic(view) // Tactile "texture"
    }
)
```

## 3. State Persistence & Resets

### Completion Flag
Once the user clicks "Let me in", the app saves a flag in the `SettingsRepository`.
```kotlin
fun onFinish() {
    settingsRepository.putBoolean(KEY_ONBOARDING_COMPLETED, true)
}
```

### Developer Reset
For debugging or testing, a "Reset onboarding" button is available in [Developer Options](../../logic/utils/developer-mode.md). This clears the completion flag and forces the wizard to appear on the next app launch.

## 4. Best Practices
- **Skip Button**: Always provide a skip button (except on the very first screen) to respect the user's time.
- **Language First**: Always place the [Language Picker](../../logic/utils/languages.md) on the very first screen so the rest of the wizard is readable.
- **Visual Continuity**: Use the same background gradients and rounded corner radius as the main app's card system.
