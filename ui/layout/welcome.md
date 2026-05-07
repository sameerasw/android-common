# Welcome & Onboarding System

A multi-step, animated onboarding experience designed to guide users through initial setup, preferences, and feature education.

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
4.  **Feature Introduction**: High-level overview using educational GIFs and [Help/Guides](file:///Users/sameerasandakelum/GIT/jetpack-common/ui/components/sheets/help-guide-sheet.md).

## 2. Interactive Welcome Step

The first screen features a spinning app logo that serves as both a brand element and an interactive toy.

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

## 3. "What's New" Integration

The onboarding UI is reused to show major version updates. When `isWhatsNewFlow = true` is passed to the `WelcomeScreen`:
- The title changes to "Welcome Back".
- It skips the legal acknowledgments and jumps straight to the **What's New** step.
- It fetches and renders the latest [GitHub Release Notes](file:///Users/sameerasandakelum/GIT/jetpack-common/logic/services/updates.md).

## 4. State Persistence & Resets

### Completion Flag
Once the user clicks "Let me in", the app saves a flag in the `SettingsRepository`.
```kotlin
fun onFinish() {
    settingsRepository.putBoolean(KEY_ONBOARDING_COMPLETED, true)
}
```

### Developer Reset
For debugging or testing, a "Reset onboarding" button is available in [Developer Options](file:///Users/sameerasandakelum/GIT/jetpack-common/logic/utils/developer-mode.md). This clears the completion flag and forces the wizard to appear on the next app launch.

## 5. Implementation Tips
- **Haptics**: Navigation buttons use standard virtual key haptics, while the logo uses custom micro-haptics for a "mechanical dial" feel.
- **GIFs**: Use Coil with `ImageDecoderDecoder` to render educational animations smoothly.
- **Language First**: Always place the [Language Picker](file:///Users/sameerasandakelum/GIT/jetpack-common/logic/utils/languages.md) on the very first screen so the rest of the wizard is readable.
