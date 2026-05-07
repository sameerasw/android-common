# Progressive Blur Implementation

This guide covers the implementation of a high-performance progressive blur effect using AGSL (Android Graphics Shading Language), including device-specific optimizations and power-saving considerations.

## 1. AGSL Shader (Progressive Blur)

The progressive blur uses a custom jittered-sampling algorithm to create a smooth transition from clear to blurred.

```kotlin
@Language("AGSL")
private val PROGRESSIVE_BLUR_SKSL = """
    uniform shader content;
    uniform float blurRadius;
    uniform float height;
    uniform float contentHeight;
    uniform int isTop;

    half4 main(float2 fragCoord) {
        float progress;
        if (isTop == 1) {
            progress = 1.0 - clamp(fragCoord.y / height, 0.0, 1.0);
        } else {
            progress = 1.0 - clamp((contentHeight - fragCoord.y) / height, 0.0, 1.0);
        }
        
        // Easing curve for smoother transition (power curve)
        progress = pow(progress, 1.5);
        
        float radius = progress * blurRadius;
        
        if (radius <= 0.0) {
            return content.eval(fragCoord);
        }

        half4 accum = half4(0.0);
        float weightSum = 0.0;
        
        // Random value for dithering based on pixel coordinates
        float dither = fract(sin(dot(fragCoord, float2(12.9898, 78.233))) * 43758.5453);
        float2 jitter = float2(dither - 0.5, fract(dither * 1.618) - 0.5);
        
        const int SAMPLES = 4; 
        float offsetScale = radius / float(SAMPLES);
        
        for (int x = -SAMPLES; x <= SAMPLES; x++) {
            for (int y = -SAMPLES; y <= SAMPLES; y++) {
                // Apply jittered sampling with dither
                float2 offset = (float2(float(x), float(y)) + jitter) * offsetScale;
                
                float distSq = dot(offset, offset);
                float radiusSq = radius * radius;
                
                if (distSq <= radiusSq) {
                    float weight = exp(-3.0 * distSq / radiusSq);
                    accum += content.eval(fragCoord + offset) * weight;
                    weightSum += weight;
                }
            }
        }
        
        return accum / weightSum;
    }
""".trimIndent()
```

## 2. Modifier Implementation

The modifier applies the shader using `RenderEffect` on Android 13+. It also includes an optional gradient overlay to enhance legibility.

```kotlin
enum class BlurDirection { TOP, BOTTOM }

fun Modifier.progressiveBlur(
    blurRadius: Float,
    height: Float,
    direction: BlurDirection = BlurDirection.TOP,
    showGradientOverlay: Boolean = true
): Modifier = composed {
    val overlayColor = MaterialTheme.colorScheme.surfaceContainer.copy(alpha = 0.65f)
    
    val blurModifier = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU && blurRadius > 0f) {
        Modifier.graphicsLayer {
            val shader = RuntimeShader(PROGRESSIVE_BLUR_SKSL)
            shader.setFloatUniform("blurRadius", blurRadius)
            shader.setFloatUniform("height", height)
            shader.setFloatUniform("contentHeight", size.height)
            shader.setIntUniform("isTop", if (direction == BlurDirection.TOP) 1 else 0)
            
            renderEffect = RenderEffect.createRuntimeShaderEffect(shader, "content")
                .asComposeRenderEffect()
        }
    } else Modifier

    val gradientModifier = if (showGradientOverlay) {
        Modifier.drawWithContent {
            drawContent()
            val brush = when (direction) {
                BlurDirection.TOP -> Brush.verticalGradient(
                    colors = listOf(overlayColor, Color.Transparent),
                    endY = height
                )
                BlurDirection.BOTTOM -> Brush.verticalGradient(
                    colors = listOf(Color.Transparent, overlayColor),
                    startY = size.height - height
                )
            }
            drawRect(brush = brush)
        }
    } else Modifier

    this.then(blurModifier).then(gradientModifier)
}
```

## 3. Device & Battery Optimizations

### Samsung Exception
Some Samsung devices on Android 15 (One UI 7) or below have a broken native blur implementation that causes a gray screen overlay. Blur should be disabled for these devices.

```kotlin
fun isBlurProblematicDevice(): Boolean {
    return Build.MANUFACTURER.equals("samsung", ignoreCase = true) && 
           Build.VERSION.SDK_INT <= 35 // Android 15 or below
}
```

### Power Saving Mode
To conserve resources, blur should be disabled when Power Saving Mode is active.

```kotlin
fun isPowerSaveMode(context: Context): Boolean {
    val powerManager = context.getSystemService(Context.POWER_SERVICE) as? PowerManager
    return powerManager?.isPowerSaveMode == true
}
```

## 4. Usage in ViewModel

It is recommended to centralize the blur availability logic in your ViewModel.

```kotlin
class MainViewModel(context: Context) : ViewModel() {
    val isBlurEnabled = mutableStateOf(false)
    val isBlurSettingEnabled = mutableStateOf(true) // User preference

    fun updateBlurState(context: Context) {
        val isProblematic = DeviceUtils.isBlurProblematicDevice()
        val isPowerSave = DeviceUtils.isPowerSaveMode(context)
        
        isBlurEnabled.value = isBlurSettingEnabled.value && !isProblematic && !isPowerSave
    }
}
```

## 5. Usage in Composable

```kotlin
val isBlurEnabled by viewModel.isBlurEnabled
val statusBarHeightPx = with(LocalDensity.current) { WindowInsets.statusBars.getTop(this).toFloat() }

Box(
    modifier = Modifier
        .fillMaxSize()
        .progressiveBlur(
            blurRadius = if (isBlurEnabled) 40f else 0f,
            height = statusBarHeightPx * 1.15f,
            direction = BlurDirection.TOP
        )
) {
    // Content
}
```
