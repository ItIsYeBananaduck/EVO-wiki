---
title: DEPLOYMENT_AI_CONFIG
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/DEPLOYMENT_AI_CONFIG.md"]
updated: 2026-07-24
---

# Enhanced AI System Deployment Configuration

## Updated Environment Variables

### Backend (app.py) Environment Variables

```bash
# Core AI Configuration
HF_TOKEN=your_huggingface_token_here
MODEL_REPO=PhilmoLSC/philmoLSC
MODEL_NAME=alice-qwen25-1.5b-q4_k_m
MODEL_CACHE_DIR=/app/model_cache
USER_PROFILES_DIR=/app/user_profiles

# AI Engine Settings
AI_ENGINE_ENABLED=true
ENHANCED_AI_ENABLED=true
PREFERENCE_LEARNING_RATE=0.05
LEARNING_ADAPTATION_SPEED=0.1
SAFETY_CONSTRAINTS_ENABLED=true
FALLBACK_MODE_ENABLED=true

# Safety Constraint Configuration
MAX_WEIGHT_INCREASE=0.10
MIN_REP_RETENTION=0.80
MIN_REST_TIME=30
MAX_VOLUME_INCREASE=0.15
FATIGUE_THRESHOLD=8.0
FORM_DEGRADATION_LIMIT=6.0

# Performance Configuration
MAX_MODEL_MEMORY_MB=1024
MODEL_TIMEOUT_SECONDS=30
CONCURRENT_REQUESTS_LIMIT=10
RECOMMENDATION_CACHE_TTL=1800  # 30 minutes
PROFILE_CACHE_TTL=3600         # 1 hour

# Database Configuration
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key

# Redis Configuration (optional caching)
REDIS_URL=redis://localhost:6379
REDIS_DB=0
REDIS_ENABLED=false

# Logging Configuration
LOG_LEVEL=INFO
AI_LOGGING_ENABLED=true
DEBUG_AI_RECOMMENDATIONS=false
LOG_USER_FEEDBACK=true
LOG_LEARNING_EVENTS=true

# API Configuration
API_VERSION=v1
CORS_ORIGINS=http://localhost:5173,https://your-frontend-domain.com
MAX_REQUEST_SIZE=10485760  # 10MB

# Monitoring
METRICS_ENABLED=true
HEALTH_CHECK_ENABLED=true
RECOMMENDATION_METRICS=true
```

### Frontend (SvelteKit) Environment Variables

```bash
# API Configuration
VITE_API_URL=https://your-backend-api.com
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key

# AI Feature Flags
VITE_AI_COACHING_ENABLED=true
VITE_ENHANCED_AI_ENABLED=true
VITE_INSIGHTS_DASHBOARD_ENABLED=true
VITE_RECOMMENDATION_UI_ENABLED=true
VITE_PREFERENCE_LEARNING_ENABLED=true

# UI Configuration
VITE_RECOMMENDATION_TIMEOUT=30000  # 30 seconds
VITE_FEEDBACK_VALIDATION=true
VITE_CONFIDENCE_THRESHOLD=0.5      # Minimum confidence to show
VITE_AUTO_SUBMIT_FEEDBACK=false
VITE_SHOW_DEBUG_INFO=false

# Analytics Configuration
VITE_ANALYTICS_ENABLED=true
VITE_AI_ANALYTICS_TRACKING=true
VITE_USER_BEHAVIOR_TRACKING=true

# Performance Settings
VITE_ENABLE_SW_CACHING=true
VITE_CACHE_AI_RESPONSES=true
VITE_OFFLINE_MODE_ENABLED=false
```

## Legacy Dockerfile Example (Not used in current deployment)

