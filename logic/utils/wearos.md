# WearOS Communication Pattern

A robust architecture for two-way communication between an Android phone and a WearOS watch, supporting real-time data synchronization and remote action triggers.

## 1. Phone-to-Watch: Data Synchronization (`DataClient`)

The `DataClient` is used for sharing persistent state. Data is automatically synchronized by the system and available even if the app is not running on one of the devices.

### Syncing Device State
Use a dedicated manager (e.g., `DeviceInfoSyncManager`) to gather system info and push it to the `SYNC_PATH`.

```kotlin
val putDataMapReq = PutDataMapRequest.create("/device_info")
val dataMap = putDataMapReq.dataMap

// Gather state
dataMap.putInt("battery_level", batteryPct)
dataMap.putBoolean("flashlight_on", isTorchOn)
dataMap.putString("device_name", Build.MODEL)
dataMap.putLong("timestamp", System.currentTimeMillis()) // Ensure the map is seen as "changed"

// High priority sync
val putDataReq = putDataMapReq.asPutDataRequest().setUrgent()
Wearable.getDataClient(context).putDataItem(putDataReq)
```

### Event-Based Triggers
To keep the watch in sync without killing the battery, trigger a sync on specific system broadcasts:
- `ACTION_BATTERY_CHANGED`
- `RINGER_MODE_CHANGED_ACTION`
- `CameraManager.TorchCallback`

## 2. Watch-to-Phone: Action Triggers (`MessageClient`)

The `MessageClient` is ideal for one-off commands where data persistence is not required.

### Handling Messages (`WearableListenerService`)
Implement a service that extends `WearableListenerService` to process incoming commands from the watch.

```kotlin
class MyWearableListenerService : WearableListenerService() {
    override fun onMessageReceived(messageEvent: MessageEvent) {
        when (messageEvent.path) {
            "/toggle_flashlight" -> {
                // Bridge to local flashlight logic
                val intent = Intent(this, FlashlightActionReceiver::class.java).apply {
                    action = FlashlightActionReceiver.ACTION_TOGGLE
                }
                sendBroadcast(intent)
            }
            "/lock_device" -> {
                // Remote lock via Accessibility or Device Admin
            }
        }
    }
}
```

## 3. Best Practices

### Debouncing Sync Updates
When multiple system events happen rapidly (e.g., rapid flashlight intensity changes), use a `Handler` or `Flow` to debounce the `performSync` calls by ~250ms to prevent excessive Wearable IPC.

### Handling "Urgent" Data
Always use `setUrgent()` for time-sensitive data like battery level or flashlight status. Non-urgent data may be delayed by the system to optimize power.

### Capability Discovery
Use the `CapabilityClient` to check if the watch app is actually installed before showing WearOS-related settings in the phone app UI.

## 4. Common Path Registry
Maintain a consistent set of paths between both projects:
- `/device_info`: Persistent state sync.
- `/request_device_info_sync`: Watch requesting a fresh update.
- `/toggle_flashlight`: Remote command.
- `/set_flashlight_intensity`: Remote command with payload (byte array).
