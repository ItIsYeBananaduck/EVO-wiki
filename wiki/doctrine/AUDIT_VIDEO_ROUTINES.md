---
title: AUDIT_VIDEO_ROUTINES
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/AUDIT_VIDEO_ROUTINES.md
updated: 2026-07-24
---

# AUDIT_VIDEO_ROUTINES.md

**Date:** January 2, 2026
**Status:** PHASE A AUDIT COMPLETE
**Scope:** Video Workout Routines with On-Device Pose Estimation

---

## AUDIT SUMMARY

✅ **All Phase A audit tasks completed successfully**

### A1: Marketplace/Product Code Structure ✅

- **Existing System:** Supabase-based marketplace with R2 storage
- **Product Model:** Training programs (not video workouts yet)
- **Storage:** Cloudflare R2 (evostorage bucket) with signed URLs
- **Integration Points:** Ready for video workout extension

### A2: Media Handling Components ✅

- **Video Assets:** MoveNet models available (Lightning ONNX, Thunder TFLite)
- **Camera Service:** Placeholder implementation (needs development)
- **FFmpeg:** Not present (needs integration for frame extraction)
- **Pose Estimation:** Models available, integration needed

### A3: AI/Session Summary Integration ✅

- **Alice Models:** Alice Lite and alice-mistral-8b-q4.gguf available
- **Session Management:** WorkoutSession model with summary capabilities
- **Intensity System:** Comprehensive intensity scoring service
- **Data Flow:** Ready for structured pose summary consumption

---

## EXISTING PRODUCT PIPELINE

### Current Marketplace Architecture

```
User → Svelte App → Supabase → R2 Storage
     ↓
Trainer → Upload → Processing → Published Content
     ↓
Purchase → Payment → Download Access → Local Storage
```

### Database Schema (Supabase)

**Current Tables:**

- `users` - User profiles and roles
- `workout_sessions` - Session tracking
- `training_programs` - Program definitions (price in cents)
- `api_cache` - Response caching
- `alice_*` tables - AI state and metrics

**Missing for Video Workouts:**

- Video-specific product tables
- Pose signature storage
- Video workout metadata

### Storage Layer (R2)

**Current Setup:**

- Bucket: `evostorage`
- Worker: `r2-importer` for secure downloads
- Auth: Supabase token exchange
- Allowed prefixes: `alice-assets/models/`, `alice-assets/onnx/`

**Integration Points:**

- Upload: Multipart upload API available
- Download: Signed token-based access
- Security: HMAC-signed tokens with TTL

---

## INTEGRATION POINTS IDENTIFIED

### 1. Product Extension Points

**File:** `app/src/routes/marketplace/[programId]/+page.svelte`

- Current: Training program purchases
- Extension: Add video workout type
- Payment: Existing Stripe integration reusable

**File:** `supabase/functions/marketplace.ts` (Supabase based)

- Actual: Supabase Edge Functions needed
- Extension: Video upload and processing

### 2. Media Handling Integration

**Pose Estimation Models:**

- `flutter_app/assets/models/movenet_singlepose_lightning.onnx`
- `flutter_app/assets/models/movenet_thunder.tflite`
- Status: ✅ Available, needs integration

**Camera Service:**

- `app/src/lib/services/mobile/CameraService.ts`
- Status: ⚠️ Placeholder only, needs implementation

**FFmpeg Integration:**

- Status: ❌ Not present, needs addition
- Purpose: Frame extraction for pose signature generation

### 3. AI/Session Summary Integration

**Alice Models:**

- `AliceAssets/gguf/alice-mistral-8b-q4.gguf`
- Status: ✅ Available for reasoning
- Integration: Ready for pose summary consumption

**Session Management:**

- `app/src/lib/models/WorkoutSession.ts`
- Status: ✅ Comprehensive session model
- Extension: Add pose summary fields

**Intensity System:**

- `app/src/lib/services/intensityService.ts`
- Status: ✅ Full intensity scoring
- Integration: Real-time intensity during video workouts

---

## MINIMAL-CHANGE PLAN

### Phase B: Data Models & Storage

**1. Extend Existing Tables (Supabase)**