```dockerfile
FROM python:3.11-slim

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copy requirements and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Install AI/ML specific dependencies
RUN pip install torch==2.0.1 --index-url https://download.pytorch.org/whl/cpu
RUN pip install transformers==4.35.0
RUN pip install scikit-learn==1.3.0
RUN pip install numpy==1.24.3
RUN pip install redis==4.6.0

# Create application directories
RUN mkdir -p /app/model_cache
RUN mkdir -p /app/user_profiles
RUN mkdir -p /app/logs

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Copy application code
COPY . .

# Set proper permissions
RUN chown -R appuser:appuser /app
RUN chmod +x /app/deploy.sh

# Switch to non-root user
USER appuser

# Set environment variables
ENV PYTHONPATH=/app
ENV MODEL_CACHE_DIR=/app/model_cache
ENV USER_PROFILES_DIR=/app/user_profiles
ENV PYTHONUNBUFFERED=1

# Health check
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Expose port
EXPOSE 8000

# Start command with enhanced AI support
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "1"]
```

## Updated docker-compose.yml

```yaml
version: "3.8"

services:
  backend:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      - HF_TOKEN=${HF_TOKEN}
      - MODEL_REPO=PhilmoLSC/philmoLSC
      - MODEL_NAME=alice-qwen25-1.5b-q4_k_m
      - AI_ENGINE_ENABLED=true
      - ENHANCED_AI_ENABLED=true
      - SUPABASE_URL=${SUPABASE_URL}
      - SUPABASE_SERVICE_ROLE_KEY=${SUPABASE_SERVICE_ROLE_KEY}
      - REDIS_URL=redis://redis:6379
      - LOG_LEVEL=INFO
    volumes:
      - model_cache:/app/model_cache
      - user_profiles:/app/user_profiles
      - ./logs:/app/logs
    depends_on:
      - redis
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    restart: unless-stopped

  frontend:
    build:
      context: ./app
      dockerfile: Dockerfile
    ports:
      - "5173:5173"
    environment:
      - VITE_API_URL=http://localhost:8000
      - VITE_SUPABASE_URL=${SUPABASE_URL}
      - VITE_SUPABASE_ANON_KEY=${SUPABASE_ANON_KEY}
      - VITE_AI_COACHING_ENABLED=true
      - VITE_ENHANCED_AI_ENABLED=true
      - VITE_INSIGHTS_DASHBOARD_ENABLED=true
    depends_on:
      - backend
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    restart: unless-stopped

volumes:
  model_cache:
  user_profiles:
  redis_data:
```

## Updated fly.toml (Fly.io Deployment)

```toml
app = "git-fit-enhanced-ai"
primary_region = "ord"

[build]
  # Docker builds are not used in the current deployment workflow.

[env]
  MODEL_CACHE_DIR = "/app/model_cache"
  USER_PROFILES_DIR = "/app/user_profiles"
  AI_ENGINE_ENABLED = "true"
  ENHANCED_AI_ENABLED = "true"
  SAFETY_CONSTRAINTS_ENABLED = "true"
  FALLBACK_MODE_ENABLED = "true"
  LOG_LEVEL = "INFO"
  PYTHONUNBUFFERED = "1"

[experimental]
  auto_rollback = true

[[services]]
  http_checks = []
  internal_port = 8000
  processes = ["app"]
  protocol = "tcp"
  script_checks = []

  [services.concurrency]
    hard_limit = 25
    soft_limit = 20
    type = "connections"

  [[services.ports]]
    force_https = true
    handlers = ["http"]
    port = 80

  [[services.ports]]
    handlers = ["tls", "http"]
    port = 443

  [[services.tcp_checks]]
    grace_period = "1s"
    interval = "15s"
    restart_limit = 0
    timeout = "2s"

# Enhanced machine configuration for AI workloads
[[vm]]
  cpu_kind = "shared"
  cpus = 2
  memory_mb = 2048

# Volume for model cache persistence
[[mounts]]
  destination = "/app/model_cache"
  source = "model_cache_vol"

[[mounts]]
  destination = "/app/user_profiles"
  source = "user_profiles_vol"

# Health checks for AI system
[checks]
  [checks.ai_health]
    grace_period = "30s"
    interval = "30s"
    method = "get"
    path = "/health"
    port = 8000
    protocol = "http"
    restart_limit = 0
    timeout = "10s"
```

