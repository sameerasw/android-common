# Liquid Ripple AGSL Shader Effect

This guide documents the implementation of a hardware-accelerated **liquid displacement ripple effect** using AGSL (`RuntimeShader` and `RenderEffect`) in Jetpack Compose, designed for action confirmations, scanning animations, and modal bottom sheet overlays.

---

## 1. Overview

The liquid ripple effect performs pixel displacement on live UI layers rather than rendering flat color circles. It distorts the background content along the ripple propagation wave vectors with decaying sinusoidal displacement, trailing sub-ripples, and specular highlights.

```
       uOrigin (cx, cy)
            *  ~~~ (wave1: primary ripple)
               ~~~~~ (wave2: trailing sub-ripple)
                 ~~~~~~~ -> propagates across layout
```

---

## 2. AGSL Shader Source (`nfc_ripple.agsl`)

Place this shader in `res/raw/nfc_ripple.agsl` (or as an `@Language("AGSL")` constant):

```glsl
uniform shader inputShader;
uniform float2 uResolution;
uniform float2 uOrigin;
uniform float uTime;
uniform float uAmplitude;
uniform float uFrequency;
uniform float uDecay;
uniform float uSpeed;

half4 main(float2 fragCoord) {
    float2 pos = fragCoord;
    float distance = length(pos - uOrigin);
    float delay = distance / uSpeed;
    float time = max(0.0, uTime - delay);
    
    // Primary ripple wave + secondary trailing sub-ripple
    float wave1 = uAmplitude * sin(uFrequency * time) * exp(-uDecay * time);
    
    float subTime = max(0.0, time - 0.22);
    float wave2 = (uAmplitude * 0.55) * sin(uFrequency * 1.15 * subTime) * exp(-(uDecay * 0.8) * subTime);
    
    float totalWave = wave1 + wave2;
    float2 n = normalize(pos - uOrigin);
    float2 newPos = pos + totalWave * n;
    
    // Dynamic wave specular highlight
    float highlight = 0.16 * (totalWave / max(1.0, uAmplitude));
    
    return inputShader.eval(newPos) + half4(highlight, highlight, highlight, 0.0);
}
```

### Parameter Tuning

| Uniform | Default Recommended | Description |
| :--- | :--- | :--- |
| `uAmplitude` | `32f * density` | Peak displacement offset in pixels |
| `uFrequency` | `12f` | Wave oscillation frequency |
| `uDecay` | `4.5f` | Exponential damping rate (higher = fades faster) |
| `uSpeed` | `1400f * density` | Propagation velocity in pixels/second |
| `uTime` | `0.0f .. 3.2f` | Elapsed animation time in seconds |

---

## 3. Jetpack Compose Integration (Bottom Sheet & Containers)

In Jetpack Compose, modal dialogs and bottom sheets run in separate overlay window surfaces. Wrapping the content container with `graphicsLayer { renderEffect = ... }` guarantees that the full layout inside the modal is distorted smoothly.

```kotlin
@Composable
fun LiquidRippleContainer(
    modifier: Modifier = Modifier,
    rippleTrigger: Int = 0,
    rippleOrigin: Offset = Offset.Zero,
    content: @Composable () -> Unit
) {
    val context = LocalContext.current
    val density = LocalDensity.current

    val animTime = remember { Animatable(0f) }

    LaunchedEffect(rippleTrigger) {
        if (rippleTrigger > 0) {
            animTime.snapTo(0f)
            animTime.animateTo(
                targetValue = 3.2f,
                animationSpec = tween(durationMillis = 3200, easing = LinearEasing)
            )
            animTime.snapTo(0f)
        }
    }

    val shader = remember {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            val code = context.resources.openRawResource(R.raw.nfc_ripple).use {
                BufferedReader(InputStreamReader(it)).readText()
            }
            RuntimeShader(code)
        } else null
    }

    var containerWindowPos by remember { mutableStateOf(Offset.Zero) }

    Box(
        modifier = modifier
            .fillMaxWidth()
            .onGloballyPositioned { coords ->
                containerWindowPos = coords.positionInWindow()
            }
            .graphicsLayer {
                val currentTime = animTime.value
                if (currentTime > 0f && currentTime < 3.2f && shader != null && Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
                    val densityVal = density.density
                    val amplitude = 32f * densityVal
                    val frequency = 12f
                    val decay = 4.5f
                    val speed = 1400f * densityVal

                    val localOriginX = if (rippleOrigin != Offset.Zero) {
                        rippleOrigin.x - containerWindowPos.x
                    } else {
                        size.width / 2f
                    }
                    val localOriginY = if (rippleOrigin != Offset.Zero) {
                        (rippleOrigin.y - containerWindowPos.y).coerceAtLeast(0f)
                    } else {
                        0f // Top edge / drag handle origin
                    }

                    shader.setFloatUniform("uResolution", size.width, size.height)
                    shader.setFloatUniform("uOrigin", localOriginX, localOriginY)
                    shader.setFloatUniform("uTime", currentTime)
                    shader.setFloatUniform("uAmplitude", amplitude)
                    shader.setFloatUniform("uFrequency", frequency)
                    shader.setFloatUniform("uDecay", decay)
                    shader.setFloatUniform("uSpeed", speed)

                    renderEffect = RenderEffect
                        .createRuntimeShaderEffect(shader, "inputShader")
                        .asComposeRenderEffect()
                } else {
                    renderEffect = null
                }
            }
    ) {
        content()
    }
}
```

