---
title: LOGIN_SCREEN_REDIRECT
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/LOGIN_SCREEN_REDIRECT.md"]
updated: 2026-07-24
---

# Login Screen Redirect Implementation

## Overview

The app already has robust authentication logic that automatically redirects users to the login screen when they are signed out. This ensures users cannot access protected content without proper authentication.

## Current Implementation

### AppRouter Authentication Flow

**Location**: `lib/routes/app_router.dart`

The authentication logic is implemented in the `AppRouter` widget:

```dart
@override
Widget build(BuildContext context) {
  // Check if Supabase is initialized before accessing it
  final client = safeSupabaseClient();

  if (client == null) {
    // Supabase is not initialized - show appropriate login screen
    debugPrint('[AppRouter] Supabase not initialized - showing login screen');
    return _isDesktop ? const DesktopPairingScreen() : const LoginScreen();
  }

  return StreamBuilder<AuthState>(
    stream: client.auth.onAuthStateChange,
    builder: (context, snapshot) {
      final session = snapshot.data?.session ?? client.auth.currentSession;

      if (snapshot.connectionState == ConnectionState.waiting && session == null) {
        return const Scaffold(body: Center(child: CircularProgressIndicator()));
      }

      if (session == null) {
        return _isDesktop ? const DesktopPairingScreen() : const LoginScreen();
      }

      final userId = session.user.id;
      return _PostAuthRouter(userId: userId);
    },
  );
}
```

### Key Features

#### 1. **Real-time Authentication Monitoring**

- **Stream-based**: Uses `client.auth.onAuthStateChange` stream
- **Immediate response**: Reacts instantly to authentication state changes
- **Connection state handling**: Shows loading indicator while checking auth state

#### 2. **Platform-Specific Login Screens**

- **Desktop apps**: Uses `DesktopPairingScreen`
- **Mobile apps**: Uses `LoginScreen`
- **Automatic detection**: `_isDesktop` flag determines appropriate screen

#### 3. **Session Validation**

- **Null session check**: `if (session == null)` redirects to login
- **Session persistence**: Uses `client.auth.currentSession` as fallback
- **User ID extraction**: `session.user.id` for authenticated users

#### 4. **Supabase Initialization Check**

- **Safe client access**: `safeSupabaseClient()` handles null cases
- **Configuration validation**: Checks if SUPABASE_URL and SUPABASE_ANON_KEY are set
- **Graceful fallback**: Shows login screen if Supabase is not initialized

## Authentication Flow

### 1. **App Startup**

```
App starts → AppRouter.build() → Check Supabase client → StreamBuilder<AuthState>
```

### 2. **Authentication State Change**

```
AuthStateChange event → StreamBuilder rebuild → Check session → Route accordingly
```

### 3. **User Signs Out**

```
Sign out action → AuthStateChange (null session) → StreamBuilder → LoginScreen
```

### 4. **User Signs In**

```
Sign in action → AuthStateChange (valid session) → StreamBuilder → _PostAuthRouter
```

## Platform-Specific Behavior

### Desktop Apps (macOS, Windows, Linux)

- **Login screen**: `DesktopPairingScreen`
- **Pairing flow**: Device code authentication
- **Trainer access**: Specialized trainer authentication

### Mobile Apps (iOS, Android)

- **Login screen**: `LoginScreen`
- **Email/password**: Traditional authentication
- **Social login**: Apple, Google, etc.

## Post-Authentication Routing

### \_PostAuthRouter

Once authenticated, users are routed to `_PostAuthRouter` which handles:

1. **AI Bootstrap Check**: Determines if AI model loading is needed
2. **Desktop Role Check**: Validates trainer permissions for desktop app
3. **Onboarding**: Shows onboarding if first-time user
4. **Main App**: Routes to appropriate main screen

```dart
class _PostAuthRouter extends StatefulWidget {
  const _PostAuthRouter({required this.userId});
  final String userId;

  @override
  void initState() {
    super.initState();
    _needsBootstrapFuture = _needsAiBootstrap();
    if (_isDesktop) {
      _desktopRoleCheckFuture = _fetchAppUser();
    }
  }
}
```

## Edge Cases Handled

### 1. **Supabase Not Initialized**

- **Cause**: Missing or invalid environment variables
- **Behavior**: Shows login screen with debug message
- **Debug output**: `[AppRouter] Supabase not initialized - showing login screen`

### 2. **Network Issues**

- **Behavior**: Shows loading indicator during connection state
- **Fallback**: Uses cached session if available
- **Timeout**: Eventually shows login screen if no session

