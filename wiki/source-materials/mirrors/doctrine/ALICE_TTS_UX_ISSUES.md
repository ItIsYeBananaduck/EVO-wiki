---
title: ALICE_TTS_UX_ISSUES
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/ALICE_TTS_UX_ISSUES.md"]
updated: 2026-07-24
---

# Alice TTS & App UX Issues - Diagnosis & Fixes

> **Issues Reported:**
>
> 1. Kokoro TTS not working (falling back to system voice)
> 2. TTS toggle needed when audio device connected
> 3. White screen for 2-3s after app idle
> 4. OAuth doesn't auto-return to app after completion

---

## Issue 1: Kokoro TTS Not Working

### Current Behavior

- User hears "bad voice" (system TTS fallback)
- Kokoro falls back to system TTS with error messages

### Root Causes Identified

#### A. ONNX Runtime Import Conditional

**Location:** `KokoroTtsPlugin.swift:6-8`

```swift
#if canImport(onnxruntime_objc)
    import onnxruntime_objc
#endif
```

**Problem:** If ONNX Runtime framework isn't properly linked, the entire Kokoro synthesis code is disabled at compile time.

**Diagnosis Steps:**

1. Check if `onnxruntime.framework` is in `ios/Frameworks/`
2. Check Xcode project settings → General → Frameworks, Libraries, and Embedded Content
3. Check build logs for framework loading errors

#### B. Asset File Locations

**Location:** `KokoroTtsPlugin.swift:211-265`

The plugin checks 3 locations in priority order:

1. **App Group**: `group.biz.lsctech.adaptivefit/EVO/ModelStore/AliceAssets/onnx/`
2. **Documents/EVO**: `Documents/EVO/ModelStore/AliceAssets/onnx/`
3. **Legacy**: `Documents/AliceAssets/onnx/`

**Required Files:**

- `kokoro-v1-fp16.onnx` (ONNX model)
- `voice_af_bella.bin` (voice embedding)
- `tokenizer.json` (tokenizer data)

**Diagnosis Command:**

```bash
# Check if files exist in App Group
xcrun simctl get_app_container <device_id> biz.lsctech.adaptivefit data

# Check file sizes
ls -lh <app_container>/Library/Application\ Support/group.biz.lsctech.adaptivefit/EVO/ModelStore/AliceAssets/onnx/
```

#### C. CoreML Fallback Issue

**Location:** `KokoroTtsPlugin.swift:347-353`

```swift
do {
    try sessionOptions.appendCoreMLExecutionProvider(...)
    usedCoreML = true
} catch {
    print("[KokoroTts] WARNING: CoreML not available, using CPU: \(error)")
}
```

**Problem:** If CoreML fails to initialize, it falls back to CPU which may be too slow or cause inference failures.

**Fix:** Add better error handling and diagnostics:

```swift
} catch {
    print("[KokoroTts] CoreML provider failed: \(error.localizedDescription)")
    if let nsError = error as NSError? {
        print("[KokoroTts] CoreML error details - Domain: \(nsError.domain), Code: \(nsError.code)")
    }
    // Try CPU fallback
}
```

#### D. Tokenization Vocabulary Incomplete

**Location:** `KokoroTtsPlugin.swift:24-33`

```swift
private let vocab: [Character: Int64] = [
    ";": 1, ":": 2, ",": 3, ".": 4, "!": 5, "?": 6,
    // ... only basic characters
]
```

**Problem:** The hardcoded vocab only has ~40 characters. Real Kokoro uses `tokenizer.json` with full phoneme vocabulary. Unknown characters fall back to space token (16), producing poor audio.

**Fix Options:**

1. **Load tokenizer.json** (currently not implemented):

```swift
private func loadTokenizer() {
    guard let tokenizerPath = self.tokenizerPath else { return }
    // Parse tokenizer.json and build full vocab
}
```

2. **Use phonetic conversion** for better fallback
3. **Log tokenization failures** to diagnose

