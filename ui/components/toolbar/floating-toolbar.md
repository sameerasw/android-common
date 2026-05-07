# Floating Bottom Toolbar

A highly adaptive and animated floating bottom toolbar designed for modern Jetpack Compose applications. It supports two primary modes: Tabbed navigation and Standard action bar.

## Features

- **Tabbed Mode**: Animated expanding labels for the selected item.
- **Standard Mode**: Contextual back button and title support.
- **Adaptive Sizing**: Automatically hides labels on compact screens or large accessibility font scales.
- **Experimental M3 Expressive**: Uses `HorizontalFloatingToolbar` for a premium feel.
- **Badge Support**: Built-in red dot badge for notification/update awareness.
- **Haptic Integration**: Native haptic feedback on interaction.

## Implementation

### 1. Data Model

```kotlin
data class ToolbarItem(
    val iconRes: Int,
    val labelRes: Int,
    val onClick: () -> Unit,
    val hasBadge: Boolean = false
)
```

### 2. Composable Component

```kotlin
@OptIn(ExperimentalMaterial3Api::class, ExperimentalMaterial3ExpressiveApi::class)
@Composable
fun EssentialsFloatingToolbar(
    modifier: Modifier = Modifier,
    // Tabbed Mode
    items: List<ToolbarItem> = emptyList(),
    selectedIndex: Int = -1,
    // Standard Mode
    title: String? = null,
    onBackClick: (() -> Unit)? = null,
    // Action Slots
    floatingActionButton: (@Composable () -> Unit)? = null,
    scrollBehavior: FloatingToolbarScrollBehavior? = null,
    expanded: Boolean = true
) {
    val view = LocalView.current
    val configuration = LocalConfiguration.current
    val fontScale = LocalDensity.current.fontScale
    
    // Adaptive logic: Hide labels if font is too large or screen is too narrow
    val shouldHideLabel = fontScale > 1.25f || (configuration.screenWidthDp < 400 && items.size > 3)

    HorizontalFloatingToolbar(
        modifier = modifier
            .windowInsetsPadding(WindowInsets.navigationBars)
            .padding(horizontal = 16.dp),
        expanded = expanded,
        floatingActionButton = floatingActionButton ?: {},
        scrollBehavior = scrollBehavior,
        colors = FloatingToolbarDefaults.vibrantFloatingToolbarColors(
            toolbarContentColor = MaterialTheme.colorScheme.onSurface,
            toolbarContainerColor = MaterialTheme.colorScheme.primary,
        ),
        content = {
            if (onBackClick != null) {
                // STANDARD MODE: Back button and Title
                IconButton(
                    onClick = {
                        HapticUtil.performVirtualKeyHaptic(view)
                        onBackClick()
                    },
                    modifier = Modifier.size(48.dp),
                    colors = IconButtonDefaults.filledIconButtonColors(
                        contentColor = MaterialTheme.colorScheme.primary,
                        containerColor = MaterialTheme.colorScheme.background
                    )
                ) {
                    Icon(
                        painter = painterResource(id = R.drawable.rounded_arrow_back_24),
                        contentDescription = "Back",
                        modifier = Modifier.size(24.dp)
                    )
                }

                if (title != null) {
                    Spacer(modifier = Modifier.width(8.dp))
                    Text(
                        text = title,
                        style = MaterialTheme.typography.titleMedium,
                        color = MaterialTheme.colorScheme.background,
                        maxLines = 1,
                        overflow = TextOverflow.Ellipsis,
                        modifier = Modifier
                            .widthIn(max = 250.dp)
                            .padding(horizontal = 8.dp)
                    )
                }
            } else {
                // TABBED MODE: Expanding labels
                items.forEachIndexed { index, item ->
                    val isSelected = selectedIndex == index

                    val itemWidth by animateDpAsState(
                        targetValue = if (expanded || isSelected) 48.dp else 0.dp,
                        animationSpec = spring(dampingRatio = Spring.DampingRatioMediumBouncy, stiffness = Spring.StiffnessLow),
                        label = "item_width"
                    )

                    val labelWidth by animateDpAsState(
                        targetValue = if (isSelected && !shouldHideLabel) 80.dp else 0.dp,
                        animationSpec = spring(dampingRatio = Spring.DampingRatioMediumBouncy, stiffness = Spring.StiffnessLow),
                        label = "label_width"
                    )

                    if (itemWidth > 0.dp || isSelected) {
                        IconButton(
                            onClick = {
                                HapticUtil.performVirtualKeyHaptic(view)
                                item.onClick()
                            },
                            modifier = Modifier
                                .width(itemWidth + labelWidth)
                                .height(48.dp),
                            colors = if (isSelected) {
                                IconButtonDefaults.filledIconButtonColors(
                                    contentColor = MaterialTheme.colorScheme.primary,
                                    containerColor = MaterialTheme.colorScheme.background
                                )
                            } else {
                                IconButtonDefaults.iconButtonColors(
                                    contentColor = MaterialTheme.colorScheme.background,
                                    containerColor = MaterialTheme.colorScheme.primary
                                )
                            }
                        ) {
                            Row(
                                verticalAlignment = Alignment.CenterVertically,
                                horizontalArrangement = Arrangement.Center,
                                modifier = Modifier.padding(horizontal = 8.dp)
                            ) {
                                Box {
                                    Icon(
                                        painter = painterResource(id = item.iconRes),
                                        contentDescription = stringResource(id = item.labelRes),
                                        modifier = Modifier.size(24.dp)
                                    )
                                    if (item.hasBadge) {
                                        Canvas(modifier = Modifier.size(8.dp).align(Alignment.TopEnd)) {
                                            drawCircle(color = Color.Red)
                                        }
                                    }
                                }
                                if (isSelected && !shouldHideLabel) {
                                    Spacer(modifier = Modifier.width(8.dp))
                                    Text(
                                        text = stringResource(id = item.labelRes),
                                        style = MaterialTheme.typography.labelLarge,
                                        maxLines = 1,
                                        color = MaterialTheme.colorScheme.primary
                                    )
                                }
                            }
                        }
                        
                        if (index < items.size - 1) {
                            Spacer(modifier = Modifier.width(8.dp))
                        }
                    }
                }
            }
        }
    )
}
```

## Usage Example

### Navigation Tabs

```kotlin
EssentialsFloatingToolbar(
    selectedIndex = currentPage,
    items = listOf(
        ToolbarItem(R.drawable.ic_home, R.string.tab_home, { currentPage = 0 }),
        ToolbarItem(R.drawable.ic_settings, R.string.tab_settings, { currentPage = 1 })
    ),
    floatingActionButton = {
        FloatingActionButton(onClick = { /* Action */ }) {
            Icon(Icons.Default.Add, contentDescription = null)
        }
    }
)
```

### Contextual Title

```kotlin
EssentialsFloatingToolbar(
    title = "Settings",
    onBackClick = { finish() },
    fabAction = { /* ... */ },
    fabIconRes = R.drawable.ic_save
)
```
