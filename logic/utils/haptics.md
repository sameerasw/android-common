# Haptic Feedback Utility

A centralized and robust haptic feedback system that ensures consistent tactile responses throughout the application. It supports both in-app UI interactions and background service feedback.

## 1. Core Logic

### HapticFeedbackType Enum
Defines different "feelings" for haptic responses.

```kotlin
enum class HapticFeedbackType {
    NONE,
    SUBTLE,
    DOUBLE,
    CLICK,
    TICK
}
```

### HapticUtil Object
The main entry point for UI-driven haptics. It includes app-wide toggling and device-specific optimizations (including API 30+ primitives).

```kotlin
object HapticUtil {
    val isAppHapticsEnabled = mutableStateOf(true)

    // Standard UI Tap (Keyboard feel)
    fun performUIHaptic(view: View) {
        if (!isAppHapticsEnabled.value) return
        view.performHapticFeedback(HapticFeedbackConstants.KEYBOARD_TAP)
    }

    // Light Tick (Clock/Selection feel)
    fun performLightHaptic(view: View) {
        if (!isAppHapticsEnabled.value) return
        view.performHapticFeedback(HapticFeedbackConstants.CLOCK_TICK)
    }

    // Heavy/Virtual Key (Button press feel)
    fun performHeavyHaptic(view: View) {
        if (!isAppHapticsEnabled.value) return
        view.performHapticFeedback(HapticFeedbackConstants.VIRTUAL_KEY)
    }

    // Micro Haptic (Extremely subtle tick)
    fun performMicroHaptic(view: View) {
        performCustomHaptic(view, 0.02f)
    }

    /**
     * Advanced custom haptic using Primitives (API 30+)
     * Falls back to amplitude control (API 26+) or standard ticks.
     */
    fun performCustomHaptic(view: View, strength: Float) {
        if (!isAppHapticsEnabled.value) return
        val vibrator = getVibrator(view.context)

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
            if (vibrator.areAllPrimitivesSupported(VibrationEffect.Composition.PRIMITIVE_CLICK)) {
                val effect = VibrationEffect.startComposition()
                    .addPrimitive(VibrationEffect.Composition.PRIMITIVE_CLICK, strength)
                    .compose()
                vibrator.vibrate(effect, VibrationAttributes.createForUsage(VibrationAttributes.USAGE_TOUCH))
                return
            }
        }
        
        // Fallback for older APIs
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O && vibrator.hasAmplitudeControl()) {
            val amplitude = (strength * strength * 255).toInt().coerceIn(1, 255)
            vibrator.vibrate(VibrationEffect.createOneShot(12, amplitude))
        } else {
            view.performHapticFeedback(if (strength < 0.5f) HapticFeedbackConstants.CLOCK_TICK else HapticFeedbackConstants.KEYBOARD_TAP)
        }
    }
}
```

## 2. Background Service Haptics

For non-UI contexts (like Accessibility Services or Tiles), use the Context-based vibrator helper.

```kotlin
fun performHapticFeedback(vibrator: Vibrator, feedbackType: HapticFeedbackType) {
    if (!vibrator.hasVibrator()) return

    when (feedbackType) {
        HapticFeedbackType.SUBTLE -> vibrator.vibrate(VibrationEffect.createPredefined(VibrationEffect.EFFECT_TICK))
        HapticFeedbackType.DOUBLE -> {
            val pattern = longArrayOf(0, 40, 60, 40)
            val amplitudes = intArrayOf(0, 180, 0, 220)
            vibrator.vibrate(VibrationEffect.createWaveform(pattern, amplitudes, -1))
        }
        HapticFeedbackType.CLICK -> {
            val pattern = longArrayOf(0, 50, 60, 30)
            val amplitudes = intArrayOf(0, 200, 0, 150)
            vibrator.vibrate(VibrationEffect.createWaveform(pattern, amplitudes, -1))
        }
        // ...
    }
}
```

## 3. Use Cases

### Standard Button
Always add haptics to button clicks for a premium feel.
```kotlin
Button(onClick = { 
    HapticUtil.performUIHaptic(view)
    onAction() 
}) { /* ... */ }
```

### Slider / Progress
Use `performSliderHaptic` or `performMicroHaptic` during value changes.
```kotlin
Slider(
    value = val,
    onValueChange = { 
        if (it != val) HapticUtil.performSliderHaptic(view)
        val = it
    }
)
```

### Long Press / Success
Use `performHeavyHaptic` for significant actions like long-press or completion.

## 4. Settings Integration
The utility includes helper methods to persist the user's haptic preference.

```kotlin
fun saveAppHapticsEnabled(context: Context, enabled: Boolean) {
    val prefs = context.getSharedPreferences("app_prefs", Context.MODE_PRIVATE)
    prefs.edit().putBoolean("app_haptics_enabled", enabled).apply()
    isAppHapticsEnabled.value = enabled
}
```