### Recommended Fixes

#### Fix 1: Add Diagnostic Logging

**File:** `KokoroTtsPlugin.swift`

Add to `checkAvailability`:

```swift
private func checkAvailability(result: @escaping FlutterResult) {
    synthesisQueue.async { [weak self] in
        guard let self = self else {
            DispatchQueue.main.async { result(false) }
            return
        }

        print("[KokoroTts] === AVAILABILITY CHECK ===")
        print("[KokoroTts] ONNX Runtime available: \(canImport(onnxruntime_objc))")

        let available = self.checkAssetsPresent()
        print("[KokoroTts] Assets present: \(available)")

        if available && !self.isInitialized {
            print("[KokoroTts] Initializing ONNX session...")
            self.initializeOnnxSession()
            print("[KokoroTts] Initialization complete: \(self.isInitialized)")
            if let error = self.lastInitError {
                print("[KokoroTts] Last init error: \(error)")
            }
        }

        DispatchQueue.main.async {
            result(available && self.isInitialized)
        }
    }
}
```

#### Fix 2: Better Error Reporting to Flutter

**File:** `KokoroTtsPlugin.swift` → `speak` method

```swift
case "speak":
    guard let args = call.arguments as? [String: Any] else {
        result(FlutterError(code: "INVALID_ARGS", message: "Missing arguments", details: nil))
        return
    }
    let text = args["text"] as? String ?? ""
    let speed = args["speed"] as? Double ?? 1.0

    // Return diagnostic info on failure
    speak(text: text, speed: speed) { success in
        if success {
            result(true)
        } else {
            result(FlutterError(
                code: "SYNTHESIS_FAILED",
                message: self.lastInitError ?? "Unknown error",
                details: self.getDiagnosticStatus()
            ))
        }
    }
```

#### Fix 3: Implement Tokenizer.json Loading

**File:** `KokoroTtsPlugin.swift`

```swift
private var fullVocab: [String: Int64] = [:]

private func loadTokenizer() {
    guard let tokenizerPath = self.tokenizerPath else {
        print("[KokoroTts] Tokenizer path not set")
        return
    }

    do {
        let data = try Data(contentsOf: URL(fileURLWithPath: tokenizerPath))
        let json = try JSONSerialization.jsonObject(with: data) as? [String: Any]

        if let model = json?["model"] as? [String: Any],
           let vocab = model["vocab"] as? [String: Int] {
            print("[KokoroTts] Loaded tokenizer vocab with \(vocab.count) tokens")
            self.fullVocab = vocab.mapValues { Int64($0) }
        }
    } catch {
        print("[KokoroTts] Failed to load tokenizer: \(error)")
    }
}
```

---

## Issue 2: TTS Toggle with Audio Device

### Current Behavior

✅ **Already Implemented!**

**Location:** `alice_chat_screen.dart:903-978`

The UI already has:

- Volume icon button (green when ON, gray when OFF)
- PopupMenu with "Auto-speak responses" toggle
- "Allow speaking without device" option when no audio device connected

### Current Logic

```dart
bool _shouldAutoSpeak() {
  return _autoSpeak && (_hasAudioDevice || _allowSpeakWithoutAudioDevice);
}
```

### Potential Improvements

#### Enhancement 1: More Visible Toggle

Make the toggle more prominent - users may not know it exists in a popup menu.

**Add to AppBar actions:**

```dart
actions: [
  // Direct toggle button (no popup)
  IconButton(
    icon: Icon(
      _autoSpeak ? Icons.volume_up : Icons.volume_off,
      color: _autoSpeak ? Colors.greenAccent : Colors.grey,
    ),
    onPressed: () {
      setState(() {
        _autoSpeak = !_autoSpeak;
      });
      _saveUserPreferences();
      showGlassSnackBar(
        context,
        message: _autoSpeak
            ? 'Alice voice enabled'
            : 'Alice voice disabled',
        icon: _autoSpeak ? Icons.volume_up : Icons.volume_off,
      );
    },
    tooltip: 'Toggle Alice voice',
  ),
  // ... existing popup menu for advanced options
]
```

