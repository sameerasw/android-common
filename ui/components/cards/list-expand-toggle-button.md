# Expandable Section Toggle Button Pattern (`ListExpandToggleButton`)

An interactive expandable toggle button built with **Material 3 Expressive** design. It serves as an expand/collapse toggle for lists, sections, or QS tile setting option panels while supporting **Translation Mode** long-press interactions to translate both expanded and collapsed state labels.

## 1. Overview

`ListExpandToggleButton` provides:
- **Animated Chevron**: Smoothly rotates 180° when toggled between collapsed and expanded states.
- **Dynamic Text Support**: Displays different strings/string resources based on `isExpanded` state.
- **Translation Mode Long-Press Support**: When global Translation Mode is enabled, long-pressing anywhere on the full-width row container triggers a `SegmentedDropdownMenu` with options to view string translation keys.
- **Haptic Feedback**: Standard virtual key haptics on button click.

---

## 2. Implementation Code (`ListExpandToggleButton.kt`)

```kotlin
package com.sameerasw.essentials.ui.components.buttons

import androidx.compose.animation.core.animateFloatAsState
import androidx.compose.foundation.ExperimentalFoundationApi
import androidx.compose.foundation.combinedClickable
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.PaddingValues
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.layout.width
import androidx.compose.material3.Button
import androidx.compose.material3.ButtonDefaults
import androidx.compose.material3.Icon
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.graphicsLayer
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.platform.LocalView
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import com.sameerasw.essentials.R
import com.sameerasw.essentials.translation.TranslationManager
import com.sameerasw.essentials.translation.ui.TranslationBottomSheet
import com.sameerasw.essentials.translation.ui.TranslationMenuItems
import com.sameerasw.essentials.ui.components.menus.SegmentedDropdownMenu
import com.sameerasw.essentials.utils.HapticUtil

@OptIn(ExperimentalFoundationApi::class)
@Composable
fun ListExpandToggleButton(
    isExpanded: Boolean,
    onToggle: () -> Unit,
    modifier: Modifier = Modifier,
    title: Any = R.string.action_show_top_apps,
    description: Any? = R.string.action_show_all,
    expandedText: String? = null,
    collapsedText: String? = null
) {
    val context = LocalContext.current
    val view = LocalView.current
    val isTranslationModeActive by TranslationManager.isTranslationModeEnabled

    var showMenu by remember { mutableStateOf(false) }
    var translationSheetKey by remember { mutableStateOf<String?>(null) }

    val rotationDegree by animateFloatAsState(
        targetValue = if (isExpanded) 180f else 0f,
        label = "list_expand_chevron_rotation"
    )

    val expText = expandedText ?: when (title) {
        is Int -> stringResource(title)
        is String -> title
        else -> title.toString()
    }
    val colText = collapsedText ?: when (description) {
        is Int -> stringResource(description)
        is String -> description
        null -> expText
        else -> description.toString()
    }

    Box(
        modifier = modifier
            .fillMaxWidth()
            .combinedClickable(
                onClick = {},
                onLongClick = if (isTranslationModeActive) {
                    {
                        HapticUtil.performVirtualKeyHaptic(view)
                        showMenu = true
                    }
                } else null
            )
    ) {
        Button(
            onClick = {
                HapticUtil.performVirtualKeyHaptic(view)
                onToggle()
            },
            modifier = Modifier.padding(start = 4.dp, top = 4.dp),
            colors = ButtonDefaults.buttonColors(
                containerColor = MaterialTheme.colorScheme.surfaceBright,
                contentColor = MaterialTheme.colorScheme.primary
            ),
            contentPadding = PaddingValues(horizontal = 20.dp, vertical = 14.dp)
        ) {
            Icon(
                painter = painterResource(id = R.drawable.rounded_keyboard_arrow_down_24),
                contentDescription = null,
                tint = MaterialTheme.colorScheme.primary,
                modifier = Modifier
                    .size(22.dp)
                    .graphicsLayer { rotationZ = rotationDegree }
            )
            Spacer(modifier = Modifier.width(10.dp))
            Text(
                text = if (isExpanded) expText else colText,
                style = MaterialTheme.typography.bodyMedium,
                fontWeight = FontWeight.SemiBold
            )
        }

        SegmentedDropdownMenu(
            expanded = showMenu,
            onDismissRequest = { showMenu = false }
        ) {
            TranslationMenuItems(
                title = title,
                description = description,
                onSelectKey = { key ->
                    showMenu = false
                    translationSheetKey = key
                }
            )
        }
    }

    val targetKey = translationSheetKey
    if (targetKey != null) {
        val resolvedKey = remember(targetKey) {
            TranslationManager.resolveKey(context, targetKey) ?: targetKey
        }
        TranslationBottomSheet(
            stringKey = resolvedKey,
            onDismissRequest = { translationSheetKey = null }
        )
    }
}
```

---

## 3. Key Design Choices & Architecture

### Full-Width Box Container (`fillMaxWidth()`)
Wrapping the `Button` in a `Box(modifier = modifier.fillMaxWidth().combinedClickable(...))` ensures:
1. **Unmodified Button Touch Target**: Direct taps on the `Button` trigger `onToggle()` with native `Button` styling and feedback.
2. **Translation Mode Receiver**: Long-pressing anywhere along the row area outside or around the button captures `onLongClick` when Translation Mode is enabled without interfering with standard button click listeners.

### Flexibly Typed Resource Titles (`title: Any`)
Accepts either `Int` (string resource ID e.g., `R.string.action_show_all`) or `String` values. Passing raw `Int` resource IDs enables `TranslationManager.resolveKey` to automatically resolve string key names for translation bottom sheets.

---

## 4. Usage Snippets

### A. List Expansion (e.g., App List)
```kotlin
var showAllApps by remember { mutableStateOf(false) }

ListExpandToggleButton(
    isExpanded = showAllApps,
    onToggle = { showAllApps = !showAllApps },
    title = R.string.action_show_top_apps,
    description = R.string.action_show_all
)
```

### B. Toggleable Subsystem Lists (e.g., Wakeup Logs)
```kotlin
var showWakeups by remember { mutableStateOf(false) }

ListExpandToggleButton(
    isExpanded = showWakeups,
    onToggle = { showWakeups = !showWakeups },
    title = R.string.action_hide_wakeups,
    description = R.string.action_show_wakeups
)
```

### C. Expandable Sub-Settings Options Section
```kotlin
var showOptions by remember { mutableStateOf(false) }

ListExpandToggleButton(
    isExpanded = showOptions,
    onToggle = { showOptions = !showOptions },
    title = R.string.action_charging_qs_tile_options,
    description = null
)
```
