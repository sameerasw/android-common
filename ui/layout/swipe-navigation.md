# Swipe Navigation Pattern

A high-level navigation strategy using `HorizontalPager` to allow users to switch between main app sections with fluid, natural gestures.

## 1. Implementation (`HorizontalPager`)

The core of swipe navigation is the `HorizontalPager` from the Foundation library. It manages the page state and handles the touch gestures.

```kotlin
val pagerState = rememberPagerState(
    initialPage = initialPage,
    pageCount = { tabs.size }
)

HorizontalPager(
    state = pagerState,
    modifier = Modifier.fillMaxSize()
) { page ->
    when (page) {
        0 -> ConnectScreen()
        1 -> RemoteScreen()
        // ...
    }
}
```

## 2. Pager-Toolbar Synchronization

To keep the UI in sync, the `pagerState.currentPage` is used to drive the active state of the [Floating Toolbar](../components/toolbar/floating-toolbar.md).

### Bidirectional Sync
- **Swipe -> Toolbar**: The toolbar observes `pagerState.currentPage` to highlight the correct tab.
- **Click -> Pager**: Tapping a toolbar icon calls `pagerState.animateScrollToPage(index)`.

```kotlin
// Syncing Haptics on Page Change
LaunchedEffect(pagerState.currentPage) {
    snapshotFlow { pagerState.currentPage }.collect { _ ->
        HapticUtil.performLightTick(haptics)
    }
}
```

## 3. Swipe-Aware Effects

In the `airsync-android` project, we utilize several visual effects to make the swipe feel "integrated":

### Progressive Blur
As the user swipes, the top and bottom of the screen can have a [Progressive Blur](progressive-blur.md) that stays fixed while the content moves underneath.

```kotlin
HorizontalPager(
    modifier = Modifier
        .fillMaxSize()
        .progressiveBlur(direction = BlurDirection.TOP)
        .progressiveBlur(direction = BlurDirection.BOTTOM),
    state = pagerState
)
```

### Scroll-Aware Component Visibility
If a screen has its own internal scroll state, we can hide the FAB or other UI elements when scrolling down, but ensure they are visible when switching pages.

## 4. Initial Tab Logic
Using `LaunchedEffect` and `withTimeoutOrNull`, the app can decide which tab to show first based on connection state or specific launch intents (Shortcuts).

```kotlin
val targetPage = when (uiState.defaultTab) {
    "remote" -> 1
    "clipboard" -> 2
    else -> 0
}
pagerState.scrollToPage(targetPage)
```

## 5. Implementation Tips
- **Haptic Feedback**: Always add a light tick (`HapticUtil.performLightTick`) when the `currentPage` changes to give tactile confirmation of the swipe.
- **Preloading**: `HorizontalPager` preloads adjacent pages by default; ensure your screens are lightweight or use `LaunchedEffect` for heavy data loading.
- **Edge-to-Edge**: Ensure the pager fills the entire screen and uses `windowInsetsPadding` only where necessary to maintain a seamless swipe experience.
