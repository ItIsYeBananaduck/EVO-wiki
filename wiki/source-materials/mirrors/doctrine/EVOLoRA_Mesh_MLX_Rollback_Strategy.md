---
title: EVOLoRA_Mesh_MLX_Rollback_Strategy
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-deprecated/EVOLoRA_Mesh_MLX_Rollback_Strategy.md"]
updated: 2026-07-24
---

# EVOLoRA Mesh MLX/iOS Rollback Strategy

**Platform**: iOS (MLX format - safetensors)
**Challenge**: iOS uses safetensors directly (no conversion), but still needs rollback capability if adapter fails to load or causes issues.

---

## MLX/iOS Adapter Storage

### Storage Structure

```
AliceAssets/adapters/user/
├── user_lora.safetensors          ← Current (active)
└── user_lora.safetensors.backup    ← Previous version (for rollback)
```

**Key Difference from Android**:

- iOS: Safetensors only (no GGUF conversion)
- Simpler: Just backup/restore safetensors file

---

## Rollback Scenarios

### Scenario 1: Adapter Fails to Load

**Problem**: New adapter file is corrupt or incompatible

**Detection**:

- MLX engine fails to load adapter
- Returns error or throws exception

**Rollback**:

- Restore from `.backup` file
- Retry loading with backup adapter

### Scenario 2: Adapter Causes Inference Errors

**Problem**: Adapter loads but causes runtime errors during inference

**Detection**:

- Inference fails repeatedly
- Error rate increases
- Model crashes

**Rollback**:

- Mark current adapter as problematic
- Restore from `.backup`
- Continue with previous adapter

### Scenario 3: Adapter Validation Fails

**Problem**: Adapter file is invalid (wrong format, missing metadata)

**Detection**:

- Validate adapter before loading
- Check file integrity, format, size

**Rollback**:

- Don't load invalid adapter
- Restore from `.backup` if current is invalid

---

## Implementation

### LoRA Adapter Manager (iOS)

**File**: `flutter_app/lib/features/alice/domain/lora_adapter_manager.dart` (MODIFY)

```dart
class LoRAAdapterManager {
  /// Stores a LoRA adapter file with backup/versioning (iOS safetensors).
  Future<void> storeAdapter(
    LoRAKind kind,
    Uint8List data, {
    String? trainerId,
    bool createBackup = true,
  }) async {
    final String path = getAdapterPath(kind, trainerId: trainerId);

    // For iOS: Use safetensors format
    final String format = Platform.isIOS ? 'safetensors' : 'gguf';
    final String fullPath = path.replaceAll('.gguf', '.$format');

    // Step 1: Create backup of current adapter (if exists)
    if (createBackup && await adapterExists(kind, trainerId: trainerId)) {
      await _createBackup(fullPath);
    }

    // Step 2: Write new adapter
    if (_useSharedModelStoreSaf) {
      final String relativePath = p.relative(fullPath, from: baseDir);
      await sharedModelStore!.writeBytes(
        relativePath,
        data,
        mime: 'application/octet-stream',
        overwrite: true,
      );
    } else {
      final File file = File(fullPath);
      if (!await file.parent.exists()) {
        await file.parent.create(recursive: true);
      }
      await file.writeAsBytes(data, flush: true);
    }

    debugPrint('[LoRAAdapterManager] Stored ${kind.key} adapter at $fullPath');
  }

  /// Creates a backup of the current adapter (iOS safetensors).
  Future<void> _createBackup(String currentPath) async {
    final String backupPath = '$currentPath.backup';

    try {
      if (_useSharedModelStoreSaf) {
        final String relativeCurrent = p.relative(currentPath, from: baseDir);
        final String relativeBackup = p.relative(backupPath, from: baseDir);

        // Read current
        final Uint8List? data = await sharedModelStore!.readFileAsBytes(relativeCurrent);
        if (data != null) {
          // Delete old backup if exists
          await sharedModelStore!.deleteFileIfExists(relativeBackup);

          // Write backup
          await sharedModelStore!.writeBytes(
            relativeBackup,
            data,
            mime: 'application/octet-stream',
            overwrite: true,
          );
        }
      } else {
        final File currentFile = File(currentPath);
        final File backupFile = File(backupPath);

        if (await currentFile.exists()) {
          // Delete old backup if exists
          if (await backupFile.exists()) {
            await backupFile.delete();
          }

          // Copy current to backup
          await currentFile.copy(backupPath);
        }
      }

      debugPrint('[LoRAAdapterManager] Created backup: $backupPath');
    } catch (e) {
      debugPrint('[LoRAAdapterManager] WARNING: Failed to create backup: $e');
      // Continue anyway - backup is optional
    }
  }

  /// Restores adapter from backup (iOS safetensors).
  Future<bool> restoreFromBackup(LoRAKind kind, {String? trainerId}) async {
    final String path = getAdapterPath(kind, trainerId: trainerId);
    final String format = Platform.isIOS ? 'safetensors' : 'gguf';
    final String fullPath = path.replaceAll('.gguf', '.$format');
    final String backupPath = '$fullPath.backup';

    try {
      if (_useSharedModelStoreSaf) {
        final String relativeBackup = p.relative(backupPath, from: baseDir);
        final String relativePath = p.relative(fullPath, from: baseDir);

        if (!await sharedModelStore!.fileExists(relativeBackup)) {
          return false; // No backup available
        }

        // Read backup
        final Uint8List? data = await sharedModelStore!.readFileAsBytes(relativeBackup);
        if (data != null) {
          // Restore backup
          await sharedModelStore!.writeBytes(
            relativePath,
            data,
            mime: 'application/octet-stream',
            overwrite: true,
          );
          return true;
        }
      } else {
        final File backupFile = File(backupPath);
        final File currentFile = File(fullPath);

        if (!await backupFile.exists()) {
          return false; // No backup available
        }

        // Restore backup
        await backupFile.copy(fullPath);
        return true;
      }
    } catch (e) {
      debugPrint('[LoRAAdapterManager] ERROR: Failed to restore from backup: $e');
      return false;
    }

    return false;
  }
}
```

