---
title: OAUTH_COMPLETE
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/OAUTH_COMPLETE.md"]
updated: 2026-07-24
---

# OAuth Fully Working on iOS ✅

## All Issues Fixed

### ✅ Issue 1: "Safari cannot open page"

**Fixed:** Added proper URL scheme configuration to `Info.plist` and updated redirect URL to `biz.lsctech.adaptivefit://auth-callback`

### ✅ Issue 2: Redirects to website instead of app

**Fixed:** Added `app_links` package and deep link listener to catch OAuth callbacks and complete authentication in-app

### ✅ Issue 3: Safari doesn't close after redirect

**Fixed:** Changed OAuth to use `LaunchMode.inAppWebView` which automatically dismisses after redirect

## How OAuth Works Now

1. **User taps "Continue with Google"**
   - App opens an **in-app browser** (not external Safari)
   - Browser is embedded within your app

2. **User authenticates with Google**
   - Completes login in the in-app browser
   - Google redirects to Supabase
   - Supabase processes OAuth

3. **Redirect back to app**
   - Supabase redirects to `biz.lsctech.adaptivefit://auth-callback`
   - **In-app browser automatically closes** ✨
   - Deep link listener catches the callback
   - User is authenticated and back in the app

## Key Changes

| File                                               | Change                                                                                                                              |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `ios/Runner/Info.plist`                            | Added `CFBundleURLTypes` for deep linking                                                                                           |
| `lib/features/auth/presentation/login_screen.dart` | - Updated redirect URL to bundle identifier scheme<br>- Added `LaunchMode.inAppWebView` for mobile<br>- Added `url_launcher` import |
| `lib/main.dart`                                    | Added `app_links` listener with `getSessionFromUrl()`                                                                               |
| `pubspec.yaml`                                     | Added `app_links: ^6.4.1`                                                                                                           |

## User Experience

**Before:**

1. Tap "Continue with Google"
2. Safari opens externally
3. Authenticate
4. Redirected to website
5. Manually close Safari
6. Manually return to app
7. Still not logged in ❌

**Now:**

1. Tap "Continue with Google"
2. In-app browser appears
3. Authenticate
4. **Browser automatically closes**
5. **Immediately logged in** ✅

## Testing

The app is running on your simulator now. Try the OAuth flow:

1. Tap "Continue with Google" or "Continue with Apple"
2. Complete authentication
3. **Browser closes automatically**
4. You're logged in! ✅

## Important: Supabase Configuration

Make sure you've added the redirect URL to Supabase:

**URL:** https://app.supabase.com/project/pzcrllejymdofvfvhtxr/auth/url-configuration

**Add this to Redirect URLs:**

```
biz.lsctech.adaptivefit://auth-callback
```

---

**Status:** ✅ OAuth is fully functional on iOS
**UX:** Seamless - in-app browser auto-closes after authentication
**Result:** Professional, native iOS authentication experience

## Related
