---
title: EVOLoRA_Mesh_Adapter_Versioning_Strategy
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOLoRA_Mesh_Adapter_Versioning_Strategy.md"]
updated: 2026-07-24
---

# EVOLoRA Mesh Adapter Versioning & Cleanup Strategy

**Problem**: When a new LoRA adapter is created (daily for U/T), what happens to the old adapter file? Overwrite? Delete? Version?

**Solution**: Overwrite old adapter, but keep a backup for rollback capability.

---

## Storage Strategy

### On-Device Adapter Storage

```
AliceAssets/adapters/
├── user/
│   ├── user_lora.safetensors          ← Current (iOS)
│   ├── user_lora.gguf                 ← Current (Android)
│   ├── user_lora.safetensors.backup   ← Previous version (iOS backup)
│   └── user_lora.gguf.backup          ← Previous version (Android backup)
├── trainer/
│   ├── {trainerId}_lora.safetensors
│   ├── {trainerId}_lora.gguf
│   ├── {trainerId}_lora.safetensors.backup
│   └── {trainerId}_lora.gguf.backup
└── global/
    ├── global_user_lora.safetensors
    ├── global_user_lora.gguf
    ├── global_user_lora.safetensors.backup
    └── global_user_lora.gguf.backup
```

### Versioning Approach

**Strategy**: Keep current + one backup (last version)

**Process**:

1. **Before writing new adapter**:
   - If current adapter exists → Move to `.backup`
   - Delete old `.backup` if exists (only keep last version)

2. **Write new adapter**:
   - Write new adapter as current
   - New adapter becomes active

3. **Rollback capability**:
   - If new adapter fails/corrupts → Restore from `.backup`
   - Manual rollback option (if needed)

**Benefits**:

- ✅ Simple (only 2 files: current + backup)
- ✅ Minimal storage (only 1 backup, not full history)
- ✅ Rollback capability (if new adapter fails)
- ✅ Automatic cleanup (old backup deleted when new one created)

---

## Implementation

### LoRA Adapter Manager Update

**File**: `flutter_app/lib/features/alice/domain/lora_adapter_manager.dart` (MODIFY)

```dart
class LoRAAdapterManager {
  /// Stores a LoRA adapter file with backup/versioning.
  Future<void> storeAdapter(
    LoRAKind kind,
    Uint8List data, {
    String? trainerId,
    bool createBackup = true,
  }) async {
    final String path = getAdapterPath(kind, trainerId: trainerId);

    // Step 1: Create backup of current adapter (if exists)
    if (createBackup && await adapterExists(kind, trainerId: trainerId)) {
      await _createBackup(path);
    }

    // Step 2: Write new adapter
    if (_useSharedModelStoreSaf) {
      final String relativePath = p.relative(path, from: baseDir);
      await sharedModelStore!.writeBytes(
        relativePath,
        data,
        mime: 'application/octet-stream',
        overwrite: true,
      );
    } else {
      final File file = File(path);
      if (!await file.parent.exists()) {
        await file.parent.create(recursive: true);
      }
      await file.writeAsBytes(data, flush: true);
    }

    debugPrint('[LoRAAdapterManager] Stored ${kind.key} adapter at $path');
  }

  /// Creates a backup of the current adapter.
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

  /// Restores adapter from backup (if available).
  Future<bool> restoreFromBackup(LoRAKind kind, {String? trainerId}) async {
    final String path = getAdapterPath(kind, trainerId: trainerId);
    final String backupPath = '$path.backup';

    try {
      if (_useSharedModelStoreSaf) {
        final String relativeBackup = p.relative(backupPath, from: baseDir);
        final String relativePath = p.relative(path, from: baseDir);

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
        final File currentFile = File(path);

        if (!await backupFile.exists()) {
          return false; // No backup available
        }

        // Restore backup
        await backupFile.copy(path);
        return true;
      }
    } catch (e) {
      debugPrint('[LoRAAdapterManager] ERROR: Failed to restore from backup: $e');
      return false;
    }

    return false;
  }

  /// Deletes old backup (cleanup).
  Future<void> deleteBackup(LoRAKind kind, {String? trainerId}) async {
    final String path = getAdapterPath(kind, trainerId: trainerId);
    final String backupPath = '$path.backup';

    try {
      if (_useSharedModelStoreSaf) {
        final String relativeBackup = p.relative(backupPath, from: baseDir);
        await sharedModelStore!.deleteFileIfExists(relativeBackup);
      } else {
        final File backupFile = File(backupPath);
        if (await backupFile.exists()) {
          await backupFile.delete();
        }
      }
    } catch (e) {
      debugPrint('[LoRAAdapterManager] WARNING: Failed to delete backup: $e');
    }
  }
}
```

---

## On-Device Conversion Flow

### Android: Safetensors → GGUF Conversion

**File**: `app/src/lib/services/ml/qloraTrainer.ts` (MODIFY)