```sql
-- Extend training_programs table
ALTER TABLE training_programs ADD COLUMN program_type TEXT DEFAULT 'traditional' CHECK (program_type IN ('traditional', 'video_workout'));
ALTER TABLE training_programs ADD COLUMN video_url TEXT;
ALTER TABLE training_programs ADD COLUMN pose_signature_url TEXT;
ALTER TABLE training_programs ADD COLUMN video_duration_ms INTEGER;
ALTER TABLE training_programs ADD COLUMN video_metadata JSONB;
```

**2. Add Video-Specific Tables**

```sql
-- Video workout assets
CREATE TABLE video_workout_assets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  program_id UUID REFERENCES training_programs(id) ON DELETE CASCADE,
  asset_type TEXT CHECK (asset_type IN ('video', 'pose_signature', 'manifest', 'thumbnail')),
  r2_key TEXT NOT NULL,
  file_size INTEGER,
  checksum TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

**3. Storage Integration**

- Extend R2 allowed prefixes for video content
- Add video workout asset organization
- Implement secure upload/download flows

### Phase C: Trainer Flow

**1. Extend Upload Interface**

- Modify existing program creation UI
- Add video file upload component
- Integrate FFmpeg for frame extraction

**2. Pose Signature Generation**

- On-device processing using MoveNet
- Store signature in R2 alongside video
- Generate manifest with metadata

**3. Publishing Flow**

- Extend existing marketplace publishing
- Add video-specific metadata
- Maintain current payment integration

### Phase D: User Experience

**1. Library Extension**

- Extend existing program library
- Add video workout type indicator
- Reuse download management system

**2. Playback Interface**

- New video player component
- Overlay pose estimation visualization
- Real-time intensity display

**3. Camera Integration**

- Implement camera service for live pose
- Compare with reference signature
- Maintain privacy (no recording)

---

## RISKS + MITIGATIONS

### Technical Risks

**Risk 1: FFmpeg Integration Complexity**

- **Mitigation:** Use WASM build or server-side processing
- **Fallback:** Manual frame specification for trainers

**Risk 2: MoveNet Performance**

- **Mitigation:** Model selection based on device capability
- **Fallback:** Reduced FPS for older devices

**Risk 3: Storage Costs**

- **Mitigation:** Compress pose signatures, efficient video encoding
- **Monitoring:** R2 usage tracking and alerts

**Risk 4: Real-time Processing Performance**

- **Mitigation:** Adaptive quality, background processing
- **Fallback:** Pose estimation only at key intervals

### Privacy Risks

**Risk 1: Camera Data Exposure**

- **Mitigation:** On-device processing only, no upload
- **Validation:** Privacy audit before deployment

**Risk 2: Pose Data Reversibility**

- **Mitigation:** Store only normalized coordinates, no video
- **Validation:** Security review of data storage

### Business Risks

**Risk 1: Trainer Adoption**

- **Mitigation:** Simple upload flow, clear value proposition
- **Validation:** Beta testing with trainer community

**Risk 2: User Experience Complexity**

- **Mitigation:** Progressive disclosure, optional features
- **Validation:** User testing across device types

---

## FEATURE-FLAG PLAN

### Phase 1: Core Infrastructure (Always On)

- Database schema updates
- R2 storage extensions
- Basic video upload/download

### Phase 2: Trainer Tools (Feature Flag: `VIDEO_WORKOUT_UPLOAD`)

- Trainer upload interface
- Pose signature generation
- Video publishing flow

### Phase 3: User Experience (Feature Flag: `VIDEO_WORKOUT_PLAYBACK`)

- Video workout library
- Playback interface
- Basic pose estimation

### Phase 4: Advanced Features (Feature Flag: `VIDEO_WORKOUT_POSE_ANALYSIS`)

- Live pose comparison
- Real-time feedback
- Advanced analytics

---

## STORAGE ARCHITECTURE

### R2 Organization

```
evostorage/
├── alice-assets/
│   ├── models/
│   └── onnx/
├── video-workouts/
│   ├── videos/
│   │   └── {program_id}/
│   │       ├── workout_video.mp4
│   │       └── thumbnail.jpg
│   ├── pose-signatures/
│   │   └── {program_id}/
│   │       └── pose_signature.json
│   └── manifests/
│       └── {program_id}/
│           └── workout_manifest.json
└── user-downloads/
    └── {user_id}/
        └── {program_id}/
            ├── workout_video.mp4
            └── pose_signature.json
