# Job Portal Recommendation System

## Table of Contents
- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Machine Learning Algorithms](#machine-learning-algorithms)
  - [PRO Model](#pro-model)
  - [ULTRA PRO Model](#ultra-pro-model)
- [API Documentation](#api-documentation)
- [Installation & Setup](#installation--setup)
- [Database Schema](#database-schema)
- [Subscription Tiers](#subscription-tiers)
- [Testing](#testing)

---

## Overview

An intelligent job recommendation system built with Django and scikit-learn that provides personalized job recommendations based on user skills and subscription level. The system features two distinct recommendation algorithms (PRO and ULTRA PRO) with varying levels of sophistication.

### Key Features
- **Dual-Model ML System**: PRO and ULTRA PRO recommendation engines
- **Skill-Based Matching**: Intelligent matching based on user skills
- **Subscription-Based Access**: Tiered access to recommendation features
- **CSV Dataset Training**: Trains on large job market datasets
- **Database Integration**: Recommends jobs from active database listings
- **JWT Authentication**: Secure cookie-based authentication
- **Auto-retraining**: Models retrain automatically every 7 days

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Client Application                       │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                    Django REST API                           │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Authentication Layer (JWT + Cookies)                  │ │
│  └────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Views (Auth, Subscription, Recommendations)           │ │
│  └────────────────────────────────────────────────────────┘ │
└────────────────────────┬────────────────────────────────────┘
                         │
        ┌────────────────┴────────────────┐
        ▼                                  ▼
┌──────────────────┐            ┌──────────────────┐
│  Model Manager   │            │  PostgreSQL DB   │
│                  │            │                  │
│  - PRO Engine    │◄───────────┤  - Users         │
│  - ULTRA PRO     │            │  - Jobs          │
│    Engine        │            │  - Skills        │
│  - Training      │            │  - Subscriptions │
│    Logic         │            │  - UpcomingJobs  │
└──────┬───────────┘            └──────────────────┘
       │
       ▼
┌──────────────────┐
│  Training Data   │
│                  │
│  - job_market.csv│
│  - Database Jobs │
└──────────────────┘
```

---

## Machine Learning Algorithms

### Training Strategy

The system uses a **hybrid training approach**:
1. **Training Data**: Models train on ALL available jobs (CSV dataset + database jobs)
2. **Recommendations**: Models recommend ONLY database jobs (positive IDs)

This approach ensures:
- Rich feature learning from large datasets
- Relevant recommendations from active job listings
- Better clustering and skill demand analysis

### PRO Model

**Algorithm**: K-Means Clustering + Cosine Similarity

#### How It Works

1. **Feature Engineering**
   ```python
   # Convert skills to binary vectors using MultiLabelBinarizer
   skills = ['python', 'django', 'postgresql']
   vector = [1, 0, 1, 1, 0, 0, ...]  # One-hot encoded
   ```

2. **Clustering**
   ```python
   # Group similar jobs using K-Means (6-8 clusters)
   kmeans = KMeans(n_clusters=6, random_state=42)
   job_clusters = kmeans.fit_predict(job_vectors)
   ```

3. **Similarity Calculation**
   ```python
   # Cosine similarity between user skills and job requirements
   similarity = cosine_similarity(user_vector, job_vectors)
   ```

4. **Scoring Formula**
   ```
   Score = Number of Matching Skills
   Sort by: Descending order of matching skills
   ```

#### PRO Model Features
- Simple skill matching
- Binary match/no-match logic
- Fast inference (~0.003s)
- Suitable for users who want quick matches

#### Code Structure
```python
class ProJobRecommendationEngine:
    def prepare_and_train(self, jobs_list):
        # Create skill vectors
        # Train K-means clustering
        
    def recommend(self, seeker_skills, top_n=10):
        # Calculate cosine similarity
        # Filter database jobs only (ID > 0)
        # Sort by matching skills count
        # Return top N recommendations
```

---

### ULTRA PRO Model

**Algorithm**: Advanced K-Means + Multi-Factor Scoring

#### How It Works

1. **Enhanced Feature Engineering**
   ```python
   # Same as PRO but with additional metadata
   - Skill demand scores (market analysis)
   - Job cluster information
   - Quality scoring
   ```

2. **Market Demand Analysis**
   ```python
   # Calculate how often each skill appears in job market
   skill_demand_scores = {
       'python': 0.75,      # 75% of jobs require Python
       'javascript': 0.60,  # 60% require JavaScript
       'docker': 0.45       # 45% require Docker
   }
   ```

3. **Advanced Clustering**
   ```python
   # More clusters for finer granularity (8-10 clusters)
   kmeans = KMeans(n_clusters=10, random_state=42)
   ```

4. **Multi-Factor Scoring**
   ```
   Ultra Pro Score = 
       0.40 × Cosine Similarity +
       0.30 × Quality Score +
       0.20 × Market Demand Score +
       0.10 × Cluster Proximity Score
   
   Quality Score = Base Match % + Bonus for High-Demand Skills
   ```

5. **Job Tier Classification**
   ```
   - Perfect Match: 80%+ skill match
   - Strong Match: 60-79% skill match
   - Good Match: 40-59% skill match
   - Growth Opportunity: <40% match but high quality score
   - Stretch Role: Low match percentage
   ```

#### ULTRA PRO Model Features
- Multi-dimensional scoring
- Market demand analysis
- Skill gap identification
- Career growth potential assessment
- High-demand skill highlighting

#### Advanced Scoring Example
```python
Job: Senior Python Developer
User Skills: ['python', 'django', 'docker']
Job Skills: ['python', 'django', 'postgresql', 'redis']

Matching: ['python', 'django']  (2/4 = 50%)
Missing: ['postgresql', 'redis']

Calculations:
- Cosine Similarity: 0.82
- Base Match: 50%
- Demand Bonus: +15% (python and django are high-demand)
- Quality Score: 65%
- Market Demand: 70% (job requires popular skills)
- Cluster Score: 0.85

Final Score: 0.40(82) + 0.30(65) + 0.20(70) + 0.10(85) = 75.3

Tier: "Strong Match"
Career Growth: "High" (only 2 skills to learn)
```

#### Code Structure
```python
class UltraProJobRecommendationEngine:
    def prepare_and_train(self, jobs_list):
        # Calculate skill demand scores
        # Create enhanced feature vectors
        # Train K-means with more clusters
        
    def calculate_skill_match_quality(self, user_skills, job_skills):
        # Base match percentage
        # High-demand skill bonus
        # Quality score calculation
        
    def recommend_ultra_pro(self, seeker_skills, top_n=10):
        # Multi-factor scoring
        # Job tier classification
        # Skill gap analysis
        # Sort by Ultra Pro score
```

---

## API Documentation

### Base URL
```
http://localhost:8000/api/
```

### Authentication
All protected endpoints require JWT token in cookies.

Cookie Name: `auth_token`
Expiry: 7 days

---

### Authentication Endpoints

#### 1. Sign Up
```http
POST /api/signup/
Content-Type: application/json
```

**Request Body:**
```json
{
  "full_name": "John Doe",
  "email": "john@example.com",
  "phone": "1234567890",
  "password": "securepassword"
}
```

**Response:** `201 Created`
```json
{
  "message": "Signup successful",
  "user": {
    "id": 1,
    "full_name": "John Doe",
    "email": "john@example.com",
    "phone": "1234567890",
    "subscription_level": "none",
    "subscription_expiry": null,
    "created_at": "2024-02-11T10:30:00Z"
  }
}
```

**Cookie Set:** `auth_token` (HttpOnly, 7 days)

---

#### 2. Login
```http
POST /api/login/
Content-Type: application/json
```

**Request Body:**
```json
{
  "email": "john@example.com",
  "password": "securepassword"
}
```

**Response:** `200 OK`
```json
{
  "message": "Login successful",
  "user": {
    "id": 1,
    "full_name": "John Doe",
    "email": "john@example.com",
    "subscription_level": "pro",
    "subscription_expiry": "2024-03-11T10:30:00Z",
    "is_subscription_active": true,
    "created_at": "2024-02-11T10:30:00Z"
  }
}
```

---

#### 3. Logout
```http
POST /api/logout/
Cookie: auth_token=<token>
```

**Response:** `200 OK`
```json
{
  "message": "Logout successful"
}
```

---

#### 4. Get Current User
```http
GET /api/user/
Cookie: auth_token=<token>
```

**Response:** `200 OK`
```json
{
  "user": {
    "id": 1,
    "full_name": "John Doe",
    "email": "john@example.com",
    "phone": "1234567890",
    "subscription_level": "pro",
    "subscription_status": "active",
    "has_premium_access": true,
    "subscription_expiry": "2024-03-11T10:30:00Z",
    "is_subscription_active": true,
    "skills": ["python", "javascript", "docker"],
    "has_skills": true,
    "created_at": "2024-02-11T10:30:00Z",
    "can_access_recommendations": true
  }
}
```

---

### Subscription Endpoints

#### 5. Update Subscription
```http
POST /api/subscription/upgrade/
Cookie: auth_token=<token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "subscription_level": "pro"  // or "ultra_pro"
}
```

**Valid Upgrade Paths:**
- `none` → `pro` or `ultra_pro`
- `pro` → `ultra_pro`
- `ultra_pro` → (no upgrades)

**Response:** `200 OK`
```json
{
  "message": "Subscription updated successfully",
  "user": {
    "id": 1,
    "full_name": "John Doe",
    "email": "john@example.com",
    "subscription_level": "pro",
    "subscription_expiry": "2024-03-13T10:30:00Z",
    "updated_at": "2024-02-11T10:30:00Z"
  }
}
```

---

#### 6. Check Subscription Status
```http
GET /api/subscription/check/
Cookie: auth_token=<token>
```

**Response:** `200 OK`
```json
{
  "user": {
    "id": 1,
    "full_name": "John Doe",
    "subscription_level": "pro",
    "subscription_status": "active",
    "subscription_expiry": "2024-03-13T10:30:00Z",
    "is_subscription_active": true
  }
}
```

---

### Skill Management Endpoints

#### 7. Add Single Skill
```http
POST /api/skills/user/add/
Cookie: auth_token=<token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "skill_name": "python"
}
```

**Response:** `201 Created`
```json
{
  "message": "Skill added successfully",
  "skill": {
    "id": 5,
    "name": "python",
    "description": "",
    "is_new": true
  }
}
```

---

#### 8. Add Multiple Skills
```http
POST /api/skills/user/add-multiple/
Cookie: auth_token=<token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "skills": ["python", "javascript", "docker", "postgresql"]
}
```

**Response:** `200 OK`
```json
{
  "message": "Skills processed successfully",
  "added_skills": ["python", "docker"],
  "existing_skills": ["javascript", "postgresql"],
  "total_added": 2,
  "total_existing": 2,
  "errors": null
}
```

---

### Recommendation Endpoints

#### 9. Get Recommendations (PRO or ULTRA PRO)
```http
GET /api/recommendations/?top_n=10
Cookie: auth_token=<token>
```

**Query Parameters:**
- `top_n` (optional): Number of recommendations (default: 10, max: 50)

**PRO User Response:** `200 OK`
```json
{
  "user": {
    "id": 1,
    "name": "John Doe",
    "subscription_level": "pro"
  },
  "model_type": "PRO",
  "model": "PRO",
  "seeker_skills": ["python", "javascript", "docker"],
  "total_jobs_analyzed": 1500,
  "filtered_to": "database",
  "total_recommendations": 15,
  "subscription_level": "pro",
  "user_skills_used": ["python", "javascript", "docker"],
  "recommendations": [
    {
      "job_id": 101,
      "title": "Senior Python Developer",
      "company": "TechCorp",
      "salary": "$120k-$160k",
      "location": "San Francisco, CA",
      "source": "database",
      "similarity_score": 85.3,
      "match_percentage": 75.0,
      "required_skills": ["python", "django", "postgresql", "redis"],
      "matching_skills": ["python"],
      "missing_skills": ["django", "postgresql", "redis"],
      "num_matching": 1,
      "num_missing": 3
    }
  ]
}
```

**ULTRA PRO User Response:** `200 OK`
```json
{
  "user": {
    "id": 2,
    "name": "Jane Smith",
    "subscription_level": "ultra_pro"
  },
  "model_type": "ULTRA_PRO",
  "model": "ULTRA_PRO",
  "seeker_skills": ["python", "javascript", "docker"],
  "total_jobs_analyzed": 1500,
  "filtered_to": "database",
  "total_recommendations": 15,
  "subscription_level": "ultra_pro",
  "user_skills_used": ["python", "javascript", "docker"],
  "recommendations": [
    {
      "job_id": 101,
      "title": "Senior Python Developer",
      "salary": "$120k-$160k",
      "location": "San Francisco, CA",
      "company": "TechCorp",
      "source": "database",
      "match_tier": "Strong Match",
      "ultra_pro_score": 78.5,
      "match_percentage": 75.0,
      "quality_score": 82.5,
      "market_demand_score": 85.0,
      "required_skills": ["python", "django", "postgresql", "redis"],
      "matching_skills": ["python"],
      "missing_skills": ["django", "postgresql", "redis"],
      "high_demand_skills": ["python"],
      "num_matching": 1,
      "num_missing": 3,
      "career_growth_potential": "High"
    }
  ]
}
```

**No Subscription Response:** `403 Forbidden`
```json
{
  "error": "Subscription required",
  "message": "You need a Pro or Ultra Pro subscription to access job recommendations.",
  "upgrade_url": "/api/subscription/upgrade/"
}
```

---

#### 10. Get Model Status (Public)
```http
GET /api/model/status/
```

**Response:** `200 OK`
```json
{
  "status": "trained",
  "last_trained": "2024-02-11T10:00:00Z",
  "database_jobs": 25,
  "dataset_jobs": 1475,
  "total_training_jobs": 1500,
  "models": {
    "pro": {
      "exists": true,
      "n_clusters": 8,
      "skills": 156
    },
    "ultra_pro": {
      "exists": true,
      "n_clusters": 10,
      "skills": 156
    }
  },
  "dataset_file_exists": true
}
```

---

#### 11. Retrain Models (Admin)
```http
POST /api/model/retrain/
Cookie: auth_token=<token>
```

**Response:** `200 OK`
```json
{
  "message": "Both PRO and ULTRA PRO models retrained successfully",
  "total_jobs": 1500,
  "models": {
    "pro": {
      "clusters": 8,
      "skills": 156
    },
    "ultra_pro": {
      "clusters": 10,
      "skills": 156
    }
  }
}
```

---

### ULTRA PRO Exclusive Endpoints

#### 12. Get Upcoming Jobs (ULTRA PRO Only)
```http
GET /api/upcomingjobs/?limit=20&offset=0
Cookie: auth_token=<token>
```

**Query Parameters:**
- `limit` (optional): Number of jobs to return (default: 20)
- `offset` (optional): Pagination offset (default: 0)

**Response:** `200 OK`
```json
{
  "user_id": 2,
  "subscription_level": "ultra_pro",
  "upcoming_jobs": [
    {
      "id": 50,
      "title": "Lead DevOps Engineer",
      "description": "We are looking for...",
      "location": "Austin, TX",
      "salary_range": "$150k-$200k",
      "scheduled_date": "2024-02-20T09:00:00Z",
      "days_until_posting": 9,
      "status": "active",
      "skills": ["kubernetes", "docker", "aws", "terraform"],
      "total_skills": 4,
      "created_at": "2024-02-10T15:00:00Z",
      "updated_at": "2024-02-10T15:00:00Z"
    }
  ],
  "total_jobs": 45,
  "limit": 20,
  "offset": 0,
  "has_more": true
}
```

**Non-ULTRA PRO Response:** `403 Forbidden`
```json
{
  "error": "Ultra Pro subscription required",
  "message": "This feature is only available for Ultra Pro subscribers.",
  "upgrade_url": "/api/subscription/update/"
}
```

---

## Installation & Setup

### Prerequisites
```bash
- Python 3.8+
- PostgreSQL 12+
- pip
```

### 1. Install Dependencies
```bash
pip install django==4.2.7
pip install psycopg2-binary
pip install djangorestframework
pip install django-cors-headers
pip install pyjwt
pip install scikit-learn
pip install numpy
```

### 2. Database Configuration

**settings.py:**
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'job_portal_db',
        'USER': 'postgres',
        'PASSWORD': 'your_password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

### 3. Database Setup

**SQL Schema:**
```sql
-- Create database
CREATE DATABASE job_portal_db;

-- Create tables
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(200) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    password VARCHAR(255) NOT NULL,
    subscription_level VARCHAR(20) DEFAULT 'none',
    subscription_expiry TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE skills (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE user_skills (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    skill_id INTEGER REFERENCES skills(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, skill_id)
);

CREATE TABLE jobs (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    description TEXT NOT NULL,
    location VARCHAR(200),
    salary_range VARCHAR(100),
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE job_skills (
    id SERIAL PRIMARY KEY,
    job_id INTEGER REFERENCES jobs(id) ON DELETE CASCADE,
    skill_id INTEGER REFERENCES skills(id) ON DELETE CASCADE,
    UNIQUE(job_id, skill_id)
);

CREATE TABLE upcoming_jobs (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    description TEXT NOT NULL,
    location VARCHAR(200),
    salary_range VARCHAR(100),
    scheduled_date TIMESTAMP NOT NULL,
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE upcoming_job_skills (
    id SERIAL PRIMARY KEY,
    upcoming_job_id INTEGER REFERENCES upcoming_jobs(id) ON DELETE CASCADE,
    skill_id INTEGER REFERENCES skills(id) ON DELETE CASCADE,
    UNIQUE(upcoming_job_id, skill_id)
);
```

### 4. Prepare Dataset

Place your `job_market.csv` in the project root directory.

**Expected CSV Format:**
```csv
job_title,company,location,skills,salary_min,salary_max
Senior Python Developer,TechCorp,Remote,"python,django,postgresql",100000,150000
Frontend Engineer,WebCo,New York,"javascript,react,css",90000,130000
```

### 5. Run Migrations
```bash
python manage.py migrate
```

### 6. Train Models

**Option A: Automatic (on first API call)**
Models train automatically when first recommendation is requested.

**Option B: Manual**
```bash
# Run test script
python test_pro_model.py

# Or use API
curl -X POST http://localhost:8000/api/model/retrain/ \
  -H "Cookie: auth_token=YOUR_TOKEN"
```

### 7. Start Server
```bash
python manage.py runserver
```

---

## Database Schema

### ER Diagram
```
┌─────────────┐         ┌──────────────┐         ┌─────────────┐
│    Users    │         │  User_Skills │         │   Skills    │
├─────────────┤         ├──────────────┤         ├─────────────┤
│ id (PK)     │────┐    │ id (PK)      │    ┌────│ id (PK)     │
│ full_name   │    └───→│ user_id (FK) │    │    │ name        │
│ email       │         │ skill_id (FK)│←───┘    │ description │
│ phone       │         └──────────────┘         └─────────────┘
│ password    │                                         │
│ sub_level   │                                         │
│ sub_expiry  │         ┌──────────────┐               │
└─────────────┘         │  Job_Skills  │               │
                        ├──────────────┤               │
┌─────────────┐         │ id (PK)      │               │
│    Jobs     │────┐    │ job_id (FK)  │               │
├─────────────┤    └───→│ skill_id (FK)│←──────────────┘
│ id (PK)     │         └──────────────┘
│ title       │
│ description │         ┌──────────────────────┐
│ location    │         │ Upcoming_Job_Skills  │
│ salary_range│         ├──────────────────────┤
│ status      │         │ id (PK)              │
└─────────────┘         │ upcoming_job_id (FK) │
                        │ skill_id (FK)        │←───────┐
┌──────────────────┐    └──────────────────────┘        │
│  Upcoming_Jobs   │───┐                                 │
├──────────────────┤   └─────────────────────────────────┘
│ id (PK)          │
│ title            │
│ description      │
│ location         │
│ salary_range     │
│ scheduled_date   │
│ status           │
└──────────────────┘
```

---

## Subscription Tiers

### Comparison Table

| Feature | None | PRO | ULTRA PRO |
|---------|------|-----|-----------|
| **Pricing** | Free | $9.99/mo | $19.99/mo |
| **Job Recommendations** | ❌ | ✅ | ✅ |
| **Recommendation Quality** | - | Basic | Advanced |
| **Scoring Algorithm** | - | Simple Match | Multi-Factor |
| **Market Insights** | ❌ | ❌ | ✅ |
| **Skill Gap Analysis** | ❌ | ❌ | ✅ |
| **Career Growth Recommendations** | ❌ | ❌ | ✅ |
| **High-Demand Skill Highlighting** | ❌ | ❌ | ✅ |
| **Upcoming Job Alerts** | ❌ | ❌ | ✅ |
| **Early Access to Jobs** | ❌ | ❌ | ✅ |
| **Max Recommendations** | - | 50 | 50 |

### Subscription Logic

```python
# Check active subscription
def get_active_subscription_level(user):
    if user.subscription_level == 'none':
        return 'none'
    
    if user.subscription_expiry is None:
        return 'none'
    
    if datetime.now(timezone.utc) >= user.subscription_expiry:
        return 'none'  # Expired
    
    return user.subscription_level  # 'pro' or 'ultra_pro'
```

### Upgrade Rules
- Users can upgrade from `none` to any paid tier
- PRO users can upgrade to ULTRA PRO
- Downgrades require waiting for subscription expiry
- Subscriptions last 30 days from upgrade date

---

## Testing

### Unit Test: PRO Model
```bash
python test_pro_model.py
```

**Expected Output:**
```
🔵 PRO MODEL TEST
==========================================

📊 Training Data:
   - Database jobs: 5
   - Dataset jobs: 1495
   - Total training: 1500

✓ PRO Model trained: 1500 jobs, 156 skills
✅ Training completed in 2.45s
✅ Recommendations generated in 0.003s

📋 PRO RECOMMENDATIONS (DATABASE JOBS ONLY)
==========================================

1. Senior Python Backend Developer
   ID: 1001
   Company: TechDB Corp
   Location: San Francisco, CA
   Salary: $120k-$160k
   Source: DATABASE ⭐
   📊 Match: 3/4 skills (75.0%)
   ✅ You have: python, postgresql, docker
   📚 You need: django
```

### API Testing with cURL

**1. Sign Up**
```bash
curl -X POST http://localhost:8000/api/signup/ \
  -H "Content-Type: application/json" \
  -d '{
    "full_name": "Test User",
    "email": "test@example.com",
    "password": "testpass123"
  }' \
  -c cookies.txt
```

**2. Add Skills**
```bash
curl -X POST http://localhost:8000/api/skills/user/add-multiple/ \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "skills": ["python", "javascript", "docker", "postgresql"]
  }'
```

**3. Upgrade to PRO**
```bash
curl -X POST http://localhost:8000/api/subscription/upgrade/ \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "subscription_level": "pro"
  }'
```

**4. Get Recommendations**
```bash
curl -X GET http://localhost:8000/api/recommendations/?top_n=5 \
  -b cookies.txt
```

### Testing ULTRA PRO Features

**1. Upgrade to ULTRA PRO**
```bash
curl -X POST http://localhost:8000/api/subscription/upgrade/ \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "subscription_level": "ultra_pro"
  }'
```

**2. Get Advanced Recommendations**
```bash
curl -X GET http://localhost:8000/api/recommendations/ \
  -b cookies.txt
```

**3. Get Upcoming Jobs**
```bash
curl -X GET http://localhost:8000/api/upcomingjobs/?limit=10 \
  -b cookies.txt
```

---

## Model Performance

### Training Metrics
```
Dataset Size: 1,500 jobs
Unique Skills: 156
Training Time: ~2.5 seconds
Model Size: ~450 KB (PRO), ~520 KB (ULTRA PRO)
```

### Inference Performance
```
PRO Model: ~3ms per user
ULTRA PRO Model: ~5ms per user
```

### Accuracy Considerations
- Models retrain every 7 days
- Accuracy improves with more database jobs
- Skill coverage depends on dataset quality
- Real-world testing recommended for production

---

## Troubleshooting

### Models Not Training
```bash
# Check dataset file exists
ls -la job_market.csv

# Check database connection
python manage.py dbshell

# Force retrain
curl -X POST http://localhost:8000/api/model/retrain/ \
  -b cookies.txt
```

### No Recommendations Returned
```bash
# Check if database has jobs
SELECT COUNT(*) FROM jobs WHERE status='active';

# Check user has skills
SELECT s.name FROM user_skills us 
JOIN skills s ON us.skill_id = s.id 
WHERE us.user_id = 1;

# Check subscription is active
SELECT subscription_level, subscription_expiry FROM users WHERE id=1;
```

### Cookie Authentication Issues
```bash
# Verify cookie is being set
curl -v http://localhost:8000/api/login/ \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"testpass123"}' \
  -c cookies.txt

# Check cookie file
cat cookies.txt

# Ensure SameSite and Secure settings match environment
# Development: secure=False, samesite='Lax'
# Production: secure=True, samesite='Strict'
```

---

## File Structure

```
project/
├── api/
│   ├── __init__.py
│   ├── models.py                    # Django models
│   ├── views.py                     # Auth & skill endpoints
│   ├── recommendation_views.py      # Recommendation endpoints
│   ├── ultra_pro_views.py           # ULTRA PRO endpoints
│   ├── urls.py                      # API routes
│   ├── ml_model_manager.py          # Model orchestration
│   ├── ml_pro_engine.py             # PRO algorithm
│   ├── ml_ultra_pro_engine.py       # ULTRA PRO algorithm
│   └── dataset_processor.py         # CSV processing
├── j_p/
│   ├── settings.py                  # Django settings
│   └── urls.py                      # Project URLs
├── job_market.csv                   # Training dataset
├── pro_model.pkl                    # Trained PRO model
├── ultra_pro_model.pkl              # Trained ULTRA PRO model
├── training_metadata.json           # Training info
├── processed_jobs.json              # Cached dataset
└── test_pro_model.py                # Testing script
```

---

## Production Deployment Checklist

- [ ] Change `SECRET_KEY` in settings.py
- [ ] Set `DEBUG = False`
- [ ] Configure `ALLOWED_HOSTS`
- [ ] Set `secure=True` for cookies (HTTPS)
- [ ] Use environment variables for sensitive data
- [ ] Set up automatic model retraining (cron job)
- [ ] Configure database backups
- [ ] Set up monitoring and logging
- [ ] Load test recommendation endpoints
- [ ] Implement rate limiting
- [ ] Set up CDN for static files
- [ ] Configure CORS properly
- [ ] Enable database connection pooling

---

## Support & Contact

For issues, questions, or contributions:
- Email: support@jobportal.com
- Documentation: https://docs.jobportal.com
- GitHub: https://github.com/yourorg/job-portal

---
