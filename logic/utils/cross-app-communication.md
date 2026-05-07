# Cross-App Communication Pattern

A secure and lightweight architecture for sharing data and triggering actions between two separate applications (e.g., `Essentials` and `AirSync`) using the **Signature-Protected Broadcast Bridge**.

## 1. Secure Permission Definition

To ensure that only your authorized applications can communicate, define a custom permission in the "owner" app (usually the one receiving the most data).

### Manifest Definition (Essentials)
The `protectionLevel="signature"` ensures that only apps signed with the same developer certificate can use or trigger this bridge.

```xml
<permission 
    android:name="com.sameerasw.permission.ESSENTIALS_AIRSYNC_BRIDGE" 
    android:protectionLevel="signature" />
<uses-permission android:name="com.sameerasw.permission.ESSENTIALS_AIRSYNC_BRIDGE" />
```

## 2. Sending Data (`airsync`)

The sender app uses **Explicit Broadcasts** combined with the custom permission to send data securely.

### Explicit Targeting
Always set the package name and the required permission when sending the broadcast.

```kotlin
val intent = Intent("com.sameerasw.essentials.action.UPDATE_MAC_BATTERY").apply {
    putExtra("level", batteryLevel)
    putExtra("isConnected", true)
    setPackage("com.sameerasw.essentials") // Explicitly target Essentials
}
context.sendBroadcast(intent, "com.sameerasw.permission.ESSENTIALS_AIRSYNC_BRIDGE")
```

## 3. Receiving Data (`essentials`)

The receiver app registers a `BroadcastReceiver` in its manifest, protected by the same signature permission.

### Receiver Declaration
```xml
<receiver
    android:name=".services.receivers.AirSyncBridgeReceiver"
    android:exported="true"
    android:permission="com.sameerasw.permission.ESSENTIALS_AIRSYNC_BRIDGE">
    <intent-filter>
        <action android:name="com.sameerasw.essentials.action.UPDATE_MAC_BATTERY" />
    </intent-filter>
</receiver>
```

## 4. Async Processing in Receivers

Since `onReceive` runs on the main thread and has a short timeout, use `goAsync()` combined with a `CoroutineScope` for any long-running tasks (like updating DataStore or Widgets).

```kotlin
override fun onReceive(context: Context, intent: Intent) {
    val pendingResult = goAsync()
    CoroutineScope(Dispatchers.IO).launch {
        try {
            val level = intent.getIntExtra("level", -1)
            // Process data...
            updateWidgets(context, level)
        } finally {
            pendingResult.finish()
        }
    }
}
```

## 5. Bidirectional Requests

If one app needs to request data from the other (instead of just waiting for an update), it can send a "Request" broadcast which the other app responds to using the same bridge pattern.

### Request Pattern
1. `essentials` sends `ACTION_REQUEST_MAC_BATTERY`.
2. `airsync` receives the request and immediately sends `ACTION_UPDATE_MAC_BATTERY` with the current state.

## 6. Implementation Tips
- **Package Visibility**: If targeting API 30+, add `<queries>` to your manifest to ensure the apps can see each other.
- **Graceful Failure**: Always check if the other app is installed before attempting to trigger complex logic.
- **Data Marshalling**: Keep Intent extras simple (Primitives, Strings, or Parcelables) to ensure fast IPC (Inter-Process Communication).
