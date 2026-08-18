---
title: EVOLoRA_Mesh_Training_Data_Spec
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOLoRA_Mesh_Training_Data_Spec.md"]
updated: 2026-07-24
---

# EVOLoRA Mesh LoRA Adapter Training Data Specification

**Critical Requirement**: All LoRAs are **knowledge-only** (not tone/personality).
The base model (`alice-phi3-q4.gguf`) already contains Alice's personality and tone.

---

## Adapter Types & Training Data

### U (User Adapter) - Per-User Knowledge

**Purpose**: Encode user-specific knowledge, preferences, and patterns.

**Training Data Sources**:

1. **User Workout History**
   - Exercise preferences (favorite exercises, avoided exercises)
   - Typical rep ranges, weights, rest periods
   - Workout frequency and patterns
   - Progression history (PRs, plateaus, deloads)

2. **User Responses & Interactions**
   - Questions the user frequently asks
   - Topics they're interested in (nutrition, recovery, specific goals)
   - Communication style preferences (brief vs detailed)
   - Past conversations and context

3. **User Profile Data**
   - Goals (strength, hypertrophy, endurance, weight loss)
   - Experience level
   - Equipment available
   - Time constraints
   - Injury history / limitations

4. **User Plan Adherence**
   - Which exercises they complete vs skip
   - How they modify plans
   - Divergence patterns (intended vs executed)

**Training Format**:

```json
{
  "instruction": "User prefers 3x8-10 rep ranges for upper body",
  "context": "User has been training for 2 years, focuses on hypertrophy",
  "response": "For your upper body hypertrophy goals, 3 sets of 8-10 reps is ideal. You can progress by adding weight when you hit 10 reps on all sets."
}
```

**Data Collection**:

- Extract from `OnDeviceWorkoutLog` (workout history)
- Extract from `TrainingPlanRecord` (plan adherence)
- Extract from chat messages (user questions/preferences)
- Aggregate over user's lifetime in app

---

### T (Trainer Adapter) - Per-Trainer Knowledge

**Purpose**: Encode trainer-specific coaching knowledge, plan templates, and client patterns.

**Training Data Sources**:

1. **Trainer Plan Templates**
   - Trainer's preferred program structures
   - Exercise selection patterns
   - Periodization strategies
   - Progression schemes

2. **Trainer Edits & Modifications**
   - How trainer modifies user plans
   - Trainer's approval/rejection patterns
   - Trainer's plan adjustments for specific clients
   - Trainer's safety interventions

3. **Trainer Client Patterns**
   - Patterns trainer observes across their clients
   - Common modifications trainer makes
   - Trainer's coaching philosophy (evidence-based, intuitive, etc.)

4. **Trainer Communication Style**
   - How trainer explains concepts
   - Trainer's preferred terminology
   - Trainer's approach to motivation/accountability

**Training Format**:

```json
{
  "instruction": "Trainer modifies plan for client with knee injury",
  "context": "Client has patellar tendinitis, trainer replaces squats with leg press",
  "response": "I've replaced back squats with leg press to reduce knee stress while maintaining quad development. Use 3x10-12 with controlled tempo."
}
```

**Data Collection**:

- Extract from `trainer_approvals` table (approval decisions)
- Extract from plan modification logs (when trainer edits plans)
- Extract from trainer-side chat messages
- Aggregate trainer's actions across all clients

**Note**: Each trainer gets their own T adapter (per-trainer LoRA).

---

### GU (Global User Adapter) - Aggregated User Knowledge

**Purpose**: Encode common patterns, best practices, and knowledge shared across all users.

**Training Data Sources**:

1. **Aggregated User Patterns**
   - Common progression patterns (what works for most users)
   - Typical beginner mistakes and corrections
   - Common goal-oriented approaches
   - Effective exercise combinations

2. **General Fitness Knowledge**
   - Evidence-based training principles
   - Common exercise form cues
   - Nutrition basics (macros, timing)
   - Recovery best practices

3. **Cross-User Insights**
   - What works for similar user profiles
   - Common adaptations for common goals
   - Typical timelines for progress

