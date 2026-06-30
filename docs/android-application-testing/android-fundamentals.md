# Android Fundamentals

## Android Platform Architecture

Android is a layered stack. Key layers include the Linux kernel, Hardware Abstraction Layer (HAL), Android Runtime (ART), native libraries, and the Application Framework.

## Important Directories

| Directory                          | Description |
|------------------------------------|-------------|
| /data/data                         | Contains all the applications installed by the user |
| /data/user/0                       | Contains data that only the app can access |
| /data/app                          | Contains the APKs of user-installed applications |
| /system/app                        | Pre-installed applications of the device |
| /system/bin                        | Binary files |
| /data/local/tmp                    | World-writable directory |
| /data/system                       | System configuration files |
| /etc/apns-conf.xml                 | Default APN configurations |
| /data/misc/wifi                    | WiFi configuration files |
| /data/misc/user/0/cacerts-added    | User certificate store |
| /etc/security/cacerts              | System certificate store (no non-root access) |
| /sdcard                            | Symbolic link to media directories (DCIM, Downloads, etc.) |

## App Project Structure

**App folder:**
- manifest: Essential metadata (package name, components, permissions, network config, API level)
- java: Application Java/Kotlin source code
- res: UI strings, images, layouts, other assets

**Gradle Scripts:**
- build.gradle: Build configuration, dependencies, build types, ProGuard settings
- proguard-rules.pro: Custom ProGuard rules

## ADB Commands

| Command | Description |
|---------|-------------|
| adb help | List all commands |
| adb kill-server | Kill the adb server |
| adb devices | List connected devices |
| adb root | Restart adbd with root permissions |
| adb install <apk> | Install an app |
| adb push <local> <remote> | Copy file/dir to device |
| adb pull <remote> <local> | Copy file/dir from device |
| adb logcat [options] [filter] | View device logs |

## Testing Methodology

1. Planning and Environment Setup
2. Enumeration and Information Gathering
3. Static Analysis
4. Dynamic Analysis
5. Documenting and Reporting

## Common Tools

- **ADB** — communicate with devices
- **JADX** — decompile and view source
- **APKTool** — disassemble, edit, recompile APKs
- **Ghidra** — reverse native code
- **Burp Suite** — intercept traffic
- **Frida** — dynamic instrumentation
- **Autopsy** — forensics

**Automation:**
- MobSF, Drozer, Qark, Objection, Medusa, Androbugs
