# Theming & Pitch Black Pattern

A flexible theming system based on **Material 3 Expressive**, supporting Dynamic Color (Monet) and a specialized "Pitch Black" mode for AMOLED displays.

## 1. Material 3 Dynamic Color

The app utilizes `dynamicDarkColorScheme` and `dynamicLightColorScheme` on Android 12+ (SDK 31) to match the system's wallpaper-based colors.

```kotlin
val colorScheme = when {
    dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
        val context = LocalContext.current
        if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
    }
    darkTheme -> DarkColorScheme
    else -> LightColorScheme
}
```

## 2. Pitch Black (AMOLED) Implementation

Pitch Black mode is implemented by intercepting the `ColorScheme` and forcing all surface and background containers to pure black (`#000000`). This preserves the dynamic accent colors (Primary, Secondary) while ensuring maximum battery savings on AMOLED screens.

### Token Overrides
When `pitchBlackTheme` is enabled, the following tokens must be overridden:

```kotlin
val pitchBlackScheme = baseScheme.copy(
    background = Color.Black,
    surface = Color.Black,
    surfaceContainer = Color.Black,
    surfaceContainerLowest = Color.Black,
    surfaceContainerLow = Color.Black,
    surfaceContainerHigh = Color.Black,
    surfaceContainerHighest = Color.Black,
    surfaceVariant = Color.Black
)
```

## 3. Custom Typography

The app uses **Google Sans Flex** with variable font support.

### Variable Font Axes
We utilize the `ROND` (Roundness) axis for specific UI elements to create a softer, more premium look.

```kotlin
val GoogleSansFlexRounded = FontFamily(
    Font(
        R.font.google_sans_flex,
        variationSettings = FontVariation.Settings(
            FontVariation.Setting("ROND", 100f) // Max roundness
        )
    )
)
```

## 4. Component Shapes

To match the Material 3 Expressive aesthetic, we use extra-large corner radii for containers and cards.

```kotlin
val Shapes = Shapes(
    extraSmall = RoundedCornerShape(4.dp),
    small = RoundedCornerShape(8.dp),
    medium = RoundedCornerShape(12.dp),
    large = RoundedCornerShape(16.dp),
    extraLarge = RoundedCornerShape(28.dp) // Signature rounded look
)
```

## 5. Implementation Tips

- **Surface Tints**: In Pitch Black mode, ensure that `surfaceTint` is still applied or handled carefully, as Material 3 normally uses elevation-based tinting which can look "gray" on black backgrounds.
- **Contrast Ratios**: When overriding background to black, verify that `onBackground` and `onSurface` colors (usually from the dynamic scheme) still provide sufficient contrast.
- **Status Bar**: Always update the `SystemUiController` (or `WindowInsetsController`) to ensure the status bar icons are visible against the pitch black background.
