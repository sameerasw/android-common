# Segmented Picker Pattern

A premium, interactive selection component based on **Material 3 Expressive** design, used for toggling between a small set of options (e.g., Haptic intensities, Language modes, or Theme styles).

<img width="608" height="298" alt="CleanShot-scrcpy-Sameera Pixel 7-20260507-4  15 41@2x" src="https://github.com/user-attachments/assets/c93cd1fb-b8b2-4596-b752-82d083b0d2c3" />


## 1. Core Component (`SegmentedPicker`)

The `SegmentedPicker` uses a connected button group aesthetic where multiple `ToggleButton` components are joined together to form a single cohesive "pill".

### Connected Shapes
We utilize `ButtonGroupDefaults` to automatically calculate the correct corner radii for each segment:
- **Leading**: Rounded on the left.
- **Middle**: Rectangular (no roundness on inner edges).
- **Trailing**: Rounded on the right.

```kotlin
shapes = when (index) {
    0 -> ButtonGroupDefaults.connectedLeadingButtonShapes()
    items.lastIndex -> ButtonGroupDefaults.connectedTrailingButtonShapes()
    else -> ButtonGroupDefaults.connectedMiddleButtonShapes()
}
```

### Layout Strategy
Segments are placed inside a `Row` with `ButtonGroupDefaults.ConnectedSpaceBetween` to ensure they are visually fused.

```kotlin
Row(
    modifier = modifier
        .background(MaterialTheme.colorScheme.surfaceBright, RoundedCornerShape(12.dp))
        .padding(10.dp),
    horizontalArrangement = Arrangement.spacedBy(ButtonGroupDefaults.ConnectedSpaceBetween),
) {
    // ToggleButtons...
}
```

## 2. Generic Implementation (`<T>`)

The picker is designed to be data-agnostic using generics and provider lambdas. This allows it to handle Enums, Strings, or custom Data Classes seamlessly.

```kotlin
fun <T> SegmentedPicker(
    items: List<T>,
    selectedItem: T,
    onItemSelected: (T) -> Unit,
    labelProvider: (T) -> String,
    iconProvider: (@Composable (T) -> Unit)? = null
)
```

## 3. High-Level Wrapper (`ConfigPickerItem`)

In settings screens, the picker is often wrapped in a `ConfigPickerItem` list item. This provides:
- **Title & Description**: Context for the setting.
- **Current Selection Status**: A visual chip showing the active choice.
- **Interaction**: Tapping the item can expand a menu, show a bottom sheet, or reveal the segmented picker inline.

## 4. Haptic Feedback

To ensure a "physical" feel, every selection change triggers a UI haptic pulse.

```kotlin
onCheckedChange = {
    HapticUtil.performUIHaptic(view)
    onItemSelected(item)
}
```

## 5. Usage Examples
- **[Haptic Feedback](../../logic/utils/haptics.md)**: Selecting between "Subtle", "Click", and "Double".
- **Language Picker**: Toggling between system default and custom locales.
- **Refresh Rate**: Choosing between "Fixed" and "Range" modes.
