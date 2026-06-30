# Dynamic Testing

## Enumerating Local Storage (with Root)

```bash
adb root
adb shell
dumpsys activity activities | grep VisibleActivityProcess
ls -l /data/data/com.example.myapp
```

Examples:
- SQLite databases: `sqlite3 ... ; .tables ; select * from ...;`
- Shared preferences: edit XML directly to flip boolean flags (e.g., pinLockStatus).
- External storage: `/sdcard/Android/data/<package>/`

## Exported Activities / Components

- Components with `exported="true"` in manifest can be started directly:
  ```bash
  adb shell am start -n com.example.myapp/.NoteContentActivity --es filename "Note_xxx.txt"
  ```
- Useful when logic (e.g., decryption or access without PIN) is present.

## Insecure Logging

- Use logcat with filters (`adb logcat '*:D' | grep 'tag'`)
- Sensitive data (decrypted notes, tokens) sometimes logged before proper protection is applied.
- Combine with exported components to trigger the code path.

## Pending Intents and Other Issues

- Look for mutable pending intents (`FLAG_MUTABLE`) that can be hijacked or modified.
- Inspect code for intent handling that can be abused to access protected functionality.

## General Dynamic Workflow

- Root or Frida for deeper runtime inspection.
- Monitor filesystem changes, network, and logs while exercising the app.
- Test exported components, deep links, and intent-based attacks.
- Combine with static findings (e.g., use JADX to locate the method, then trigger via adb/am).
