# Feature Info Bottom Sheet

A contextual bottom sheet used to explain specific features, their purpose, and their required permissions.

## 1. Adaptive Header

The header reuses the **Pastel Icon** pattern from the [Feature Card](../cards/list-items.md) to maintain visual continuity.

```kotlin
Row(
    modifier = Modifier.padding(start = 8.dp),
    verticalAlignment = Alignment.CenterVertically,
    horizontalArrangement = Arrangement.spacedBy(8.dp)
) {
    Box(
        modifier = Modifier
            .size(40.dp)
            .background(
                color = ColorUtil.getPastelColorFor(featureTitle),
                shape = CircleShape
            ),
        contentAlignment = Alignment.Center
    ) {
        Icon(
            painter = painterResource(feature.iconRes),
            tint = ColorUtil.getVibrantColorFor(featureTitle)
        )
    }

    Text(text = featureTitle, style = MaterialTheme.typography.titleMedium)
}
```

## 2. Permission Badges

If a feature requires permissions, we display them as `AssistChip` items in a `LazyRow` at the bottom of the sheet. This gives the user a quick overview of requirements without cluttering the description.

```kotlin
if (feature.permissionKeys.isNotEmpty()) {
    LazyRow(
        horizontalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(permissions) { permission ->
            AssistChip(
                onClick = { /* Explain permission */ },
                label = { Text(permission.title) },
                leadingIcon = { Icon(painterResource(permission.iconRes), ...) },
                colors = AssistChipDefaults.assistChipColors(
                    containerColor = MaterialTheme.colorScheme.surfaceContainerHigh
                )
            )
        }
    }
}
```

## 3. Usage

This sheet is typically triggered from a "What is this?" or "Help" menu item on a [Feature Card](../cards/list-items.md).

```kotlin
onHelpClick = {
    selectedFeature = feature
    showHelpSheet = true
}
```
