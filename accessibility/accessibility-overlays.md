# Accessibility & System Overlays

A specialized pattern for drawing UI elements over the entire system, including the status bar, navigation bar, and lock screen, using a combination of `WindowManager` and `AccessibilityService`.

## 1. The Overlay Mechanism

Overlays are created by adding a custom `View` directly to the `WindowManager`.

### Layout Parameters
To ensure the overlay covers the entire screen and ignores touch events (if needed), use these specific flags:

```kotlin
val params = WindowManager.LayoutParams(
    WindowManager.LayoutParams.MATCH_PARENT,
    WindowManager.LayoutParams.MATCH_PARENT,
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) 
        WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY 
    else 
        WindowManager.LayoutParams.TYPE_PHONE,
    WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE or
            WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE or
            WindowManager.LayoutParams.FLAG_LAYOUT_IN_SCREEN or
            WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS or
            WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED,
    PixelFormat.TRANSLUCENT
)
```

## 2. Accessibility Delegation Pattern

Starting with Android 12, standard `Service` overlays have restricted visibility over system windows (like the status bar and lock screen). To achieve "true" full-screen coverage, the app delegates the drawing task to an `AccessibilityService`.

### Why use an Accessibility Service?
- **Higher Elevation**: Accessibility overlays reside in a higher window layer than standard app overlays.
- **Lock Screen Access**: They can draw over the Keyguard/Lock screen without being dismissed.
- **System Bar Coverage**: They can reliably draw over the status and navigation bars.

### How it works
1.  A standard service (e.g., `NotificationLightingService`) receives an event.
2.  It checks if the `AccessibilityService` is enabled.
3.  If enabled, it sends an `Intent` with the overlay configuration to the Accessibility Service.
4.  The Accessibility Service handles the `WindowManager.addView()` call.

```kotlin
// In NotificationLightingService
if (isAccessibilityServiceEnabled()) {
    val intent = Intent(context, MyAccessibilityService::class.java).apply {
        action = "SHOW_OVERLAY"
        putExtra("color", resolvedColor)
    }
    context.startService(intent) // Deliver config to the elevated service
    stopSelf() // Standard service can now stop
}
```

## 3. Native System Ripples

Instead of drawing custom views, you can trigger native Android animations (like the charging ripple) using shell commands via [Shizuku/Root](file:///Users/sameerasandakelum/GIT/jetpack-common/permissions/shizuku.md).

```kotlin
// Charging ripple from the center
ShellUtils.runCommand(context, "cmd statusbar charging-ripple")

// Custom auth ripple at specific coordinates
ShellUtils.runCommand(context, "cmd statusbar auth-ripple custom $x $y")
```

## 4. Lifecycle & Stability

### Sticky Services
Overlay services should be `START_STICKY` and run in the foreground to prevent being killed by the system.

### Screen State Handling
Register a `BroadcastReceiver` for `ACTION_SCREEN_OFF` and `ACTION_SCREEN_ON` to refresh or remove overlays as needed. This is particularly important for "Ambient Display" or "Always On" lighting features.

## 5. Implementation Tips
- **Fade Transitions**: Never show or hide an overlay abruptly. Always use a `ValueAnimator` to fade the alpha for a premium feel.
- **Corner Radius**: Detect the device's corner radius (or provide a slider in settings) so the overlay stroke perfectly matches the physical screen corners.
- **Hardware Acceleration**: Always enable `FLAG_HARDWARE_ACCELERATED` for smooth animations, especially when using complex shaders or blurring.