## Package.json Updates (Frontend)

```json
{
  "name": "git-fit-frontend",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "build": "vite build",
    "dev": "vite dev",
    "preview": "vite preview",
    "test": "vitest run",
    "test:ai": "vitest run ai-coaching.test.ts",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage",
    "check": "svelte-kit sync && svelte-check --tsconfig ./tsconfig.json",
    "check:watch": "svelte-kit sync && svelte-check --tsconfig ./tsconfig.json --watch",
    "lint": "prettier --plugin-search-dir . --check . && eslint .",
    "format": "prettier --plugin-search-dir . --write .",
    "deploy": "npm run build && npx cap sync",
    "test:ai-integration": "npm run test:ai && npm run test:coverage"
  },
  "devDependencies": {
    "@sveltejs/adapter-auto": "^2.0.0",
    "@sveltejs/kit": "^1.20.4",
    "@testing-library/svelte": "^4.0.3",
    "@vitest/coverage-c8": "^0.33.0",
    "eslint": "^8.28.0",
    "eslint-plugin-svelte": "^2.30.0",
    "jsdom": "^22.1.0",
    "prettier": "^2.8.0",
    "prettier-plugin-svelte": "^2.10.1",
    "svelte": "^4.0.5",
    "svelte-check": "^3.4.3",
    "tslib": "^2.4.1",
    "typescript": "^5.0.0",
    "vite": "^4.4.2",
    "vitest": "^0.34.0"
  },
  "type": "module",
  "dependencies": {
    "@capacitor/android": "^5.0.0",
    "@capacitor/core": "^5.0.0",
    "@capacitor/ios": "^5.0.0",
    "@supabase/supabase-js": "^2.0.0"
  }
}
```

## Requirements.txt Updates (Backend)

```txt
# Core FastAPI dependencies
fastapi==0.104.1
uvicorn[standard]==0.23.2
pydantic==2.4.2
python-multipart==0.0.6

# AI/ML dependencies
torch==2.0.1
transformers==4.35.0
scikit-learn==1.3.0
numpy==1.24.3
huggingface-hub==0.17.3

# Data processing
pandas==2.1.1
scipy==1.11.3

# Caching and storage
redis==4.6.0
pickle-mixin==1.0.2

# HTTP and networking
httpx==0.25.0
requests==2.31.0

# Development and testing
pytest==7.4.2
pytest-asyncio==0.21.1
pytest-cov==4.1.0

# Logging and monitoring
structlog==23.1.0
python-json-logger==2.0.7

# Environment management
python-dotenv==1.0.0

# Security
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4

# CORS middleware
fastapi-cors==0.0.6
```

## Monitoring and Health Checks

### Health Check Endpoint

```python
@app.get("/health")
async def health_check():
    """Comprehensive health check for AI system"""
    health_status = {
        "status": "healthy",
        "timestamp": datetime.now().isoformat(),
        "ai_engine": {
            "status": "unknown",
            "model_loaded": False,
            "enhanced_engine": False
        },
        "database": {
            "status": "unknown"
        },
        "cache": {
            "status": "unknown"
        }
    }

    # Check AI engine status
    try:
        ai_engine = get_enhanced_engine()
        if ai_engine:
            health_status["ai_engine"]["status"] = "healthy"
            health_status["ai_engine"]["enhanced_engine"] = True
            health_status["ai_engine"]["model_loaded"] = ai_engine.model is not None
        else:
            health_status["ai_engine"]["status"] = "degraded"
    except Exception as e:
        health_status["ai_engine"]["status"] = "unhealthy"
        health_status["ai_engine"]["error"] = str(e)

    # Check database connectivity (if applicable)
    try:
        # Add database health check here
        health_status["database"]["status"] = "healthy"
    except Exception as e:
        health_status["database"]["status"] = "unhealthy"
        health_status["database"]["error"] = str(e)

    # Check cache status
    try:
        # Add Redis/cache health check here
        health_status["cache"]["status"] = "healthy"
    except Exception as e:
        health_status["cache"]["status"] = "unhealthy"
        health_status["cache"]["error"] = str(e)

    # Determine overall status
    if (health_status["ai_engine"]["status"] == "unhealthy" or
        health_status["database"]["status"] == "unhealthy"):
        health_status["status"] = "unhealthy"
    elif (health_status["ai_engine"]["status"] == "degraded"):
        health_status["status"] = "degraded"

    return health_status
```

