# Reusable Top App Bar

A flexible and feature-rich Top App Bar implementation that supports large flexible headers, profile menus, search, and update notifications.

## Features

- **Flexible Sizing**: Automatically adjusts between `TopAppBar` and `LargeFlexibleTopAppBar` based on needs.
- **Dynamic Title/Subtitle**: Supports both title and subtitle with beta label integration.
- **Contextual Actions**: Built-in support for search, settings, help, and update badges.
- **GitHub Integration**: Specialized profile icon slot for GitHub users with a built-in dropdown menu.
- **Haptic Feedback**: Standardized haptics for all interactive icons.

## Implementation

### Composable Component

```kotlin
@OptIn(ExperimentalMaterial3Api::class, ExperimentalMaterial3ExpressiveApi::class)
@Composable
fun ReusableTopAppBar(
    title: Any, // Can be Int (Resource ID) or String
    subtitle: Any? = null,
    hasBack: Boolean = false,
    onBackClick: (() -> Unit)? = null,
    hasSettings: Boolean = false,
    onSettingsClick: (() -> Unit)? = null,
    hasUpdateAvailable: Boolean = false,
    onUpdateClick: (() -> Unit)? = null,
    gitHubUser: GitHubUser? = null,
    onSignOutClick: (() -> Unit)? = null,
    scrollBehavior: TopAppBarScrollBehavior? = null,
    isSmall: Boolean = false,
    actions: @Composable RowScope.() -> Unit = {}
) {
    // Title Content with Beta Label support
    val titleContent: @Composable () -> Unit = {
        Column {
            Text(resolve(title), maxLines = 1, overflow = TextOverflow.Ellipsis)
            if (subtitle != null) {
                Text(
                    resolve(subtitle),
                    style = MaterialTheme.typography.bodyMedium,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
        }
    }

    // Actions including Profile, Update, and Settings
    val actionsContent: @Composable RowScope.() -> Unit = {
        actions()
        
        if (hasUpdateAvailable) {
            IconButton(onClick = onUpdateClick ?: {}) {
                Box {
                    Icon(painterResource(R.drawable.rounded_mobile_arrow_down_24), "Update")
                    Box(Modifier.size(10.dp).align(Alignment.TopEnd).background(Color.Red, CircleShape))
                }
            }
        }

        if (gitHubUser != null) {
            // Profile menu implementation...
        }

        if (hasSettings) {
            IconButton(onClick = onSettingsClick ?: {}) {
                Icon(painterResource(R.drawable.rounded_settings_heart_24), "Settings")
            }
        }
    }

    if (isSmall) {
        TopAppBar(
            title = titleContent,
            navigationIcon = { /* Back button if hasBack */ },
            actions = actionsContent,
            scrollBehavior = scrollBehavior
        )
    } else {
        LargeFlexibleTopAppBar(
            title = titleContent,
            navigationIcon = { /* Back button if hasBack */ },
            actions = actionsContent,
            scrollBehavior = scrollBehavior,
            expandedHeight = if (subtitle != null) 152.dp else 120.dp
        )
    }
}
```

## Usage Examples

### Standard Page Header

```kotlin
ReusableTopAppBar(
    title = R.string.title_settings,
    hasBack = true,
    onBackClick = { navController.popBackStack() }
)
```

### Main Screen with Profile & Updates

```kotlin
ReusableTopAppBar(
    title = "App Name",
    subtitle = "Welcome back!",
    hasUpdateAvailable = true,
    gitHubUser = currentUser,
    onSettingsClick = { /* Navigate to settings */ }
)
```
