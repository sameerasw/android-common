# Shell Command Execution Pattern

A unified system for executing privileged `adb` and `shell` commands by abstracting away the differences between **Shizuku** and **Root** (Magisk/KernelSU).

## 1. Unified Abstraction (`ShellUtils`)

The app uses `ShellUtils` as a high-level wrapper. This allows the rest of the app to call shell commands without worrying about whether the user has granted Root or Shizuku permissions.

```kotlin
object ShellUtils {
    fun runCommand(context: Context, command: String) {
        if (isRootEnabled(context)) {
            RootUtils.runCommand(command)
        } else {
            ShizukuUtils.runCommand(command)
        }
    }
}
```

## 2. Common Command Patterns

The following are the most frequent use cases for shell commands in our ecosystem.

### System UI Triggers (`cmd statusbar`)
Trigger native system animations that are usually inaccessible to third-party apps.

- **Charging Ripple**: `cmd statusbar charging-ripple`
- **Auth Ripple**: `cmd statusbar auth-ripple custom $x $y` (Requires coordinates)
- **Dismiss Keyboard**: `cmd statusbar dismiss-keyboard`

### Settings Manipulation (`settings put`)
Modify system-level toggles that are typically marked as "Secure" or "Global".

- **Airplane Mode**: `settings put global airplane_mode_on 1`
- **Mono Audio**: `settings put system master_mono 1`
- **Window Animation Scale**: `settings put global window_animation_scale 0.5`

### Permission Management (`pm grant`)
Granting sensitive permissions to our own app or others without a PC.

- `pm grant <package_name> android.permission.WRITE_SECURE_SETTINGS`
- `pm grant <package_name> android.permission.DUMP`

### Package Control (`pm disable/enable`)
Forcefully enabling or disabling apps/components (used for features like App Freezer).

- `pm disable-user --user 0 <package_name>`
- `pm enable <package_name>`

## 3. Advanced Interaction (`runCommandWithOutput`)

When we need to retrieve data from the system (e.g., checking if a specific service is running), we use output capture.

```kotlin
fun isServiceActive(serviceName: String): Boolean {
    val output = ShellUtils.runCommandWithOutput(context, "service check $serviceName")
    return output?.contains("Service $serviceName: found") == true
}
```

## 4. Implementation Tips

- **Background Execution**: Shell commands involve IPC (Inter-Process Communication) and can block the main thread. Always wrap them in `Dispatchers.IO`.
- **User Preference**: Always provide a "Use Root" toggle in settings. If disabled, default to [Shizuku Integration](file:///Users/sameerasandakelum/GIT/jetpack-common/permissions/shizuku.md).
- **Escape Commands**: Be careful with command strings. If a package name or coordinate is dynamic, ensure it's properly sanitized to prevent injection.
- **Fire and Forget**: For animations like ripples, use a fire-and-forget approach to avoid waiting for the process to terminate.