---

## MLX Engine Integration

### MLX Engine (iOS) - Adapter Loading with Rollback

**File**: `flutter_app/ios/Runner/MLXEngine.swift` (NEW or MODIFY)

```swift
@objc class MLXEngine: NSObject {
    @objc static let shared = MLXEngine()

    private var loadedAdapters: [String: Any] = [:]

    /// Load LoRA adapter with rollback capability.
    @objc func loadLoRAAdapter(
        path: String,
        scale: Float,
        kind: String,
        withRollback: Bool = true
    ) -> Bool {
        // Step 1: Validate adapter file exists and is readable
        guard FileManager.default.fileExists(atPath: path) else {
            print("[MLXEngine] ERROR: Adapter file not found: \(path)")
            if withRollback {
                return _tryRestoreFromBackup(path: path, kind: kind)
            }
            return false
        }

        // Step 2: Validate adapter file (check size, format)
        guard _validateAdapterFile(path: path) else {
            print("[MLXEngine] ERROR: Adapter file validation failed: \(path)")
            if withRollback {
                return _tryRestoreFromBackup(path: path, kind: kind)
            }
            return false
        }

        // Step 3: Try to load adapter
        do {
            let adapter = try _loadMLXAdapter(path: path)

            // Step 4: Apply adapter with scale
            _applyAdapter(adapter: adapter, scale: scale, kind: kind)

            loadedAdapters[kind] = adapter
            print("[MLXEngine] ✅ Loaded adapter: \(kind) from \(path)")
            return true

        } catch {
            print("[MLXEngine] ERROR: Failed to load adapter: \(error)")

            // Step 5: Rollback if loading failed
            if withRollback {
                return _tryRestoreFromBackup(path: path, kind: kind)
            }
            return false
        }
    }

    /// Validate adapter file before loading.
    private func _validateAdapterFile(path: String) -> Bool {
        guard let attributes = try? FileManager.default.attributesOfItem(atPath: path),
              let fileSize = attributes[.size] as? Int64 else {
            return false
        }

        // Check file size (should be > 0 and < 100MB)
        guard fileSize > 0 && fileSize < 100 * 1024 * 1024 else {
            print("[MLXEngine] ERROR: Invalid adapter file size: \(fileSize)")
            return false
        }

        // Check file extension (should be .safetensors for iOS)
        guard path.hasSuffix(".safetensors") else {
            print("[MLXEngine] ERROR: Invalid adapter file format: \(path)")
            return false
        }

        return true
    }

    /// Load MLX adapter from safetensors file.
    private func _loadMLXAdapter(path: String) throws -> Any {
        // Use MLX Python runtime to load adapter
        // This is a placeholder - actual implementation depends on MLX integration
        let python = Python.import("mlx_lm")
        let adapter = try python.load_lora_weights(path)
        return adapter
    }

    /// Apply adapter to model with scale.
    private func _applyAdapter(adapter: Any, scale: Float, kind: String) {
        // Apply adapter to MLX model
        // This is a placeholder - actual implementation depends on MLX integration
        print("[MLXEngine] Applying adapter \(kind) with scale \(scale)")
    }

    /// Try to restore adapter from backup.
    private func _tryRestoreFromBackup(path: String, kind: String) -> Bool {
        let backupPath = "\(path).backup"

        guard FileManager.default.fileExists(atPath: backupPath) else {
            print("[MLXEngine] ERROR: No backup available for \(kind)")
            return false
        }

        print("[MLXEngine] Attempting rollback from backup: \(backupPath)")

        // Restore backup file
        do {
            try FileManager.default.removeItem(atPath: path)
            try FileManager.default.copyItem(atPath: backupPath, toPath: path)
            print("[MLXEngine] ✅ Restored adapter from backup")

            // Try loading restored adapter
            return loadLoRAAdapter(
                path: path,
                scale: 1.0, // Default scale
                kind: kind,
                withRollback: false // Don't try rollback again
            )
        } catch {
            print("[MLXEngine] ERROR: Failed to restore from backup: \(error)")
            return false
        }
    }

    /// Test adapter before making it active (optional validation step).
    @objc func validateAdapter(path: String) -> Bool {
        // Try loading adapter in test mode
        // If it loads successfully, it's valid
        return loadLoRAAdapter(
            path: path,
            scale: 1.0,
            kind: "test",
            withRollback: false
        )
    }
}
```