#### Enhancement 2: Show Status in Chat

Add a subtle indicator when TTS is active/muted.

---

## Issue 3: White Screen After App Idle

### Current Behavior

- App shows white screen for 2-3 seconds after being idle
- Then suddenly loads

### Root Cause

Flutter's default splash screen may have expired, but main UI isn't ready yet.

### Diagnosis

**Check these files:**

1. `ios/Runner/Assets.xcassets/LaunchImage.imageset/` - iOS splash
2. `ios/Runner/Base.lproj/LaunchScreen.storyboard` - iOS launch screen
3. `android/app/src/main/res/drawable/launch_background.xml` - Android splash

### Fixes

#### Fix 1: Extend Splash Screen Duration

**File:** `ios/Runner/AppDelegate.swift`

Add a loading indicator that persists until app is ready:

```swift
override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
) -> Bool {
    // Show loading indicator
    if let controller = window?.rootViewController as? FlutterViewController {
        let loadingView = createLoadingView()
        controller.view.addSubview(loadingView)
        loadingView.tag = 999 // For later removal

        // Remove after Flutter signals ready
        controller.engine?.binaryMessenger
            .setMessageHandlerOnChannel("app/ready") { _, _ in
                loadingView.removeFromSuperview()
            }
    }

    // ... rest of initialization
}

private func createLoadingView() -> UIView {
    let view = UIView(frame: UIScreen.main.bounds)
    view.backgroundColor = .black

    let activityIndicator = UIActivityIndicatorView(style: .large)
    activityIndicator.color = .white
    activityIndicator.center = view.center
    activityIndicator.startAnimating()
    view.addSubview(activityIndicator)

    return view
}
```

#### Fix 2: Signal Ready from Flutter

**File:** `lib/main.dart`

Add after all initialization completes:

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // ... all initialization ...

  runApp(const GitFitApp());

  // Signal native side that Flutter is ready
  WidgetsBinding.instance.addPostFrameCallback((_) {
    const MethodChannel('app/ready').invokeMethod('ready');
  });
}
```

#### Fix 3: Use flutter_native_splash Package

**pubspec.yaml:**

```yaml
dependencies:
  flutter_native_splash: ^2.3.10

flutter_native_splash:
  color: "#000000"
  image: assets/images/logo.png
  android: true
  ios: true
  web: false
  android_gravity: center
  ios_content_mode: center
```

Then run:

```bash
dart run flutter_native_splash:create
```

---

## Issue 4: OAuth Doesn't Auto-Return to App

### Current Implementation

**File:** `lib/main.dart:126-138`

```dart
// Set up deep link handling for OAuth callbacks
if (!kIsWeb && (Platform.isIOS || Platform.isAndroid)) {
  try {
    final appLinks = AppLinks();
    appLinks.uriLinkStream.listen((uri) {
      debugPrint('[main] Deep link received: $uri');
      // Supabase will automatically handle the OAuth callback
      Supabase.instance.client.auth.getSessionFromUrl(uri);
    });
  } catch (e) {
    debugPrint('[main] Failed to initialize AppLinks: $e');
  }
}
```

### Problem

The deep link handler is set up, but there are several potential issues:

#### Issue A: Deep Link Not Registered Properly

**Check:** `ios/Runner/Info.plist`

Should have:

```xml
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleTypeRole</key>
    <string>Editor</string>
    <key>CFBundleURLName</key>
    <string>biz.lsctech.adaptivefit</string>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>adaptivefit</string>
      <string>biz.lsctech.adaptivefit</string>
    </array>
  </dict>
</array>
```

#### Issue B: Supabase Redirect URL Mismatch

**Check Supabase Dashboard:**

1. Go to Authentication → URL Configuration
2. Ensure these redirect URLs are added:
   - `adaptivefit://auth-callback`
   - `biz.lsctech.adaptivefit://auth-callback`

