# Rounded Card Container

A specialized container designed to group related list items or components into a visually distinct, rounded section. This is the foundation for all settings and feature groups in the application.

## Features

- **Implicit Clipping**: Automatically clips children to its rounded corners.
- **Customizable Spacing**: Uses `Arrangement.spacedBy` for consistent internal padding between items.
- **Surface Integration**: Typically used with a background color (like `surfaceContainer`) to create a "card group" effect.
- **Uniform Radius**: Defaults to a 24.dp radius, aligning with Material 3 expressive design.

## Implementation

```kotlin
@Composable
fun RoundedCardContainer(
    modifier: Modifier = Modifier,
    spacing: Dp = 2.dp,
    cornerRadius: Dp = 24.dp,
    containerColor: Color = MaterialTheme.colorScheme.surfaceBright,
    content: @Composable ColumnScope.() -> Unit
) {
    Column(
        modifier = modifier
            .clip(RoundedCornerShape(cornerRadius))
            .background(containerColor),
        verticalArrangement = Arrangement.spacedBy(spacing),
        content = content
    )
}
```

## Usage

The `RoundedCardContainer` is most effective when used to wrap multiple [List Items](../cards/list-items.md).

```kotlin
RoundedCardContainer {
    IconToggleItem(...)
    IconToggleItem(...)
    FeatureCard(...)
}
```

### Layout Tip
When using this inside a `Column`, it's common to add a label above the container:

```kotlin
Text(
    text = "App Settings",
    style = MaterialTheme.typography.titleMedium,
    modifier = Modifier.padding(start = 16.dp, top = 16.dp, bottom = 8.dp)
)
RoundedCardContainer {
    // Items
}
```