```

### Security Model

**Upload:**

- Trainer authentication via Supabase
- Multipart upload with progress tracking
- Virus scanning and validation

**Download:**

- User purchase verification
- Time-limited signed URLs
- Local device encryption

**Privacy:**

- Camera frames processed on-device only
- No video or camera data uploaded
- Pose data stored as normalized coordinates

---

## AI INTEGRATION ARCHITECTURE

### Alice Model Integration

```
Video Workout Session
    ↓
Live Pose Estimation (MoveNet)
    ↓
Pose Comparison Engine
    ↓
Session Summary JSON
    ↓
Alice Model (Lite or Full)
    ↓
End-of-Session Text Feedback
```

### Data Flow

**Input to Alice:**

```json
{
  "schema_version": 1,
  "routine_id": "program_123",
  "pose_match_avg": 0.82,
  "pose_match_min": 0.54,
  "hotspots": [{ "joint": "left_hip", "severity": 0.72 }],
  "timing_drift": { "avg_ms": 180, "max_ms": 420 },
  "intensity": {
    "source": "wearable|rpe",
    "score": 0.74,
    "rpe": "moderate"
  },
  "duration_minutes": 28.5,
  "confidence": 0.89
}
```

**Output from Alice:**

- End-of-session text summary
- Form recommendations
- Improvement suggestions

---

## PERFORMANCE CONSIDERATIONS

### Device Capability Detection

**High-end Devices:**

- MoveNet Thunder (TFLite)
- Full FPS pose estimation (15-30 fps)
- Real-time comparison

**Mid-range Devices:**

- MoveNet Lightning (ONNX)
- Reduced FPS (10-15 fps)
- Interval-based comparison

**Low-end Devices:**

- Minimal pose estimation (5-10 fps)
- Key frame comparison only
- Simplified feedback

### Storage Optimization

**Video Compression:**

- H.264 encoding with adaptive bitrate
- Multiple quality levels
- Progressive download

**Pose Signature Optimization:**

- Delta compression for frame sequences
- Joint subset for complex movements
- Configurable precision levels

---

## VALIDATION CRITERIA

### Phase B Validation

- [ ] Database migrations successful
- [ ] R2 upload/download working
- [ ] Pose signature schema stable
- [ ] Manifest format validated

### Phase C Validation

- [ ] Video upload flow complete
- [ ] Pose signature generation accurate
- [ ] Trainer publishing functional
- [ ] Privacy requirements met

### Phase D Validation

- [ ] Video playback smooth
- [ ] Pose estimation real-time
- [ ] Alice integration working
- [ ] Cross-device compatibility

### Phase E Validation

- [ ] Session summary accurate
- [ ] Alice feedback relevant
- [ ] Performance acceptable
- [ ] User experience positive

---

## NEXT STEPS

### Immediate Actions (Phase B)

1. Create Supabase migrations for video workout tables
2. Extend R2 worker for video asset handling
3. Define pose signature and manifest schemas
4. Implement basic video upload/download APIs

### Medium Term (Phase C-D)

1. Implement camera service and pose estimation
2. Create trainer upload interface
3. Build video workout player
4. Integrate Alice model for summaries

### Long Term (Phase E-F)

1. Advanced pose analysis features
2. Performance optimization
3. User testing and validation
4. Feature flag rollout strategy

---

## CONCLUSION

The audit reveals a solid foundation for Video Workout Routines implementation:

✅ **Strengths:**

- Existing marketplace infrastructure
- Available AI models and pose estimation
- Comprehensive session management
- Secure storage and authentication

⚠️ **Areas Needing Work:**

- Camera service implementation
- FFmpeg integration for frame extraction
- Video-specific data models
- Real-time pose processing

🎯 **Recommended Approach:**

- Minimal-change extension of existing systems
- Feature-flag rollout for risk mitigation
- Privacy-first design with on-device processing
- Progressive enhancement based on device capabilities

The architecture supports the specified requirements while maintaining compatibility with existing systems and following the minimal-change principle.

## Related

^[source-materials/mirrors/doctrine/AUDIT_VIDEO_ROUTINES.md]