### Metrics Endpoint

```python
@app.get("/metrics")
async def metrics():
    """AI system metrics for monitoring"""
    return {
        "recommendations_served": get_recommendation_count(),
        "average_confidence": get_average_confidence(),
        "user_acceptance_rate": get_acceptance_rate(),
        "model_response_time_ms": get_avg_response_time(),
        "active_users": get_active_user_count(),
        "learning_events_processed": get_learning_events_count()
    }
```

## Production Optimization Settings

### Nginx Configuration (if using reverse proxy)

```nginx
upstream git_fit_backend {
    server localhost:8000;
    keepalive 32;
}

server {
    listen 80;
    server_name your-domain.com;

    # AI model file caching
    location ~* \.(model|bin|safetensors)$ {
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    # API routes
    location /api/ {
        proxy_pass http://git_fit_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;

        # Longer timeout for AI processing
        proxy_read_timeout 60s;
        proxy_send_timeout 60s;
    }

    # Static files
    location / {
        root /var/www/html;
        try_files $uri $uri/ /index.html;

        # Enable gzip compression
        gzip on;
        gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
    }
}
```

### Deployment Script Updates

```bash
#!/bin/bash
# Enhanced deployment script with AI system support

set -e

echo "🤖 Deploying Enhanced AI System..."

# Environment validation
if [ -z "$HF_TOKEN" ]; then
    echo "❌ HF_TOKEN environment variable is required"
    exit 1
fi

if [ -z "$SUPABASE_URL" ]; then
    echo "❌ SUPABASE_URL environment variable is required"
    exit 1
fi

# Build and test AI components
echo "🧪 Running AI system tests..."
pytest app/tests/test_ai_backend.py -v
cd app && npm run test:ai && cd ..

# Build containers
echo "🏗️ Building containers..."
docker-compose build

# Deploy database schema updates
echo "🗄️ Updating database schema..."
pnpm supabase db push

# Deploy to production
echo "🚀 Deploying to production..."
if command -v flyctl &> /dev/null; then
    flyctl deploy
else
    docker-compose up -d
fi

# Health check
echo "🩺 Performing health checks..."
sleep 30
curl -f http://localhost:8000/health || {
    echo "❌ Health check failed"
    exit 1
}

echo "✅ Enhanced AI System deployed successfully!"
echo "🔗 API Health: http://localhost:8000/health"
echo "📊 Metrics: http://localhost:8000/metrics"
```

## Monitoring Setup

### Prometheus Metrics (Optional)

```python
from prometheus_client import Counter, Histogram, Gauge
import time

# Metrics
RECOMMENDATION_COUNTER = Counter('ai_recommendations_total', 'Total AI recommendations served')
RECOMMENDATION_HISTOGRAM = Histogram('ai_recommendation_duration_seconds', 'AI recommendation processing time')
CONFIDENCE_GAUGE = Gauge('ai_recommendation_confidence', 'Current average recommendation confidence')
ACCEPTANCE_RATE_GAUGE = Gauge('ai_acceptance_rate', 'Current recommendation acceptance rate')

@RECOMMENDATION_HISTOGRAM.time()
async def get_personalized_recommendation_with_metrics(request: RecommendationRequest):
    RECOMMENDATION_COUNTER.inc()

    start_time = time.time()
    recommendation = await get_personalized_recommendation(request)
    processing_time = time.time() - start_time

    CONFIDENCE_GAUGE.set(recommendation.confidence)

    return recommendation
```

This deployment configuration ensures the Enhanced AI Coaching System is properly configured for production use with comprehensive monitoring, caching, and optimization settings.

## Related

^[{src_rel}]
