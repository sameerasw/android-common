# Deep Search Pattern

A robust search implementation that indexes the entire application's features and sub-settings, allowing users to find specific options that are nested deep within menus.

## 1. Data Model

The `SearchableItem` represents a single result. It contains the metadata needed to display the result and the navigation keys needed to deep-link to it.

```kotlin
data class SearchableItem(
    val title: String,
    val description: String,
    val category: String,
    val icon: Int?,
    val featureKey: String, // The main feature/activity ID
    val parentFeature: String? = null,
    val targetSettingHighlightKey: String? = null, // The specific setting to scroll to
    val keywords: List<String> = emptyList(),
    val isBeta: Boolean = false
)
```

## 2. The Search Registry

Instead of crawling the UI, the search system uses a registry-based approach. It iterates through a global list of features and their defined sub-settings.

### Indexing Logic

```kotlin
object SearchRegistry {
    fun search(context: Context, query: String): List<SearchableItem> {
        val q = query.trim().lowercase()
        if (q.isEmpty()) return emptyList()

        val allItems = mutableListOf<SearchableItem>()

        FeatureRegistry.ALL_FEATURES.forEach { feature ->
            // 1. Index the feature itself
            allItems.add(SearchableItem(
                title = context.getString(feature.title),
                description = context.getString(feature.description),
                featureKey = feature.id,
                // ...
            ))

            // 2. Index sub-settings defined within the feature
            feature.searchableSettings.forEach { setting ->
                allItems.add(SearchableItem(
                    title = context.getString(setting.title),
                    description = context.getString(setting.description),
                    featureKey = feature.id,
                    targetSettingHighlightKey = setting.targetSettingHighlightKey,
                    keywords = setting.getKeywords(context),
                    // ...
                ))
            }
        }

        // 3. Filter and Rank
        return allItems.filter { item ->
            item.title.lowercase().contains(q) ||
            item.description.lowercase().contains(q) ||
            item.keywords.any { it.lowercase().contains(q) }
        }.sortedByDescending { it.title.lowercase().startsWith(q) }
    }
}
```

## 3. ViewModel Integration

The ViewModel maintains the search state and triggers the search logic.

```kotlin
class MainViewModel : ViewModel() {
    val searchQuery = mutableStateOf("")
    val searchResults = mutableStateOf<List<SearchableItem>>(emptyList())
    val isSearching = mutableStateOf(false)

    fun onSearchQueryChanged(query: String, context: Context) {
        searchQuery.value = query
        if (query.isBlank()) {
            searchResults.value = emptyList()
            return
        }

        // Trigger search (can be debounced for performance)
        isSearching.value = true
        searchResults.value = SearchRegistry.search(context, query)
        isSearching.value = false
    }
}
```

## 4. UI & Deep Linking

When a search result is clicked, the app navigates to the target feature activity and passes the `highlight_setting` key.

### Navigation Logic
```kotlin
onClick = { result ->
    val intent = Intent(context, FeatureSettingsActivity::class.java).apply {
        putExtra("feature_id", result.featureKey)
        putExtra("highlight_setting", result.targetSettingHighlightKey)
    }
    context.startActivity(intent)
}
```

### Highlighting in Destination
Inside the destination Activity/Composable:
1.  Read the `highlight_setting` extra.
2.  Find the corresponding list item.
3.  Scroll to it and apply a temporary highlight (e.g., a brief background pulse).

## 5. Benefits

- **Performance**: Instant filtering of hundreds of items since they are already in memory.
- **Discoverability**: Users can find nested settings like "Edge Lighting Colors" directly from the main screen.
- **Keywords**: Supports synonyms (e.g., searching "hide" finds "Status Bar Icon Visibility").