4. **Public Knowledge Base**
   - Exercise science facts
   - Safety guidelines
   - General programming principles

**Training Format**:

```json
{
  "instruction": "How should beginners progress in strength training?",
  "context": "General user population, no specific user context",
  "response": "Beginners should focus on linear progression: add 2.5-5lbs per session for upper body, 5-10lbs for lower body. Start with 3x5-8 reps, 3-4 days per week."
}
```

**Data Collection**:

- Aggregate from all users (anonymized)
- Extract common patterns from workout logs
- Include general fitness knowledge from training datasets
- Federated learning aggregation (weekly/monthly)

**Privacy**: Must be anonymized - no user-identifiable information.

---

### GT (Global Trainer Adapter) - Aggregated Trainer Knowledge

**Purpose**: Encode best practices, patterns, and knowledge shared across all trainers.

**Training Data Sources**:

1. **Aggregated Trainer Patterns**
   - Common coaching strategies that work
   - Typical plan structures trainers use
   - Common modifications trainers make
   - Effective trainer-client communication patterns

2. **Trainer Best Practices**
   - Evidence-based coaching approaches
   - Common periodization strategies
   - Safety protocols trainers follow
   - Client management strategies

3. **Cross-Trainer Insights**
   - What experienced trainers do differently
   - Common trainer mistakes and corrections
   - Effective trainer interventions

4. **Professional Knowledge**
   - Exercise science principles
   - Program design best practices
   - Client assessment approaches

**Training Format**:

```json
{
  "instruction": "How do experienced trainers handle client plateaus?",
  "context": "General trainer knowledge, aggregated patterns",
  "response": "Experienced trainers typically: 1) Deload 10-20% for 1 week, 2) Change exercise selection or rep ranges, 3) Assess recovery/sleep/stress, 4) Re-evaluate goals. The key is systematic troubleshooting."
}
```

**Data Collection**:

- Aggregate from all trainers (anonymized)
- Extract patterns from trainer approvals/edits
- Include professional coaching knowledge
- Federated learning aggregation (weekly/monthly)

**Privacy**: Must be anonymized - no trainer-identifiable information.

---

## Training Data Requirements

### Knowledge-Only Constraint

**What to Include**:

