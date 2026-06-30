# Static Testing

## Extracting APK Files

- Third-party stores (e.g., APKCombo) or APK Export tools.
- Manual via ADB:
  ```bash
  adb shell pm list packages | grep myapp
  adb shell pm path com.example.myapp
  adb pull /data/app/com.example.myapp-1/base.apk .
  ```

## Disassembling the APK

**APKTool:**
```bash
apktool d myapp.apk
```

### Reviewing AndroidManifest.xml

**Permissions** (common dangerous ones and risks):
- READ_SMS / SEND_SMS / RECEIVE_SMS — intercept OTPs
- READ_CALL_LOG / WRITE_CALL_LOG
- READ_CONTACTS / WRITE_CONTACTS
- ACCESS_FINE_LOCATION / COARSE_LOCATION
- READ/WRITE_EXTERNAL_STORAGE
- GET_ACCOUNTS, CAMERA, RECORD_AUDIO
- INSTALL_PACKAGES, SYSTEM_ALERT_WINDOW

**Application class, Network Security Config, Activities:**
- `android:exported="true"` on activities — callable from outside (ADB).
- Intent filters define entry points.

## Smali (Dalvik Bytecode)

Human-readable .dex disassembly.

Example and instruction table provided below.

### Data Types and Reference Types

Full tables for primitives (V, Z, B, S, C, I, J, F, D) and references (L, [ ).

### Fields, Registers, and Method Signatures

Detailed mapping of p/v registers, parameter passing, and examples.

## Reading Hardcoded Strings and Secrets

Search locations: res/xml, res/values/strings.xml, lib/, assets/js.

Use:
```bash
apktool d myapp.apk
grep -Rnw './myapp' -e 'password' -e 'api_key'
strings lib/... | grep -E "[a-zA-Z0-9_-]{60,}"
```

Common secrets: API keys, DB creds, OAuth tokens, crypto keys, PII, URLs, debug info.

## Bad Cryptography

Look for encrypt/decrypt functions and hardcoded keys/IVs.

## Reversing Hybrid Apps

- Locate JS bundles in assets (after JADX or apktool).
- Beautify: `js-beautify ...`
- Hermes bytecode: use hermes-dec tools.

## Obfuscation and Deobfuscation

- ProGuard / R8 (minifyEnabled true + rules).
- Tools like LSParanoid deobfuscator.
- Manual tracing in JADX.

## Native Code (Shared Objects / SO files)

- `strings lib/... | grep ...`
- Ghidra for analysis (import, analyze, use Symbol Tree / Decompiler).
- DLLs (Xamarin etc.): use pyxamstore, ILSpy.

## Patching and Rebuilding

```bash
apktool d myapp.apk
# edit smali
apktool b myapp
# sign
keytool ... ; zipalign ... ; apksigner sign ...
```


