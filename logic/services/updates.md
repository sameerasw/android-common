# In-App Update System

A robust, GitHub-based self-update mechanism that supports semantic versioning, pre-release tracking, and rich release notes with Markdown.

## 1. Update Repository (GitHub Integration)

The `UpdateRepository` interacts with the GitHub Releases API to fetch the latest version info.

### API Endpoints
- **Stable**: `https://api.github.com/repos/{owner}/{repo}/releases/latest`
- **Pre-release**: `https://api.github.com/repos/{owner}/{repo}/releases` (fetches a list and finds the highest semantic version).

### Logic Overview
1.  Fetch JSON from GitHub.
2.  Parse `tag_name` and `body` (release notes).
3.  Identify the APK asset (e.g., `app-release.apk`) to extract the `browser_download_url`.
4.  Compare with the current app version using `SemanticVersion`.

## 2. Semantic Versioning

To ensure accurate update detection (e.g., `1.0.0-beta.1` < `1.0.0`), we use a custom `SemanticVersion` parser.

```kotlin
private data class SemanticVersion(
    val major: Int,
    val minor: Int,
    val patch: Int,
    val preRelease: String? = null
) : Comparable<SemanticVersion> {
    override fun compareTo(other: SemanticVersion): Int {
        if (major != other.major) return major - other.major
        if (minor != other.minor) return minor - other.minor
        if (patch != other.patch) return patch - other.patch
        
        // Pre-release logic: 1.0.0-beta < 1.0.0
        if (preRelease == null && other.preRelease != null) return 1
        if (preRelease != null && other.preRelease == null) return -1
        return preRelease?.compareTo(other.preRelease ?: "") ?: 0
    }
}
```

## 3. Update Bottom Sheet

When an update is found, it is presented in a premium `ModalBottomSheet`.

### Key Features
- **Markdown Notes**: Uses a `SimpleMarkdown` component to render the GitHub release body.
- **Beta Warning**: Displays a prominent warning if the detected update is a pre-release (beta/alpha/rc).
- **Direct Action**: Primary "Download APK" button and an optional "View on GitHub" outlined button.

```kotlin
@Composable
fun UpdateBottomSheet(updateInfo: UpdateInfo) {
    ModalBottomSheet(...) {
        Column(...) {
            // Version Header
            Text(text = "Update Available", style = MaterialTheme.typography.headlineSmall)
            
            // Release Notes Card
            RoundedCardContainer {
                SimpleMarkdown(content = updateInfo.releaseNotes)
            }
            
            // Actions
            Button(onClick = { openUrl(updateInfo.downloadUrl) }) {
                Text("Download APK")
            }
        }
    }
}
```

## 4. Background Checks & Debouncing

To avoid hitting GitHub API rate limits and redundant checks, the `MainViewModel` implements basic debouncing.

```kotlin
fun checkForUpdates(manual: Boolean = false) {
    if (!manual) {
        val currentTime = System.currentTimeMillis()
        // Limit background checks to once every 15 minutes
        if (currentTime - lastUpdateCheckTime < 900000) return
    }
    
    viewModelScope.launch {
        val result = repository.checkForUpdates(...)
        if (result?.isUpdateAvailable == true) {
            updateInfo.value = result
            // Show notification or sheet
        }
    }
}
```

## 5. Summary of Benefits
- **No Play Store Dependency**: Perfect for apps distributed via GitHub or external sites.
- **Rich Content**: Users see exactly what changed via Markdown release notes.
- **Flexible**: Supports both stable and pre-release channels via a simple toggle in the app settings.