- ✅ Factual knowledge (exercise science, programming principles)
- ✅ User/trainer preferences and patterns
- ✅ Historical data (what worked, what didn't)
- ✅ Context-specific knowledge (goals, equipment, limitations)

**What to Exclude**:

- ❌ Tone/personality (already in base model)
- ❌ Response style (already in base model)
- ❌ Greeting patterns (already in base model)
- ❌ Emotional responses (already in base model)

### Data Format

All training data should follow this structure:

```json
{
  "instruction": "Context-specific question or scenario",
  "context": "Relevant background (user profile, trainer style, etc.)",
  "response": "Knowledge-based answer (facts, patterns, recommendations)"
}
```

### Data Collection Pipeline

1. **User Data (U adapter)**

   ```
   OnDeviceWorkoutLog → Extract patterns → Format as training examples
   TrainingPlanRecord → Extract preferences → Format as training examples
   Chat messages → Extract questions/context → Format as training examples
   ```

2. **Trainer Data (T adapter)**

   ```
   trainer_approvals → Extract decisions → Format as training examples
   Plan modifications → Extract edits → Format as training examples
   Trainer chat → Extract coaching patterns → Format as training examples
   ```

3. **Global Data (GU/GT adapters)**
   ```
   Aggregate all users (anonymized) → Extract common patterns
   Aggregate all trainers (anonymized) → Extract common patterns
   Federated learning → Weekly aggregation → Format as training examples
   ```

---

## Training Process

### Step 1: Data Extraction

**For U adapter (per user)**:

```python
# Extract from user's workout logs
user_logs = get_user_workout_logs(user_id)
patterns = extract_patterns(user_logs)  # Exercise preferences, rep ranges, etc.
training_data = format_as_training_examples(patterns)
```

**For T adapter (per trainer)**:

```python
# Extract from trainer's actions
trainer_approvals = get_trainer_approvals(trainer_id)
trainer_edits = get_trainer_plan_edits(trainer_id)
patterns = extract_trainer_patterns(trainer_approvals, trainer_edits)
training_data = format_as_training_examples(patterns)
```

**For GU adapter (global user)**:

```python
# Aggregate across all users (anonymized)
all_user_logs = get_all_user_logs_anonymized()
common_patterns = extract_common_patterns(all_user_logs)
training_data = format_as_training_examples(common_patterns)
```

**For GT adapter (global trainer)**:

```python
# Aggregate across all trainers (anonymized)
all_trainer_actions = get_all_trainer_actions_anonymized()
common_patterns = extract_common_patterns(all_trainer_actions)
training_data = format_as_training_examples(common_patterns)
```

### Step 2: Training

Use existing training pipeline (`train_phi3_alice.py`) but:

- Train on knowledge-only data (not tone/personality)
- Use QLoRA (4-bit) for efficiency
- Lower learning rate (focus on knowledge, not style)

### Step 3: Export & Upload

1. Export LoRA adapter to GGUF format
2. Upload to R2 storage:
   - `alice-assets/adapters/user/{userId}_lora.gguf` (U)
   - `alice-assets/adapters/trainer/{trainerId}_lora.gguf` (T)
   - `alice-assets/adapters/global/user/global_user_lora.gguf` (GU)
   - `alice-assets/adapters/global/trainer/global_trainer_lora.gguf` (GT)

---

## Example Training Data

### U Adapter Example

```json
{
  "instruction": "User typically does 3x8-10 for bench press, prefers barbell over dumbbell",
  "context": "User has been training for 2 years, focuses on upper body strength",
  "response": "For your bench press, maintain 3 sets of 8-10 reps. When you hit 10 reps on all sets, add 5lbs. Stick with barbell since you prefer it and it allows heavier loading."
}
```

### T Adapter Example

```json
{
  "instruction": "Trainer modifies plan: replaces back squats with front squats for client with lower back issues",
  "context": "Client reported lower back pain after back squats, trainer prefers front-loaded squats for this case",
  "response": "I've replaced back squats with front squats to reduce lower back stress while maintaining quad development. Use 3x8-10 with focus on upright torso position."
}
```

### GU Adapter Example

```json
{
  "instruction": "What's the typical progression for beginners in strength training?",
  "context": "General user population, no specific user",
  "response": "Beginners should use linear progression: add 2.5-5lbs per session for upper body, 5-10lbs for lower body. Start with 3x5-8 reps, train 3-4 days per week with full-body or upper/lower split."
}
```

### GT Adapter Example

```json
{
  "instruction": "How do experienced trainers handle client plateaus in strength gains?",
  "context": "General trainer knowledge, aggregated patterns",
  "response": "Experienced trainers typically: 1) Deload 10-20% for 1 week to allow recovery, 2) Change exercise selection or rep ranges to provide new stimulus, 3) Assess recovery factors (sleep, stress, nutrition), 4) Re-evaluate goals and program structure. Systematic troubleshooting is key."
}
```

---

## Data Privacy & Compliance

### User Data (U adapter)

- ✅ Stored on-device (never uploaded)
- ✅ Trained locally (federated learning)
- ✅ Only adapter weights uploaded (not raw data)

### Trainer Data (T adapter)

- ✅ Trainer owns their data
- ✅ Can opt-out of aggregation
- ✅ Only adapter weights shared (not raw data)

### Global Data (GU/GT adapters)

- ✅ Fully anonymized
- ✅ No user/trainer identifiers
- ✅ Only aggregated patterns
- ✅ Federated learning aggregation

---

## Next Steps

1. **Build data extraction pipeline** for each adapter type
2. **Create training dataset builders** that format data correctly
3. **Set up federated learning** for GU/GT aggregation
4. **Train initial adapters** on available data
5. **Upload to R2** and test with MeshRouter

---

## References

- Compliance Report: `docs/audits/EVOLoRA_Mesh_Compliance_Report.md`
- Implementation Plan: `docs/EVOLoRA_Mesh_Implementation_Plan.md`
- Training Script: `training/train_phi3_alice.py`

## Related

^[{src_rel}]
