# About & Community Components

Premium UI components used to display application information, developer credits, and community links.

## 1. About Section Component

A centralized component that displays the app version, a description, and the developer's avatar. It also includes a `FlowRow` of social and support buttons.

### Key Features
- **Version Detection**: Automatically fetches `versionName` from the package manager.
- **Developer Avatar**: Uses a `clip(RoundedCornerShape(16.dp))` image.
- **Social Links**: Provides buttons for Website, GitHub, Email, Telegram, and Support (Buy Me a Coffee).
- **Other Apps**: Displays a secondary section with links to other projects by the developer.

### Developer Mode Trigger
The avatar is wrapped in a `combinedClickable` modifier, which triggers a callback on long-press. This is the primary entry point for enabling hidden developer settings.

```kotlin
Image(
    modifier = Modifier
        .combinedClickable(
            onClick = { /* Simple view */ },
            onLongClick = { onAvatarLongClick() } // Triggers Developer Mode
        )
)
```

## 2. MadebySameerasw Community Card

A visually rich promotional card for the developer's community.

### Design Elements
- **Overlapping Avatar**: A `CircleShape` avatar that overlaps the banner image using an `offset(y = 32.dp)`.
- **Layered Borders**: Uses multiple borders (surface color, subtle accent, and sharp inner stroke) to create a high-end "glow" effect.
- **Banner Image**: A full-width `ContentScale.Crop` image as the background header.
- **Typography**: Uses `GoogleSansFlexRounded` for the community title.

```kotlin
Surface(
    modifier = modifier.clip(RoundedCornerShape(32.dp)),
    color = MaterialTheme.colorScheme.surfaceBright
) {
    Column {
        Box {
            Image(painterResource(R.drawable.banner), ...)
            Avatar(modifier = Modifier.align(Alignment.BottomCenter).offset(y = 32.dp))
        }
        Spacer(modifier = Modifier.height(42.dp))
        CommunityInfo(...)
    }
}
```

## 3. Usage Pattern

These components are typically placed at the bottom of the **Settings** screen inside a `RoundedCardContainer`.

```kotlin
RoundedCardContainer {
    AboutSection(
        onAvatarLongClick = { viewModel.toggleDeveloperMode() }
    )
}
```
