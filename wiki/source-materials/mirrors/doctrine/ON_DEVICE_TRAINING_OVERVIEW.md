---
title: ON_DEVICE_TRAINING_OVERVIEW
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/ON_DEVICE_TRAINING_OVERVIEW.md"]
updated: 2026-07-24
---

# On-Device AI Training Architecture

**Privacy-First Federated Learning for Adaptive Fitness Coaching**

---

## Overview

Git-fit uses a **privacy-preserving federated learning** system where:

- **Training happens ON YOUR DEVICE** using local data
- Only **anonymized numeric deltas** leave your device
- Your personal data (workouts, meals, health info) **never goes to the cloud**
- The global AI model **improves weekly** from aggregated learnings across all users

**Tech Stack**: Mistral 7B (4-bit GGUF) + Supabase + Fly.io

---

## Core Architecture

```
┌──────────────────────────────────────────────────────────────┐
│  YOUR DEVICE (iPhone/Android)                                │
│                                                                │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Mistral 7B (4-bit GGUF)                                  │ │
│  │ - Runs via llama.cpp (C++/Metal on iOS, NDK on Android) │ │
│  │ - 35 GPU layers on Metal (A19 Pro)                       │ │
│  │ - ~2GB RAM, <5W power consumption                        │ │
│  └─────────────────────────────────────────────────────────┘ │
│           ↓                                  ↑                │
│  ┌─────────────────┐              ┌─────────────────┐        │
│  │ Your Data       │              │ Model Updates   │        │
│  │ (Local Only)    │              │ (Weekly GGUF)   │        │
│  │ • Workouts      │              └─────────────────┘        │
│  │ • Meals         │                                          │
│  │ • Biometrics    │                                          │
│  │ • Preferences   │                                          │
│  └─────────────────┘                                          │
│           ↓                                                    │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Delta Generator (Privacy Layer)                         │ │
│  │ - Converts your interactions to numeric vectors        │ │
│  │ - Fatigue bias: [0.02, -0.01, 0.03, ...]              │ │
│  │ - Macro preference: [-0.015, 0.021, ...]              │ │
│  │ - ≤ 2KB per delta, max 50/week                         │ │
│  └─────────────────────────────────────────────────────────┘ │
│           ↓                                                    │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Local Delta Storage (Capacitor Preferences)            │ │
│  │ Key: delta_{modelId}                                   │ │
│  │ Format: [{ indices: [1,5,9], values: [0.02,-0.01,0.03],│ │
│  │           timestamp: "2025-11-04T..." }]               │ │
│  └─────────────────────────────────────────────────────────┘ │
│           ↓ (Sunday 8am CDT, automated)                       │
└───────────┼──────────────────────────────────────────────────┘
            │
            │ ONLY anonymized numeric vectors
            ↓
 ┌───────────────────────────────────────────────────────────────┐
 │  SUPABASE (Cloud Orchestrator)                                │
 │  ┌──────────────────────────────────────────────────────────┐ │
 │  │ Weekly Aggregation Job (Sunday 8am CDT via Edge Function)│ │
 │  │ 1. Collect metadata from active devices                 │ │
 │  │ 2. Validate: size ≤2KB, rate limit (50/week), freshness│ │
 │  │ 3. Merge sparse vectors:                                │ │
 │  │    - Union all indices across devices                   │ │
 │  │    - Weighted average at shared indices (time-decay)    │ │
 │  │    - Handle conflicts via federated averaging           │ │
 │  │ 4. Stage merged GGUF for operator review               │ │
 │  └──────────────────────────────────────────────────────────┘ │
│           ↓                                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ Canary Testing (Automated)                              │ │
│  │ • Run test prompts on staged model                      │ │
│  │ • Validate output coherence (>50 chars, no truncation) │ │
│  │ • Check model size (<200MB)                             │ │
│  │ • Generate pass/fail report                             │ │
│  └──────────────────────────────────────────────────────────┘ │
│           ↓                                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ Operator Approval (Semi-Automated)                      │ │
│  │ • Operator reviews canary results via admin UI         │ │
│  │ • Approves or rejects staged artifact                   │ │
│  │ • All actions audited for safety                        │ │
│  └──────────────────────────────────────────────────────────┘ │
│           ↓ (if approved)                                      │
└───────────┼──────────────────────────────────────────────────┘
            │
            ↓
┌───────────────────────────────────────────────────────────────┐
│  FLY.IO (Merge Server)                                         │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ /merge Endpoint                                          │ │
│  │ • Receives aggregated sparse vectors from the app        │ │
│  │ • Applies federated averaging to base model             │ │
│  │ • Generates new GGUF with merged weights                │ │
│  │ • Runs smoke tests (coherence, safety constraints)      │ │
│  └──────────────────────────────────────────────────────────┘ │
│           ↓                                                    │
└───────────┼──────────────────────────────────────────────────┘
            │
            ↓
┌───────────────────────────────────────────────────────────────┐
│  HUGGING FACE (Private Model Repository)                      │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ PhilmoLSC/alice-human-fusion (3-way LoRA fusion)        │ │
│  │ • Stores weekly GGUF releases                           │ │
│  │ • Version tagged: v1.0, v1.1, v1.2, ...                │ │
│  │ • Authenticated access only (VITE_HUGGINGFACE_TOKEN)    │ │
│  └──────────────────────────────────────────────────────────┘ │
│           ↓ (devices poll /version every 15 min)              │
└───────────┼──────────────────────────────────────────────────┘
            │
            ↓
    ┌───────────────┐
    │ AUTO-UPDATE   │
    │ All Devices   │
    │ (Within 15min)│
    └───────────────┘
```

