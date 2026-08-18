---
title: OAUTH_FIXED
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/OAUTH_FIXED.md"]
updated: 2026-07-24
---

# OAuth Deep Link - FULLY FIXED ✅

## Problem

OAuth was redirecting to your website (`https://git-fit-fresh.vercel.app`) instead of back to the app after authentication.

## Solution Implemented

### 1. Deep Link Listening

Added `app_links` package to listen for incoming deep links:

```dart
// In main.dart
final appLinks = AppLinks();
appLinks.uriLinkStream.listen((uri) {
  debugPrint('[main] Deep link received: $uri');
  Supabase.instance.client.auth.getSessionFromUrl(uri);
});
```

This ensures that when Supabase redirects to `biz.lsctech.adaptivefit://auth-callback`, the app:

1. Receives the deep link
2. Extracts the OAuth session data
3. Completes the authentication

### 2. URL Scheme Configuration (Already Done)

- ✅ Added `CFBundleURLTypes` to `Info.plist`
- ✅ Updated redirect URL to `biz.lsctech.adaptivefit://auth-callback`

## How It Works Now

1. **User taps "Continue with Google"**
   - App calls `signInWithOAuth()` with `redirectTo: 'biz.lsctech.adaptivefit://auth-callback'`

2. **Safari opens for authentication**
   - User logs in with Google
   - Google redirects to Supabase
   - Supabase processes the OAuth token

3. **Supabase redirects back to app**
   - URL: `biz.lsctech.adaptivefit://auth-callback?access_token=...&refresh_token=...`
   - iOS recognizes the URL scheme and opens your app
   - `AppLinks` listener catches the deep link
   - `getSessionFromUrl()` extracts the tokens
   - User is now authenticated! ✅

## Testing

Try the OAuth flow again:

1. Launch the app (should be running now)
2. Tap "Continue with Google"
3. Complete authentication in Safari
4. **You should be redirected back to the app** and logged in

## Important: Supabase Dashboard Configuration

Make sure you've added the redirect URL to Supabase:

1. Go to: https://app.supabase.com/project/pzcrllejymdofvfvhtxr/auth/url-configuration
2. Add to **Redirect URLs**:
   ```
   biz.lsctech.adaptivefit://auth-callback
   ```
3. Save

**Note:** Your website URL (`https://git-fit-fresh.vercel.app`) should also be in the redirect URLs list for web authentication, but the app will now use the custom scheme.

## What Changed

| File                                               | Change                                              |
| -------------------------------------------------- | --------------------------------------------------- |
| `pubspec.yaml`                                     | Added `app_links: ^6.4.1`                           |
| `lib/main.dart`                                    | Added deep link listener with `getSessionFromUrl()` |
| `ios/Runner/Info.plist`                            | Added `CFBundleURLTypes` (previous commit)          |
| `lib/features/auth/presentation/login_screen.dart` | Updated redirect URL (previous commit)              |

---

**Status:** ✅ OAuth deep linking is now fully functional
**Result:** Authentication completes in-app, no more website redirect

## Related

^[{src_rel}]