**Check:** `lib/features/auth/presentation/login_screen.dart:167`

```dart
final redirectTo = isMobilePlatform
    ? 'biz.lsctech.adaptivefit://auth-callback'
    : null;
```

#### Issue C: Deep Link Timing Issue

The listener is set up in `main()`, but if OAuth completes before the listener is ready, the callback is missed.

### Fixes

#### Fix 1: Handle Deep Link in AppDelegate

**File:** `ios/Runner/AppDelegate.swift`

Add URL handling:

```swift
override func application(
    _ app: UIApplication,
    open url: URL,
    options: [UIApplication.OpenURLOptionsKey: Any] = [:]
) -> Bool {
    print("[AppDelegate] Deep link opened: \(url)")

    // Store URL for Flutter to process
    if let controller = window?.rootViewController as? FlutterViewController {
        let channel = FlutterMethodChannel(
            name: "app_links",
            binaryMessenger: controller.binaryMessenger
        )
        channel.invokeMethod("onUrl", arguments: url.absoluteString)
    }

    return super.application(app, open: url, options: options)
}
```

#### Fix 2: Navigate After OAuth Success

**File:** `lib/main.dart`

Update deep link handler to navigate:

```dart
appLinks.uriLinkStream.listen((uri) async {
  debugPrint('[main] Deep link received: $uri');

  try {
    // Process OAuth callback
    await Supabase.instance.client.auth.getSessionFromUrl(uri);

    // Get current session
    final session = Supabase.instance.client.auth.currentSession;

    if (session != null) {
      debugPrint('[main] OAuth success - session established');
      // Navigate to home screen (or wherever appropriate)
      // This requires access to Navigator, so might need to be in a widget
    } else {
      debugPrint('[main] OAuth callback processed but no session');
    }
  } catch (e) {
    debugPrint('[main] OAuth callback error: $e');
  }
});
```

#### Fix 3: Add AuthStateChange Listener

**File:** `lib/main.dart` or root widget

```dart
Supabase.instance.client.auth.onAuthStateChange.listen((data) {
  final event = data.event;
  debugPrint('[Auth] State changed: $event');

  if (event == AuthChangeEvent.signedIn) {
    debugPrint('[Auth] User signed in - navigating to home');
    // Navigate to home screen
    // This requires BuildContext, so implement in a StatefulWidget
  }
});
```

---

## Testing Checklist

### Kokoro TTS

- [ ] Check ONNX Runtime framework is linked
- [ ] Verify asset files exist in correct location
- [ ] Run app and check console for `[KokoroTts]` logs
- [ ] Test speak method returns detailed error if fails
- [ ] Test tokenizer.json loading

### TTS Toggle

- [ ] Verify volume icon shows current state
- [ ] Test toggle changes state and saves preference
- [ ] Test "allow without device" option works
- [ ] Consider adding more prominent toggle button

### White Screen

- [ ] Implement extended splash screen
- [ ] Test app cold start
- [ ] Test app resume from background after idle
- [ ] Verify smooth transition to main UI

### OAuth Redirect

- [ ] Verify deep link URLs in Info.plist
- [ ] Verify redirect URLs in Supabase dashboard
- [ ] Test OAuth flow end-to-end
- [ ] Verify app returns automatically after auth
- [ ] Check console for deep link logs

---

## Quick Wins (Implement First)

1. **Add Kokoro Diagnostics** - Show why TTS fails in UI
2. **Make TTS Toggle More Obvious** - Users don't know it exists
3. **Fix OAuth Navigation** - Add auth state listener to auto-navigate
4. **Add Loading Indicator** - Remove white screen with flutter_native_splash

---

## Next Steps

1. Run diagnostics to confirm root cause of Kokoro failure
2. Implement enhanced logging
3. Test each fix individually
4. Document results and update as needed

## Related
