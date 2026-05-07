# List Item Variants

A collection of standardized list items designed to be used inside the [Rounded Card Container](../containers/rounded-card-container.md). These components handle haptic feedback, adaptive layouts, and consistent Material 3 styling.

## 1. Icon Toggle Item (`IconToggleItem`)

The workhorse for settings screens. It supports a leading icon, title, optional description, and a trailing switch.

### Key Features
- **Toggle Mode**: Show/Hide the trailing switch with `showToggle`.
- **Haptic Feedback**: Integrated `HapticUtil.performVirtualKeyHaptic` on interaction.
- **Disabled State**: Supports `onDisabledClick` to explain why a feature is unavailable (e.g., missing root/permissions).
- **Secondary Action**: Can open another activity/dialog when `onClick` is provided while still having a toggle.

### Usage
```kotlin
IconToggleItem(
    iconRes = R.drawable.ic_vibrate,
    title = "Haptic Feedback",
    description = "Feel the clicks",
    isChecked = isEnabled,
    onCheckedChange = { viewModel.setHaptics(it) }
)
```

## 2. Feature Card (`FeatureCard`)

A more prominent variant often used for main feature toggles. It features **Colorful Pastel Icons** and a more complex menu system.

### Key Features
- **Pastel Icons**: Automatically generates a soft background color and vibrant icon tint based on the title.
- **Contextual Menu**: Long-press triggers a `SegmentedDropdownMenu` for actions like "Pin" or "Help".
- **Dynamic Blur**: When the long-press menu is open, other cards in the list can blur/fade for focus (via `LocalMenuStateManager`).
- **Beta Badge**: Built-in indicator for experimental features.

### Implementation: Pastel Icons
The color logic ensures icons feel unique and premium without manual color assignment.

```kotlin
// Inside FeatureCard's leadingContent
Box(
    modifier = Modifier
        .size(40.dp)
        .background(
            color = ColorUtil.getPastelColorFor(title),
            shape = CircleShape
        ),
    contentAlignment = Alignment.Center
) {
    Icon(
        painter = painterResource(id = iconRes),
        tint = ColorUtil.getVibrantColorFor(title)
    )
}
```

## 3. Implementation Logic

### Color Utility
Used to generate consistent colors from any string or ID.

```kotlin
object ColorUtil {
    private val pastelColors = listOf(...) // Soft colors

    fun getPastelColorFor(key: Any): Color {
        val index = abs(key.hashCode()) % pastelColors.size
        return pastelColors[index]
    }

    fun getVibrantColorFor(key: Any): Color {
        val baseColor = getPastelColorFor(key)
        // Adjust saturation/value for the icon...
        return vibrantColor
    }
}
```

## 4. Design Guidelines

- **Grouping**: Always wrap list items in a `RoundedCardContainer`.
- **Haptics**: Always call `HapticUtil` in the `onClick` or `onCheckedChange` callbacks.
- **Interactions**:
    - Use `IconToggleItem` for simple boolean settings.
    - Use `FeatureCard` for high-level features that have sub-settings or documentation.
    - If a feature requires root or a specific permission, use the `enabled = false` and `onDisabledClick` pattern to show a guidance sheet.