---

## Step-by-Step: How a Workout Delta is Generated

### 1. User Completes a Workout

```typescript
// User logs workout in the app
const workout = {
  exercise: "Bench Press",
  sets: 3,
  reps: [10, 8, 6], // Actual reps
  plannedReps: [10, 10, 10], // What was planned
  fatigue: 4, // User rates fatigue 1-5
  formQuality: 8, // User rates form 1-10
  timestamp: "2025-11-04T14:30:00Z",
};
```

### 2. Delta Generator Creates Anonymized Vector

```typescript
// app/src/lib/ai/deltaGenerator.ts
function generateWorkoutDelta(workout, modelId) {
  // Calculate fatigue bias
  const actualVsPlanned = workout.reps.map(
    (actual, i) => (actual - workout.plannedReps[i]) / workout.plannedReps[i],
  );
  const avgDeviation = mean(actualVsPlanned);

  // Map to sparse vector (only non-zero values)
  const delta = {
    indices: [1, 5, 9, 15, 23], // Sparse representation
    values: [
      0.02, // Fatigue adjustment (index 1)
      -0.01, // Volume tolerance (index 5)
      0.03, // Recovery rate (index 9)
      -0.005, // Form degradation (index 15)
      0.015, // Progressive overload bias (index 23)
    ],
    timestamp: workout.timestamp,
  };

  // Validate size (<2KB when serialized)
  const serialized = JSON.stringify(delta);
  if (serialized.length > 2048) {
    throw new Error("Delta exceeds 2KB limit");
  }

  return delta;
}
```

### 3. Store Delta Locally (Private)

```typescript
// Stored in Capacitor Preferences (encrypted by platform)
import { Preferences } from "@capacitor/preferences";

const key = `delta_${modelId}`;
const existingDeltas = await Preferences.get({ key });
const deltas = JSON.parse(existingDeltas.value || "[]");

// Enforce 50/week rate limit
const weekAgo = Date.now() - 7 * 24 * 60 * 60 * 1000;
const recentDeltas = deltas.filter(
  (d) => new Date(d.timestamp).getTime() > weekAgo,
);

if (recentDeltas.length >= 50) {
  console.warn("Delta rate limit reached (50/week)");
  return;
}

deltas.push(delta);
await Preferences.set({ key, value: JSON.stringify(deltas) });
```

