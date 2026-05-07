# Splash Screen Implementation

A comprehensive guide to implementing the modern Android Splash Screen API with custom exit animations and branding.

<img width="500" height="1111" alt="screen-20260507-162433-1778151268818" src="https://github.com/user-attachments/assets/862e7cb3-2f53-49c0-a780-eb440ab7ef80" />

## 1. Branding & Configuration

The splash screen is configured using a specialized theme that inherits from `Theme.SplashScreen`.

### XML Theme (`res/values/splash.xml`)

```xml
<resources>
    <style name="Theme.App.Splash" parent="Theme.SplashScreen">
        <!-- Background color of the splash screen -->
        <item name="windowSplashScreenBackground">@color/app_window_background</item>
        
        <!-- The icon to display in the center -->
        <item name="windowSplashScreenAnimatedIcon">@drawable/ic_splash_icon</item>
        
        <!-- Duration the splash screen stays on screen -->
        <item name="windowSplashScreenAnimationDuration">1000</item>
        
        <!-- The theme to switch to after the splash screen is dismissed -->
        <item name="postSplashScreenTheme">@style/Theme.App.Main</item>
    </style>
</resources>
```

### Manifest Integration
Apply the splash theme to your starting Activity.

```xml
<activity
    android:name=".MainActivity"
    android:theme="@style/Theme.App.Splash">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

## 2. Activity Integration

Initialize the splash screen before `super.onCreate`.

```kotlin
class MainActivity : AppCompatActivity() {
    private var isAppReady = false

    override fun onCreate(savedInstanceState: Bundle?) {
        val splashScreen = installSplashScreen()
        super.onCreate(savedInstanceState)
        
        // Keep splash screen visible while app is loading (e.g., database/auth checks)
        splashScreen.setKeepOnScreenCondition { !isAppReady }
        
        setupExitAnimation(splashScreen)
    }
}
```

## 3. Exit Animation

To provide a seamless transition, we use a custom exit animation that scales and fades the splash screen.

```kotlin
private fun setupExitAnimation(splashScreen: SplashScreen) {
    splashScreen.setOnExitAnimationListener { splashScreenViewProvider ->
        val splashScreenView = splashScreenViewProvider.view
        val splashIcon = try { splashScreenViewProvider.iconView } catch (e: Exception) { null }

        // 1. Fade out the entire splash screen view
        val fadeOut = ObjectAnimator.ofFloat(splashScreenView, "alpha", 1f, 0f).apply {
            interpolator = AnticipateInterpolator()
            duration = 750
        }

        fadeOut.doOnEnd {
            splashScreenViewProvider.remove()
            // CRITICAL: Re-apply edge-to-edge logic after removal to ensure 
            // the system windows are handled correctly by the main app theme.
            enableEdgeToEdge()
        }

        // 2. Safely animate the icon (Scale Down)
        if (splashIcon != null) {
            val scaleX = ObjectAnimator.ofFloat(splashIcon, "scaleX", 1f, 0.5f)
            val scaleY = ObjectAnimator.ofFloat(splashIcon, "scaleY", 1f, 0.5f)
            
            listOf(scaleX, scaleY).forEach { 
                it.interpolator = AnticipateInterpolator()
                it.duration = 750
                it.start()
            }
        }
        
        fadeOut.start()
    }
}
```

### OEM Device Considerations
Some OEM implementations (like Samsung One UI or Xiaomi) may not provide an `iconView` in the `splashScreenViewProvider`, which can lead to `NullPointerExceptions`. Always wrap icon animations in a null check and try-catch block.

## 4. Optional Improvements

### Rotation Animation
For extra flair, you can add a rotation to the icon during exit:
```kotlin
val rotate = ObjectAnimator.ofFloat(splashIcon, "rotation", 0f, -90f).apply {
    interpolator = AnticipateInterpolator()
    duration = 750
}
rotate.start()
```

### Branding Consistency
Ensure the `windowSplashScreenBackground` matches the `app_window_background` used in your main theme to prevent "color jumping" when the splash screen transitions to the app content.
