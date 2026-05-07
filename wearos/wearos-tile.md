# WearOS Tiles Pattern

Implementation of glanceable information on the watch face using the **Protolayout** library.

## 1. Service Setup (`SuspendingTileService`)

Using `SuspendingTileService` (from Horologist) simplifies resource and layout management by providing coroutine support.

```kotlin
class MainTileService : SuspendingTileService() {
    override suspend fun tileRequest(requestParams: TileRequest) = tile(requestParams, this)
    override suspend fun resourcesRequest(requestParams: ResourcesRequest) = resources(requestParams)
}
```

## 2. Layout Structure (`EdgeContentLayout`)

For circular screens, `EdgeContentLayout` is the standard pattern to ensure content fits within the safe area while utilizing the curved edges for progress indicators.

### Key Elements
- **Edge Content**: Usually a `CircularProgressIndicator` following the screen curvature.
- **Primary Label**: A small header text.
- **Content**: The main body, often a `Column` of events or data.

```kotlin
val edgeContentLayout = EdgeContentLayout.Builder(deviceConfiguration)
    .setEdgeContent(progressIndicator)
    .setPrimaryLabelTextContent(headerText)
    .setContent(mainContent)
    .build()
```

## 3. Dynamic Data & Colors

Tiles are not standard Compose; they use `LayoutElementBuilders`. Colors must be converted to `argb` format.

```kotlin
val lightAccent = ThemeUtil.getLightAccentColor(themeColor)
Text.Builder(context, "Hello")
    .setColor(argb(lightAccent))
    .setTypography(Typography.TYPOGRAPHY_BODY1)
    .build()
```

## 4. Interaction (`LaunchAction`)

Tapping the tile should deep-link into a specific screen of the main WearOS app.

```kotlin
val openAppIntent = ActionBuilders.LaunchAction.Builder()
    .setAndroidActivity(
        ActionBuilders.AndroidActivity.Builder()
            .setPackageName(context.packageName)
            .setClassName("com.sameerasw.essentials.presentation.MainActivity")
            .addKeyToExtraMapping("navigate_to", stringExtra("schedule"))
            .build()
    )
    .build()
```

## 5. Implementation Tips
- **Resource Versioning**: Always update the `RESOURCES_VERSION` string when changing image assets to ensure the system flushes the old cache.
- **Responsive Layouts**: Use `setResponsiveContentInsetEnabled(true)` to automatically adjust padding based on the device's chin or bezel.
- **Preview Support**: Implement `@Preview` with `TilePreviewData` to iterate quickly in the IDE.

## See Also
- **[WearOS Communication Logic](../logic/utils/wearos.md)**: Details on how tiles fetch data from the phone.