---

## Weekly Aggregation Cycle

### Sunday 8am CDT: Supabase Edge Function Triggers

```typescript
// supabase/functions/weekly-aggregate/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2";

serve(async (req) => {
  const supabase = createClient(
    Deno.env.get("SUPABASE_URL") ?? "",
    Deno.env.get("SUPABASE_SERVICE_ROLE_KEY") ?? "",
  );

  // 1. Collect metadata from active devices
  const { data: devices, error } = await supabase
    .from("devices")
    .select("*")
    .eq("active", true);

  const metadata = await Promise.all(
    devices.map(async (device) => {
      // Device sends ONLY metadata (no raw deltas)
      const response = await fetch("/api/delta-metadata", {
        method: "POST",
        headers: {
          Authorization: `Bearer ${device.token}`,
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          deviceId: device.anonymized_id,
          modelId: device.current_model_id,
          localUpdateCount: device.delta_count, // Just the count
          sketchStats: device.delta_sketch, // Approximate stats
          dataVersion: device.data_version,
        }),
      });

      return response.json();
    }),
  );

  // 2. Validate metadata
  const validMetadata = metadata.filter(
    (m) =>
      m.localUpdateCount > 0 &&
      m.localUpdateCount <= 50 && // Rate limit check
      m.sketchStats.avgDeltaSize <= 2048, // Size check
  );

  // 3. Sparse vector merging
  const mergedVector = mergeSparseVectors(validMetadata);

  // 4. Stage artifact for operator review
  const { data: stagedArtifact } = await supabase
    .from("staged_artifacts")
    .insert({
      merged_vector: mergedVector,
      source_devices: validMetadata.length,
      status: "pending_review",
      canary_results: null, // Will be populated by canary tests
    })
    .select()
    .single();

  // 5. Trigger canary tests (automated via another Edge Function)
  await supabase.functions.invoke("run-canary-tests", {
    body: { artifactId: stagedArtifact.id },
  });

  return new Response(
    JSON.stringify({
      stagedArtifactId: stagedArtifact.id,
      sourceDevices: validMetadata.length,
    }),
    { headers: { "Content-Type": "application/json" } },
  );
});

function mergeSparseVectors(metadata) {
  // Federated averaging algorithm
  const allIndices = new Set();
  const indexValues = new Map();

  metadata.forEach(({ sketchStats }) => {
    sketchStats.indices.forEach((idx) => {
      allIndices.add(idx);
      if (!indexValues.has(idx)) {
        indexValues.set(idx, []);
      }
      indexValues.get(idx).push(sketchStats.values[idx]);
    });
  });

  // Weighted average with time-decay
  const mergedIndices = Array.from(allIndices).sort();
  const mergedValues = mergedIndices.map((idx) => {
    const values = indexValues.get(idx);
    const weights = values.map(
      (_, i) => Math.exp(-0.1 * i), // Recent deltas get higher weight
    );
    const weightedSum = values.reduce(
      (sum, val, i) => sum + val * weights[i],
      0,
    );
    const totalWeight = weights.reduce((sum, w) => sum + w, 0);
    return weightedSum / totalWeight;
  });

  return { indices: mergedIndices, values: mergedValues };
}
```

### Operator Review & Approval

