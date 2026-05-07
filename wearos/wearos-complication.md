# WearOS Complications Pattern

Implementation of small, modular data providers that can be placed directly on the watch face.

## 1. Service Setup (`SuspendingComplicationDataSourceService`)

The core provider must extend `SuspendingComplicationDataSourceService` to handle asynchronous data fetching (e.g., from `SharedPreferences` updated by the phone bridge).

```kotlin
class BatteryComplicationService : SuspendingComplicationDataSourceService() {
    override suspend fun onComplicationRequest(request: ComplicationRequest): ComplicationData {
        val phoneBattery = prefs.getInt("phone_battery_level", -1)
        return createComplicationData(watchBattery, phoneBattery, request.complicationType)
    }
}
```

## 2. Dynamic Icon Generation (`Canvas`)

For complex complications (like showing both Watch and Phone battery), we use a `Canvas` to draw a custom `MonochromaticImage`.

### Dual Progress Pattern
- **Outer Ring**: Watch battery level.
- **Inner Ring**: Phone battery level.
- **Implementation**: `Canvas.drawArc()` with different radii.

```kotlin
val bitmap = Bitmap.createBitmap(128, 128, Bitmap.Config.ARGB_8888)
val canvas = Canvas(bitmap)
// Draw outer arc...
// Draw inner arc...
val icon = Icon.createWithBitmap(bitmap)
val monochromaticImage = MonochromaticImage.Builder(icon).build()
```

## 3. Complication Types Support

To ensure compatibility with different watch faces, implement support for multiple `ComplicationType`s:

- **SHORT_TEXT**: Standard for most watch face slots.
- **RANGED_VALUE**: Best for progress indicators (0-100%).
- **MONOCHROMATIC_IMAGE**: For icon-only slots.

## 4. Interaction (`TapAction`)

Every complication should have a `tapAction` that deep-links the user to the relevant screen in the main app.

```kotlin
private fun getTapAction(): PendingIntent {
    val intent = Intent(this, MainActivity::class.java).apply {
        putExtra("navigate_to", "your_android")
    }
    return PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_IMMUTABLE)
}
```

## 5. Implementation Tips
- **Preview Data**: Always override `getPreviewData()` to provide sensible defaults for the watch face picker UI.
- **Update Frequency**: Complications are updated by the system; ensure your data is ready in `SharedPreferences` so the update is instantaneous when requested.
- **Monochromatic Requirement**: Ensure custom bitmaps used for `MonochromaticImage` are purely white with an alpha channel; the system will tint them automatically.

## See Also
- **[WearOS Communication Logic](../logic/utils/wearos.md)**: Details on how data is synced from the phone to the watch.
