# Bottom Sheet Design System

Standardized design and layout for all Modal Bottom Sheets in the application. We use Material 3 `ModalBottomSheet` with consistent container colors, rounded corners, and padding.

## 1. Base Structure

Every bottom sheet follows a standard structure to ensure a premium feel:
- **Container Color**: `surfaceContainerHigh` or `surfaceContainer`.
- **Top Padding**: Managed by the sheet's drag handle (default M3).
- **Horizontal Padding**: 16.dp for content.
- **Scroll Behavior**: Usually wrapped in a `LazyColumn` for content that might exceed screen height.

### Basic Template

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun StandardBottomSheet(
    onDismissRequest: () -> Unit
) {
    val sheetState = rememberModalBottomSheetState()

    ModalBottomSheet(
        onDismissRequest = onDismissRequest,
        sheetState = sheetState,
        containerColor = MaterialTheme.colorScheme.surfaceContainerHigh,
        // Optional: windowInsets = BottomSheetDefaults.windowInsets
    ) {
        Column(
            modifier = Modifier
                .fillMaxWidth()
                .padding(horizontal = 16.dp)
                .padding(bottom = 32.dp), // Extra bottom padding for navigation bar
            verticalArrangement = Arrangement.spacedBy(16.dp)
        ) {
            Text(
                text = "Sheet Title",
                style = MaterialTheme.typography.headlineSmall,
                fontWeight = FontWeight.Bold
            )
            
            // Content goes here
            RoundedCardContainer {
                // Settings or Info
            }
        }
    }
}
```

## 2. Shared Components inside Sheets

### Rounded Card Containers
We frequently use `RoundedCardContainer` inside sheets to group items, often with a `surfaceBright` background to contrast against the sheet's `surfaceContainerHigh`.

### Action Buttons
Buttons at the bottom of sheets are typically centered or use `FlowRow` for multiple social/support links.

```kotlin
FlowRow(
    modifier = Modifier.fillMaxWidth(),
    horizontalArrangement = Arrangement.Center,
    verticalArrangement = Arrangement.spacedBy(8.dp)
) {
    Button(onClick = { /* Primary */ }) { ... }
    OutlinedButton(onClick = { /* Secondary */ }) { ... }
}
```

## 3. Best Practices

- **Haptics**: Trigger haptic feedback when the sheet is opened or when a significant action inside the sheet is performed.
- **Edge-to-Edge**: Ensure `windowInsets` are handled if the sheet content needs to go behind the navigation bar.
- **State Preservation**: Use `rememberSaveable` for any user input inside the sheet.