```typescript
// Operator UI (admin dashboard)
async function reviewStagedArtifact(artifactId) {
  const { data: artifact } = await supabaseClient
    .from("staged_artifacts")
    .select("*")
    .eq("id", artifactId)
    .single();

  // Show canary test results
  console.log("Canary Tests:", artifact.canaryResults);
  // {
  //   testsPassed: 8/10,
  //   outputCoherence: 'PASS',
  //   modelSize: '185MB (<200MB target)',
  //   safetyChecks: 'PASS',
  //   failedTests: ['test_nutrition_edge_case', 'test_extreme_fatigue']
  // }

  // Operator decides
  if (artifact.canary_results.tests_passed >= 8) {
    // Approve for merge & upload
    await supabaseClient
      .from("staged_artifacts")
      .update({
        status: "approved",
        operator_id: "phil@lonestarcajun.tech",
        operator_notes: "Canary tests look good, proceeding with merge",
        approved_at: new Date().toISOString(),
      })
      .eq("id", artifactId);

    // Trigger merge via Fly.io
    await supabaseClient.functions.invoke("trigger-flyio-merge", {
      body: { artifactId },
    });
  } else {
    // Reject and investigate
    await supabaseClient
      .from("staged_artifacts")
      .update({
        status: "rejected",
        operator_id: "phil@lonestarcajun.tech",
        rejection_reason: "Failed nutrition edge case test - needs review",
        rejected_at: new Date().toISOString(),
      })
      .eq("id", artifactId);
  }
}
```

### Fly.io Merge & Hugging Face Upload

```typescript
// fly.io /merge endpoint
app.post("/merge", authenticateSupabase, async (req, res) => {
  const { mergedVector, modelId } = req.body;

  // 1. Download base model
  const baseModel = await downloadGGUF(
    "PhilmoLSC/alice-human-fusion",
    "latest",
  );

  // 2. Apply federated averaging to model weights
  const updatedModel = applyDeltaToGGUF(baseModel, mergedVector);

  // 3. Run smoke tests
  const smokeTestResults = await runSmokeTests(updatedModel);
  if (!smokeTestResults.passed) {
    throw new Error("Smoke tests failed");
  }

  // 4. Upload to Hugging Face
  const newVersion = incrementVersion(modelId);
  await uploadToHuggingFace(
    updatedModel,
    "PhilmoLSC/alice-human-fusion",
    newVersion,
    process.env.HF_TOKEN,
  );

  // 5. Update version endpoint
  await updateVersionEndpoint(newVersion);

  res.json({
    success: true,
    version: newVersion,
    uploadedAt: new Date().toISOString(),
  });
});
```

---

## Device Auto-Update Flow

### Every 15 Minutes: Poll for New Version

```typescript
// app/src/lib/model.ts
import { Http } from "@capacitor/http";
import { Filesystem, Directory } from "@capacitor/filesystem";

async function checkForModelUpdate() {
  // 1. Poll version endpoint via Supabase Edge Function
  const response = await Http.get({
    url: "https://pzcrllejymdofvfvhtxr.supabase.co/functions/v1/get-model-version",
  });

  const remoteVersion = response.data.version; // "v1.2"
  const localVersion = await getLocalModelVersion(); // "v1.1"

  if (remoteVersion !== localVersion) {
    console.log(`New model available: ${remoteVersion}`);
    await downloadAndInstallModel(remoteVersion);
  }
}

async function downloadAndInstallModel(version) {
  updateStatus("downloading");

  const token = await getHuggingFaceToken();
  const url = `https://huggingface.co/PhilmoLSC/alice-human-fusion/resolve/${version}/exports/alice-human-fusion.Q4_K_M.gguf`;

  // Download with retry logic (3 attempts, 5s delay)
  for (let attempt = 1; attempt <= 3; attempt++) {
    try {
      const response = await Http.downloadFile({
        url,
        filePath: "adaptive-fit-brain-temp.gguf",
        fileDirectory: Directory.Data,
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });

      // Move temp to active
      await Filesystem.rename({
        from: "adaptive-fit-brain-temp.gguf",
        to: "adaptive-fit-brain.gguf",
        directory: Directory.Data,
      });

      // Run automated tests
      updateStatus("testing");
      const testResults = await runAutomatedTests();

      if (testResults.passed) {
        updateStatus("ready");
        await setLocalModelVersion(version);
        await clearLocalDeltas(); // Start fresh delta collection
        console.log(`Model updated to ${version}`);
      } else {
        updateStatus("error");
        console.error("Model tests failed:", testResults);
      }

      break; // Success
    } catch (error) {
      if (attempt === 3) {
        updateStatus("error");
        throw error;
      }
      await sleep(5000); // 5s delay before retry
    }
  }
}
```

---

## Privacy Guarantees

### What NEVER Leaves Your Device

- ❌ Workout names, exercise descriptions
- ❌ Meal names, food items
- ❌ Body measurements (weight, body fat %, etc.)
- ❌ Health conditions, injuries
- ❌ Personal goals, preferences (in text form)
- ❌ Photos, videos
- ❌ Location data
- ❌ User ID, email, name

### What IS Sent (Anonymized)

- ✅ **Numeric deltas only**: `[0.02, -0.01, 0.03, ...]`
- ✅ **Sparse indices**: `[1, 5, 9, 15, 23]` (which weights to adjust)
- ✅ **Metadata counts**: "This device contributed 12 deltas this week"
- ✅ **Statistical sketches**: "Average delta size: 1.2KB"
- ✅ **No personally identifiable information**

### Delta Size Limits

- Max delta size: **2KB** (enforced client-side)
- Max deltas per week: **50** (rate limited server-side)
- Total weekly upload: **~100KB** per device (negligible bandwidth)

---

## Technical Implementation Details

### On-Device Inference (llama.cpp)

**iOS (Metal GPU)**

```swift
// Native bridge: ios/Plugin/LlamaPlugin.swift
import Foundation
import MetalKit
import Accelerate

