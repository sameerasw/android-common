# Edge-to-Edge Implementation

This guide covers the implementation of adaptive edge-to-edge support in Jetpack Compose, ensuring the app content draws behind system bars while handling insets correctly.

<img width="1672" height="266" alt="CleanShot-scrcpy-Sameera Pixel 7-20260507-4  21 18@2x" src="https://github.com/user-attachments/assets/ac631b33-c97c-4cbc-9f7a-fb21199b7890" />

## Prerequisites

- Project **MUST** use Android Jetpack Compose.
- Project **MUST** target SDK 35 or later.

## Implementation Steps

### 1. Enable Edge-to-Edge in Activity
Call `enableEdgeToEdge()` in your Activity's `onCreate` before `super.onCreate` (or immediately before `setContent`).

```kotlin
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        val splashScreen = installSplashScreen()
        enableEdgeToEdge()
        super.onCreate(savedInstanceState)
        setContent {
            MyTheme {
                // Content
            }
        }
    }
}
```

> [!IMPORTANT]
> **Android 15+ (SDK 35) Deprecations**:
> - Do **not** use `WindowCompat.setDecorFitsSystemWindows(window, false)` — use `enableEdgeToEdge()` instead.
> - Do **not** set `android:statusBarColor` or `android:navigationBarColor` in `themes.xml` (deprecated in Android 15). Keep them managed automatically by `enableEdgeToEdge()`.
> - Do **not** restrict `android:screenOrientation="portrait"` on activities unless strictly required (Android 16 ignores orientation restrictions on large screen / foldable devices).

### 2. Update Manifest
Add `android:windowSoftInputMode="adjustResize"` to your Activity in `AndroidManifest.xml` to handle the IME correctly.

```xml
<activity
    android:name=".MainActivity"
    android:windowSoftInputMode="adjustResize">
</activity>
```

### 3. Handle System Insets

#### Using Scaffold (Preferred)
Pass `PaddingValues` to the content lambda.

```kotlin
Scaffold { innerPadding ->
    Box(modifier = Modifier.padding(innerPadding)) {
        // Content
    }
}
```

#### Manual Inset Padding
For components outside a Scaffold, use window inset modifiers.

```kotlin
Box(
    modifier = Modifier
        .fillMaxSize()
        .safeDrawingPadding() // Applies padding for status and navigation bars
) {
    // Content
}
```

### 4. Handling Lists
Apply insets to `contentPadding` of scrollable components instead of `Modifier.padding()`. This allows content to scroll behind system bars while keeping the first/last items accessible.

```kotlin
LazyColumn(
    contentPadding = WindowInsets.systemBars.asPaddingValues()
) {
    // Items
}
```

### 5. IME (Keyboard) Support
Ensure the IME doesn't obscure input fields. Use `imePadding()` or `WindowInsets.ime`.

```kotlin
Column(
    modifier = Modifier
        .fillMaxSize()
        .imePadding() // Adjusts padding when keyboard appears
        .verticalScroll(rememberScrollState())
) {
    TextField(value = text, onValueChange = { text = it })
}
```

### 6. System Bar Contrast
When using `ComponentActivity.enableEdgeToEdge()`, system bar icon colors are handled automatically based on the theme. 

If using a bottom bar (e.g., `NavigationBar`), disable contrast enforcement to allow colors to extend fully:

```kotlin
window.isNavigationBarContrastEnforced = false
```

## Dialogs
To make a full-screen Dialog edge-to-edge, set `decorFitsSystemWindows = false` in `DialogProperties`.

```kotlin
Dialog(
    onDismissRequest = { /* ... */ },
    properties = DialogProperties(
        usePlatformDefaultWidth = false,
        decorFitsSystemWindows = false
    )
) {
    // Full screen edge-to-edge content
}
```

## Checklist
- [ ] `enableEdgeToEdge()` called in Activity before `super.onCreate` / `setContent`.
- [ ] No deprecated `WindowCompat.setDecorFitsSystemWindows()` used.
- [ ] No `android:statusBarColor` or `android:navigationBarColor` defined in `themes.xml`.
- [ ] `adjustResize` set in Manifest for IME resizing.
- [ ] Lists use `contentPadding` for insets.
- [ ] TextFields handle `imePadding`.
- [ ] Bottom bars have `isNavigationBarContrastEnforced = false` if applicable.
- [ ] No unnecessary `android:screenOrientation="portrait"` locks for Android 16 large-screen support.
