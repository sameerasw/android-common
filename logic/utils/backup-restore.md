# Backup & Restore Pattern

A robust system for exporting and importing the entire application state by serializing multiple `SharedPreferences` files into a single, type-safe JSON container.

## 1. Multi-File Export Strategy

Instead of backing up a single preference file, the app aggregates all functional preference domains (e.g., `essentials_prefs`, `caffeinate_prefs`, `live_wallpaper_prefs`) into a unified JSON structure.

### JSON Wrapper Format
Each entry in the JSON includes the data type to ensure that when the data is restored, it is written back using the correct `putInt`, `putBoolean`, etc., methods.

```json
{
  "essentials_prefs": {
    "haptic_feedback_enabled": {
      "type": "Boolean",
      "value": true
    },
    "flashlight_intensity": {
      "type": "Int",
      "value": 5
    }
  }
}
```

## 2. Data Sanitization

To keep backups clean and stable, transient or volatile data is filtered out during the export process.

### Filtered Data Types
- **Discovered Channels**: System-provided notification channel IDs (e.g., `snooze_discovered_channels`).
- **Device States**: Battery levels or connection statuses (e.g., `mac_battery_level`).
- **Temporary Selections**: Cached app selections that might change across device restores.

## 3. Type-Safe Restore Logic

When importing the JSON, the app must handle JSON's default behavior of parsing numbers as `Double`.

### Casting Strategy
The restore logic identifies the "type" field and performs the necessary conversion before writing to disk:

```kotlin
when (itemType) {
    "Int" -> putInt(key, (itemValue as Double).toInt())
    "Long" -> putLong(key, (itemValue as Double).toLong())
    "Float" -> putFloat(key, (itemValue as Double).toFloat())
    "StringSet" -> putStringSet(key, (itemValue as List<String>).toSet())
}
```

### Atomic Preference Reset
Before applying the imported settings, the app calls `clear()` on each preference file to ensure no "ghost" settings from the current session leak into the restored state.

## 4. Internal Data Migrations

If a feature's data structure has changed between app versions (e.g., moving from individual keys to a combined JSON list), repositories use a `migrateIfNeeded()` pattern during initialization.

```kotlin
private fun migrateIfNeeded() {
    if (prefs.contains("legacy_lat") && !prefs.contains("new_json_format")) {
        val legacyData = extractLegacyData()
        val converted = convertToNewFormat(legacyData)
        saveInNewFormat(converted)
        clearLegacyKeys()
    }
}
```

## 5. Implementation Tips

- **IO Streams**: Always use `OutputStream` and `InputStream` (e.g., via `ActivityForResult` with `CREATE_DOCUMENT`) to let the user choose the backup location (local storage, cloud, etc.).
- **GSON Configuration**: Use a consistent `Gson` instance with `@Keep` annotations on domain models to prevent ProGuard/R8 from breaking JSON serialization.
- **Immediate Sync**: After a restore, call `syncSystemSettingsWithSaved()` to immediately apply system-level changes (like animation scales or font weights) without requiring an app restart.