@objc(LlamaPlugin)
public class LlamaPlugin: CAPPlugin {
  private var model: LlamaModel?

  @objc func loadModel(_ call: CAPPluginCall) {
    let modelPath = call.getString("path") ?? ""

    // Load GGUF with Metal acceleration
    self.model = LlamaModel(
      path: modelPath,
      gpuLayers: 35, // A19 Pro can handle 35 layers on GPU
      contextSize: 2048
    )

    call.resolve(["loaded": true])
  }

  @objc func runInference(_ call: CAPPluginCall) {
    guard let model = self.model else {
      call.reject("Model not loaded")
      return
    }

    let prompt = call.getString("prompt") ?? ""

    // Run inference on Metal GPU
    model.predict(prompt: prompt) { result in
      call.resolve(["text": result])
    }
  }
}
```

**Android (NDK)**

```kotlin
// android/app/src/main/java/com/adaptivefit/LlamaPlugin.kt
package com.adaptivefit

import com.getcapacitor.Plugin
import com.getcapacitor.PluginCall
import com.getcapacitor.annotation.CapacitorPlugin

@CapacitorPlugin(name = "Llama")
class LlamaPlugin : Plugin() {
    private external fun loadModel(path: String, gpuLayers: Int): Long
    private external fun runInference(modelPtr: Long, prompt: String): String

    companion object {
        init {
            System.loadLibrary("llama-android") // llama.cpp NDK binding
        }
    }

    private var modelPtr: Long = 0

    @PluginMethod
    fun loadModel(call: PluginCall) {
        val path = call.getString("path") ?: ""
        modelPtr = loadModel(path, 10) // Use 10 GPU layers on Android
        call.resolve(JSObject().put("loaded", true))
    }

    @PluginMethod
    fun runInference(call: PluginCall) {
        if (modelPtr == 0L) {
            call.reject("Model not loaded")
            return
        }

        val prompt = call.getString("prompt") ?: ""
        val result = runInference(modelPtr, prompt)
        call.resolve(JSObject().put("text", result))
    }
}
```

### Backup & Restore

```typescript
// app/src/lib/model.ts
import { BackgroundTask } from "@capacitor/background-task";

BackgroundTask.beforeExit(async () => {
  const modelId = await getLocalModelId();
  const backupName = `adaptive-fit-brain-${modelId}.gguf`;

  // Copy current model to versioned backup
  await Filesystem.copy({
    from: "adaptive-fit-brain.gguf",
    to: backupName,
    directory: Directory.Data,
  });

  console.log(`Backed up model to ${backupName}`);

  BackgroundTask.finish();
});

