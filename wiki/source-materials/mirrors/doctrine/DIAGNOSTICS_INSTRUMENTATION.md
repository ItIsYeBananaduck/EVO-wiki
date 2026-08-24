---
title: DIAGNOSTICS_INSTRUMENTATION
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/DIAGNOSTICS_INSTRUMENTATION.md"]
updated: 2026-07-24
---

# Diagnostics Instrumentation Code Snippets

## 1. Instrument AliceAssetDownloadManager

Add to `alice_asset_download_manager.dart`:

```dart
import '../core/diagnostics/diag_log.dart';
import '../core/diagnostics/model_pipeline_stage.dart';

// At the start of _downloadAsset method, before HTTP request:
DiagLog.instance.setStage(ModelPipelineStage.network);
DiagLog.instance.log(
  'Starting download: $assetLabel',
  data: {
    'storage_path': asset.storagePath,
    'expected_size': expectedSizeBytes,
    'worker_url_host': Uri.parse(_workerBaseUrl).host,
  },
);

// Perform HEAD request first (add this method):
Future<Map<String, String>?> _performHeadRequest(Uri uri) async {
  try {
    final client = HttpClient();
    final request = await client.headUrl(uri);
    request.headers.add('User-Agent', 'AliceApp/1.0');

    final response = await request.close();
    final headers = <String, String>{};
    response.headers.forEach((key, values) {
      headers[key] = values.join(', ');
    });

    DiagLog.instance.log(
      'HEAD response: ${response.statusCode}',
      data: {
        'status_code': response.statusCode,
        'content_length': headers['content-length'],
        'content_type': headers['content-type'],
        'server_date': headers['date'],
      },
      level: response.statusCode >= 400 ? DiagLevel.error : DiagLevel.info,
    );

    client.close();
    return headers;
  } catch (e) {
    DiagLog.instance.log('HEAD request failed: $e', level: DiagLevel.warn);
    return null;
  }
}

// In _downloadAsset, after getting download URL:
final headHeaders = await _performHeadRequest(uri);
if (headHeaders != null && headHeaders['content-type']?.contains('text/html') == true) {
  DiagLog.instance.log(
    'Download URL returned HTML (likely error page)',
    level: DiagLevel.error,
    data: {'status': response.statusCode},
  );
  throw Exception('Download URL returned HTML error page');
}

// Before starting download stream:
DiagLog.instance.setStage(ModelPipelineStage.network);
final downloadStartTime = DateTime.now();

// During download, log progress periodically:
if (offset % (10 * 1024 * 1024) == 0) { // Every 10MB
  DiagLog.instance.log(
    'Download progress: ${(offset / (expectedSizeBytes ?? 1) * 100).toStringAsFixed(1)}%',
    data: {'bytes_received': offset, 'total_bytes': expectedSizeBytes},
  );
}

// After download completes, before file write:
DiagLog.instance.log(
  'Download complete',
  data: {
    'bytes_received': offset,
    'expected_bytes': expectedSizeBytes,
    'duration_ms': DateTime.now().difference(downloadStartTime).inMilliseconds,
  },
);

// DISK CHECK: Before file write:
DiagLog.instance.setStage(ModelPipelineStage.disk);
final targetFile = File(targetPath);
final targetDir = targetFile.parent;

DiagLog.instance.log(
  'Preparing disk write',
  data: {
    'target_path': targetPath,
    'target_dir': targetDir.path,
    'dir_exists': await targetDir.exists(),
  },
);

// Ensure directory exists:
if (!await targetDir.exists()) {
  await targetDir.create(recursive: true);
  DiagLog.instance.log('Created directory: ${targetDir.path}');
}

// After file write completes:
await targetFile.writeAsBytes(bytes);
final fileSize = await targetFile.length();

DiagLog.instance.log(
  'File written to disk',
  data: {
    'file_path': targetPath,
    'file_size': fileSize,
    'expected_size': expectedSizeBytes,
  },
);

// Fail-fast: Check file size
if (expectedSizeBytes != null && fileSize < expectedSizeBytes * 0.9) {
  DiagLog.instance.log(
    'File size mismatch: expected ${expectedSizeBytes}, got $fileSize',
    level: DiagLevel.error,
  );
  throw Exception('File size mismatch: expected ${expectedSizeBytes}, got $fileSize');
}

// VERIFY CHECK:
DiagLog.instance.setStage(ModelPipelineStage.verify);

// Optional: Verify file header/magic bytes for GGUF
if (asset.storagePath.endsWith('.gguf')) {
  final file = File(targetPath);
  final header = await file.openRead(0, 4).toList();
  final magic = header.expand((e) => e).toList();
  // GGUF magic bytes: [0x47, 0x47, 0x55, 0x46] = "GGUF"
  if (magic.length >= 4 &&
      magic[0] == 0x47 && magic[1] == 0x47 &&
      magic[2] == 0x55 && magic[3] == 0x46) {
    DiagLog.instance.log('GGUF magic bytes verified');
  } else {
    DiagLog.instance.log(
      'GGUF magic bytes mismatch',
      level: DiagLevel.error,
      data: {'header_bytes': magic.take(4).map((b) => '0x${b.toRadixString(16)}').join(', ')},
    );
    throw Exception('Invalid GGUF file: magic bytes mismatch');
  }
}

DiagLog.instance.log('Verification complete');
// Stage remains at DISK until model init
```