---

## 4. View & Activity Level Controller (`NfcRippleEffect`)

For triggering the ripple on standard Android `View` or Activity `decorView`:

```kotlin
@RequiresApi(Build.VERSION_CODES.TIRAMISU)
class NfcRippleEffect(view: View) {

    private val viewRef = WeakReference(view)
    private val shader: RuntimeShader

    private var animator: ValueAnimator? = null

    init {
        val shaderCode = loadShaderSource(view.context)
        shader = RuntimeShader(shaderCode)
    }

    private fun loadShaderSource(context: Context): String {
        return context.resources.openRawResource(R.raw.nfc_ripple).use { stream ->
            BufferedReader(InputStreamReader(stream)).readText()
        }
    }

    fun animate(cx: Float, cy: Float, durationSec: Float = 3.2f) {
        val view = viewRef.get() ?: return
        val displayMetrics = view.resources.displayMetrics
        val density = displayMetrics.density

        val width = view.width.toFloat().takeIf { it > 0 } ?: displayMetrics.widthPixels.toFloat()
        val height = view.height.toFloat().takeIf { it > 0 } ?: displayMetrics.heightPixels.toFloat()

        val amplitude = 32f * density
        val frequency = 12f
        val decay = 4.5f
        val speed = 1400f * density

        shader.setFloatUniform("uResolution", width, height)
        shader.setFloatUniform("uOrigin", cx, cy)
        shader.setFloatUniform("uAmplitude", amplitude)
        shader.setFloatUniform("uFrequency", frequency)
        shader.setFloatUniform("uDecay", decay)
        shader.setFloatUniform("uSpeed", speed)

        animator?.cancel()
        animator = ValueAnimator.ofFloat(0f, durationSec).apply {
            duration = (durationSec * 1000f).toLong()
            interpolator = LinearInterpolator()
            addUpdateListener { anim ->
                val time = anim.animatedValue as Float
                shader.setFloatUniform("uTime", time)
                val effect = RenderEffect.createRuntimeShaderEffect(shader, "inputShader")
                view.setRenderEffect(effect)
                view.invalidate()
            }
            addListener(object : AnimatorListenerAdapter() {
                override fun onAnimationEnd(animation: Animator) {
                    view.setRenderEffect(null)
                    view.invalidate()
                }
            })
            start()
        }
    }

    companion object {
        fun triggerOnView(view: View, cx: Float, cy: Float) {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
                val targetView = view.rootView ?: view
                var effect = targetView.getTag(R.id.ripple_effect_tag) as? NfcRippleEffect
                if (effect == null) {
                    effect = NfcRippleEffect(targetView)
                    targetView.setTag(R.id.ripple_effect_tag, effect)
                }
                effect.animate(cx, cy)
            }
        }
    }
}
```

---

## 5. Best Practices & Compatibility

1. **API Level Guard**: Always guard AGSL with `Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU` (Android 13+).
2. **Coordinate Normalization**: Translate global window offsets (`coordinates.positionInWindow()`) to the local container coordinates (`pos - containerPos`) to ensure the wave center matches the user's touch or target element.
3. **No Garbage Collection Pressure**: Instantiate `RuntimeShader` once (via `remember` or `init`), mutating only its uniforms on every frame tick.
4. **Haptic Integration**: Accompany the liquid wave trigger with a heavy/custom haptic pulse (`HapticUtil.performHeavyHaptic(view)` or `HapticUtil.performCustomHaptic(...)`).
