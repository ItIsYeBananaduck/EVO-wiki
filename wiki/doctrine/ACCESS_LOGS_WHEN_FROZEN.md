---
title: ACCESS_LOGS_WHEN_FROZEN
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/ACCESS_LOGS_WHEN_FROZEN.md"]
updated: 2026-07-24
---

# Accessing Logs When App is Frozen

If the app freezes before you can interact with it, here are ways to access the logs:

## Method 1: Automatic Log Viewer (NEW)

The app now **automatically shows the log viewer** on startup if there are recent errors or crashes.

**What happens:**

- App starts normally
- After 3 seconds, if errors are detected, log viewer opens automatically
- You can see logs even if the app freezes after that

## Method 2: Files App (iOS)

Logs are now copied to a **shareable location** that you can access via the **Files app**:

1. **Open Files app** on your iPhone
2. **Tap "On My iPhone"** (or "Browse")
3. **Look for your app's folder** (EVOtraining or similar)
4. **Find these files:**
   - `crash_logs.txt` - Full logs
   - `CRASH_LOG_SUMMARY.txt` - Summary

**Location in Files app:**

```
On My iPhone → [Your App] → crash_logs.txt
```

## Method 3: Share Sheet (If App Partially Works)

If the app starts but freezes later:

1. **Wait for automatic log viewer** (opens after 3 seconds if errors detected)
2. **Or triple-tap quickly** before it freezes
3. **Copy logs** using the copy button
4. **Paste in Notes app** or email

## Method 4: System Crash Logs

If the app completely crashes (not just freezes):

1. **Settings → Privacy & Security → Analytics & Improvements → Analytics Data**
2. **Look for entries starting with "EVOtraining" or your app name**
3. **Tap to view crash report**

## Method 5: TestFlight Crash Reports

If using TestFlight:

1. **App Store Connect → TestFlight → Crashes**
2. **Download crash reports**

## What the Stack Trace Shows

Your stack trace shows:

```
#14 0x0000000104c134a4 in specialized LlamaEngine._processGeneration(...)
```

This means the freeze is happening **during generation** (when processing a chat request), not during initialization.

**Key points:**

- The app loaded successfully (model loaded, context created)
- The freeze happens when trying to generate a response
- This is likely in the decode loop or token generation

## Next Steps

1. **Check automatic log viewer** - It should open automatically on next startup
2. **Check Files app** - Logs are copied there every 5 seconds
3. **Look for these in logs:**
   - `_processGeneration` entries
   - `safeDecode` calls
   - `llama_decode` calls
   - Any `TIMED OUT` or `INVARIANT_VIOLATION` messages

## If App Freezes Immediately

If the app freezes **immediately on startup** (before log viewer can open):

1. **Force quit the app**
2. **Open Files app**
3. **Navigate to app folder**
4. **Read `CRASH_LOG_SUMMARY.txt`** - This is updated every 10 seconds and contains recent entries

The summary file is your best bet because:

- It's updated automatically
- It contains the most recent entries
- It's easier to read than full logs
- It works even if app is frozen

## Related

^[{src_rel}]
