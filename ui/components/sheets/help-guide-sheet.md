# Help & Guides Bottom Sheet

A specialized bottom sheet used for long-form instructions, tutorials, and support links. It features **Expandable Sections** to keep the interface clean while offering deep information.

<img width="996" height="1078" alt="CleanShot-scrcpy-Sameera Pixel 7-20260507-4  13 27@2x" src="https://github.com/user-attachments/assets/6fdab110-76d2-4d19-87a5-13ab64c1e535" />


## 1. Expandable Sections Pattern

We use a combination of `Card` and `AnimatedVisibility` to create smooth expansion animations.

### Data Structure
```kotlin
data class InstructionSection(
    val title: String,
    val iconRes: Int,
    val description: String? = null,
    val steps: List<InstructionStep> = emptyList(),
    val links: List<Pair<String, String>> = emptyList()
)
```

### Expandable Component
```kotlin
@Composable
fun ExpandableGuideSection(section: InstructionSection) {
    var expanded by remember { mutableStateOf(false) }
    val rotation by animateFloatAsState(if (expanded) 180f else 0f)

    Card(
        modifier = Modifier.clickable { expanded = !expanded },
        colors = CardDefaults.cardColors(
            containerColor = if (expanded) 
                MaterialTheme.colorScheme.surfaceBright 
            else 
                MaterialTheme.colorScheme.surfaceContainerLow
        )
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            // Header Row (Icon + Title + Arrow)
            Row(...) {
                Icon(painterResource(section.iconRes), ...)
                Text(section.title, ...)
                Icon(Icons.Rounded.ArrowDropDown, Modifier.rotate(rotation))
            }

            // Animated Content
            AnimatedVisibility(visible = expanded) {
                Column(modifier = Modifier.padding(top = 24.dp)) {
                    if (section.description != null) {
                        Text(section.description, ...)
                    }
                    
                    // Render Steps with Images
                    section.steps.forEach { step ->
                        InstructionStepItem(step)
                    }
                }
            }
        }
    }
}
```

## 2. Instruction Steps with Images

To make guides clear, we use a vertical list of instructions followed by a `clip` and `ContentScale.FillWidth` image.

```kotlin
@Composable
private fun InstructionStepItem(step: InstructionStep) {
    Column(...) {
        Text(step.instruction, ...)
        Image(
            painter = painterResource(step.imageRes),
            contentDescription = null,
            modifier = Modifier
                .fillMaxWidth()
                .clip(RoundedCornerShape(12.dp)),
            contentScale = ContentScale.FillWidth
        )
    }
}
```

## 3. Social & Support Links

At the bottom of the sheet, we use a `FlowRow` to provide quick access to external resources like GitHub, Telegram, or Email.

```kotlin
FlowRow(
    horizontalArrangement = Arrangement.Center,
    maxItemsInEachRow = 3
) {
    Button(onClick = { openGitHub() }) { ... }
    OutlinedButton(onClick = { openTelegram() }) { ... }
}
```

## 4. Usage Tips
- **Group Sections**: Use a `RoundedCardContainer` to wrap all `ExpandableGuideSection` items for a unified look.
- **Lazy Loading**: Since guides can be very long with many images, use `LazyColumn` for the entire sheet content to maintain performance.
