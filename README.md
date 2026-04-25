# awesome-agent-android

A guide for AI coding agents (Claude Code, Gemini CLI, Codex, Cursor, etc.) on **what's possible** when building and debugging Android apps from a shell — and the gotchas that bite during real autonomous loops.

This is intentionally not a copy-paste cookbook. Exact flags and command names drift with tool versions. Agents are good at discovering current syntax via `--help`, READMEs, and trial. What agents *can't* easily discover is what capabilities exist in the first place, and where the sharp edges are.

The promise of this guide: after reading it, an agent should have a complete mental model of the Android dev loop and know which tool to reach for next — without needing to look up flags in advance.

---

## Why this exists

Agents fail at Android in predictable ways:

1. **They don't know which tool exists for a job.** They reach for screenshots when a structured layout dump is available. They reach for raw `gradle` when the project ships a wrapper. They reach for full rebuilds when static APK analysis would answer the question in a second.
2. **They use destructive shortcuts** without knowing they're destructive — most notably the new `android run --apks=…`, which silently uninstalls the existing app on install failure.
3. **They don't read logs effectively.** They `grep` for the wrong patterns or stop at the first `E` line instead of finding the `Caused by:` root.
4. **They assume the GUI is the source of truth.** Most things Android Studio shows you visually are exposed as text by `adb`, `apkanalyzer`, and `android`. Agents don't need a screen.

The rest of this guide is the inverse of those failure modes.

---

## What's possible from a shell

### Build, install, launch, observe

Every Android app has the same lifecycle and an agent can drive every step:

- **Build** — projects ship a `gradlew` wrapper. Use it, never bare `gradle`. The wrapper pins the right Gradle version. `assembleDebug` is the standard target for an unsigned, runnable APK.
- **Install** — `adb install` puts the APK on a device or emulator. `-r` reinstalls in place.
- **Launch** — `adb shell am start` invokes the activity manager to start an activity. The activity must be declared in the manifest with a `LAUNCHER` intent-filter.
- **Observe** — three independent channels: a screenshot (PNG), a layout tree (structured JSON), and `adb logcat` for runtime logs. Use whichever answers your question fastest.

### Inspect a compiled APK without running it

`apkanalyzer` reads APKs as static bytes. From an APK alone you can list every class and method in the dex, list cross-references between symbols, print the merged manifest after AGP processes it, and break down APK size by file. This is the fastest way to answer "is class X actually compiled in?" or "did the manifest merger drop my permission?" — much faster than rebuilding and re-running.

For full decompiled source (function bodies, not just signatures), `jadx` produces readable Java/Kotlin from any APK including third-party ones.

### Read the live UI as data, not pixels

Google's `android` CLI exposes a `layout` subcommand that returns the current foreground UI as JSON: text content, resource IDs, click targets, and pixel coordinates per element. There's also a `--diff` mode that returns only what changed since the last call, which is ideal for tight UI iteration loops.

Prefer this over screenshots when the question is "did my button render with the right text/id." Reach for screenshots only when pixel layout actually matters.

### Manage emulators

The `android emulator` subcommand creates, starts, stops, and lists virtual devices. `adb` complements it — `adb devices` confirms the emulator is online, `adb shell getprop sys.boot_completed` confirms Android itself finished booting (the emulator is "online" before the OS is ready).

For tight iteration loops, leave the emulator running between runs. Cold boot is 30–60 seconds.

### Read runtime logs

`adb logcat` streams every log message from the device. The `-d` flag dumps and exits (no streaming), which is what you usually want from a script. Filter by package, log level, or pattern. For crashes, the magic words to grep for are `AndroidRuntime`, `FATAL`, and `Caused by:` — the last one names the root exception.

---

## The dev loop, conceptually

```
edit → build → install → launch → observe → diagnose → edit
```

Each step is one tool. Agents loop through this autonomously by chaining commands and reading the output of the next step.

The most important habit: **after every launch, observe before assuming success.** Apps can install and "launch" but immediately crash back to the launcher. Always confirm the foreground activity is yours, not the system launcher, before moving on.

---

## Crash diagnosis as a discipline

When something goes wrong, the answer is in `logcat` 95% of the time. The discipline:

1. Clear logcat (`-c`) before launching, so the only entries are from this run.
2. Launch.
3. Wait a few seconds.
4. Dump logcat, filter to crash patterns.
5. Find the **`Caused by:`** line — that names the root exception. The lines above it are the framework's call stack; the lines below it are the chain of wrapped causes.
6. Map the exception to a likely root.

Common roots worth memorizing:

| Exception | Likely root cause |
|---|---|
| `ClassNotFoundException` on an activity | Manifest declares a class name that doesn't exist in the dex (typo, renamed class, ProGuard stripped it). Verify with `apkanalyzer dex references`. |
| `Resources$NotFoundException` | A layout/drawable/string ID is referenced but the resource isn't in the build. Often a stale build — clean and rebuild. |
| `InflateException` | XML layout error. Logcat lines around the exception name the file and line number. |
| `ActivityNotFoundException` | No matching activity in the manifest, or the `LAUNCHER` intent-filter is missing. |
| `SecurityException: Permission Denial` | Missing `<uses-permission>` declaration, or runtime permission not requested. |

When you can't tell from logcat alone whether the manifest matches the code, fall back to static analysis on the APK.

---

## Gotchas

These are non-obvious rules that no `--help` will tell you. Memorize them.

### `android run --apks=…` is destructive on install failure

It tries to install + launch atomically. On install failure it leaves the app **uninstalled** (force-stopped + `pkg removed`). Subsequent attempts to launch will fail with "Activity class does not exist," sending you down a wrong-cause rabbit hole. Always split: `adb install` then `adb shell am start`.

