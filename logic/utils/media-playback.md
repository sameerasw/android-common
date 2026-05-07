# Media Playback Extraction Pattern

A robust system for listening to system-wide media playback, extracting metadata, and performing deep interactions (like "Liking" songs) across different media players (Spotify, YouTube Music, etc.).

## 1. Accessing Media Sessions

The primary entry point is the `NotificationListenerService`. When a media notification is posted, we extract the `MediaSession.Token`.

```kotlin
val token = extras.getParcelable(Notification.EXTRA_MEDIA_SESSION) as? MediaSession.Token
if (token != null) {
    val controller = MediaController(context, token)
    // Now we have access to metadata and playback state
}
```

## 2. Metadata Extraction

### Basic Info
Retrieve song titles, artists, and playback status directly from the controller.

```kotlin
val metadata = controller.metadata
val title = metadata?.getString(MediaMetadata.METADATA_KEY_TITLE)
val artist = metadata?.getString(MediaMetadata.METADATA_KEY_ARTIST)
val isPlaying = controller.playbackState?.state == PlaybackState.STATE_PLAYING
```

### Album Art Dictionary
Extracting high-resolution Bitmaps from system services is expensive. We use a caching strategy to avoid redundant processing.

1.  **Extract**: Get the Bitmap from `METADATA_KEY_ALBUM_ART` or `METADATA_KEY_ART`.
2.  **Hash**: Create a unique key using `Title + Artist`.
3.  **Cache**: Save the Bitmap to `cacheDir` as a PNG.
4.  **Cleanup**: Maintain a rolling cache (e.g., keep only the last 3-5 unique tracks).

## 3. Deep Interaction: The "Like" Heuristics

Different apps implement the "Like" button differently. To support "Like" globally (e.g., via a [Quick Settings Tile](file:///Users/sameerasandakelum/GIT/jetpack-common/logic/services/qs-tiles.md) or [Ambient Glance](file:///Users/sameerasandakelum/GIT/jetpack-common/accessibility/accessibility-overlays.md)), we use three layers of detection:

### Layer 1: Native Rating API
Checking if the app supports the standard `METADATA_KEY_USER_RATING`.
```kotlin
val rating = metadata.getRating(MediaMetadata.METADATA_KEY_USER_RATING)
val isLiked = rating?.hasHeart() == true || rating?.isThumbUp == true
```

### Layer 2: Custom Playback Actions
Many apps (like Spotify) use `CustomAction` objects within the `PlaybackState`. We scan these for keywords.
- **Keywords**: "Like", "Heart", "Favorite", "Love", "ThumbsUp".
- **Detecting "Already Liked"**: Look for "Unlike", "Unheart", or "Remove from collection".

### Layer 3: Notification Action Fallback
As a last resort, we scan the physical buttons in the notification shade.
```kotlin
val actions = sbn.notification.actions
val likeAction = actions.find { it.title.contains("Like", ignoreCase = true) }
likeAction?.actionIntent?.send()
```

## 4. Triggering Actions

Once the correct "Like" action is identified, we execute it using the appropriate channel:
- **Priority**: `transportControls.sendCustomAction` (Fastest/Cleanest).
- **Fallback**: `PendingIntent.send()` (Reliable for older apps).

## 5. Implementation Tips
- **Android Auto Protection**: Always check `AppUtil.isAndroidAutoRunning()` before triggering media-related overlays (like Ambient Glance) to avoid distracting the driver.
- **Bypass Interactive Check**: For features like "Ambient Music Glance", we often need to extract media info even when the screen is off and the device is non-interactive.
- **Interactive Detection**: Use `PlaybackState.STATE_PLAYING` to prioritize the active player if multiple media apps are open.
