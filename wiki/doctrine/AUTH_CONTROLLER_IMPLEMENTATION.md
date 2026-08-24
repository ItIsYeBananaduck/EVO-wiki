---
title: AUTH_CONTROLLER_IMPLEMENTATION
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/AUTH_CONTROLLER_IMPLEMENTATION.md
updated: 2026-07-24
---

# AuthController Implementation

## Problem Solved

The app had an intermittent bug where returning users would hang indefinitely on the loading screen. The root cause was:

1. **Race conditions**: Multiple components reading `supabase.auth.currentSession` directly
2. **No timeout protection**: Session refresh could hang forever
3. **Concurrent initialization**: Splash screen + router + retry button could initialize auth simultaneously
4. **No deterministic state**: UI relied on Supabase's internal state which could be inconsistent

## Solution Overview

AuthController provides deterministic auth state management with these guarantees:

- **Never hangs**: Always resolves to `signedIn` or `signedOut` within 10 seconds
- **Single source of truth**: UI reads `AuthController.state`, never Supabase directly
- **Race condition protection**: Prevents concurrent initialization
- **Time-boxed fallback**: 10-second timeout with safe fallback logic

## Architecture

### Core Components

#### 1. AuthState Enum

```dart
enum AuthState {
  loading,    // Temporary state during initialization
  signedIn,   // User authenticated, session valid
  signedOut,  // No valid session
}
```

#### 2. AuthController

- Owns auth state exclusively
- Single auth listener (`onAuthStateChange`)
- Time-boxed initialization (10 seconds)
- Concurrent init protection
- Automatic fallback logic

#### 3. UI Integration

- AppRouter uses `Consumer<AuthController>` for reactive updates
- Loading screen with retry button
- No direct Supabase session access in UI

### Key Features

#### Time-Boxed Initialization

```dart
// 10-second timeout fallback
Timer(const Duration(seconds: 10), () {
  if (_state == AuthState.loading) {
    _forceResolution();
  }
});
```

#### Concurrent Init Protection

```dart
if (_isInitializing) {
  debugPrint('[AuthController] initAuth: already initializing, skipping');
  return;
}
```

#### Single Auth Listener

```dart
_authSubscription ??= client.auth.onAuthStateChange.listen(_handleAuthStateChange);
```

#### Fallback Logic

1. Try `refreshSession()` once
2. If still unresolved, force `signedOut`
3. UI always gets deterministic state

## File Structure

```
lib/features/auth/
├── domain/
│   ├── auth_controller.dart      # Main AuthController implementation
│   └── app_user.dart            # User model (unchanged)
└── presentation/
    ├── auth_loading_screen.dart    # Loading screen with retry
    ├── auth_retry_button.dart      # Reusable retry component
    ├── login_screen.dart           # Login screen (unchanged)
    ├── desktop_pairing_screen.dart # Desktop pairing (unchanged)
    └── trainer_required_screen.dart # Trainer guard (unchanged)
```

## Integration Points

### main.dart Changes

```dart
// Initialize AuthController globally
authController = AuthController();

// Provider setup
ChangeNotifierProvider<AuthController>(
  create: (_) => authController,
  child: MaterialApp(...),
);

// Early initialization
WidgetsBinding.instance.addPostFrameCallback((_) {
  authController.initAuth();
});
```

### AppRouter Changes

```dart
// Before: Direct Supabase access
StreamBuilder<AuthState>(
  stream: client.auth.onAuthStateChange,
  builder: (context, snapshot) {
    final session = snapshot.data?.session ?? client.auth.currentSession;
    // ... routing logic
  },
)

// After: AuthController-driven
Consumer<AuthController>(
  builder: (context, authController, child) {
    switch (authController.state) {
      case AuthState.loading:
        return const CircularProgressIndicator();
      case AuthState.signedOut:
        return const LoginScreen();
      case AuthState.signedIn:
        return _PostAuthRouter(userId: userId);
    }
  },
)
```

## Why This Prevents Auth Hangs

### 1. Deterministic State Machine

- **Before**: UI read Supabase's internal state (could be inconsistent)
- **After**: UI reads AuthController.state (always deterministic)

### 2. Timeout Protection

- **Before**: Session refresh could hang indefinitely
- **After**: 10-second timeout forces resolution

### 3. Race Condition Elimination

- **Before**: Multiple components could initialize auth simultaneously
- **After**: `_isInitializing` guard prevents concurrent calls

### 4. Single Listener

- **Before**: Multiple auth listeners could cause inconsistent state
- **After**: Single listener with centralized state management

### 5. Safe Fallback

- **Before**: No fallback when refresh failed
- **After**: Automatic fallback to `signedOut` with retry option

## Usage Examples

### Basic Usage in UI

```dart
Consumer<AuthController>(
  builder: (context, authController, child) {
    if (authController.isLoading) {
      return const CircularProgressIndicator();
    } else if (authController.isSignedIn) {
      return const HomeScreen();
    } else {
      return const LoginScreen();
    }
  },
)
```

### Manual Retry

```dart
// From retry button
await context.read<AuthController>().retry();

// From custom logic
await authController.retry();
```

### Sign Out

```dart
// Deterministic sign out
context.read<AuthController>().signOut();
```

## Testing Strategy

### Unit Tests

- Test state transitions
- Test timeout behavior
- Test concurrent init protection
- Test fallback logic

### Integration Tests

- Test app startup flow
- Test retry functionality
- Test OAuth callback handling
- Test network failure scenarios

## Migration Guide

### For Existing Code

1. Replace direct `supabase.auth.currentSession` reads with `AuthController.state`
2. Replace `StreamBuilder<AuthState>` with `Consumer<AuthController>`
3. Remove duplicate auth listeners
4. Add retry buttons to loading screens

### Common Patterns

#### Before (Problematic)

```dart
final session = supabase.auth.currentSession;
if (session == null) {
  return LoginScreen();
}
```

#### After (Deterministic)

```dart
final authState = context.watch<AuthController>().state;
if (authState == AuthState.signedOut) {
  return LoginScreen();
}
```

## Performance Considerations

- **Single listener**: Reduces overhead vs multiple listeners
- **Lazy initialization**: AuthController created once, reused
- **Efficient state updates**: Only notifies when state actually changes
- **Timeout cleanup**: Timer cancelled when state resolves

## Debugging

### Logging

AuthController includes comprehensive logging:

```dart
debugPrint('[AuthController] State transition: loading -> signedIn');
debugPrint('[AuthController] initAuth: TIMEOUT - forcing resolution');
```

### Common Issues

1. **Stuck in loading**: Check network connectivity, timeout will resolve
2. **Immediate signedOut**: Check Supabase configuration in .env
3. **OAuth not working**: Verify deep link configuration

## Future Enhancements

1. **Offline support**: Cache auth state for offline usage
2. **Biometric auth**: Extend AuthController for local authentication
3. **Session warnings**: Warn users before session expires
4. **Analytics**: Track auth state transitions for debugging

## Conclusion

AuthController eliminates the auth hang bug by providing deterministic, time-boxed authentication state management. The key insight is that UI should never depend directly on Supabase's internal state - instead, it should read from a controlled state machine that guarantees resolution.

This implementation provides:

- **Reliability**: Never hangs indefinitely
- **Predictability**: Always resolves to a terminal state
- **User experience**: Retry button for manual intervention
- **Maintainability**: Centralized auth logic
- **Testability**: Clear state machine for testing

## Related

^[source-materials/mirrors/doctrine/AUTH_CONTROLLER_IMPLEMENTATION.md]