## 2. Add Hidden Gesture to ai_bootstrap_screen.dart

```dart
import '../core/diagnostics/diagnostics_overlay.dart';

class _AiBootstrapScreenState extends State<AiBootstrapScreen> {
  int _logoTapCount = 0;
  DateTime? _lastTapTime;

  // In build method, wrap the logo/avatar widget:
  Widget _buildLogoWithGesture(Widget logo) {
    return GestureDetector(
      onTap: () {
        final now = DateTime.now();
        if (_lastTapTime == null ||
            now.difference(_lastTapTime!) > const Duration(seconds: 2)) {
          _logoTapCount = 1;
        } else {
          _logoTapCount++;
        }
        _lastTapTime = now;

        if (_logoTapCount >= 7) {
          _logoTapCount = 0;
          Navigator.push(
            context,
            MaterialPageRoute(builder: (_) => const DiagnosticsOverlay()),
          ).then((result) {
            if (result == 'retry') {
              // Restart pipeline
              _init();
            }
          });
        }
      },
      child: logo,
    );
  }
}
```

## 3. Error Categorization Helper

Add to a new file or diag_log.dart:

```dart
enum DiagErrorCategory {
  network,
  auth,
  disk,
  integrity,
  initialization,
  unknown,
}

DiagErrorCategory _categorizeError(dynamic error, int? statusCode) {
  if (statusCode != null) {
    if (statusCode == 401 || statusCode == 403) {
      return DiagErrorCategory.auth;
    }
    if (statusCode == 404) {
      return DiagErrorCategory.network;
    }
    if (statusCode >= 500) {
      return DiagErrorCategory.network;
    }
  }

  final errorStr = error.toString().toLowerCase();
  if (errorStr.contains('permission') || errorStr.contains('denied')) {
    return DiagErrorCategory.disk;
  }
  if (errorStr.contains('corrupted') || errorStr.contains('invalid')) {
    return DiagErrorCategory.integrity;
  }
  if (errorStr.contains('init') || errorStr.contains('load')) {
    return DiagErrorCategory.initialization;
  }

  return DiagErrorCategory.unknown;
}

String _getUserFriendlyError(DiagErrorCategory category, int? statusCode) {
  switch (category) {
    case DiagErrorCategory.auth:
      return 'Access denied or link expired. Please retry.';
    case DiagErrorCategory.network:
      if (statusCode == 404) {
        return 'Model not found. Please check configuration.';
      }
      return 'Network error. Please check connection and retry.';
    case DiagErrorCategory.disk:
      return 'Storage error. Please check available space.';
    case DiagErrorCategory.integrity:
      return 'File corrupted or incomplete. Please retry.';
    case DiagErrorCategory.initialization:
      return 'Model initialization failed. Please retry.';
    case DiagErrorCategory.unknown:
      return 'An error occurred. See diagnostics for details.';
  }
}
```

## 4. R2 Connectivity Test

Add to diagnostics_overlay.dart:

```dart
Future<void> _testR2Connectivity() async {
  DiagLog.instance.log('Testing R2 connectivity');

  // Use a known small test file from the same bucket
  // TODO: Replace with actual test endpoint
  final testUrl = 'https://r2-importer.evoapp.workers.dev/test-ping';

  try {
    final client = HttpClient();
    final request = await client.getUrl(Uri.parse(testUrl));
    final startTime = DateTime.now();
    final response = await request.close();
    final duration = DateTime.now().difference(startTime);

    final headers = <String, String>{};
    response.headers.forEach((key, values) {
      headers[key] = values.join(', ');
    });

    final body = await response.transform(utf8.decoder).join();

    DiagLog.instance.log(
      'R2 test: ${response.statusCode}',
      data: {
        'status_code': response.statusCode,
        'duration_ms': duration.inMilliseconds,
        'content_length': headers['content-length'],
        'latency_ms': duration.inMilliseconds,
      },
      level: response.statusCode == 200 ? DiagLevel.info : DiagLevel.warn,
    );

    client.close();
  } catch (e) {
    DiagLog.instance.log(
      'R2 connectivity test failed: $e',
      level: DiagLevel.error,
    );
  }
}
```

## 5. Required Dependencies (add to pubspec.yaml)

```yaml
dependencies:
  device_info_plus: ^9.1.0
  package_info_plus: ^5.0.0
  share_plus: ^7.2.0
  path_provider: ^2.1.0 # Should already exist
```

## Related