### `android emulator start` takes a positional AVD name

Not a `--device=` flag. The CLI's broader pattern of `--device` for other subcommands is misleading.

### Kotlin reserved words break package declarations

`in`, `is`, `object`, `package`, `class` are reserved. If your reverse-DNS package starts with `in.` (common for Indian developers), the Kotlin source needs backticks: `package \`in\`.example.app`. The directory layout is fine without escaping.

### "Broken pipe (32)" on install is usually transient

Retry once. If it persists, restart the adb server. Not a code problem.

### `AppsFilter ... BLOCKED` in logcat is not your bug

Android 11+ enforces package visibility. Every install logs a flurry of these. They're other apps being told they can't see yours. Skip them.

### `android run` (without `--apks`) is launch-only

It assumes the app is already installed. If no activity is declared in the manifest, it'll report "App loaded ... No matching components found for type ACTIVITY" — which is a launch-time failure, not a build or install failure. Don't conflate them.

### `gradlew assembleDebug` and `gradlew installDebug` differ

`assembleDebug` builds only. `installDebug` builds and installs. The latter is convenient but bypasses your chance to inspect the APK before deploying. For loops where static analysis matters, prefer the explicit two-step.

### The emulator may have other apps installed

Especially shared dev emulators or images carried over from prior projects. When `android layout` returns surprising content, your activity isn't necessarily broken — another app may be in the foreground. Always confirm the resumed activity first via `adb shell dumpsys activity activities | grep ResumedActivity`.

### `JAVA_HOME` matters for Gradle

The Gradle wrapper invokes the JVM at `JAVA_HOME` (or system default). Android Studio bundles a JDK that's known to work with the AGP version. If builds fail with cryptic Kotlin daemon errors, suspect a JDK version mismatch — point `JAVA_HOME` at Android Studio's bundled JDK.

---

## Agent prompt patterns

These are *shapes* an agent should recognize, not literal scripts. The agent picks the exact tool calls.

### "Is this app working?"

Build → install → launch → wait briefly → confirm the foreground activity is the target one (not the launcher). If it's the launcher, the app crashed — pivot to crash diagnosis.

### "Why did it crash?"

Pull the relevant slice of logcat from this run. Find the `Caused by:` chain. Map to a likely root using the table above. If the root is a `ClassNotFoundException` or similar manifest-vs-code mismatch, verify against the APK with static analysis instead of guessing.

### "Did my change land in the binary?"

Don't relaunch — inspect the APK. List dex packages or grep references for the symbol you changed. If the symbol isn't in the dex, the build didn't pick up your change (cache, bad path, ProGuard stripped it).

### "Iterate on the UI until it looks right"

Loop: edit → rebuild → install → launch → dump layout (not screenshot) → check for expected resource IDs and text → repeat. Reach for a screenshot only when pixel layout is the actual question.

### "Migrate this codebase to X"

If "X" is one of the supported migration topics (Compose, Navigation 3, edge-to-edge, AGP 9, Play Billing, R8 cleanup), use the corresponding [Android Skill](https://github.com/android/skills) — these are Google-authored playbooks installed via `android skills add`.

---

## Setup, briefly

This guide assumes you have:

- The Android SDK (via Android Studio or standalone command-line tools).
- Google's `android` CLI, the new agent-facing tool. Announcement and downloads: [d.android.com/tools/agents](https://developer.android.com/tools/agents).
- Skills installed for your agent of choice via `android skills add --agent=<name>`. List skills with `android skills list`.
- Standard env vars (`ANDROID_HOME`, `JAVA_HOME`) and platform-tools / cmdline-tools / emulator on `PATH`. Once these resolve, every CLI in this guide is invokable by bare name.

The agent should verify its environment with one or two probes (`which adb`, `android info`) at the start of any Android session and surface what's missing rather than failing later.

---

## Vendor neutrality

Nothing in this guide is specific to one agent platform. The tools — `adb`, `apkanalyzer`, `gradlew`, `emulator`, the new `android` CLI — produce text I/O that any agent can consume.

Agent-specific bits are limited to where skills install:

- **Claude Code** — `~/.claude/skills/`
- **Gemini CLI / Antigravity** — `~/.gemini/antigravity/skills/`
- **Codex / Cursor / Windsurf** — varies; consult each platform's skill or rules-file conventions.

Use `android skills add --agent=<name>` to target the right directory.

---

## What's out of scope

- **Native debugging** (`gdb`, `lldb` attach). Rarely needed by an agent; see Android NDK docs if it is.
- **Profiling.** `simpleperf`, Android Studio Profiler, `systrace` — these are human-shaped tools. An agent should usually surface "this is a profiling question, ask a human" rather than try to drive them.
- **Real device debugging.** Same `adb` commands, but assumes USB debugging is enabled and a device is connected. Emulator loops are typically faster.
- **Release signing and store deploys.** Production deploy is human-in-the-loop territory. An agent shouldn't autonomously upload to Play.

---

## Credits

- Google's [Android CLI announcement](https://android-developers.googleblog.com/2026/04/build-android-apps-3x-faster-using-any-agent.html)
- The [android/skills repository](https://github.com/android/skills)

This guide is independent and not affiliated with Google.

## License

MIT — see [LICENSE](./LICENSE).

## Contributing

Pull requests welcome. Particularly valuable: additional crash patterns and root causes, agent prompt patterns that worked in practice, and anti-patterns to add to the gotchas list. Keep contributions principle-shaped, not command-shaped — this guide stays useful by *not* being a moving target.
