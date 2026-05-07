# Shizuku Integration Pattern

Shizuku allows regular apps to use system APIs directly with `adb` or `root` privileges. In our implementation, we use Shizuku as a non-intrusive alternative to Root for executing privileged shell commands and granting sensitive permissions.

## 1. Lifecycle Management (`ShizukuUtils`)

Shizuku is binder-based. We must listen for the binder to be received or "dead" (if the Shizuku service stops).

### Initialization
```kotlin
fun initialize() {
    Shizuku.addBinderReceivedListener {
        binder = Shizuku.getBinder()
        if (Shizuku.checkSelfPermission() != PackageManager.PERMISSION_GRANTED) {
            Shizuku.requestPermission(REQUEST_CODE)
        }
    }
    Shizuku.addBinderDeadListener { binder = null }
}
```

## 2. Privileged Shell Commands

We use `IShizukuService` to spawn new processes with `shell` privileges.

```kotlin
fun runCommand(command: String) {
    if (!hasPermission()) return
    
    val service = IShizukuService.Stub.asInterface(Shizuku.getBinder())
    try {
        // Run as 'sh -c [command]'
        val process = service.newProcess(arrayOf("sh", "-c", command), null, "/")
        process?.waitFor()
    } catch (e: RemoteException) {
        Log.e("Shizuku", "Execution failed", e)
    }
}
```

### Usage: Granting Permissions
One of the most powerful use cases is granting `WRITE_SECURE_SETTINGS` to the app itself without requiring the user to use a PC.

```kotlin
fun grantSecureSettings() {
    runCommand("pm grant ${context.packageName} android.permission.WRITE_SECURE_SETTINGS")
}
```

## 3. Advanced: Accessing System Binders

Shizuku can provide access to system services that are usually hidden from third-party apps.

```kotlin
fun getSystemBinder(name: String): IBinder? {
    val service = IShizukuService.Stub.asInterface(Shizuku.getBinder())
    return service.getSystemBinder(name) // e.g., "statusbar" or "activity"
}
```

## 4. The Dual-Mode Abstraction (`ShellUtils`)

To ensure the app works for both Root and Shizuku users, we use `ShellUtils` as a high-level wrapper.

| Feature | Shizuku | Root |
| :--- | :--- | :--- |
| **Setup** | Requires Shizuku App | Requires Magisk/KernelSU |
| **Speed** | Medium (Binder IPC) | High (Direct su) |
| **Safety** | High (Managed by Shizuku) | Low (Full system access) |

### Unified Call Pattern
```kotlin
fun toggleAirplaneMode(enabled: Boolean) {
    val state = if (enabled) 1 else 0
    // ShellUtils automatically detects if it should use Root or Shizuku
    ShellUtils.runCommand(context, "settings put global airplane_mode_on $state")
}
```

## 5. Implementation Tips
- **Haptic Warnings**: Always perform a heavy haptic feedback before requesting Shizuku permissions, as it's a high-privilege request.
- **Availability Checks**: Always check `isShizukuAvailable()` before showing Shizuku-related UI elements.
- **User Choice**: In the app settings, provide a toggle to switch between Shizuku and Root modes if both are available.