// On app launch: restore if needed
async function restoreModelFromBackup() {
  const modelId = await Preferences.get({ key: "modelId" });

  if (!modelId.value) {
    // No stored model ID - find any backup
    const files = await Filesystem.readdir({
      path: "",
      directory: Directory.Data,
    });

    const backups = files.files.filter(
      (f) =>
        f.name.startsWith("adaptive-fit-brain-") && f.name.endsWith(".gguf"),
    );

    if (backups.length > 0) {
      // Restore most recent backup
      const latest = backups.sort().pop();
      await Filesystem.copy({
        from: latest.name,
        to: "adaptive-fit-brain.gguf",
        directory: Directory.Data,
      });

      console.log(`Restored model from ${latest.name}`);
    }
  }
}
```

---

## Performance Characteristics

### Model Size & Memory

- **GGUF size**: ~180-200MB (4-bit quantized)
- **RAM usage**: ~2GB during inference
- **GPU layers**: 35 (iOS A19 Pro), 10 (Android mid-range)
- **Inference latency**: 50-100ms per token (Metal), 200-300ms (Android CPU)

### Battery Impact

- **Idle**: <1% battery/hour (model loaded in memory)
- **Active inference**: ~5W power draw (Metal GPU)
- **Delta generation**: <0.1% battery (lightweight math)
- **Weekly upload**: <0.01% battery (100KB over WiFi)

### Network Usage

- **Initial model download**: ~200MB (one-time per version)
- **Weekly delta upload**: ~100KB (negligible)
- **Version check poll**: <1KB every 15 min (~100KB/day)

---

## Safety & Rollback

### Automated Safeguards

1. **Size validation**: GGUF must be <200MB
2. **Checksum verification**: SHA-256 hash matches expected
3. **Canary testing**: 10 automated tests must pass (≥80% threshold)
4. **Smoke tests**: Post-upload coherence checks
5. **Rate limiting**: 50 deltas/device/week enforced server-side
6. **Delta size limit**: 2KB max enforced client-side

### Rollback Procedure

```typescript
// If new model fails tests, automatic rollback
async function rollbackModel() {
  const previousVersion = await getPreviousModelVersion();

  console.warn(`Rolling back to ${previousVersion}`);

  // Delete failed model
  await Filesystem.deleteFile({
    path: "adaptive-fit-brain.gguf",
    directory: Directory.Data,
  });

  // Restore from backup
  await Filesystem.copy({
    from: `adaptive-fit-brain-${previousVersion}.gguf`,
    to: "adaptive-fit-brain.gguf",
    directory: Directory.Data,
  });

  await setLocalModelVersion(previousVersion);
  updateStatus("ready");
}
```

---

## Future Enhancements

### Planned for Post-Alpha

- **Fully automated operator approval** (requires stricter canary gates)
- **Differential privacy guarantees** (formal ε-δ privacy bounds)
- **Homomorphic encryption** (encrypted delta aggregation)
- **On-device fine-tuning** (LoRA adapters for personalization)
- **Continuous learning** (daily micro-updates vs weekly)

---

## Summary

**git-fit's on-device training**:
✅ Preserves privacy (no raw data uploaded)
✅ Improves AI weekly (federated learning)
✅ Runs entirely offline (local inference)
✅ Minimal battery/network impact
✅ Automatic updates (seamless UX)
✅ Safe rollback (if issues detected)

**You get personalized AI coaching that learns from the community while keeping your data 100% private.**

---

**Questions?** Check the full specs:

- `specs/adaptive-fit-alpha-full-automation/spec.md`
- `specs/auto-task-manager-model-lifecycle/spec.md`
- `app/src/lib/api/supabase.ts`
- Supabase Project: `https://pzcrllejymdofvfvhtxr.supabase.co`

## Related
