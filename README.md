# awesome-agent-android

A practical, vendor-neutral guide for AI coding agents (Claude Code, Gemini CLI, Codex, Cursor, etc.) to **build, install, launch, and debug Android apps end-to-end from a shell** — no GUI, no human-in-the-loop.

The Android Studio GUI was designed for human developers. Agents need text I/O, deterministic commands, and structured output. This guide documents the toolchain that makes that work, with the gotchas that bite during real autonomous loops.

---

## Why this exists

Agents fail at Android in predictable ways:

1. **They reach for `gradle` directly** — but most projects ship a `gradlew` wrapper with a pinned version, and bare `gradle` invocations silently use a different one.
2. **They try to launch apps with `adb shell am start`** without first installing the APK, or without confirming the activity is declared in the manifest.
3. **They take screenshots when a layout dump would tell them more**, in less time, with structured data they can actually parse.
4. **They use `android run --apks=…` as a one-shot install+launch** and don't know it leaves the app uninstalled when it fails.
5. **They can't read crash logs** because nobody told them `adb logcat` exists, or they grep for the wrong patterns.

This guide fixes all five.

---

## Quick install

### 1. Android SDK

If you're on macOS or Linux and don't already have Android Studio installed:

```bash
# macOS — install Android Studio (bundles the SDK)
brew install --cask android-studio
# Or get the SDK alone via cmdline-tools: https://developer.android.com/studio#command-tools

# Linux — download the cmdline-tools zip from https://developer.android.com/studio
```

After install, the SDK lives at:
- macOS: `~/Library/Android/sdk`
- Linux: `~/Android/Sdk`
- Windows: `%LOCALAPPDATA%\Android\Sdk`

### 2. Google's `android` CLI (the new agent-facing tool)

```bash
# macOS (Apple Silicon)
curl -fsSL https://dl.google.com/android/cli/latest/darwin_arm64/install.sh | bash

# Linux x86_64
curl -fsSL https://dl.google.com/android/cli/latest/linux_x86_64/install.sh | bash
```

The default install location is `/usr/local/bin/android` (requires sudo). To install to `~/.local/bin` without sudo, download the binary directly:

```bash
curl -fsSL https://dl.google.com/android/cli/latest/darwin_arm64/android \
  -o ~/.local/bin/android && chmod +x ~/.local/bin/android
ANDROID_CLI_FRESH_INSTALL=1 ~/.local/bin/android   # one-time bootstrap
```

### 3. Environment variables and PATH

Add to `~/.zshrc` or `~/.bashrc`:

```bash
export ANDROID_HOME="$HOME/Library/Android/sdk"   # Linux: $HOME/Android/Sdk
export ANDROID_SDK_ROOT="$ANDROID_HOME"
export PATH="$PATH:$ANDROID_HOME/platform-tools:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/emulator"
```

Verify:
```bash
which adb apkanalyzer sdkmanager avdmanager emulator android
```

### 4. Install agent skills

Google ships markdown "skills" that activate when prompts match topics like "migrate to Compose" or "fix edge-to-edge bug." Install them for your agent of choice:

```bash
# Claude Code
android skills add --all --agent=claude-code

# Gemini CLI
android skills add --all --agent=gemini

# Codex
android skills add --all --agent=codex
```

Available skills include `android-cli`, `navigation-3`, `edge-to-edge`, `migrate-xml-views-to-jetpack-compose`, `agp-9-upgrade`, `play-billing-library-version-upgrade`, `r8-analyzer`. List with `android skills list`.

---

## The core loop

Every Android dev cycle reduces to:

```
build → install → launch → observe → diagnose → edit → repeat
```

### Build

```bash
./gradlew assembleDebug
# APK: app/build/outputs/apk/debug/app-debug.apk
```

Always use the project's `./gradlew`, never bare `gradle`.

### Install

```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

`-r` reinstalls if already present. Don't use `android run --apks=…` for this — see [Gotchas](#gotchas).

### Launch

```bash
adb shell am start -n <package>/.<MainActivity>
# e.g. adb shell am start -n com.example.app/.MainActivity
```

The activity must be declared in `AndroidManifest.xml` with a `LAUNCHER` intent-filter:

```xml
<activity android:name=".MainActivity" android:exported="true">
  <intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
  </intent-filter>
