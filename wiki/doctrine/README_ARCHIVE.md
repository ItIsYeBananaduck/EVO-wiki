---
title: README_ARCHIVE
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/README_ARCHIVE.md
updated: 2026-07-24
---

# Archive and Upload to TestFlight Script

## Quick Usage

```bash
cd flutter_app
./scripts/archive_and_upload_testflight.sh
```

## Prerequisites

### Option 1: App Store Connect API Key (Recommended)

Set these environment variables:

```bash
export APP_STORE_CONNECT_API_KEY_ID="your-key-id"
export APP_STORE_CONNECT_API_ISSUER="your-issuer-uuid"
export APP_STORE_CONNECT_API_KEY="path/to/AuthKey_xxxxx.p8"
```

Or create `.env` file in `flutter_app/` directory:

```bash
APP_STORE_CONNECT_API_KEY_ID=your-key-id
APP_STORE_CONNECT_API_ISSUER=your-issuer-uuid
APP_STORE_CONNECT_API_KEY=/path/to/AuthKey_xxxxx.p8
```

### Option 2: Interactive Authentication

The script will prompt for your Apple ID password:

```bash
export APPLE_ID="your-apple-id@example.com"
./scripts/archive_and_upload_testflight.sh
```

## What the Script Does

1. **Flutter Build** (if Flutter is in PATH):
   - Cleans build
   - Gets dependencies
   - Builds iOS release

2. **Xcode Archive**:
   - Archives for generic iOS device
   - Uses automatic signing
   - Creates `.xcarchive` file

3. **Export IPA**:
   - Exports IPA from archive
   - Configures for App Store distribution
   - Includes symbols for crash reporting

4. **Upload to TestFlight**:
   - Uploads IPA to App Store Connect
   - Uses API key or interactive auth
   - Provides upload status

## Troubleshooting

### Flutter Not Found

- Script will skip Flutter build and use Xcode directly
- Works fine if Xcode workspace is up to date

### Signing Issues

- Make sure signing is configured in Xcode
- Team ID must be set in project settings
- Script will detect and use your team

### Upload Fails

- Check App Store Connect API credentials
- Or use Xcode Organizer to upload manually:
  1. Window → Organizer
  2. Select archive
  3. Distribute App → TestFlight

### Archive Fails

- Check Xcode build logs
- Verify workspace builds in Xcode
- May need to configure signing manually first

## Output Files

- **Archive**: `~/Desktop/Runner-YYYYMMDD-HHMMSS.xcarchive`
- **IPA**: `~/Desktop/Export-YYYYMMDD-HHMMSS/Runner.ipa`
- **Export Options**: `~/Desktop/ExportOptions.plist`
- **Logs**: `/tmp/xcode_*.log`

## Next Steps

After upload:

1. Go to App Store Connect → TestFlight
2. Wait for processing (10-30 minutes)
3. Add testers if needed
4. Test on iPhone!

## Related

^[source-materials/mirrors/doctrine/README_ARCHIVE.md]
