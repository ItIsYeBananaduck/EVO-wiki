---
title: AI_SYSTEM_DOCUMENTATION
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/AI_SYSTEM_DOCUMENTATION.md
updated: 2026-07-24
---

# Enhanced AI Coaching System - Complete Documentation

## Table of Contents

1. [System Architecture Overview](#system-architecture-overview)
2. [AI Engine Components](#ai-engine-components)
3. [Preference Learning Algorithms](#preference-learning-algorithms)
4. [API Endpoints Documentation](#api-endpoints-documentation)
5. [Frontend Integration Guide](#frontend-integration-guide)
6. [Database Schema Documentation](#database-schema-documentation)
7. [Deployment Configuration](#deployment-configuration)
8. [User Guide](#user-guide)
9. [Development Guide](#development-guide)
10. [Troubleshooting](#troubleshooting)

---

## System Architecture Overview

### High-Level Architecture

The Enhanced AI Coaching System consists of five main components:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   SvelteKit     │    │   Flutter App   │    │  Alice AI       │
│   Marketplace   │    │   + llama.cpp   │◄──►│  Qwen2.5-1.5B   │
│                 │    │                 │    │                 │
├─────────────────┤    ├─────────────────┤    └─────────────────┘
│• AI Coaching    │    │• Enhanced AI    │              │
│  Service        │    │  Engine         │              ▼
│• Recommendation │    │• Safety         │    ┌─────────────────┐
│  UI Components  │    │  Constraints    │    │   Model Cache   │
│• Insights       │    │• Preference     │    │   & Fallbacks   │
│  Dashboard      │    │  Learning       │    └─────────────────┘
└─────────────────┘    └─────────────────┘
         │                       │
         ▼                       ▼
┌─────────────────┐    ┌─────────────────┐
│    Supabase     │    │   File System   │
│   Database      │    │  User Profiles  │
│                 │    │                 │
│• AI Preferences │    │• Pickle Storage │
│• Workout Tweaks │    │• Learning Data  │
│• Learning Events│    │• Model Versions │
│• Session Data   │    └─────────────────┘
└─────────────────┘
```

### Data Flow

1. **User Action** → Frontend captures workout context and exercise data
2. **Context Analysis** → AI Engine analyzes user energy, preferences, and constraints
3. **AI Processing** → Alice (Qwen2.5-1.5B) generates personalized recommendation
4. **Safety Validation** → Safety constraints applied (weight ≤110%, reps ≥80%)
5. **Recommendation Display** → Frontend shows recommendation with confidence score
6. **User Feedback** → User accepts/modifies/skips with detailed ratings
7. **Learning Update** → Preferences updated using exponential moving averages
8. **Data Storage** → All interactions stored for continuous learning

---

## AI Engine Components

### Core Classes

#### UserPreferenceProfile

```python
@dataclass
class UserPreferenceProfile:
    user_id: str
    preferred_intensity: float  # 1-10 scale
    volume_tolerance: float     # 1-10 scale
    rest_time_preference: int   # seconds
    exercise_variety: float     # 1-10 scale
    progression_rate: float     # 1-10 scale
    form_focus: float          # 1-10 scale
    time_constraints: Optional[Dict]

    # Feedback patterns (0-1 scale)
    acceptance_rate: float
    modification_frequency: float
    skip_rate: float

    # Confidence scores (0-1 scale)
    workout_confidence: float
    exercise_confidence: float
    intensity_confidence: float

    # Learning metadata
    total_interactions: int
    last_updated: str
    learning_rate: float
    adaptation_speed: float = 0.1
```

#### WorkoutContext

```python
@dataclass
class WorkoutContext:
    time_of_day: str           # morning, afternoon, evening
    day_of_week: int          # 1-7 (Monday-Sunday)
    user_energy: Optional[int] # 1-10 scale
    user_motivation: Optional[int]
    available_time: Optional[int]  # minutes
    equipment_availability: Optional[List[str]]
    gym_crowding: Optional[str]    # low, medium, high
    previous_performance: Optional[Dict]
    wearable_data: Optional[Dict]
```

#### AIRecommendation

```python
@dataclass
class AIRecommendation:
    type: str                 # increase_weight, adjust_reps, etc.
    original_value: Any
    suggested_value: Any
    confidence: float         # 0-1 scale
    reasoning: str
    factors: List[str]
    expected_outcome: str
    risk_assessment: str      # low, medium, high
    alternative_options: Optional[List[Dict]]
```

### Safety Constraints

The AI engine implements strict safety constraints:

```python
SAFETY_CONSTRAINTS = {
    'MAX_WEIGHT_INCREASE': 0.10,    # 10% max increase
    'MIN_REP_RETENTION': 0.80,      # 80% min reps
    'MIN_REST_TIME': 30,            # 30 seconds minimum
    'MAX_VOLUME_INCREASE': 0.15,    # 15% max volume
    'FATIGUE_THRESHOLD': 8.0,       # Above 8/10 triggers caution
    'FORM_DEGRADATION_LIMIT': 6.0   # Below 6/10 triggers reduction
}
```

### Fallback Mechanisms

When the AI model is unavailable:

1. **Rule-based logic** for common scenarios
2. **Conservative recommendations** with lower confidence
3. **Safety-first approach** prioritizing injury prevention
4. **Graceful degradation** with user notification

---

## Preference Learning Algorithms

### Exponential Moving Average (EMA)

User preferences are updated using EMA to balance historical data with recent feedback:

```python
def update_preference(self, current_value: float, new_signal: float, learning_rate: float = 0.05) -> float:
    """
    Update preference using exponential moving average

    Args:
        current_value: Current preference value (0-10)
        new_signal: New feedback signal (0-10)
        learning_rate: Learning rate (0-1, default 0.05)

    Returns:
        Updated preference value
    """
    return current_value + learning_rate * (new_signal - current_value)
```

### Learning Signal Extraction

From user feedback, the system extracts learning signals:

```python
def extract_learning_signals(self, feedback: Dict) -> Dict:
    signals = {}

    # Intensity preference from difficulty rating
    if feedback.get('difficulty_rating'):
        target_difficulty = 7.0  # Sweet spot
        if feedback['difficulty_rating'] < 5:
            signals['intensity_increase'] = 1.0
        elif feedback['difficulty_rating'] > 8:
            signals['intensity_decrease'] = 1.0

    # Volume tolerance from completion and perceived exertion
    if feedback.get('perceived_exertion') and feedback.get('completion_time'):
        if feedback['perceived_exertion'] < 6 and feedback['completion_time'] < expected_time:
            signals['volume_increase'] = 0.8

    # Form focus from form quality ratings
    if feedback.get('form_quality') and feedback['form_quality'] < 7:
        signals['form_emphasis'] = 1.0

    return signals
```

### Confidence Scoring

Confidence in recommendations is calculated based on:

```python
def calculate_confidence(self, user_profile: UserPreferenceProfile, context: WorkoutContext) -> float:
    base_confidence = 0.7

    # Increase confidence based on interaction history
    interaction_bonus = min(0.2, user_profile.total_interactions * 0.001)

    # Adjust for context familiarity
    time_familiarity = 0.1 if context.time_of_day in user_profile.preferred_times else 0

    # Account for recent acceptance rate
    acceptance_factor = user_profile.acceptance_rate * 0.2

    # Reduce confidence for edge cases
    energy_penalty = 0.1 if (context.user_energy or 5) < 4 else 0

    confidence = base_confidence + interaction_bonus + time_familiarity + acceptance_factor - energy_penalty

    return max(0.3, min(1.0, confidence))  # Clamp between 0.3 and 1.0
```

---

## API Endpoints Documentation

### POST /recommendation

Get personalized AI recommendation for an exercise.

**Request Body:**

```json
{
  "user_id": "string",
  "exercise_data": {
    "exercise_name": "string",
    "planned_sets": "integer",
    "planned_reps": "integer",
    "planned_weight": "number",
    "current_set": "integer"
  },
  "context": {
    "time_of_day": "morning|afternoon|evening",
    "day_of_week": "integer (1-7)",
    "user_energy": "integer (1-10, optional)",
    "user_motivation": "integer (1-10, optional)",
    "available_time": "integer (minutes, optional)",
    "equipment_availability": "array of strings (optional)",
    "gym_crowding": "low|medium|high (optional)"
  },
  "event_type": "string"
}
```

**Response:**

```json
{
  "success": true,
  "recommendation": {
    "id": "string",
    "type": "increase_weight|decrease_weight|adjust_reps|modify_rest|change_exercise|add_warmup|suggest_form_focus",
    "original_value": "any",
    "suggested_value": "any",
    "confidence": "float (0-1)",
    "reasoning": "string",
    "factors": "array of strings",
    "expected_outcome": "string",
    "risk_assessment": "low|medium|high",
    "alternative_options": "array (optional)"
  },
  "user_id": "string",
  "event_type": "string",
  "timestamp": "string (ISO 8601)"
}
```

### POST /feedback

Submit user feedback on a recommendation.

**Request Body:**

```json
{
  "user_id": "string",
  "tweak_id": "string",
  "feedback": {
    "accepted": "boolean",
    "modified": "boolean",
    "skipped": "boolean",
    "difficulty_rating": "integer (1-10, optional)",
    "effectiveness_rating": "integer (1-10, optional)",
    "custom_adjustment": "any (optional)",
    "completion_time": "integer (seconds, optional)",
    "perceived_exertion": "integer (1-10, optional)",
    "form_quality": "integer (1-10, optional)",
    "notes": "string (optional)"
  }
}
```

**Response:**

```json
{
  "success": true,
  "message": "Feedback processed successfully",
  "user_id": "string",
  "tweak_id": "string",
  "timestamp": "string (ISO 8601)"
}
```

### GET /user/{user_id}/insights

Get comprehensive user insights and analytics.

**Response:**

```json
{
  "success": true,
  "insights": {
    "preference_summary": {
      "preferred_intensity": "float",
      "volume_tolerance": "float",
      "progression_rate": "float",
      "workout_style": "string"
    },
    "performance_trends": {
      "acceptance_rate": "float (0-1)",
      "improvement_rate": "float (0-1)",
      "consistency_score": "float (0-1)",
      "weak_areas": "array of strings",
      "strong_areas": "array of strings"
    },
    "ai_effectiveness": {
      "recommendation_accuracy": "float (0-1)",
      "user_satisfaction": "float (0-1)",
      "personalization_level": "float (0-1)"
    },
    "next_focus_areas": "array of strings"
  },
  "generated_at": "string (ISO 8601)"
}
```

### GET /user/{user_id}/profile

Get user preference profile.

**Response:**

```json
{
  "success": true,
  "profile": {
    "user_id": "string",
    "preferred_intensity": "float",
    "volume_tolerance": "float",
    "rest_time_preference": "integer",
    "exercise_variety": "float",
    "progression_rate": "float",
    "form_focus": "float",
    "time_constraints": "object (optional)",
    "acceptance_rate": "float (0-1)",
    "modification_frequency": "float (0-1)",
    "skip_rate": "float (0-1)",
    "workout_confidence": "float (0-1)",
    "exercise_confidence": "float (0-1)",
    "intensity_confidence": "float (0-1)",
    "total_interactions": "integer",
    "last_updated": "string (ISO 8601)",
    "learning_rate": "float"
  },
  "generated_at": "string (ISO 8601)"
}
```

### POST /event (Legacy)

Legacy endpoint for backward compatibility.

**Request Body:**

```json
{
  "user_id": "string",
  "event": "string",
  "user_data": "object",
  "context": "object"
}
```

---

## Frontend Integration Guide

### Service Layer Usage

```typescript
import {
  aiCoaching,
  createExerciseData,
  createFeedback,
  getWorkoutContext,
} from "./lib/ai-coaching";

// Get recommendation
const exerciseData = createExerciseData("Bench Press", 3, 10, 135, 1);
const context = getWorkoutContext();
context.user_energy = 8;
context.user_motivation = 7;

try {
  const recommendation = await aiCoaching.getRecommendation(
    "user123",
    exerciseData,
    context,
  );
  console.log("Recommendation:", recommendation);
} catch (error) {
  console.error("Failed to get recommendation:", error);
}

// Submit feedback
const feedback = createFeedback("user123", recommendation.id, true, {
  difficultyRating: 7,
  effectivenessRating: 8,
  perceivedExertion: 6,
  formQuality: 9,
  notes: "Felt perfect!",
});

await aiCoaching.provideFeedback(feedback);
```

### Component Integration

```svelte
<!-- In your workout component -->
<script lang="ts">
  import AIRecommendationCard from '$lib/components/AIRecommendationCard.svelte';
  import { createExerciseData, getWorkoutContext } from '$lib/ai-coaching.js';

  let userId = 'current_user_id';
  let exerciseData = createExerciseData('Squat', 3, 8, 185);
  let workoutContext = getWorkoutContext();

  // Update context with user inputs
  workoutContext.user_energy = 7;
  workoutContext.available_time = 45;

  function handleFeedbackSubmitted(feedback) {
    console.log('Feedback submitted:', feedback);
    // Update UI, show success message, etc.
  }
</script>

<AIRecommendationCard
  {userId}
  {exerciseData}
  {workoutContext}
  onFeedbackSubmitted={handleFeedbackSubmitted}
/>
```

### Insights Dashboard

```svelte
<!-- User insights page -->
<script lang="ts">
  import AIInsightsDashboard from '$lib/components/AIInsightsDashboard.svelte';

  export let userId: string;
</script>

<div class="insights-page">
  <h1>Your AI Coaching Insights</h1>
  <AIInsightsDashboard {userId} />
</div>
```

---

## Database Schema Documentation

### AI User Preferences Table (`aiUserPreferences`)

Stores comprehensive user preference profiles for AI personalization.

**Schema:**

```typescript
aiUserPreferences: {
  userId: string;           // User identifier
  preferences: {
    preferred_intensity: number;     // 1-10 intensity preference
    volume_tolerance: number;        // 1-10 volume tolerance
    rest_time_preference: number;    // Preferred rest time (seconds)
    exercise_variety: number;        // 1-10 variety preference
    progression_rate: number;        // 1-10 progression speed
    form_focus: number;             // 1-10 form importance
    time_constraints?: {
      typical_session_length: number;
      preferred_times: string[];
      rush_tolerance: number;
    };
  };
  feedback_patterns: {
    acceptance_rate: number;         // 0-1 recommendation acceptance
    modification_frequency: number;  // 0-1 modification rate
    skip_rate: number;              // 0-1 skip rate
  };
  confidence_scores: {
    workout_confidence: number;      // 0-1 overall confidence
    exercise_confidence: number;     // 0-1 exercise-specific confidence
    intensity_confidence: number;    // 0-1 intensity confidence
  };
  learning_metadata: {
    total_interactions: number;
    last_updated: string;
    learning_rate: number;
    adaptation_speed: number;
  };
  createdAt: number;
  updatedAt: number;
}
```

**Indexes:**

- `by_userId` on `userId`
- `by_updatedAt` on `updatedAt`

### AI Workout Tweaks Table (`aiWorkoutTweaks`)

Stores all AI recommendations and user responses for learning.

**Schema:**

```typescript
aiWorkoutTweaks: {
  userId: string;
  tweakId: string;         // Unique recommendation ID
  exercise: {
    name: string;
    type: string;
    muscle_groups: string[];
    difficulty_level?: number;
  };
  recommendation: {
    type: string;
    original_value: any;
    suggested_value: any;
    confidence: number;
    reasoning: string;
    factors: string[];
    expected_outcome: string;
    risk_assessment: string;
    alternative_options?: any[];
  };
  context: {
    workout_phase: string;
    time_of_day: string;
    day_of_week: number;
    user_energy?: number;
    user_motivation?: number;
    available_time?: number;
    equipment_availability?: string[];
    gym_crowding?: string;
    environmental_factors?: {
      temperature?: number;
      noise_level?: string;
      crowding?: string;
    };
  };
  user_response?: {
    accepted: boolean;
    modified: boolean;
    skipped: boolean;
    actual_value?: any;
    feedback_ratings?: {
      difficulty: number;
      effectiveness: number;
      satisfaction: number;
      perceived_exertion: number;
      form_quality: number;
    };
    completion_time?: number;
    notes?: string;
    responded_at?: number;
  };
  createdAt: number;
  updatedAt: number;
}
```

**Indexes:**

- `by_userId` on `userId`
- `by_tweakId` on `tweakId`
- `by_createdAt` on `createdAt`

### AI Learning Events Table (`aiLearningEvents`)

Tracks learning events for AI model improvement.

**Schema:**

```typescript
aiLearningEvents: {
  userId: string;
  eventType: string;       // feedback_processed, preference_updated, etc.
  eventData: {
    exercise_name?: string;
    recommendation_type?: string;
    user_action?: string;
    context_factors?: string[];
    performance_metrics?: {
      accuracy?: number;
      user_satisfaction?: number;
      outcome_quality?: number;
    };
    learning_signals?: {
      preference_shift?: number;
      confidence_change?: number;
      pattern_strength?: number;
    };
  };
  metadata: {
    model_version: string;
    algorithm_version: string;
    confidence_threshold: number;
    processing_time?: number;
  };
  timestamp: number;
}
```

**Indexes:**

- `by_userId` on `userId`
- `by_eventType` on `eventType`
- `by_timestamp` on `timestamp`

### AI Model Versions Table (`aiModelVersions`)

Tracks AI model versions and their performance metrics.

**Schema:**

```typescript
aiModelVersions: {
  version: string;
  modelType: string;       // qwen25-1.5b, alice, etc.
  capabilities: string[];
  performance_metrics: {
    accuracy: number;
    response_time: number;
    memory_usage: number;
    personalization_score: number;
  };
  deployment_info: {
    deployed_at: string;
    environment: string;
    config_hash: string;
  };
  active: boolean;
  createdAt: number;
}
```

**Indexes:**

- `by_modelType` on `modelType`
- `by_version` on `version`
- `by_active` on `active`

### Workout Sessions Table (`workoutSessions`)

Comprehensive workout session data with AI interaction tracking.

**Schema:**

```typescript
workoutSessions: {
  userId: string;
  sessionId: string;
  sessionData: {
    workout_type: string;
    duration: number;
    total_exercises: number;
    completed_exercises: number;
    ai_recommendations_count: number;
    ai_recommendations_accepted: number;
    overall_difficulty: number;
    overall_satisfaction: number;
    energy_before?: number;
    energy_after?: number;
    perceived_exertion?: number;
    notes?: string;
  };
  exercises: {
    name: string;
    sets: number;
    reps: number[];
    weights: number[];
    rest_times: number[];
    ai_tweaks_applied: number;
    completion_quality: number;
  }[];
  wearableData?: {
    heart_rate_avg?: number;
    heart_rate_max?: number;
    heart_rate_zones?: {
      zone1: number;
      zone2: number;
      zone3: number;
      zone4: number;
      zone5: number;
    };
    calories_burned?: number;
    steps?: number;
    active_time?: number;
  };
  createdAt: number;
  updatedAt: number;
}
```

**Indexes:**

- `by_userId` on `userId`
- `by_sessionId` on `sessionId`
- `by_createdAt` on `createdAt`

---

## Deployment Configuration

### Environment Variables

**Backend (FastAPI):**

```bash
# AI Model Configuration
HF_TOKEN=your_huggingface_token
MODEL_REPO=PhilmoLSC/philmoLSC
MODEL_NAME=alice-qwen25-1.5b-q4_k_m
MODEL_CACHE_DIR=/app/model_cache

# AI Engine Settings
AI_ENGINE_ENABLED=true
PREFERENCE_LEARNING_RATE=0.05
SAFETY_CONSTRAINTS_ENABLED=true
FALLBACK_MODE_ENABLED=true

# Database Configuration
DATABASE_URL=your_database_connection_string
REDIS_URL=your_redis_url_for_caching

# Performance Settings
MAX_MODEL_MEMORY_MB=1024
MODEL_TIMEOUT_SECONDS=30
CONCURRENT_REQUESTS_LIMIT=10

# Logging
LOG_LEVEL=INFO
AI_LOGGING_ENABLED=true
```

**Frontend (SvelteKit):**

```bash
# API Configuration
VITE_API_URL=https://your-backend-api.com
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key

# Feature Flags
VITE_AI_COACHING_ENABLED=true
VITE_INSIGHTS_DASHBOARD_ENABLED=true
VITE_RECOMMENDATION_UI_ENABLED=true

# Analytics
VITE_ANALYTICS_ENABLED=true
VITE_AI_ANALYTICS_TRACKING=true
```

### Docker Configuration

**Backend Dockerfile additions:**

```dockerfile
# Install AI/ML dependencies
RUN pip install torch==2.0.1 --index-url https://download.pytorch.org/whl/cpu
RUN pip install transformers==4.35.0
RUN pip install scikit-learn==1.3.0

# Create model cache directory
RUN mkdir -p /app/model_cache
RUN mkdir -p /app/user_profiles

# Set model cache permissions
RUN chown -R app:app /app/model_cache
RUN chown -R app:app /app/user_profiles

# Environment variables
ENV MODEL_CACHE_DIR=/app/model_cache
ENV USER_PROFILES_DIR=/app/user_profiles
ENV PYTHONPATH=/app
```

### Production Optimizations

**AI Model Optimization:**

- Use CPU-optimized PyTorch for cost efficiency
- Implement model quantization for faster inference
- Enable model caching with Redis
- Batch multiple requests when possible

**Database Optimization:**

- Implement proper indexing on AI tables
- Use connection pooling for database access
- Regular cleanup of old learning events
- Partition large tables by date

**Caching Strategy:**

- Cache user profiles in Redis (1 hour TTL)
- Cache model outputs for identical requests (30 min TTL)
- Cache user insights (4 hours TTL)
- Use CDN for static AI model files

---

## User Guide

### Getting AI-Powered Recommendations

1. **Start Your Workout**: Begin any exercise in the app
2. **Context Input**: Provide energy level (1-10) and available time
3. **AI Analysis**: The system analyzes your preferences and workout context
4. **Recommendation Display**: Review the personalized suggestion with confidence score
5. **Action Choice**: Accept, modify, or skip the recommendation
6. **Feedback**: Rate difficulty, effectiveness, and perceived exertion
7. **Learning**: Your preferences are updated for better future recommendations

### Understanding Recommendations

**Confidence Scores:**

- **90-100%**: High confidence based on extensive data
- **70-89%**: Good confidence with solid user history
- **50-69%**: Moderate confidence, consider carefully
- **Below 50%**: Low confidence, use caution

**Risk Assessment:**

- **Low Risk**: Safe progression within established limits
- **Medium Risk**: Slight increase in challenge, monitor closely
- **High Risk**: Significant change, proceed with caution

**Recommendation Types:**

- **Increase Weight**: Add load for progressive overload
- **Adjust Reps**: Modify repetitions for volume/intensity balance
- **Modify Rest**: Change rest time based on recovery needs
- **Form Focus**: Emphasize technique over load progression
- **Exercise Change**: Alternative exercise for better fit

### Using the Insights Dashboard

**Preference Summary:**

- View your workout preferences on 1-10 scales
- Track how your preferences evolve over time
- Understand your workout style classification

**Performance Trends:**

- Monitor recommendation acceptance rate
- Track improvement and consistency scores
- Identify strong areas and areas for development

**AI Effectiveness:**

- See how accurately the AI predicts your preferences
- View user satisfaction scores
- Understand personalization level

**Focus Areas:**

- Get recommended areas to focus on next
- Track progress toward specific goals
- Adjust training emphasis based on insights

---

## Development Guide

### Setting Up the Development Environment

1. **Clone Repository:**

```bash
git clone <repository-url>
cd git-fit
```

2. **Install Dependencies:**

```bash
# Backend
pip install -r requirements.txt

# Frontend
cd app
npm install
```

3. **Environment Configuration:**

```bash
# Copy example environment files
cp .env.example .env
cp app/.env.example app/.env

# Configure AI model settings
export HF_TOKEN=your_token
export MODEL_REPO=PhilmoLSC/philmoLSC
```

4. **Start Development Servers:**

```bash
# Backend (Terminal 1)
uvicorn app:app --reload

# Frontend (Terminal 2)
cd app && npm run dev

# Supabase local (Terminal 3)
pnpm supabase start
```

### Testing the AI System

**Run Backend Tests:**

```bash
# Unit tests
pytest app/tests/test_ai_backend.py -v

# Integration tests
pytest app/tests/test_integration.py -v

# Performance tests
pytest app/tests/test_performance.py -v
```

**Run Frontend Tests:**

```bash
cd app
npm test ai-coaching.test.ts
npm run test:coverage
```

### Adding New AI Features

1. **Define the Feature**: Document the new capability in the AI engine
2. **Update Data Models**: Modify `UserPreferenceProfile` or `WorkoutContext` if needed
3. **Implement Logic**: Add the feature to `enhanced_ai_engine.py`
4. **Add API Endpoint**: Create new FastAPI endpoint if external access needed
5. **Update Frontend**: Add UI components and service methods
6. **Add to Database**: Create new tables or fields if persistent data needed
7. **Write Tests**: Comprehensive test coverage for the new feature
8. **Update Documentation**: Document the feature and its usage

### Debugging AI Recommendations

**Enable Debug Logging:**

```python
import logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger('enhanced_ai_engine')
```

**Trace Recommendation Flow:**

```python
def generate_personalized_recommendation(self, user_id, exercise_data, context, event_type):
    logger.debug(f"Generating recommendation for {user_id}")
    logger.debug(f"Exercise: {exercise_data}")
    logger.debug(f"Context: {context}")

    # ... rest of method

    logger.debug(f"Final recommendation: {recommendation}")
    return recommendation
```

**Validate Learning Updates:**

```python
def process_user_feedback(self, user_id, tweak_id, feedback):
    old_profile = self.get_user_profile(user_id)
    logger.debug(f"Profile before: {old_profile.preferred_intensity}")

    # ... processing logic

    new_profile = self.get_user_profile(user_id)
    logger.debug(f"Profile after: {new_profile.preferred_intensity}")
    logger.debug(f"Change: {new_profile.preferred_intensity - old_profile.preferred_intensity}")
```

---

## Troubleshooting

### Common Issues

**1. AI Model Not Loading**

_Symptoms:_ All recommendations use fallback logic, low confidence scores

_Solutions:_

- Check `HF_TOKEN` environment variable
- Verify model repository access
- Ensure sufficient disk space for model cache
- Check network connectivity to Hugging Face

```bash
# Test model download
# Model is downloaded via alice_asset_download_manager.dart from Cloudflare R2
```

**2. Preferences Not Learning**

_Symptoms:_ Recommendations don't improve over time, same suggestions repeatedly

_Solutions:_

- Verify user feedback is being processed
- Check `learning_rate` is not too low (should be 0.01-0.1)
- Ensure user profiles are being saved
- Validate feedback signal extraction

```python
# Debug preference learning
profile = engine.get_user_profile('user_id')
print(f"Interactions: {profile.total_interactions}")
print(f"Last updated: {profile.last_updated}")
print(f"Learning rate: {profile.learning_rate}")
```

**3. Low Recommendation Confidence**

_Symptoms:_ All confidence scores below 70%

_Solutions:_

- Increase user interaction history
- Provide more context data (energy, motivation)
- Check for data quality issues
- Validate confidence calculation logic

**4. Safety Constraints Too Restrictive**

_Symptoms:_ No weight increases suggested, very conservative recommendations

_Solutions:_

- Review safety constraint settings
- Check if user is consistently rating difficulty as too easy
- Validate form quality scores
- Consider adjusting constraint thresholds for experienced users

**5. Frontend Components Not Updating**

_Symptoms:_ Recommendation UI shows loading indefinitely, no data displayed

_Solutions:_

- Check API endpoint connectivity
- Verify CORS settings on backend
- Validate API response format
- Check browser network tab for errors

```typescript
// Debug API calls
const recommendation = await aiCoaching.getRecommendation(
  userId,
  exerciseData,
  context,
);
console.log("API Response:", recommendation);
```

### Performance Issues

**High Response Times:**

_Causes:_

- Model inference bottleneck
- Database query performance
- Large user profile data

_Solutions:_

- Implement model output caching
- Optimize database queries with indexes
- Use connection pooling
- Consider model quantization

**Memory Usage:**

_Causes:_

- Model loaded in memory
- Large user profile cache
- Memory leaks in learning updates

_Solutions:_

- Monitor memory usage patterns
- Implement profile cleanup
- Use CPU-only PyTorch
- Optimize data structures

### Data Quality Issues

**Inconsistent User Behavior:**

_Symptoms:_ Conflicting feedback patterns, erratic preference changes

_Solutions:_

- Implement feedback validation
- Use weighted averaging for outliers
- Add confidence intervals to preferences
- Consider user context changes

**Missing Context Data:**

_Symptoms:_ Recommendations based on incomplete information

_Solutions:_

- Implement smart defaults
- Use historical averages for missing data
- Prompt users for important context
- Graceful degradation strategies

### Integration Issues

**Database Connection Problems:**

- Verify connection strings
- Check network access to Supabase
- Validate schema migrations
- Monitor connection pool usage

**API Endpoint Failures:**

- Test endpoints individually
- Validate request/response formats
- Check authentication/authorization
- Monitor error rates and patterns

---

## Support and Maintenance

### Monitoring

**Key Metrics:**

- Recommendation acceptance rate
- Average confidence scores
- Response times
- Error rates
- User engagement with AI features

**Alerts:**

- Model inference failures
- Database connection issues
- High error rates (>5%)
- Low recommendation acceptance (<60%)

### Regular Maintenance Tasks

**Weekly:**

- Review AI performance metrics
- Analyze user feedback patterns
- Check error logs for issues
- Validate model accuracy

**Monthly:**

- Update user preference statistics
- Analyze learning algorithm effectiveness
- Review safety constraint performance
- Update documentation

**Quarterly:**

- Consider model version updates
- Evaluate new AI capabilities
- Review system architecture
- Plan performance optimizations

---

This documentation provides comprehensive coverage of the Enhanced AI Coaching System. For additional questions or support, please refer to the development team or create an issue in the repository.

## Related

^[source-materials/mirrors/doctrine/AI_SYSTEM_DOCUMENTATION.md]