</activity>
```

### Observe

```bash
android screen capture -o /tmp/state.png    # screenshot (PNG)
android layout -p                            # UI tree as pretty JSON
android layout --diff                        # only what changed since last call
adb logcat -d -t 200                         # last 200 log lines
```

**Prefer `android layout` over screenshots.** It returns structured JSON with element text, resource IDs, click targets, and pixel coordinates — far more parseable than image bytes:

```json
[
  {"text":"Hello", "center":"[539,1200]", "resource-id":"tvHello"},
  {"text":"Tap me", "interactions":["clickable"], "center":"[540,1761]", "resource-id":"btnGo"}
]
```

### Diagnose

```bash
adb logcat -d | grep -E "AndroidRuntime|FATAL|Caused by" | tail -30
```

Common causes:

| Exception | Likely root |
|---|---|
| `ClassNotFoundException: Didn't find class "...MainActivity"` | Manifest `android:name=` doesn't match the dex class name |
| `Resources$NotFoundException` | Layout/resource ID referenced but not built — clean and rebuild |
| `InflateException` | XML layout error — surrounding logcat lines name the file and line number |
| `ActivityNotFoundException` | No `<activity>` declared, or `LAUNCHER` intent-filter missing |
| `SecurityException: Permission Denial` | Missing `<uses-permission>` in manifest |

---

## Static analysis (read code without running it)

When you suspect a name mismatch or want to audit what's *actually* compiled into an APK, don't rebuild and re-run. Inspect the binary directly:

```bash
apkanalyzer dex packages app.apk                      # all classes + methods
apkanalyzer dex references app.apk | grep <Symbol>    # is X actually compiled in?
apkanalyzer manifest print app.apk                    # final merged manifest
apkanalyzer apk summary app.apk                       # size breakdown
apkanalyzer resources packages app.apk                # all resources
```

**Example: proving a class doesn't exist.** If `AndroidManifest.xml` declares `android:name=".MainActivity"` and the app crashes with `ClassNotFoundException`:

```bash
$ apkanalyzer dex references app.apk | grep MainActivity
# (empty — confirms the class isn't in the dex)

$ apkanalyzer dex packages app.apk | grep -i mainactivity
C d   com.example.app.leaMainActivity   # found! it's named differently
```

For full source decompilation (function bodies, not just signatures):

```bash
brew install jadx          # macOS
# or apt install jadx      # Linux
jadx -d /tmp/decompiled app.apk
# Browse /tmp/decompiled — readable Java/Kotlin
```

---

## Emulator management

```bash
android emulator list                                 # available AVDs
android emulator start <avd-name>                     # POSITIONAL arg (NOT --device=)
android emulator stop <avd-name>
adb devices                                           # confirm online after boot
adb shell getprop sys.boot_completed                  # "1" = ready
```

Cold boot takes 30–60s. For tight iteration, leave the emulator running between runs.

### Wait-for-boot pattern

```bash
android emulator start Pixel_API_36 &
until adb devices | grep -E "emulator-[0-9]+\s+device$" >/dev/null 2>&1; do
  sleep 3
done
# emulator is now listed but may still be booting Android itself
until [ "$(adb shell getprop sys.boot_completed | tr -d '\r')" = "1" ]; do
  sleep 2
done
echo "ready"
```

### Creating a new AVD from CLI

```bash
sdkmanager "system-images;android-34;google_apis;arm64-v8a"
avdmanager create avd -n Pixel_API_34 -k "system-images;android-34;google_apis;arm64-v8a" -d pixel
```

---

## Gotchas

### 1. `android run --apks=` is destructive on install failure

Google's `android run` tries to install + launch atomically. If the install step fails (broken pipe, signature mismatch, etc.), it leaves the app **uninstalled** — force-stopped and `pkg removed`. Then your subsequent `adb shell am start` fails with "Activity class does not exist," sending you down the wrong rabbit hole.

**Always split:**
```bash
adb install -r app.apk
adb shell am start -n pkg/.Activity
```

### 2. `android emulator start` takes a positional AVD name

Not `--device=` (despite what `--help` for related commands suggests):
```bash
android emulator start Pixel_API_36         # ✓
android emulator start --device=Pixel_API_36  # ✗
```

### 3. `in` and `package` are reserved words in Kotlin

If your package name starts with `in.` (common for Indian developers using country-coded reverse-DNS), the Kotlin source file needs backticks:
```kotlin
package `in`.example.app
```
The directory layout (`src/main/java/in/example/app/`) is fine without escaping.

