# Flashlight & Brightness Control Pattern

A sophisticated flashlight management system supporting granular brightness control, smooth fading effects, and hardware-aware protection.

## 1. Hardware Abstraction (`FlashlightUtil`)

Since flashlight intensity is an Android 13+ (SDK 33) feature, the utility provides a safe wrapper for both legacy toggling and modern strength adjustments.

### Key Hardware APIs
- `turnOnTorchWithStrengthLevel(cameraId, level)`: Sets granular brightness.
- `FLASH_INFO_STRENGTH_MAXIMUM_LEVEL`: Retrieves the maximum brightness step supported by the hardware (e.g., Pixel 7 supports 5-8 steps).

### Smooth Fading Algorithm
To provide a premium feel, the flashlight doesn't just "snap" to a level. It uses a coroutine-based loop to step through intensity levels.

```kotlin
suspend fun fadeFlashlight(from: Int, to: Int, durationMs: Long = 250L) {
    val steps = 10
    for (i in 1..steps) {
        val level = from + ((to - from) * i / steps)
        cameraManager.turnOnTorchWithStrengthLevel(cameraId, level)
        delay(durationMs / steps)
    }
}
```

## 2. Business Logic (`FlashlightHandler`)

The handler manages the flashlight state within the app's core service (`AccessibilityService`).

### Global Syncing
The system monitors `onTorchModeChanged`. If the flashlight is turned on externally (e.g., via the system Quick Settings tile), the app detects it and automatically fades the brightness up to the user's last saved intensity level.

### Proximity Protection
The flashlight pulse (for notifications) is automatically suppressed if the proximity sensor is blocked (e.g., the phone is face-down on a desk or in a pocket) to save battery and prevent unnecessary light.

## 3. Live Control Notifications

When the flashlight is active, a persistent "Control Center" notification appears.

### Android 15/16 Features
On the latest Android versions, the app utilizes advanced notification styles:
- **`ProgressStyle`**: Displays a visual brightness bar directly in the notification shade.
- **`setRequestPromotedOngoing`**: Ensures the notification stays prioritized at the top of the "Ongoing" section.
- **`setShortCriticalText`**: Shows the percentage (e.g., "80%") in the status bar/lock screen chip.

### Quick Actions
- **Adjust**: `+` and `-` buttons to step through brightness levels.
- **Kill Switch**: A prominent "Off" button to immediately extinguish the light.

## 4. Flashlight Pulse (Notification Lighting)
A dedicated feature that pulses the flashlight when a notification arrives. It uses the `fadeFlashlight` utility to create a soft "breathing" light effect rather than a harsh strobe.

## 5. Implementation Tips
- **Camera Conflict**: Always catch `CameraAccessException` with reason `CAMERA_IN_USE`. If the camera app is open, flashlight controls must be disabled or hidden.
- **Haptic Limits**: Trigger a "Double Tap" haptic feedback when the user reaches the minimum (1) or maximum brightness level.
- **Default Levels**: Use `FLASH_INFO_STRENGTH_DEFAULT_LEVEL` as the initial intensity if the user hasn't set a preference yet.