---

## Flutter Integration

### Alice Brain Service - Adapter Loading with Rollback

**File**: `flutter_app/lib/features/alice/domain/alice_brain_service.dart` (MODIFY)

```dart
class MethodChannelAliceBrainService implements AliceBrainService {
  final LoRAAdapterManager _adapterManager;

  Future<void> _loadAdapterStackWithRollback(
    List<Map<String, dynamic>> adapterStack,
  ) async {
    for (final adapterInfo in adapterStack) {
      final String path = adapterInfo['path'] as String;
      final double scale = adapterInfo['scale'] as double;
      final String kind = adapterInfo['kind'] as String;

      // Try loading adapter
      bool success = await _channel.invokeMethod('loadLoRAAdapter', {
        'path': path,
        'scale': scale,
        'kind': kind,
        'withRollback': true,
      });

      if (!success) {
        // Loading failed - try rollback
        debugPrint('[AliceBrain] Adapter loading failed, attempting rollback: $kind');

        final bool restored = await _adapterManager.restoreFromBackup(
          _mapKindToLoRAKind(kind),
        );

        if (restored) {
          // Retry loading with restored adapter
          success = await _channel.invokeMethod('loadLoRAAdapter', {
            'path': path,
            'scale': scale,
            'kind': kind,
            'withRollback': false, // Don't try rollback again
          });

          if (success) {
            debugPrint('[AliceBrain] ✅ Restored and loaded adapter: $kind');
          } else {
            debugPrint('[AliceBrain] ❌ Failed to load even after rollback: $kind');
          }
        } else {
          debugPrint('[AliceBrain] ❌ No backup available for: $kind');
        }
      }
    }
  }
}
```

---

## Training Integration

### On-Device Training with Rollback

**File**: `app/src/lib/services/ml/qloraTrainer.ts` (MODIFY)

```typescript
async runDailyTraining(): Promise<TrainingSession> {
    // ... existing training logic ...

    // After training completes
    const safetensorsPath = await saveAdapter('user_lora.safetensors');

    // For iOS: Store adapter with backup
    if (Platform.isIOS) {
        const loraAdapterManager = new LoRAAdapterManager(...);

        // Step 1: Create backup of current adapter (if exists)
        await loraAdapterManager.storeAdapter(
            LoRAKind.U,
            await readFile(safetensorsPath),
            createBackup: true
        );

        // Step 2: Validate new adapter
        const isValid = await validateAdapter(safetensorsPath);
        if (!isValid) {
            // Validation failed - restore from backup
            console.error('❌ Adapter validation failed, restoring backup');
            await loraAdapterManager.restoreFromBackup(LoRAKind.U);
            throw new Error('Adapter validation failed, restored previous adapter');
        }

        // Step 3: Test loading new adapter
        const loadSuccess = await testLoadAdapter(safetensorsPath);
        if (!loadSuccess) {
            // Loading failed - restore from backup
            console.error('❌ Adapter loading failed, restoring backup');
            await loraAdapterManager.restoreFromBackup(LoRAKind.U);
            throw new Error('Adapter loading failed, restored previous adapter');
        }

        console.log('✅ Adapter validated and loaded successfully');
    }

    // Upload delta to server (for aggregation only)
    await federatedUploader.enqueueDelta(safetensorsPath);

    return result;
}
```

---

## Error Detection & Rollback Triggers

### Automatic Rollback Triggers

1. **File Not Found**: Adapter file missing → Restore from backup
2. **Invalid Format**: Adapter file corrupt/wrong format → Restore from backup
3. **Load Failure**: MLX engine fails to load adapter → Restore from backup
4. **Validation Failure**: Adapter fails validation checks → Restore from backup
5. **Runtime Errors**: Adapter causes inference errors → Restore from backup (after threshold)

### Manual Rollback

```dart
// Manual rollback if needed
final bool restored = await loraAdapterManager.restoreFromBackup(LoRAKind.U);
if (restored) {
  // Reload adapter
  await reloadAdapter(LoRAKind.U);
}
```

---

## Summary

**MLX/iOS Rollback Strategy**:

- ✅ **Backup before overwrite**: Create `.backup` file before storing new adapter
- ✅ **Validate before use**: Check adapter file integrity before loading
- ✅ **Automatic rollback**: Restore from backup if loading/validation fails
- ✅ **Test before activate**: Optionally test adapter before making it active
- ✅ **Manual rollback**: Support manual rollback if needed

**Key Differences from Android**:

- Simpler: No conversion step (just backup/restore safetensors)
- Same backup strategy: Keep last version as `.backup`
- Same rollback triggers: File errors, load failures, validation failures

**Storage**: Same as Android (~20-100 MB per adapter with backup)

## Related

^[{src_rel}]