### 4. Emulator install: "Broken pipe (32)"

Usually a transient adb/emulator hiccup. Retry once. If persistent:
```bash
adb kill-server && adb start-server
```

### 5. `AppsFilter ... BLOCKED` in logcat is normal

Android 11+ enforces package visibility. Each install logs a flurry of "BLOCKED" entries — that's just other packages being told they can't see yours. Not a bug.

### 6. `android run` is launch-only by default

Without `--apks=`, `android run` assumes the app is already installed and tries to *launch* an activity. You'll get "App loaded: pkg / No matching components found for type ACTIVITY" if no activity is declared in the manifest. The CLI doesn't auto-build.

### 7. `gradlew assembleDebug` vs `gradlew installDebug`

`assembleDebug` builds the APK only. `installDebug` builds AND installs to the connected device. The latter is convenient but bypasses your ability to inspect the APK before deploy. For agent loops where you want to apkanalyze before installing, prefer `assembleDebug` + explicit `adb install -r`.

---

## Agent prompt patterns

Effective patterns for prompts you might give an agent (or have your meta-agent generate for sub-agents):

### "Build this app and tell me if it crashes"

```
1. cd to project root
2. ./gradlew assembleDebug
3. adb install -r app/build/outputs/apk/debug/app-debug.apk
4. adb logcat -c
5. adb shell am start -n <pkg>/.<MainActivity>
6. sleep 3
7. adb shell dumpsys activity activities | grep ResumedActivity
   - if foreground is the launcher, app crashed
8. adb logcat -d | grep -E "AndroidRuntime|FATAL|Caused by" | tail -30
9. report the "Caused by:" line as root cause
```

### "Verify the manifest matches the compiled code"

```
1. apkanalyzer manifest print app.apk | grep "android:name"
2. for each declared activity/service/receiver:
     apkanalyzer dex references app.apk | grep <ClassName>
3. report any declarations with no matching dex reference
```

### "Iterate on UI until it looks right"

```
loop:
  edit XML or Compose code
  ./gradlew assembleDebug && adb install -r app-debug.apk
  adb shell am start -n <pkg>/.<Activity>
  android layout -p > /tmp/layout.json
  if /tmp/layout.json contains expected text/resource-ids:
    android screen capture -o /tmp/final.png
    break
  else:
    diagnose by re-reading the layout JSON
```

---

## Vendor-neutral notes

This guide works across agent platforms because it relies only on standard CLI tools:

- **Claude Code** — skills install to `~/.claude/skills/`. Trigger via `--agent=claude-code`.
- **Gemini CLI / Antigravity** — skills install to `~/.gemini/antigravity/skills/`. Default if no agent flag.
- **Codex CLI** — skills install via `--agent=codex`. Skills are markdown; Codex reads them as context.
- **Cursor / Windsurf** — no native skills loader, but you can paste this guide into a `.cursorrules` or `.windsurfrules` file at the project root.

The `android` CLI itself, `adb`, `apkanalyzer`, `gradlew`, and `emulator` are agent-agnostic — they just produce text output that any agent can read.

---

## What's NOT covered (yet)

- **Native debugging** — `gdb`/`lldb` attach to running processes. Agents rarely need this; if they do, see [Android NDK debugging docs](https://developer.android.com/ndk/guides/debug).
- **Profiling** — `simpleperf`, Android Studio Profiler, `systrace`. Better suited to human inspection.
- **Real device debugging** — same `adb` commands as emulator, but you need USB debugging enabled and (usually) a USB cable. Agent loops are typically faster on emulator.
- **Play Console / signing** — release builds, signing keys, store uploads. These are deploy-time concerns and usually want human-in-the-loop.

---

## Credits

- Google's [android-cli announcement](https://android-developers.googleblog.com/2026/04/build-android-apps-3x-faster-using-any-agent.html)
- The [android/skills repository](https://github.com/android/skills)

This guide is independent and not affiliated with Google.

## License

MIT — see [LICENSE](./LICENSE).

## Contributing

Pull requests welcome. Particularly valuable:
- Linux/Windows install command corrections
- Additional crash patterns and their root causes
- Agent prompt patterns that worked well in practice
- Skills authored by the community (publish to `android/skills` upstream when possible)