### 3. **Session Expiration**

- **Automatic**: Supabase handles token refresh automatically
- **Manual**: If refresh fails, triggers sign-out flow
- **Redirect**: User is sent to login screen

### 4. **App Background/Foreground**

- **Session persistence**: Supabase maintains session across app lifecycle
- **Automatic reconnection**: StreamBuilder handles reconnection
- **State consistency**: UI updates appropriately

## Security Considerations

### 1. **Protected Routes**

All protected routes are wrapped by `AppRouter`, ensuring authentication check before any content is displayed.

### 2. **Session Validation**

- **Token verification**: Supabase validates JWT tokens
- **Session refresh**: Automatic token refresh prevents expired sessions
- **Revocation handling**: Revoked sessions are immediately invalidated

### 3. **Platform Security**

- **Desktop pairing**: Secure device code authentication
- **Mobile authentication**: Standard email/password with social login
- **Cross-platform consistency**: Same authentication backend for all platforms

## Testing Authentication Flow

### Manual Testing Checklist

- [ ] App starts with no user signed in → Shows login screen
- [ ] User signs in successfully → Routes to main app
- [ ] User signs out → Returns to login screen
- [ ] App closed and reopened with valid session → Stays logged in
- [ ] App closed and reopened with expired session → Shows login screen
- [ ] Network issues during startup → Shows loading, then appropriate screen
- [ ] Supabase not configured → Shows login screen with debug message

### Automated Testing

```dart
testWidgets('AppRouter redirects to login when signed out', (WidgetTester tester) async {
  // Mock Supabase client with null session
  when(mockSupabaseClient.auth.currentSession).thenReturn(null);

  await tester.pumpWidget(AppRouter());

  // Should show login screen
  expect(find.byType(LoginScreen), findsOneWidget);
});

testWidgets('AppRouter shows main app when signed in', (WidgetTester tester) async {
  // Mock Supabase client with valid session
  final mockSession = MockSession();
  when(mockSupabaseClient.auth.currentSession).thenReturn(mockSession);
  when(mockSupabaseClient.auth.onAuthStateChange).thenAnswer(
    Stream.value(AuthState(authResponse: mockSession)),
  );

  await tester.pumpWidget(AppRouter());

  // Should show post-auth router
  expect(find.byType(_PostAuthRouter), findsOneWidget);
});
```

## Configuration Requirements

### Environment Variables

Required in `.env` file:

```env
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-anon-key
```

### Dependencies

```yaml
dependencies:
  supabase_flutter: ^1.10.0
  flutter:
    sdk: flutter
```

## Troubleshooting

### Common Issues

#### 1. **Stuck on Loading Screen**

- **Cause**: Network issues or Supabase configuration problems
- **Solution**: Check network connectivity and environment variables
- **Debug**: Look for `[AppRouter] Supabase not initialized` message

#### 2. **Login Screen Not Showing**

- **Cause**: Authentication state not properly monitored
- **Solution**: Verify StreamBuilder is properly connected to auth stream
- **Debug**: Check console for authentication errors

#### 3. **Infinite Loading Loop**

- **Cause**: Connection state never resolves
- **Solution**: Check internet connection and Supabase service status
- **Debug**: Monitor `snapshot.connectionState` values

#### 4. **Desktop App Shows Mobile Login**

- **Cause**: Platform detection not working correctly
- **Solution**: Verify `_isDesktop` logic for your platform
- **Debug**: Check `Platform.isMacOS`, `Platform.isWindows`, `Platform.isLinux`

## Future Enhancements

### Phase 1: Enhanced Error Handling

- Better error messages for specific auth failures
- Network connectivity indicators
- Retry mechanisms for failed auth attempts

### Phase 2: Biometric Authentication

- Face ID / Touch ID integration
- Platform-specific biometric flows
- Fallback to traditional auth

### Phase 3: Multi-Device Sync

- Session synchronization across devices
- Device management interface
- Remote sign-out capabilities

## Conclusion

The authentication system is already robustly implemented with automatic login screen redirection. The `AppRouter` widget ensures that users are always properly authenticated before accessing any protected content, providing a secure and seamless user experience across all platforms.

### Key Benefits

- ✅ **Automatic redirect** to login screen when signed out
- ✅ **Real-time monitoring** of authentication state
- ✅ **Platform-specific** login screens for optimal UX
- ✅ **Graceful handling** of network and configuration issues
- ✅ **Secure routing** that protects all app content

The implementation follows best practices for authentication flow and provides a solid foundation for secure user access control.

## Related

^[{src_rel}]
