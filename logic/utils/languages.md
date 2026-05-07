# In-App Language Switching

A standardized approach for implementing per-app language selection that integrates with Android 13+ system settings and provides a custom in-app picker.

## 1. Language Data Model

We maintain a list of supported languages with their ISO codes, English names, and native names for better UX.

```kotlin
data class Language(
    val code: String, 
    val name: String, 
    val nativeName: String
)

object LanguageUtils {
    val languages = listOf(
        Language("en", "English", "English"),
        Language("si", "Sinhala", "සිංහල"),
        Language("es", "Spanish", "Español"),
        // ...
    )
}
```

## 2. Switching Logic

The application uses `AppCompatDelegate` to manage locales. This ensures compatibility with Android 13's per-app language features while providing a consistent experience on older versions.

```kotlin
fun setAppLanguage(languageCode: String) {
    // 1. Update local state/persistence if needed
    appLanguage.value = languageCode
    
    // 2. Apply locale via AppCompatDelegate
    val appLocale: LocaleListCompat = LocaleListCompat.forLanguageTags(languageCode)
    AppCompatDelegate.setApplicationLocales(appLocale)
}
```

### Benefits of `AppCompatDelegate`:
- **System Sync**: Changes made in the app are reflected in System Settings > Languages > App Languages.
- **Persistence**: Android handles the persistence of the selected locale automatically when using this API.
- **No Manual Restart**: The Activity is recreated automatically to apply the new resources.

## 3. Language Picker UI

A custom picker component using Material 3 `ExposedDropdownMenuBox`.

### Key Design Choices
- **Native Names**: Always display the native name (e.g., "සිංහල") so users can identify their language even if they don't understand the current app language.
- **Haptic Feedback**: Triggers a virtual key haptic on selection.
- **ListItem Integration**: Designed to fit perfectly inside a [Rounded Card Container](file:///Users/sameerasandakelum/GIT/jetpack-common/ui/components/containers/rounded-card-container.md).

```kotlin
@Composable
fun LanguagePicker(
    selectedLanguageCode: String,
    onLanguageSelected: (String) -> Unit
) {
    val selectedLanguage = languages.find { it.code == selectedLanguageCode }
    
    ListItem(
        leadingContent = { Icon(painterResource(R.drawable.ic_globe), ...) },
        trailingContent = {
            ExposedDropdownMenuBox(...) {
                OutlinedTextField(
                    value = "${selectedLanguage.nativeName} (${selectedLanguage.name})",
                    readOnly = true,
                    // ...
                )
                ExposedDropdownMenu(...) {
                    languages.forEach { lang ->
                        DropdownMenuItem(
                            text = { Text("${lang.nativeName} (${lang.name})") },
                            onClick = { onLanguageSelected(lang.code) }
                        )
                    }
                }
            }
        },
        content = { Text("Language") }
    )
}
```

## 4. Implementation Tips
- **Pre-selection**: On the first launch (Onboarding), show the `LanguagePicker` as the very first step to ensure the user can navigate the setup process in their preferred language.
- **Locale Sync**: In the `MainViewModel` init, check `AppCompatDelegate.getApplicationLocales()` to sync the app's internal state with the system's per-app language setting.