```typescript
async runDailyTraining(): Promise<TrainingSession> {
    // ... existing training logic ...

    // After training completes
    const safetensorsPath = await saveAdapter('user_lora.safetensors');

    // On-device conversion for Android
    if (Platform.isAndroid) {
        const baseModelPath = await getBaseModelPath(); // alice-phi3-q4.gguf
        const ggufPath = 'AliceAssets/adapters/user/user_lora.gguf';

        // Step 1: Create backup of current GGUF (if exists)
        const loraAdapterManager = new LoRAAdapterManager(...);
        await loraAdapterManager.storeAdapter(
            LoRAKind.U,
            await readFile(safetensorsPath), // Store safetensors first
            createBackup: true
        );

        // Step 2: Convert safetensors → GGUF on-device
        const conversionSuccess = await LlamaPlugin.convertLoRAToGGUF(
            safetensorsPath,
            baseModelPath,
            ggufPath
        );

        if (!conversionSuccess) {
            // Conversion failed - restore from backup
            console.error('❌ GGUF conversion failed, restoring backup');
            await loraAdapterManager.restoreFromBackup(LoRAKind.U);
            throw new Error('Failed to convert adapter to GGUF');
        }

        // Step 3: Store GGUF adapter (with backup)
        await loraAdapterManager.storeAdapter(
            LoRAKind.U,
            await readFile(ggufPath),
            createBackup: true
        );

        console.log('✅ Converted adapter to GGUF on-device');
    }

    // iOS: Use safetensors directly (no conversion)
    // Backup already created above

    // Upload delta to server (for aggregation only)
    await federatedUploader.enqueueDelta(safetensorsPath);

    return result;
}
```

---

## Storage Management

### Disk Space Considerations

**Adapter Sizes**:

- Safetensors: ~10-50 MB (depending on LoRA rank)
- GGUF: ~10-50 MB (similar size)
- Backup: Same size as current

**Total Storage per Adapter**:

- Current: ~10-50 MB
- Backup: ~10-50 MB
- **Total**: ~20-100 MB per adapter

**For 4 adapters (U, T, GU, GT)**:

- Total: ~80-400 MB (with backups)
- Acceptable for mobile devices ✅

### Cleanup Strategy

**Automatic Cleanup**:

- Old backup deleted when new backup created (only keep last version)
- No manual cleanup needed

**Manual Cleanup** (if needed):

```dart
// Delete backup if storage is tight
await loraAdapterManager.deleteBackup(LoRAKind.U);
```

**Storage Monitoring**:

```dart
Future<int> getAdapterStorageUsage() async {
  int totalBytes = 0;

  for (final kind in LoRAKind.values) {
    final path = getAdapterPath(kind);
    final backupPath = '$path.backup';

    if (await File(path).exists()) {
      totalBytes += await File(path).length();
    }
    if (await File(backupPath).exists()) {
      totalBytes += await File(backupPath).length();
    }
  }

  return totalBytes;
}
```

---

## Error Handling & Rollback

### Conversion Failure

**Scenario**: GGUF conversion fails on Android

**Process**:

1. Backup already created (before conversion)
2. Conversion fails
3. Restore from backup automatically
4. Log error, continue with previous adapter

**Code**:

```typescript
try {
  await convertToGGUF();
} catch (error) {
  // Restore from backup
  await loraAdapterManager.restoreFromBackup(LoRAKind.U);
  throw new Error("Conversion failed, restored previous adapter");
}
```

### Corrupt Adapter Detection

**Scenario**: New adapter is corrupt/invalid

**Process**:

1. Validate adapter after conversion
2. If invalid → Restore from backup
3. Log error, continue with previous adapter

**Code**:

```typescript
// Validate adapter
const isValid = await validateAdapter(ggufPath);
if (!isValid) {
  await loraAdapterManager.restoreFromBackup(LoRAKind.U);
  throw new Error("Adapter validation failed, restored previous adapter");
}
```

---

## Summary

**Storage Strategy**:

- ✅ **Current adapter**: Active version (overwritten on update)
- ✅ **Backup adapter**: Previous version (`.backup` suffix)
- ✅ **Automatic cleanup**: Old backup deleted when new backup created
- ✅ **Rollback capability**: Restore from backup if conversion fails

**Process**:

1. Before writing new adapter → Move current to `.backup`
2. Write new adapter → New becomes current
3. If conversion fails → Restore from `.backup`
4. Old backup deleted → Only keep last version

**Storage Impact**:

- ~20-100 MB per adapter (current + backup)
- ~80-400 MB total for 4 adapters
- Acceptable for mobile devices ✅

**Benefits**:

- ✅ Simple (only 2 files per adapter)
- ✅ Minimal storage (only 1 backup)
- ✅ Rollback capability (if conversion fails)
- ✅ Automatic cleanup (no manual intervention)

## Related

^[{src_rel}]
