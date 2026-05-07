# Quick Settings Tiles Pattern

A standardized framework for implementing custom Quick Settings (QS) tiles with built-in haptic feedback, permission handling, and root/shell fallback.

## 1. The Base Interface (`BaseTileService`)

All QS tiles in the application inherit from `BaseTileService`, which abstracts away the complexity of tile lifecycle, IPC caching, and secure settings management.

### Key Features
- **Immediate Feedback**: Triggers haptics via `HapticUtil` and updates the tile state to "Working..." immediately on click.
- **Permission Guard**: Automatically checks `hasFeaturePermission()` and disables the tile if required permissions (like `WRITE_SECURE_SETTINGS`) are missing.
- **Settings Fallback**: Provides protected methods to read/write `Secure` and `Global` settings with automatic fallback to **Shell/Root** commands if the standard Android API fails.
- **Background Scopes**: Integrated `CoroutineScope` for offloading work from the UI thread.

### Abstract Methods
Every tile must implement:
- `onTileClick()`: The actual logic performed when the tile is toggled.
- `getTileLabel()`: The primary text on the tile.
- `getTileSubtitle()`: Secondary text (e.g., "On", "Off", or "Permission missing").
- `getTileState()`: Returns `STATE_ACTIVE` or `STATE_INACTIVE`.
- `hasFeaturePermission()`: Logic to check if the tile can be used.

## 2. Core Implementation Logic

```kotlin
abstract class BaseTileService : TileService() {
    override fun onClick() {
        // 1. Immediate Haptic Feedback
        HapticUtil.performHapticForService(this)
        
        if (!hasFeaturePermission() || isProcessing) return

        // 2. Visual "Processing" State
        isProcessing = true
        updateTile()

        // 3. Offload Work
        serviceScope.launch {
            onTileClick()
            withContext(Dispatchers.Main) {
                isProcessing = false
                updateTile()
            }
        }
    }
}
```

### Settings Fallback Example
This pattern ensures tiles work even on devices where the standard API is restricted.

```kotlin
protected fun putSecureInt(key: String, value: Int) {
    try {
        Settings.Secure.putInt(contentResolver, key, value)
    } catch (e: Exception) {
        // Fallback to Shell if the app doesn't have WRITE_SECURE_SETTINGS
        ShellUtils.runCommand(this, "settings put secure $key $value")
    }
}
```

## 3. Concrete Example: Mono Audio

```kotlin
class MonoAudioTileService : BaseTileService() {
    override fun getTileLabel() = "Mono Audio"
    override fun getTileSubtitle() = if (isEnabled()) "On" else "Off"
    
    override fun getTileState() = if (isEnabled()) Tile.STATE_ACTIVE else Tile.STATE_INACTIVE

    override fun onTileClick() {
        val newState = if (isEnabled()) 0 else 1
        putSecureInt("master_mono", newState)
    }

    override fun hasFeaturePermission() = PermissionUtils.canWriteSecureSettings(this) || ShellUtils.isAvailable(this)
}
```

## 4. Manifest Configuration

Each tile service must be declared in the `AndroidManifest.xml` with the appropriate permission and intent-filter.

```xml
<service
    android:name=".services.tiles.MonoAudioTileService"
    android:icon="@drawable/ic_mono"
    android:label="Mono Audio"
    android:permission="android.permission.BIND_QUICK_SETTINGS_TILE">
    <intent-filter>
        <action android:name="android.service.quicksettings.action.QS_TILE" />
    </intent-filter>
</service>
```

## 5. UI Integration
Per user-defined rules, all added QS tiles should also be exposed in the "Quick Settings Tiles" section of the app's settings UI, allowing users to discover and learn how to add them to their system panel.
