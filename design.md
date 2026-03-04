# FixIt Hub – AI-Powered Civic Intelligence Platform

## Design Document v1.0

## Table of Contents

- [1. System Overview](#1-system-overview)
- [2. Core Objectives](#2-core-objectives)
- [3. High-Level Architecture](#3-high-level-architecture)
- [4. Functional Modules](#4-functional-modules)
- [5. Complete Database Design](#5-complete-database-design)
- [6. API Design](#6-api-design)
- [7. Security & Non-Functional Requirements](#7-security--non-functional-requirements)
- [8. Deployment Design](#8-deployment-design)
- [9. Future Enhancements](#9-future-enhancements)
- [10. Technical Stack Summary](#10-technical-stack-summary)
- [11. Development Roadmap](#11-development-roadmap)
- [12. Success Metrics](#12-success-metrics)
- [13. Risk Mitigation](#13-risk-mitigation)
- [14. Compliance & Standards](#14-compliance--standards)
- [15. Conclusion](#15-conclusion)
- [Appendix A: Glossary](#appendix-a-glossary)
- [Appendix B: API Quick Reference](#appendix-b-api-quick-reference)
- [Appendix C: Database ER Diagram](#appendix-c-database-er-diagram)
- [Appendix D: Sample AI Prompts](#appendix-d-sample-ai-prompts)
- [Document Information](#document-information)

---

## 1. System Overview

### 1.1 Introduction

**FixIt Hub** is a cloud-native, AI-powered civic intelligence platform designed specifically for India's diverse urban and semi-urban landscapes. The system enables citizens to report critical infrastructure issues including potholes, garbage overflow, water leakage, broken streetlights, and road damage through an intuitive web interface.

### 1.2 Problem Statement

Traditional civic complaint systems suffer from:

- Reactive rather than proactive governance
- Language barriers in multilingual regions
- Lack of transparency in resolution tracking
- No predictive analytics for infrastructure stress zones
- Poor citizen engagement due to lack of feedback loops

### 1.3 Solution Approach

The platform transforms raw citizen complaints into structured, actionable intelligence through:

- **AI-powered multilingual normalization** supporting 22+ Indian languages
- **Territory-based automatic routing** using PostGIS spatial queries
- **Asynchronous AI pipeline** via Celery workers for non-blocking operations
- **Duplicate detection** using spatial proximity and perceptual image hashing
- **Risk-based prioritization** combining age, votes, and severity
- **Fraud-resistant resolution proof** with GPS and timestamp validation
- **Offline-first PWA** with IndexedDB and background sync
- **Predictive risk scoring** to identify emerging infrastructure stress zones
- **Weekly AI-generated executive reports** for decision-makers
- **Transparent resolution tracking** with citizen reopen capabilities
- **Cloud-native architecture** ensuring scalability and reliability

### 1.4 Key Differentiators

> - **Geospatial Intelligence**: PostGIS + GiST indexes for O(log n) territory routing
> - **Asynchronous AI Pipeline**: Non-blocking Celery workers prevent API bottlenecks
> - **Duplicate Detection**: Multi-factor similarity (spatial + temporal + visual)
> - **Risk-Based Prioritization**: Dynamic scoring for officer dashboards
> - **Fraud-Resistant Proofs**: GPS metadata + timestamp + hash verification
> - **Offline-First PWA**: IndexedDB + service workers + background sync
> - **Smart Territory Routing**: Automatic assignment with workload balancing
> - **Public Transparency**: Real-time dashboards with open data API
> - **Spam Prevention**: Rate limiting + phone verification + reputation scoring
> - **Scalable Storage**: Pre-signed URLs + CDN delivery + image optimization
> - **Modular Services**: Loosely coupled microservices for independent scaling
> - **Comprehensive Monitoring**: Prometheus + Grafana + structured logging

---

## 2. Core Objectives

1. **Centralized Civic Issue Reporting**: Single platform for all infrastructure complaints
2. **Territory-Based Automatic Routing**: Geographic jurisdiction mapping for instant routing
3. **Transparent Resolution Tracking**: Real-time status updates with proof of resolution
4. **AI-Based Intelligence**: Multilingual normalization, classification, and insights
5. **Predictive Analytics**: Risk scoring to identify emerging problem zones
6. **Proximity Intelligence**: Cluster detection and geographic heatmaps
7. **Executive Reporting**: Weekly AI-generated summaries for governance
8. **Scalable Cloud Architecture**: Modern cloud infrastructure for national-scale deployment

---

## 3. High-Level Architecture

### 3.1 Architecture Overview

The system follows a modern three-tier cloud-native architecture deployed on AWS:

```mermaid
flowchart LR
  UI[React + Vite] --> Edge[Vercel CDN]
  Edge --> Gateway[Render Load Balancer]
  Gateway --> API[FastAPI Backend]
  API --> Appwrite[Appwrite Cloud]
  Appwrite --> DB[(Appwrite Database)]
  Appwrite --> Storage[(Appwrite Storage)]
  Appwrite --> Auth[Appwrite Auth]
  API --> AI[Amazon Bedrock or External AI]
  API --> Logs[Render Logs + Metrics]
```

### 3.2 Component Description

#### 3.2.0 Geospatial Architecture (PostGIS)

**Technology Stack:**
- PostgreSQL 15.5 with PostGIS 3.4.1 extension
- GiST (Generalized Search Tree) spatial indexes
- GEOS 3.12 library for geometric operations

**Territory Mapping:**

```sql
-- Territory boundaries table
CREATE TABLE territories (
    id UUID PRIMARY KEY,
    territory_code VARCHAR(50) UNIQUE,
    territory_name VARCHAR(255),
    territory_type VARCHAR(50), -- ward, district, zone
    boundary GEOMETRY(POLYGON, 4326), -- WGS84 coordinate system
    assigned_officers UUID[],
    department VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Create spatial index for O(log n) lookups
CREATE INDEX idx_territories_boundary 
ON territories USING GIST(boundary);

-- Point-in-polygon query (< 10ms for 1000+ territories)
SELECT territory_code, territory_name
FROM territories
WHERE ST_Contains(
    boundary,
    ST_SetSRID(ST_MakePoint(longitude, latitude), 4326)
);
```

**Spatial Clustering (DBSCAN):**

```sql
-- Find issue clusters within 200m radius
WITH clustered_issues AS (
    SELECT 
        i.id,
        i.title,
        i.location,
        ST_ClusterDBSCAN(
            i.location,
            eps := 0.002,  -- ~200m in degrees
            minpoints := 3
        ) OVER () AS cluster_id
    FROM issues i
    WHERE i.status IN ('OPEN', 'IN_PROGRESS')
)
SELECT 
    cluster_id,
    COUNT(*) as issue_count,
    ST_Centroid(ST_Collect(location)) as cluster_center,
    array_agg(id) as issue_ids
FROM clustered_issues
WHERE cluster_id IS NOT NULL
GROUP BY cluster_id
HAVING COUNT(*) >= 3;
```

**Proximity Search:**

```sql
-- Find issues within radius (uses spatial index)
SELECT 
    id,
    title,
    ST_Distance(
        location::geography,
        ST_SetSRID(ST_MakePoint($longitude, $latitude), 4326)::geography
    ) as distance_meters
FROM issues
WHERE ST_DWithin(
    location::geography,
    ST_SetSRID(ST_MakePoint($longitude, $latitude), 4326)::geography,
    $radius_meters
)
ORDER BY distance_meters
LIMIT 20;
```

**Performance Characteristics:**
- Point-in-polygon: O(log n) with GiST index
- Proximity search: < 50ms for 100k issues
- Cluster detection: < 200ms for 10k open issues
- Territory lookup: < 10ms average

#### 3.2.1 Frontend Layer

- **Technology**: React 18.2 with Vite 5.0 build tool
- **Hosting**: Vercel (automatic deployments with CDN)
- **Features**:
  - Responsive design for mobile and desktop
  - Interactive map integration (Leaflet 1.9.4)
  - Real-time status updates
  - Multilingual UI support
  - PWA with offline capabilities

#### 3.2.2 API Gateway

- **Service**: Render Load Balancer (built-in)
- **Responsibilities**:
  - Request routing and load balancing
  - SSL/TLS termination (automatic)
  - Rate limiting and throttling
  - CORS handling
  - Request/response transformation

#### 3.2.3 Backend Application Layer

- **Technology**: FastAPI 0.109 (Python 3.11.7)
- **Hosting**: Render (containerized deployment with auto-scaling)
- **Key Libraries**:
  - Appwrite SDK 4.1.0 (authentication, database, storage)
  - Pydantic 2.5.3 (validation)
  - Pillow 10.2.0 (image processing)
  - httpx 0.26.0 (async HTTP client)
  - SQLAlchemy 2.0.25 + GeoAlchemy2 0.14.3 (spatial ORM)
  - Celery 5.3.4 (task queue)
  - prometheus-client 0.19.0 (metrics)

#### 3.2.4 Database Layer

- **Service**: Appwrite Database (NoSQL document database)
- **Configuration**:
  - Cloud-hosted with automatic backups
  - Real-time data synchronization
  - Built-in access control and permissions
  - Automatic scaling

#### 3.2.5 Object Storage

- **Service**: Appwrite Storage
- **Buckets**:
  - `issue-images`: Issue images uploaded by citizens
  - `resolution-proofs`: Resolution proof photos
- **Features**:
  - Automatic encryption at rest
  - CDN integration for fast delivery
  - Built-in access control
  - Automatic image optimization

#### 3.2.6 AI Processing Layer

**Architecture: Asynchronous Task Queue**

```
API Request → FastAPI → Redis Queue → Celery Workers → AI Service
     ↓                                      ↓
  Immediate                            Background
  Response                             Processing
```

**Technology Stack:**
- **Celery**: Distributed task queue
- **Redis**: Message broker and result backend
- **Flower**: Real-time monitoring dashboard
- **Amazon Bedrock / OpenAI**: AI model providers

**Task Queue Configuration:**

```python
# celery_config.py
from celery import Celery

app = Celery(
    'fixit_hub',
    broker='redis://localhost:6379/0',
    backend='redis://localhost:6379/1'
)

app.conf.update(
    task_serializer='json',
    result_serializer='json',
    accept_content=['json'],
    timezone='Asia/Kolkata',
    enable_utc=True,
    
    # Retry configuration
    task_acks_late=True,
    task_reject_on_worker_lost=True,
    task_default_retry_delay=60,  # 1 minute
    task_max_retries=3,
    
    # Performance tuning
    worker_prefetch_multiplier=4,
    worker_max_tasks_per_child=1000,
    
    # Queue priorities
    task_routes={
        'tasks.translate_issue': {'queue': 'high_priority'},
        'tasks.classify_issue': {'queue': 'high_priority'},
        'tasks.generate_summary': {'queue': 'low_priority'},
    }
)
```

**Async Task Implementation:**

```python
# tasks.py
from celery import Task
from celery.exceptions import Retry
import logging

logger = logging.getLogger(__name__)

class AITask(Task):
    """Base task with retry logic and error handling"""
    autoretry_for = (ConnectionError, TimeoutError)
    retry_kwargs = {'max_retries': 3}
    retry_backoff = True  # Exponential backoff
    retry_backoff_max = 600  # Max 10 minutes
    retry_jitter = True  # Add randomness to prevent thundering herd

@app.task(base=AITask, bind=True)
def translate_issue(self, issue_id, description, source_lang):
    """
    Translate issue description to English
    Non-blocking: Returns immediately, processes in background
    """
    try:
        logger.info(f"Translating issue {issue_id} from {source_lang}")
        
        # Call AI service
        translated_text = ai_service.translate(
            text=description,
            source_lang=source_lang,
            target_lang='en'
        )
        
        # Update database
        db.issues.update(issue_id, {
            'description_translated': translated_text,
            'translation_status': 'completed'
        })
        
        # Chain next task: classification
        classify_issue.delay(issue_id, translated_text)
        
        logger.info(f"Translation completed for issue {issue_id}")
        return {'status': 'success', 'issue_id': issue_id}
        
    except Exception as exc:
        logger.error(f"Translation failed for issue {issue_id}: {exc}")
        
        # Mark for manual review
        db.issues.update(issue_id, {
            'translation_status': 'failed',
            'requires_manual_review': True
        })
        
        # Retry with exponential backoff
        raise self.retry(exc=exc, countdown=60 * (2 ** self.request.retries))

@app.task(base=AITask, bind=True)
def classify_issue(self, issue_id, description):
    """Classify issue into predefined categories"""
    try:
        result = ai_service.classify(
            description=description,
            categories=['pothole', 'garbage', 'water', 'electrical', 'road', 'drainage']
        )
        
        db.issues.update(issue_id, {
            'category': result['category'],
            'category_confidence': result['confidence'],
            'ai_processed': True
        })
        
        return {'status': 'success', 'category': result['category']}
        
    except Exception as exc:
        logger.error(f"Classification failed for issue {issue_id}: {exc}")
        db.issues.update(issue_id, {'category': 'other', 'ai_processed': False})
        raise self.retry(exc=exc)

@app.task(base=AITask)
def generate_weekly_summary(week_start, week_end):
    """Generate AI executive summary (low priority, runs weekly)"""
    stats = aggregate_weekly_stats(week_start, week_end)
    summary = ai_service.generate_summary(stats)
    
    db.weekly_reports.create({
        'week_start': week_start,
        'week_end': week_end,
        'statistics': stats,
        'executive_summary': summary
    })
    
    return {'status': 'success'}
```

**API Integration (Non-Blocking):**

```python
# api/issues.py
from fastapi import APIRouter, BackgroundTasks

router = APIRouter()

@router.post("/issues")
async def create_issue(issue: IssueCreate):
    # 1. Validate and store issue immediately
    issue_id = db.issues.create({
        'title': issue.title,
        'description': issue.description,
        'location': issue.location,
        'status': 'OPEN',
        'translation_status': 'pending',
        'ai_processed': False
    })
    
    # 2. Queue AI processing (non-blocking)
    translate_issue.delay(
        issue_id=issue_id,
        description=issue.description,
        source_lang=issue.language
    )
    
    # 3. Return immediately (< 100ms response time)
    return {
        'issue_id': issue_id,
        'status': 'created',
        'message': 'Issue created. AI processing in progress.'
    }
```

**Monitoring & Observability:**

```python
# Prometheus metrics
from prometheus_client import Counter, Histogram

ai_tasks_total = Counter(
    'ai_tasks_total',
    'Total AI tasks processed',
    ['task_name', 'status']
)

ai_task_duration = Histogram(
    'ai_task_duration_seconds',
    'AI task processing time',
    ['task_name']
)

queue_length = Gauge(
    'celery_queue_length',
    'Number of tasks in queue',
    ['queue_name']
)
```

**Capabilities:**
- Multilingual translation (22+ Indian languages → English)
- Issue classification (pothole, garbage, water, electrical, road)
- Executive summary generation
- Sentiment analysis (optional)
- Automatic retry with exponential backoff
- Dead letter queue for failed tasks
- Real-time monitoring via Flower dashboard

**Performance Characteristics:**
- API response time: < 100ms (non-blocking)
- Translation time: 2-5 seconds (background)
- Classification time: 1-3 seconds (background)
- Queue throughput: 100+ tasks/second
- Worker auto-scaling: 3-20 workers based on queue depth

#### 3.2.7 Monitoring & Logging

- **Service**: Render Logs + Metrics Dashboard
- **Metrics**:
  - API response times
  - Error rates
  - Memory and CPU usage
  - Request throughput
  - AI processing latency
- **Alarms**: Automated alerts for critical failures
- **Logs**: Centralized application and access logs

### 3.3 Data Flow

#### 3.3.1 Issue Submission Flow

1. Citizen submits issue via React frontend
2. Frontend uploads image to Appwrite Storage (secure upload)
3. Frontend sends issue data to FastAPI backend
4. Backend validates and stores in Appwrite Database
5. Backend triggers async AI processing
6. AI translates and classifies issue
7. Backend updates issue document with AI metadata
8. Frontend receives confirmation and issue ID

#### 3.3.2 Resolution Flow

1. Officer marks issue as resolved with proof photo
2. Backend uploads proof to Appwrite Storage
3. Backend updates issue status and resolution timestamp
4. Citizen receives notification
5. Citizen can reopen (if within limits)

#### 3.3.3 Weekly Report Generation Flow

1. Scheduled job (cron/Render cron) triggers report generation
2. Backend aggregates weekly statistics from Appwrite Database
3. Backend computes risk scores for all wards
4. Backend sends structured data to AI for executive summary
5. AI generates natural language report
6. Backend stores report in database
7. Dashboard displays latest report

---

## 4. Functional Modules

### 4.1 Territory-Based Issue Management Module

#### 4.1.1 Problem Statement

Traditional civic complaint platforms suffer from:

- **Manual Routing**: Officials receive thousands of reports from all regions
- **Jurisdiction Confusion**: Time wasted filtering irrelevant complaints
- **Manual Forwarding**: Complaints passed between departments manually
- **Duplicate Reports**: No visibility of nearby existing complaints
- **Accountability Gaps**: Unclear responsibility for specific territories

**Result**: Slow resolution, confusion, poor coordination, and citizen frustration.

#### 4.1.2 Territory-Based Solution

FixIt Hub implements intelligent geographic jurisdiction mapping where every issue is:

1. **Geotagged** with precise GPS coordinates
2. **Mapped** to defined administrative territories
3. **Automatically Routed** to responsible authorities
4. **Filtered** so officials see only their jurisdiction

#### 4.1.3 How Territory Management Works

**Step 1: Issue Reporting with Geographic Data**

When a citizen reports an issue, the system collects:
- Photo/video evidence
- Description (multilingual)
- Automatic GPS location
- Reverse geocoded address

Example:
```
Issue: Pothole on road
Location: 19.115°N, 72.842°E
Area: Andheri East
Ward: K/East
```

**Step 2: Territory Mapping**

Cities are divided into territorial zones:
- **Ward**: Municipal ward boundaries
- **District**: Administrative districts
- **Municipality Zone**: Service zones
- **Department Jurisdiction**: Department-specific areas

Each territory is stored as a geographic boundary polygon in the database.

Example Territory Boundaries:
```javascript
{
  "ward_id": "K_EAST",
  "ward_name": "Ward K/East",
  "boundary": {
    "type": "Polygon",
    "coordinates": [[[72.84, 19.11], [72.85, 19.11], ...]]
  },
  "assigned_officers": ["officer_id_1", "officer_id_2"],
  "department": "Public Works"
}
```

**Step 3: Automatic Jurisdiction Detection**

When an issue is submitted:
1. System performs **point-in-polygon spatial query**
2. Checks: Does this GPS coordinate fall inside a territory boundary?
3. If yes: Issue → Assigned to that territory's authority

Example Flow:
```
Reported Location: 19.115°N, 72.842°E
↓
Spatial Query: Point in Polygon?
↓
Matched Territory: BMC Ward K/East
↓
Assigned Authority: Ward Officer (K/East)
↓
No manual routing required ✓
```

#### 4.1.4 Territory-Based Filtering

**For Citizens:**

Users can:
- View issues near their location (radius-based)
- Sort complaints by:
  - Nearest first
  - Severity level
  - Unresolved status
- Avoid duplicate reporting by seeing existing nearby issues

Example Citizen View:
```
Nearby Issues (within 500m)
─────────────────────────────
🔴 Pothole - 120m away - OPEN
🟡 Garbage Overflow - 300m away - IN_PROGRESS
🟢 Broken Streetlight - 450m away - RESOLVED
```

**For Government Officials:**

Officials get a jurisdiction-filtered dashboard:

Example Officer Dashboard:
```
Ward Officer Dashboard
Region: Ward K/East

Issues Summary:
- 42 Open complaints
- 18 In progress
- 7 Critical priority
- 15 Resolved this week

Filter Options:
✓ My Territory Only
□ All Territories
□ Department: Public Works
□ Status: Open
□ Priority: High
```

Officials can filter by:
- Territory/ward
- Department
- Status (open, in progress, resolved)
- Priority level
- Time range

#### 4.1.5 Proximity Intelligence

The system enables advanced proximity-based queries:

**Cluster Detection:**
Authorities can identify issue clusters:

Example:
```
Road Damage Cluster Detected
Location: Andheri Metro Road
Reports: 12 complaints within 200m radius
Category: Pothole, Road Damage
Recommendation: Single repair operation
```

**Geographic Heatmaps:**
Visual representation of issue density:
- Red zones: High complaint density
- Orange zones: Medium density
- Green zones: Low density

**Nearest Unresolved Issues:**
Officers can view:
- Nearest open issues from current location
- Route optimization for field visits
- Batch resolution planning

#### 4.1.6 Database Schema (territories collection)

```javascript
// Appwrite Database Collection: territories
{
  "$id": "unique_territory_id",
  "territory_type": "string (ward|district|zone)",
  "territory_code": "string (e.g., K_EAST)",
  "territory_name": "string",
  "city": "string",
  "state": "string",
  
  // Geographic boundary (GeoJSON format)
  "boundary": "string (JSON)",
  
  // Assigned personnel
  "assigned_officers": "string[] (array of user IDs)",
  "department": "string",
  "contact_email": "string",
  "contact_phone": "string",
  
  // Metadata
  "population": "integer (optional)",
  "area_sq_km": "float (optional)",
  
  "$createdAt": "datetime",
  "$updatedAt": "datetime"
}

// Indexes:
// - territory_code (unique)
// - city
// - department
```

#### 4.1.7 Spatial Query Implementation

**Point-in-Polygon Algorithm:**

```python
def find_territory(latitude, longitude):
    """
    Find which territory contains the given coordinates
    """
    point = {
        "type": "Point",
        "coordinates": [longitude, latitude]
    }
    
    # Query all territories
    territories = database.list_documents('territories')
    
    for territory in territories:
        boundary = json.loads(territory['boundary'])
        
        # Perform point-in-polygon check
        if point_in_polygon(point, boundary):
            return territory
    
    return None  # No territory found

def auto_assign_issue(issue_id, latitude, longitude):
    """
    Automatically assign issue to territory officer
    """
    territory = find_territory(latitude, longitude)
    
    if territory:
        # Update issue with territory and assigned officer
        database.update_document('issues', issue_id, {
            'ward': territory['territory_name'],
            'territory_id': territory['$id'],
            'assigned_to': territory['assigned_officers'][0],  # Primary officer
            'auto_assigned': True
        })
        
        # Notify officer
        notify_officer(territory['assigned_officers'][0], issue_id)
        
        return True
    else:
        # Fallback: Assign to admin for manual routing
        assign_to_admin(issue_id)
        return False
```

#### 4.1.8 Advantages of Territory-Based System

1. **Automatic Routing**
   - Issues go directly to correct authority
   - Zero manual forwarding
   - Instant assignment upon submission

2. **Faster Resolution**
   - Officials see only relevant issues
   - Reduced clutter and confusion
   - Clear accountability per territory

3. **Transparency**
   - Citizens track issues in their area
   - Visible nearby complaints
   - Community awareness

4. **Data-Driven Governance**
   - Analyze issue density by territory
   - Identify problem hotspots
   - Track recurring infrastructure failures
   - Resource allocation optimization

5. **Reduced Duplicate Complaints**
   - Nearby issues visible to citizens
   - Prevents redundant reporting
   - Cleaner data for analytics

6. **Efficient Field Operations**
   - Cluster-based resolution
   - Route optimization for officers
   - Batch processing of nearby issues

---

### 4.2 User Management Module

#### 4.2.1 User Roles

- **Public/Citizen**: Can report issues, view status, reopen resolved issues, view public dashboards
- **Government Officer**: Can resolve issues, upload proof, view analytics, manage territory, customize and schedule reports

#### 4.2.2 Registration & Authentication

- **Citizen Registration**:
  - Email/phone number
  - Password (bcrypt hashed)
  - Name and location
  - Phone verification via OTP
  - Optional: Aadhaar verification for verified badge

- **Officer Registration**:
  - Government email domain verification
  - Department and ward assignment
  - Officer approval by existing officer or self-registration with verification
  - Verified badge upon approval
  - Territory assignment

#### 4.2.3 Authentication Mechanism

- **JWT-based authentication**
- Access token (15 min expiry)
- Refresh token (7 day expiry)
- Secure HTTP-only cookies
- Role-based access control (RBAC)

#### 4.2.4 Spam & Abuse Prevention

**Multi-Layered Protection:**

```python
class SpamPreventionService:
    """Comprehensive spam and abuse prevention"""
    
    def check_submission_eligibility(self, user, issue):
        """
        Multi-factor spam detection
        Returns: (allowed: bool, reason: str)
        """
        # 1. Rate limiting
        if self.check_rate_limit(user):
            return False, "Rate limit exceeded (10 issues/day)"
        
        # 2. Phone verification
        if not user.phone_verified:
            return False, "Phone verification required"
        
        # 3. Reputation score
        if user.reputation_score < 0.3:
            return False, "Low reputation - requires admin review"
        
        # 4. Duplicate content detection
        if self.detect_duplicate_content(issue):
            return False, "Duplicate or spam content detected"
        
        # 5. Behavioral analysis
        if self.detect_suspicious_pattern(user):
            return False, "Suspicious activity pattern"
        
        return True, "Approved"
    
    def check_rate_limit(self, user):
        """10 issues per day per user"""
        today_count = db.issues.count({
            'reported_by': user.id,
            'created_at': {'$gte': datetime.now() - timedelta(days=1)}
        })
        return today_count >= 10
    
    def calculate_reputation_score(self, user):
        """
        Reputation = (successful_resolutions / total_reports) * 0.7 +
                     (1 - spam_reports / total_reports) * 0.3
        """
        issues = db.issues.find({'reported_by': user.id})
        total = len(issues)
        
        if total == 0:
            return 0.5  # Neutral for new users
        
        resolved = len([i for i in issues if i['status'] == 'RESOLVED'])
        spam_flagged = len([i for i in issues if i.get('flagged_as_spam')])
        
        success_rate = resolved / total
        spam_rate = spam_flagged / total
        
        reputation = (success_rate * 0.7) + ((1 - spam_rate) * 0.3)
        return max(0.0, min(1.0, reputation))
    
    def detect_suspicious_pattern(self, user):
        """Behavioral analysis for abuse detection"""
        recent_issues = db.issues.find({
            'reported_by': user.id,
            'created_at': {'$gte': datetime.now() - timedelta(hours=24)}
        })
        
        # Check for rapid-fire submissions
        if len(recent_issues) > 5:
            timestamps = [i['created_at'] for i in recent_issues]
            intervals = [timestamps[i+1] - timestamps[i] for i in range(len(timestamps)-1)]
            avg_interval = sum(intervals, timedelta()) / len(intervals)
            
            if avg_interval < timedelta(minutes=5):
                return True  # Suspicious: < 5 min between submissions
        
        # Check for identical descriptions
        descriptions = [i['description'] for i in recent_issues]
        if len(descriptions) != len(set(descriptions)):
            return True  # Suspicious: duplicate descriptions
        
        return False
```

**Phone Verification Flow:**

```python
@router.post("/auth/verify-phone")
async def verify_phone(phone: str):
    # Generate 6-digit OTP
    otp = generate_otp()
    
    # Store in Redis with 10-minute expiry
    redis.setex(f"otp:{phone}", 600, otp)
    
    # Send via SMS gateway
    sms_service.send(phone, f"Your FixIt Hub OTP: {otp}")
    
    return {'message': 'OTP sent'}

@router.post("/auth/confirm-otp")
async def confirm_otp(phone: str, otp: str):
    stored_otp = redis.get(f"otp:{phone}")
    
    if stored_otp and stored_otp == otp:
        db.users.update({'phone': phone}, {'phone_verified': True})
        redis.delete(f"otp:{phone}")
        return {'message': 'Phone verified'}
    
    return {'error': 'Invalid OTP'}, 400
```

#### 4.2.5 Database Schema (users collection)

```javascript
// Appwrite Database Collection: users
{
  "$id": "unique_user_id",
  "email": "string (unique, indexed)",
  "phone": "string (unique, optional)",
  "password_hash": "string (handled by Appwrite Auth)",
  "full_name": "string",
  "role": "string (citizen|officer)",
  "is_verified": "boolean",
  "phone_verified": "boolean",
  "reputation_score": "float (0.0-1.0, default 0.5)",
  
  // Officer-specific fields
  "department": "string (optional)",
  "ward_assigned": "string[] (array, optional)",
  "territories_assigned": "string[] (array of territory IDs)",
  "can_approve_officers": "boolean (default false)",
  
  // Report customization preferences
  "report_preferences": {
    "default_territories": "string[]",
    "default_categories": "string[]",
    "default_format": "string (pdf|csv|excel)",
    "email_notifications": "boolean",
    "scheduled_reports": "object[]"
  },
  
  "$createdAt": "datetime",
  "$updatedAt": "datetime"
}

// Indexes:
// - email (unique)
// - role
// - phone (unique)
```

### 4.2 Issue Reporting Module

#### 4.2.1 Issue Submission

Citizens can submit issues with:

- **Title**: Brief description (max 200 chars)
- **Description**: Detailed explanation (multilingual support)
- **Category**: Auto-classified by AI or manual selection
- **Location**: GPS coordinates or map pin
- **Image**: Photo evidence (max 5MB, JPEG/PNG)
- **Language**: Original language of submission

#### 4.2.2 Location Tagging

- Interactive map interface (Leaflet.js)
- GPS auto-detection
- Manual pin placement
- Address reverse geocoding
- Ward/zone auto-assignment based on coordinates

#### 4.2.3 Status Management

Issue lifecycle states:

- **OPEN**: Newly reported, awaiting action
- **IN_PROGRESS**: Officer assigned, work started
- **RESOLVED**: Marked complete with proof
- **REOPENED**: Citizen disputed resolution
- **CLOSED**: Final closure after reopen limit

#### 4.2.4 Database Schema (issues collection)

```javascript
// Appwrite Database Collection: issues
{
  "$id": "unique_issue_id",
  "title": "string (max 200 chars)",
  "description": "string (text)",
  "description_translated": "string (optional)",
  "category": "string",
  "category_confidence": "float",
  "status": "string (OPEN|IN_PROGRESS|RESOLVED|REOPENED|CLOSED)",
  
  // Location data
  "latitude": "float",
  "longitude": "float",
  "address": "string",
  "ward": "string",
  "city": "string",
  
  // Media
  "image_url": "string (Appwrite Storage file ID)",
  
  // User tracking
  "reported_by": "string (user ID)",
  "assigned_to": "string (user ID, optional)",
  
  // Resolution tracking
  "resolved_at": "datetime (optional)",
  "resolution_proof_url": "string (Appwrite Storage file ID, optional)",
  "resolution_notes": "string (optional)",
  
  // Reopen logic
  "reopen_count": "integer (default 0)",
  "last_reopened_at": "datetime (optional)",
  "reopen_reason": "string (optional)",
  
  // AI metadata
  "original_language": "string",
  "ai_processed": "boolean (default false)",
  
  // Timestamps
  "$createdAt": "datetime",
  "$updatedAt": "datetime"
}

// Indexes:
// - status
// - category
// - ward
// - $createdAt
// - latitude, longitude (for geospatial queries)
```

### 4.3 Resolution & Reopen Control Module

#### 4.3.1 Resolution Process

**Officer Actions**:

1. View assigned issues in dashboard
2. Update status to IN_PROGRESS
3. Complete resolution work
4. Upload proof photo (before/after comparison)
5. Add resolution notes
6. Mark as RESOLVED

**Backend Validation**:

- Proof photo required (enforced)
- Resolution notes minimum 20 characters
- Timestamp recorded
- Citizen notification triggered

#### 4.3.2 Reopen Logic

**Citizen Reopen Rights**:

- Can reopen RESOLVED issues
- Maximum 2 reopens per issue
- 36-hour cooldown between consecutive reopens
- Must provide reopen reason (min 50 chars)

**Business Rules**:
```python
def can_reopen(issue):
    if issue.reopen_count >= 2:
        return False, "Maximum reopen limit reached"
    
    if issue.last_reopened_at:
        hours_since_reopen = (now() - issue.last_reopened_at).total_seconds() / 3600
        if hours_since_reopen < 36:
            return False, f"Cooldown active. Wait {36 - hours_since_reopen:.1f} hours"
    
    if issue.status != 'RESOLVED':
        return False, "Only resolved issues can be reopened"
    
    return True, "Reopen allowed"
```

#### 4.3.3 Transparency Features

- Full resolution history visible to citizen
- All previous resolution attempts displayed
- Proof photos accessible
- Officer notes visible
- Reopen reasons logged

#### 4.3.4 Database Updates

```javascript
// Appwrite Database Collection: resolution_history
{
  "$id": "unique_resolution_id",
  "issue_id": "string (reference to issues collection)",
  "resolved_by": "string (user ID)",
  "resolved_at": "datetime",
  "proof_url": "string (Appwrite Storage file ID)",
  "notes": "string",
  "reopened": "boolean (default false)",
  "reopen_reason": "string (optional)",
  "reopened_at": "datetime (optional)",
  "$createdAt": "datetime",
  "$updatedAt": "datetime"
}

// Index on issue_id for fast lookups
```

### 4.4 AI Processing Module

#### 4.4.1 Multilingual Translation

**Supported Languages**: Hindi, Bengali, Telugu, Marathi, Tamil, Gujarati, Urdu, Kannada, Odia, Malayalam, Punjabi, Assamese, Maithili, Sanskrit, Konkani, Nepali, Sindhi, Dogri, Kashmiri, Manipuri, Bodo, Santali, English

**Translation Flow**:

1. Detect source language from user input
2. Send to AI model with prompt:
```
Translate the following civic complaint from {source_lang} to English.
Preserve technical terms and location names.

Original: {description}
```

3. Store translated text in `description_translated` field
4. Fallback: If AI fails, store original text

#### 4.4.2 Automatic Classification

**Categories**:

- Pothole
- Garbage/Waste Management
- Water Leakage/Supply
- Electrical/Streetlight
- Road Damage
- Drainage/Sewage
- Other

**Classification Prompt**:
```json
{
  "task": "classify_civic_issue",
  "description": "{translated_description}",
  "title": "{title}",
  "categories": ["pothole", "garbage", "water", "electrical", "road", "drainage", "other"],
  "output_format": {
    "category": "string",
    "confidence": "float (0-1)"
  }
}
```

**Confidence Handling**:

- Confidence > 0.8: Auto-assign category
- Confidence 0.5-0.8: Suggest to officer for confirmation
- Confidence < 0.5: Mark as "Other", require manual classification

#### 4.4.3 Weekly Executive Summary Generation

**Input Data Structure**:
```json
{
  "week_start": "2026-02-09",
  "week_end": "2026-02-15",
  "total_issues": 1247,
  "resolved_issues": 892,
  "open_issues": 355,
  "growth_rate": 12.5,
  "avg_resolution_time_hours": 48.3,
  "top_category": "pothole",
  "top_ward": "Ward 23 - Koramangala",
  "high_risk_zones": [
    {"ward": "Ward 23", "risk_score": 87.5},
    {"ward": "Ward 15", "risk_score": 76.2}
  ],
  "category_breakdown": {
    "pothole": 456,
    "garbage": 312,
    "water": 189
  }
}
```

**AI Prompt**:
```
You are a civic intelligence analyst. Generate a concise executive summary 
for government officials based on the following weekly civic issue data.

Focus on:
1. Key trends and changes
2. Areas requiring immediate attention
3. Performance metrics
4. Actionable recommendations

Data: {json_data}

Generate a 200-300 word executive summary in professional tone.
```

#### 4.4.4 Fallback Handling

**AI Service Failure Strategy**:

1. Retry with exponential backoff (3 attempts)
2. If translation fails: Store original text, flag for manual review
3. If classification fails: Assign "Other" category
4. If summary fails: Generate template-based summary
5. Log all failures to CloudWatch for monitoring

**Error Logging**:
```python
logger.error(
    "AI processing failed",
    extra={
        "issue_id": issue_id,
        "operation": "translation",
        "error": str(e),
        "retry_count": retry_count
    }
)
```

### 4.5 Analytics & Risk Scoring Module

#### 4.5.1 Risk Score Formula

**Mathematical Model**:
```
Risk Score = (w1 × R) + (w2 × T) + (w3 × G) + (w4 × S)

Where:
R = Repeated Reports Score (0-100)
T = Time Unresolved Score (0-100)
G = Growth Rate Score (0-100)
S = Severity Score (0-100)

Weights (tunable):
w1 = 0.35 (repeated reports)
w2 = 0.30 (time factor)
w3 = 0.25 (growth rate)
w4 = 0.10 (severity)
```

#### 4.5.2 Component Calculations

**Repeated Reports Score (R)**:
```python
def calculate_repeated_reports_score(ward, radius_km=1.0):
    # Count issues within radius in last 30 days
    cluster_count = count_issues_in_radius(ward, radius_km, days=30)
    
    # Normalize to 0-100
    max_threshold = 50  # 50+ issues = max score
    return min(100, (cluster_count / max_threshold) * 100)
```

**Time Unresolved Score (T)**:
```python
def calculate_time_score(ward):
    open_issues = get_open_issues(ward)
    
    if not open_issues:
        return 0
    
    avg_days_open = sum(
        (now() - issue.created_at).days for issue in open_issues
    ) / len(open_issues)
    
    # Normalize: 30+ days = max score
    return min(100, (avg_days_open / 30) * 100)
```

**Growth Rate Score (G)**:
```python
def calculate_growth_score(ward):
    current_week_count = count_issues(ward, weeks=1)
    previous_week_count = count_issues(ward, weeks=2, offset=1)
    
    if previous_week_count == 0:
        return 0
    
    growth_rate = ((current_week_count - previous_week_count) / previous_week_count) * 100
    
    # Normalize: 50%+ growth = max score
    return min(100, max(0, growth_rate * 2))
```

**Severity Score (S)**:
```python
def calculate_severity_score(ward):
    # Weight categories by severity
    severity_weights = {
        'water': 1.0,      # Critical
        'electrical': 0.9,
        'drainage': 0.8,
        'pothole': 0.7,
        'road': 0.6,
        'garbage': 0.5,
        'other': 0.3
    }
    
    issues = get_open_issues(ward)
    weighted_sum = sum(severity_weights.get(issue.category, 0.5) for issue in issues)
    
    # Normalize
    return min(100, (weighted_sum / len(issues)) * 100) if issues else 0
```

#### 4.5.3 Risk Classification

```python
def classify_risk(risk_score):
    if risk_score >= 75:
        return "CRITICAL", "red"
    elif risk_score >= 50:
        return "HIGH", "orange"
    elif risk_score >= 25:
        return "MEDIUM", "yellow"
    else:
        return "LOW", "green"
```

#### 4.5.4 Weekly Aggregation Queries

```sql
-- Weekly statistics by ward
SELECT 
    ward,
    COUNT(*) as total_issues,
    COUNT(*) FILTER (WHERE status = 'RESOLVED') as resolved_count,
    COUNT(*) FILTER (WHERE status IN ('OPEN', 'IN_PROGRESS')) as open_count,
    AVG(EXTRACT(EPOCH FROM (resolved_at - created_at))/3600) as avg_resolution_hours,
    COUNT(*) FILTER (WHERE reopen_count > 0) as reopened_count
FROM issues
WHERE created_at >= NOW() - INTERVAL '7 days'
GROUP BY ward
ORDER BY total_issues DESC;

-- Category breakdown
SELECT 
    category,
    COUNT(*) as count,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) as percentage
FROM issues
WHERE created_at >= NOW() - INTERVAL '7 days'
GROUP BY category
ORDER BY count DESC;

-- Growth rate calculation
WITH current_week AS (
    SELECT COUNT(*) as count FROM issues 
    WHERE created_at >= NOW() - INTERVAL '7 days'
),
previous_week AS (
    SELECT COUNT(*) as count FROM issues 
    WHERE created_at >= NOW() - INTERVAL '14 days' 
    AND created_at < NOW() - INTERVAL '7 days'
)
SELECT 
    current_week.count as current,
    previous_week.count as previous,
    ROUND(((current_week.count - previous_week.count) * 100.0 / NULLIF(previous_week.count, 0)), 2) as growth_rate
FROM current_week, previous_week;
```

### 4.6 Weekly Report Generator

#### 4.6.1 Report Generation Trigger

**Scheduling Options**:

- **AWS EventBridge**: Cron expression `cron(0 2 ? * MON *)` (Every Monday 2 AM IST)
- **Alternative**: Celery Beat for self-hosted scheduling

#### 4.6.2 Report Generation Process

```python
async def generate_weekly_report():
    # 1. Aggregate statistics
    stats = await aggregate_weekly_stats()
    
    # 2. Calculate risk scores for all wards
    risk_scores = await calculate_all_ward_risks()
    
    # 3. Identify top performers and problem areas
    top_ward = max(stats['wards'], key=lambda x: x['total_issues'])
    top_category = max(stats['categories'], key=lambda x: x['count'])
    
    # 4. Prepare AI input
    ai_input = {
        "week_start": stats['week_start'],
        "week_end": stats['week_end'],
        "total_issues": stats['total_issues'],
        "resolved_issues": stats['resolved_issues'],
        "growth_rate": stats['growth_rate'],
        "avg_resolution_time_hours": stats['avg_resolution_time'],
        "top_category": top_category['name'],
        "top_ward": top_ward['name'],
        "high_risk_zones": [z for z in risk_scores if z['risk_score'] >= 50]
    }
    
    # 5. Generate AI summary
    executive_summary = await ai_service.generate_summary(ai_input)
    
    # 6. Store report
    report = await db.weekly_reports.create({
        "week_start": stats['week_start'],
        "week_end": stats['week_end'],
        "statistics": stats,
        "risk_scores": risk_scores,
        "executive_summary": executive_summary,
        "generated_at": datetime.now()
    })
    
    return report
```

#### 4.6.3 Database Schema (weekly_reports table)

```sql
CREATE TABLE weekly_reports (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    week_start DATE NOT NULL,
    week_end DATE NOT NULL,
    
    -- Aggregate statistics
    total_issues INTEGER NOT NULL,
    resolved_issues INTEGER NOT NULL,
    open_issues INTEGER NOT NULL,
    reopened_issues INTEGER NOT NULL,
    
    -- Performance metrics
    avg_resolution_time_hours DECIMAL(10, 2),
    growth_rate DECIMAL(10, 2),
    
    -- Top performers
    top_category VARCHAR(50),
    top_ward VARCHAR(100),
    
    -- Risk analysis
    high_risk_wards JSONB,
    
    -- AI-generated content
    executive_summary TEXT NOT NULL,
    
    -- Full data dump
    detailed_statistics JSONB,
    
    -- Metadata
    generated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    generated_by UUID REFERENCES users(id)
);

CREATE INDEX idx_weekly_reports_week ON weekly_reports(week_start, week_end);
CREATE INDEX idx_weekly_reports_generated_at ON weekly_reports(generated_at);
```

#### 4.6.4 Report Access

- **Public Dashboard**: Latest report visible to all users
- **Historical Reports**: Accessible via `/reports` endpoint
- **Export Options**: PDF, JSON, CSV formats
- **Visualization**: Charts and graphs using Chart.js/D3.js

---

## 5. Complete Database Design

### 5.1 Schema Overview

The system uses Appwrite Database (NoSQL document database) with the following collections:

1. `users` - User authentication and profiles (managed by Appwrite Auth)
2. `issues` - Core issue tracking
3. `resolution_history` - Resolution audit trail
4. `weekly_reports` - Generated reports
5. `audit_logs` - System activity logs (optional)

### 5.2 Complete Appwrite Collections Schema

```javascript
// Collection: users
// Note: Authentication is handled by Appwrite Auth
// This collection stores additional user metadata
{
  "$id": "unique_user_id",
  "email": "string (unique, indexed)",
  "phone": "string (optional)",
  "full_name": "string",
  "role": "string (citizen|officer|admin)",
  "is_verified": "boolean",
  "department": "string (optional)",
  "ward_assigned": "string (optional)",
  "$createdAt": "datetime",
  "$updatedAt": "datetime"
}

// Collection: issues
{
  "$id": "unique_issue_id",
  "title": "string",
  "description": "string",
  "description_translated": "string (optional)",
  "category": "string",
  "category_confidence": "float",
  "status": "string",
  "latitude": "float",
  "longitude": "float",
  "address": "string",
  "ward": "string",
  "city": "string",
  "image_url": "string (Storage file ID)",
  "reported_by": "string (user ID)",
  "assigned_to": "string (user ID, optional)",
  "resolved_at": "datetime (optional)",
  "resolution_proof_url": "string (Storage file ID, optional)",
  "resolution_notes": "string (optional)",
  "reopen_count": "integer",
  "last_reopened_at": "datetime (optional)",
  "reopen_reason": "string (optional)",
  "original_language": "string",
  "ai_processed": "boolean",
  "$createdAt": "datetime",
  "$updatedAt": "datetime"
}

// Collection: resolution_history
{
  "$id": "unique_resolution_id",
  "issue_id": "string",
  "resolved_by": "string (user ID)",
  "resolved_at": "datetime",
  "proof_url": "string (Storage file ID)",
  "notes": "string",
  "reopened": "boolean",
  "reopen_reason": "string (optional)",
  "reopened_at": "datetime (optional)",
  "reopened_by": "string (user ID, optional)",
  "$createdAt": "datetime",
  "$updatedAt": "datetime"
}

// Collection: weekly_reports
{
  "$id": "unique_report_id",
  "week_start": "string (ISO date)",
  "week_end": "string (ISO date)",
  "total_issues": "integer",
  "resolved_issues": "integer",
  "open_issues": "integer",
  "reopened_issues": "integer",
  "avg_resolution_time_hours": "float",
  "growth_rate": "float",
  "top_category": "string",
  "top_ward": "string",
  "high_risk_wards": "string (JSON)",
  "executive_summary": "string",
  "detailed_statistics": "string (JSON)",
  "generated_by": "string (user ID, optional)",
  "$createdAt": "datetime",
  "$updatedAt": "datetime"
}

// Collection: audit_logs (optional)
{
  "$id": "unique_log_id",
  "user_id": "string (optional)",
  "action": "string",
  "resource_type": "string",
  "resource_id": "string",
  "details": "string (JSON)",
  "ip_address": "string",
  "user_agent": "string",
  "$createdAt": "datetime"
}

// Indexes to create in Appwrite Console:
// users: email (unique), role
// issues: status, category, ward, $createdAt, reported_by, assigned_to
// resolution_history: issue_id, resolved_at
// weekly_reports: week_start, $createdAt
// audit_logs: user_id, $createdAt
```

CREATE INDEX idx_issues_status ON issues(status);
CREATE INDEX idx_issues_category ON issues(category);
CREATE INDEX idx_issues_ward ON issues(ward);
CREATE INDEX idx_issues_created_at ON issues(created_at);
CREATE INDEX idx_issues_location ON issues(latitude, longitude);
CREATE INDEX idx_issues_reported_by ON issues(reported_by);
CREATE INDEX idx_issues_assigned_to ON issues(assigned_to);

CREATE INDEX idx_resolution_history_issue ON resolution_history(issue_id);
CREATE INDEX idx_resolution_history_resolved_at ON resolution_history(resolved_at);

CREATE INDEX idx_weekly_reports_week ON weekly_reports(week_start, week_end);
CREATE INDEX idx_weekly_reports_generated_at ON weekly_reports(generated_at);

CREATE INDEX idx_audit_logs_user ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at);
```

### 5.3 Database Optimization Strategies

#### 5.3.1 Indexing Strategy

- B-tree indexes on frequently queried columns (status, category, ward)
- Composite index on (latitude, longitude) for geospatial queries
- Partial indexes for active issues: `CREATE INDEX idx_active_issues ON issues(created_at) WHERE status IN ('OPEN', 'IN_PROGRESS')`

#### 5.3.2 Partitioning (Future)

- Partition `issues` table by created_at (monthly partitions)
- Partition `audit_logs` by created_at (weekly partitions)
- Improves query performance for time-range queries

#### 5.3.3 Connection Pooling

- Use PgBouncer or SQLAlchemy connection pool
- Pool size: 20-50 connections
- Max overflow: 10 connections
- Connection timeout: 30 seconds

---

## 6. API Design

### 6.1 API Architecture

- **Protocol**: REST over HTTPS
- **Format**: JSON
- **Authentication**: JWT Bearer tokens
- **Versioning**: URL-based (`/api/v1/...`)
- **Rate Limiting**: 100 requests/minute per user

### 6.2 Authentication Endpoints

#### POST /api/v1/auth/register

Register a new user.

**Request**:
```json
{
  "email": "citizen@example.com",
  "phone": "+919876543210",
  "password": "SecurePass123!",
  "full_name": "Rajesh Kumar",
  "role": "citizen"
}
```

**Response** (201 Created):
```json
{
  "user_id": "uuid",
  "email": "citizen@example.com",
  "full_name": "Rajesh Kumar",
  "role": "citizen",
  "is_verified": false,
  "created_at": "2026-02-15T10:30:00Z"
}
```

#### POST /api/v1/auth/login

Authenticate user and receive tokens.

**Request**:
```json
{
  "email": "citizen@example.com",
  "password": "SecurePass123!"
}
```

**Response** (200 OK):
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer",
  "expires_in": 900,
  "user": {
    "id": "uuid",
    "email": "citizen@example.com",
    "full_name": "Rajesh Kumar",
    "role": "citizen"
  }
}
```

#### POST /api/v1/auth/refresh

Refresh access token.

**Request**:
```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Response** (200 OK):
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "expires_in": 900
}
```

### 6.3 Issue Management Endpoints

#### POST /api/v1/issues

Create a new issue.

**Headers**:
```
Authorization: Bearer {access_token}
Content-Type: multipart/form-data
```

**Request** (multipart/form-data):
```
title: "Large pothole on MG Road"
description: "बड़ा गड्ढा है जो खतरनाक है" (Hindi)
latitude: 12.9716
longitude: 77.5946
language: "hi"
image: [file upload]
```

**Response** (201 Created):
```json
{
  "issue_id": "uuid",
  "title": "Large pothole on MG Road",
  "description": "बड़ा गड्ढा है जो खतरनाक है",
  "description_translated": "There is a large pothole which is dangerous",
  "category": "pothole",
  "category_confidence": 0.92,
  "status": "OPEN",
  "latitude": 12.9716,
  "longitude": 77.5946,
  "ward": "Ward 23 - Koramangala",
  "image_url": "https://s3.amazonaws.com/...",
  "reported_by": "uuid",
  "created_at": "2026-02-15T10:30:00Z"
}
```

#### GET /api/v1/issues

List issues with filtering and pagination.

**Query Parameters**:

- `status`: Filter by status (open, resolved, etc.)
- `category`: Filter by category
- `ward`: Filter by ward
- `reported_by`: Filter by user ID
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20, max: 100)
- `sort`: Sort field (created_at, updated_at)
- `order`: Sort order (asc, desc)

**Request**:
```
GET /api/v1/issues?status=OPEN&ward=Ward%2023&page=1&limit=20
```

**Response** (200 OK):
```json
{
  "total": 156,
  "page": 1,
  "limit": 20,
  "pages": 8,
  "issues": [
    {
      "issue_id": "uuid",
      "title": "Large pothole on MG Road",
      "category": "pothole",
      "status": "OPEN",
      "ward": "Ward 23 - Koramangala",
      "created_at": "2026-02-15T10:30:00Z",
      "image_url": "https://s3.amazonaws.com/..."
    }
  ]
}
```

#### GET /api/v1/issues/{issue_id}

Get detailed information about a specific issue.

**Response** (200 OK):
```json
{
  "issue_id": "uuid",
  "title": "Large pothole on MG Road",
  "description": "बड़ा गड्ढा है जो खतरनाक है",
  "description_translated": "There is a large pothole which is dangerous",
  "category": "pothole",
  "status": "OPEN",
  "latitude": 12.9716,
  "longitude": 77.5946,
  "address": "MG Road, Koramangala, Bangalore",
  "ward": "Ward 23 - Koramangala",
  "image_url": "https://s3.amazonaws.com/...",
  "reported_by": {
    "id": "uuid",
    "name": "Rajesh Kumar",
    "is_verified": true
  },
  "assigned_to": null,
  "reopen_count": 0,
  "resolution_history": [],
  "created_at": "2026-02-15T10:30:00Z",
  "updated_at": "2026-02-15T10:30:00Z"
}
```

#### PATCH /api/v1/issues/{issue_id}/resolve

Mark an issue as resolved (Officer only).

**Headers**:
```
Authorization: Bearer {access_token}
Content-Type: multipart/form-data
```

**Request**:
```
resolution_notes: "Pothole filled with asphalt on 15th Feb"
proof_image: [file upload]
```

**Response** (200 OK):
```json
{
  "issue_id": "uuid",
  "status": "RESOLVED",
  "resolved_at": "2026-02-15T14:30:00Z",
  "resolved_by": {
    "id": "uuid",
    "name": "Officer Sharma",
    "department": "Public Works"
  },
  "resolution_proof_url": "https://s3.amazonaws.com/...",
  "resolution_notes": "Pothole filled with asphalt on 15th Feb"
}
```

#### POST /api/v1/issues/{issue_id}/reopen

Reopen a resolved issue (Citizen only).

**Request**:
```json
{
  "reopen_reason": "The pothole has reappeared after rain. The repair was not done properly."
}
```

**Response** (200 OK):
```json
{
  "issue_id": "uuid",
  "status": "REOPENED",
  "reopen_count": 1,
  "last_reopened_at": "2026-02-16T10:00:00Z",
  "reopen_reason": "The pothole has reappeared after rain. The repair was not done properly.",
  "can_reopen_again": true,
  "cooldown_expires_at": "2026-02-17T22:00:00Z"
}
```

**Error Response** (400 Bad Request):
```json
{
  "error": "reopen_limit_exceeded",
  "message": "Maximum reopen limit (2) reached for this issue"
}
```

### 6.4 Analytics & Statistics Endpoints

#### GET /api/v1/stats

Get overall system statistics.

**Query Parameters**:

- `period`: Time period (week, month, year, all)
- `ward`: Filter by ward (optional)

**Response** (200 OK):
```json
{
  "period": "week",
  "total_issues": 1247,
  "resolved_issues": 892,
  "open_issues": 355,
  "resolution_rate": 71.5,
  "avg_resolution_time_hours": 48.3,
  "growth_rate": 12.5,
  "category_breakdown": {
    "pothole": 456,
    "garbage": 312,
    "water": 189,
    "electrical": 145,
    "road": 98,
    "drainage": 47
  },
  "ward_breakdown": [
    {
      "ward": "Ward 23 - Koramangala",
      "total": 87,
      "resolved": 62,
      "risk_score": 67.5
    }
  ]
}
```

#### GET /api/v1/stats/risk-zones

Get high-risk zones based on risk scoring.

**Response** (200 OK):
```json
{
  "high_risk_zones": [
    {
      "ward": "Ward 23 - Koramangala",
      "risk_score": 87.5,
      "risk_level": "CRITICAL",
      "total_issues": 87,
      "open_issues": 34,
      "avg_days_unresolved": 12.3,
      "repeated_reports": 23,
      "growth_rate": 45.2
    },
    {
      "ward": "Ward 15 - Indiranagar",
      "risk_score": 76.2,
      "risk_level": "HIGH",
      "total_issues": 65,
      "open_issues": 28,
      "avg_days_unresolved": 9.8,
      "repeated_reports": 18,
      "growth_rate": 32.1
    }
  ]
}
```

### 6.5 Weekly Report Endpoints

#### POST /api/v1/reports/generate

Generate a new weekly report (Admin only).

**Request**:
```json
{
  "week_start": "2026-02-09",
  "week_end": "2026-02-15"
}
```

**Response** (201 Created):
```json
{
  "report_id": "uuid",
  "week_start": "2026-02-09",
  "week_end": "2026-02-15",
  "status": "generating",
  "message": "Report generation started. Check status at /api/v1/reports/{report_id}"
}
```

#### GET /api/v1/reports/latest

Get the most recent weekly report.

**Response** (200 OK):
```json
{
  "report_id": "uuid",
  "week_start": "2026-02-09",
  "week_end": "2026-02-15",
  "total_issues": 1247,
  "resolved_issues": 892,
  "open_issues": 355,
  "growth_rate": 12.5,
  "avg_resolution_time_hours": 48.3,
  "top_category": "pothole",
  "top_ward": "Ward 23 - Koramangala",
  "high_risk_wards": [
    {
      "ward": "Ward 23",
      "risk_score": 87.5
    }
  ],
  "executive_summary": "This week saw a 12.5% increase in reported civic issues...",
  "generated_at": "2026-02-16T02:00:00Z"
}
```

#### GET /api/v1/reports

List all historical reports.

**Query Parameters**:

- `page`: Page number
- `limit`: Items per page

**Response** (200 OK):
```json
{
  "total": 52,
  "page": 1,
  "limit": 10,
  "reports": [
    {
      "report_id": "uuid",
      "week_start": "2026-02-09",
      "week_end": "2026-02-15",
      "total_issues": 1247,
      "growth_rate": 12.5,
      "generated_at": "2026-02-16T02:00:00Z"
    }
  ]
}
```

#### GET /api/v1/reports/{report_id}

Get a specific report by ID.

**Response**: Same as `/api/v1/reports/latest`

### 6.6 Error Response Format

All API errors follow a consistent format:

```json
{
  "error": "error_code",
  "message": "Human-readable error message",
  "details": {
    "field": "Additional context"
  },
  "timestamp": "2026-02-15T10:30:00Z"
}
```

**Common Error Codes**:

- `400`: Bad Request - Invalid input
- `401`: Unauthorized - Missing or invalid token
- `403`: Forbidden - Insufficient permissions
- `404`: Not Found - Resource doesn't exist
- `409`: Conflict - Resource state conflict
- `422`: Unprocessable Entity - Validation failed
- `429`: Too Many Requests - Rate limit exceeded
- `500`: Internal Server Error - Server-side error

---

## 7. Security & Non-Functional Requirements

### 7.1 Authentication & Authorization

#### 7.1.1 Password Security

- **Hashing Algorithm**: bcrypt with cost factor 12
- **Password Requirements**:
  - Minimum 8 characters
  - At least one uppercase letter
  - At least one lowercase letter
  - At least one number
  - At least one special character
- **Password Reset**: Email-based with time-limited tokens (1 hour expiry)

#### 7.1.2 JWT Token Security

- **Algorithm**: HS256 (HMAC with SHA-256)
- **Access Token Expiry**: 15 minutes
- **Refresh Token Expiry**: 7 days
- **Token Storage**: HTTP-only cookies (frontend) + secure storage (mobile)
- **Token Rotation**: Refresh tokens rotated on each use

#### 7.1.3 Role-Based Access Control (RBAC)

**Permission Matrix**:

| Action | Citizen | Officer | Admin |
|--------|---------|---------|-------|
| Report Issue | ✓ | ✓ | ✓ |
| View Own Issues | ✓ | ✓ | ✓ |
| View All Issues | ✗ | ✓ | ✓ |
| Resolve Issue | ✗ | ✓ | ✓ |
| Reopen Issue | ✓ (own) | ✗ | ✓ |
| View Analytics | Limited | ✓ | ✓ |
| Generate Reports | ✗ | ✗ | ✓ |
| Manage Users | ✗ | ✗ | ✓ |

### 7.2 Input Validation

#### 7.2.1 Backend Validation (Pydantic)

```python
from pydantic import BaseModel, Field, validator

class IssueCreate(BaseModel):
    title: str = Field(..., min_length=10, max_length=200)
    description: str = Field(..., min_length=20, max_length=2000)
    latitude: float = Field(..., ge=-90, le=90)
    longitude: float = Field(..., ge=-180, le=180)
    language: str = Field(..., regex="^[a-z]{2}$")
    
    @validator('title', 'description')
    def sanitize_text(cls, v):
        # Remove potentially harmful characters
        return v.strip()
```

#### 7.2.2 File Upload Validation

- **Allowed Types**: JPEG, PNG only
- **Max Size**: 5MB per image
- **Virus Scanning**: ClamAV integration (optional)
- **Image Processing**: Resize to max 1920x1080, strip EXIF data

### 7.3 Rate Limiting

**Limits by Endpoint**:

- Authentication: 5 requests/minute
- Issue Creation: 10 requests/hour per user
- Issue Listing: 100 requests/minute
- Analytics: 50 requests/minute
- Report Generation: 1 request/hour (admin only)

**Implementation**: Redis-based token bucket algorithm

### 7.4 Data Privacy & Compliance

#### 7.4.1 Personal Data Protection

- **PII Encryption**: Sensitive fields encrypted at rest
- **Data Minimization**: Collect only necessary information
- **Right to Deletion**: Users can request account deletion
- **Data Retention**: Issues retained for 5 years, then archived

#### 7.4.2 Audit Logging

All sensitive operations logged:

- User authentication attempts
- Issue status changes
- Resolution actions
- Report generation
- Admin actions

### 7.5 Scalability Considerations

#### 7.5.1 Horizontal Scaling

- **Backend**: Stateless FastAPI instances behind load balancer
- **Database**: Read replicas for analytics queries
- **Storage**: S3 auto-scales
- **Caching**: Redis for session management and rate limiting

#### 7.5.2 Performance Targets

- **API Response Time**: < 200ms (p95)
- **Page Load Time**: < 2 seconds
- **Image Upload**: < 5 seconds
- **Report Generation**: < 30 seconds
- **Concurrent Users**: 10,000+

#### 7.5.3 Caching Strategy

- **CDN**: CloudFront for static assets
- **API Caching**: Redis for frequently accessed data
  - Issue listings: 5 minute TTL
  - Statistics: 15 minute TTL
  - Weekly reports: 1 hour TTL
- **Database Query Caching**: PostgreSQL query result cache

### 7.6 Fault Tolerance

#### 7.6.1 High Availability

- **Multi-AZ Deployment**: RDS and EC2 across availability zones
- **Auto-Scaling**: EC2 Auto Scaling Groups
- **Health Checks**: ALB health checks every 30 seconds
- **Failover**: Automatic RDS failover < 2 minutes

#### 7.6.2 Backup & Recovery

- **Database Backups**: Automated daily backups, 7-day retention
- **Point-in-Time Recovery**: 5-minute granularity
- **S3 Versioning**: Enabled for all buckets
- **Disaster Recovery**: Cross-region replication (optional)

#### 7.6.3 Error Handling

- **Graceful Degradation**: System remains functional if AI service fails
- **Retry Logic**: Exponential backoff for transient failures
- **Circuit Breaker**: Prevent cascading failures
- **Fallback Mechanisms**: Template-based responses when AI unavailable

### 7.7 Monitoring & Logging

#### 7.7.1 CloudWatch Metrics

- **Application Metrics**:
  - Request count and latency
  - Error rates by endpoint
  - AI processing success rate
  - Database connection pool usage

- **Infrastructure Metrics**:
  - CPU and memory utilization
  - Network throughput
  - Disk I/O
  - RDS performance insights

#### 7.7.2 Alerting

- **Critical Alerts** (PagerDuty/SNS):
  - API error rate > 5%
  - Database connection failures
  - S3 upload failures > 10%
  - AI service unavailable > 5 minutes

- **Warning Alerts** (Email):
  - High response times (> 500ms)
  - Disk usage > 80%
  - Unusual traffic patterns

#### 7.7.3 Structured Logging

```python
import structlog

logger = structlog.get_logger()

logger.info(
    "issue_created",
    issue_id=issue.id,
    user_id=user.id,
    category=issue.category,
    ward=issue.ward,
    processing_time_ms=elapsed_ms
)
```

---

## 8. Deployment Design

### 8.1 Infrastructure as Code

#### 8.1.1 AWS CloudFormation / Terraform

Define infrastructure as code for reproducible deployments:

```hcl
# Example Terraform structure
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_rds_instance" "postgres" {
  engine               = "postgres"
  engine_version       = "15.4"
  instance_class       = "db.t3.medium"
  allocated_storage    = 100
  multi_az             = true
  backup_retention_period = 7
}

resource "aws_s3_bucket" "issues" {
  bucket = "fixit-hub-issues"
  versioning {
    enabled = true
  }
}
```

### 8.2 Containerization

#### 8.2.1 Docker Configuration

```dockerfile
# Backend Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

#### 8.2.2 Docker Compose (Development)

```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/citizen_reporter
      - AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
      - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=citizen_reporter
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### 8.3 AWS Deployment Strategy

#### 8.3.1 Option A: EC2 with Auto Scaling

```
┌─────────────────────────────────────────────────────────┐
│                    Route 53 (DNS)                       │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│              CloudFront (CDN)                           │
│              - Frontend (S3)                            │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│        Application Load Balancer (ALB)                  │
└────────┬───────────────────────────────────────┬────────┘
         │                                       │
┌────────▼────────┐                    ┌────────▼────────┐
│  EC2 Instance   │                    │  EC2 Instance   │
│  (FastAPI)      │                    │  (FastAPI)      │
│  AZ-1           │                    │  AZ-2           │
└─────────────────┘                    └─────────────────┘
         │                                       │
         └───────────────┬───────────────────────┘
                         │
                ┌────────▼────────┐
                │   RDS Primary   │
                │   (Multi-AZ)    │
                └────────┬────────┘
                         │
                ┌────────▼────────┐
                │  RDS Replica    │
                │  (Read-only)    │
                └─────────────────┘
```

**Components**:

- **Route 53**: DNS management
- **CloudFront**: CDN for frontend
- **S3**: Static website hosting
- **ALB**: Load balancing and SSL termination
- **EC2 Auto Scaling Group**: 2-10 instances
- **RDS Multi-AZ**: High availability database
- **ElastiCache Redis**: Session and cache management

#### 8.3.2 Option B: AWS App Runner (Simplified)

```
┌─────────────────────────────────────────────────────────┐
│              AWS App Runner Service                     │
│              - Auto-scaling                             │
│              - Built-in load balancing                  │
│              - Automatic deployments                    │
└────────────────────┬────────────────────────────────────┘
                     │
         ┌───────────┴───────────┐
         │                       │
┌────────▼────────┐    ┌────────▼────────┐
│   RDS Primary   │    │   S3 Buckets    │
│   (Multi-AZ)    │    │   - Images      │
└─────────────────┘    └─────────────────┘
```

**Advantages**:

- Simplified deployment
- Automatic scaling
- Lower operational overhead
- Pay-per-use pricing

### 8.4 CI/CD Pipeline

#### 8.4.1 GitHub Actions Workflow

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: |
          pip install -r requirements.txt
          pytest tests/

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker image
        run: docker build -t fixit-hub:${{ github.sha }} .
      - name: Push to ECR
        run: |
          aws ecr get-login-password | docker login --username AWS --password-stdin
          docker push fixit-hub:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to EC2
        run: |
          aws ecs update-service --cluster prod --service backend --force-new-deployment
```

#### 8.4.2 Deployment Stages

1. **Development**: Auto-deploy on push to `dev` branch
2. **Staging**: Auto-deploy on push to `staging` branch
3. **Production**: Manual approval + deploy on push to `main`

### 8.5 Environment Configuration

#### 8.5.1 Environment Variables

```bash
# Database
DATABASE_URL=postgresql://user:pass@host:5432/db
DATABASE_POOL_SIZE=20

# AWS
AWS_REGION=ap-south-1
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...
S3_BUCKET_ISSUES=fixit-hub-issues
S3_BUCKET_RESOLUTIONS=fixit-hub-resolutions

# AI Service
AI_SERVICE_PROVIDER=bedrock  # or openai
BEDROCK_MODEL_ID=anthropic.claude-v2
OPENAI_API_KEY=sk-...

# Security
JWT_SECRET_KEY=...
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=15
REFRESH_TOKEN_EXPIRE_DAYS=7

# Application
ENVIRONMENT=production
LOG_LEVEL=INFO
CORS_ORIGINS=https://fixithub.gov.in
```

### 8.6 Cost Estimation (Monthly)

**AWS Services** (Estimated for 100,000 monthly active users):

| Service | Configuration | Monthly Cost (USD) |
|---------|--------------|-------------------|
| EC2 (t3.medium × 4) | 4 instances, 24/7 | $120 |
| RDS PostgreSQL (db.t3.medium) | Multi-AZ, 100GB | $150 |
| S3 Storage | 500GB storage, 1TB transfer | $30 |
| CloudFront | 1TB data transfer | $85 |
| Application Load Balancer | 24/7 operation | $25 |
| ElastiCache Redis (cache.t3.micro) | Single node | $15 |
| CloudWatch | Logs and metrics | $20 |
| Amazon Bedrock | 1M tokens/month | $50 |
| **Total** | | **~$495/month** |

**Scaling Considerations**:

- 1M users: ~$2,000/month
- 10M users: ~$8,000/month

---

## 9. Future Enhancements

### 9.1 IoT Integration

#### 9.1.1 Smart Sensors

- **Pothole Detection**: Accelerometer-based sensors in municipal vehicles
- **Garbage Overflow**: Ultrasonic sensors in bins
- **Water Leakage**: Pressure sensors in pipelines
- **Streetlight Monitoring**: IoT-enabled streetlights

#### 9.1.2 Architecture Extension

```
IoT Devices → AWS IoT Core → Lambda → API → Database
```

**Benefits**:

- Proactive issue detection
- Reduced citizen reporting burden
- Real-time infrastructure monitoring

### 9.2 Predictive ML Model Training

#### 9.2.1 Predictive Models

- **Issue Recurrence Prediction**: Predict which resolved issues likely to recur
- **Resolution Time Estimation**: Estimate time to resolve based on category/location
- **Resource Allocation**: Optimize officer assignment based on workload
- **Seasonal Patterns**: Identify seasonal trends (monsoon-related issues)

#### 9.2.2 ML Pipeline

```
Historical Data → Feature Engineering → Model Training (SageMaker) → 
Model Deployment → Inference API → Dashboard
```

### 9.3 Geo-Spatial Heatmaps

#### 9.3.1 Visualization Features

- **Issue Density Heatmap**: Color-coded map showing issue concentration
- **Category-Specific Maps**: Separate heatmaps for each issue type
- **Temporal Animation**: Time-lapse showing issue evolution
- **Cluster Analysis**: Identify hotspots using DBSCAN algorithm

#### 9.3.2 Technology Stack

- **Frontend**: Leaflet.js with heatmap plugin
- **Backend**: PostGIS extension for spatial queries
- **Data Processing**: GeoPandas for spatial analysis

```sql
-- PostGIS spatial query example
SELECT 
    ward,
    ST_AsGeoJSON(ST_Centroid(ST_Collect(ST_MakePoint(longitude, latitude)))) as center,
    COUNT(*) as issue_count
FROM issues
WHERE created_at >= NOW() - INTERVAL '30 days'
GROUP BY ward;
```

### 9.4 SMS Integration for Rural Users

#### 9.4.1 SMS-Based Reporting

- **Toll-Free Number**: Citizens send SMS to report issues
- **Format**: `REPORT <category> <location> <description>`
- **Example**: `REPORT POTHOLE MG_ROAD Large pothole near bus stop`

#### 9.4.2 SMS Gateway Integration

- **Service**: AWS SNS or Twilio
- **Flow**: SMS → Gateway → Lambda → API → Database
- **Response**: Confirmation SMS with issue ID

#### 9.4.3 Voice Call Support

- **IVR System**: Interactive Voice Response for illiterate users
- **Language Support**: Regional language voice prompts
- **Recording**: Voice description converted to text via AWS Transcribe

### 9.5 Mobile Application Expansion

#### 9.5.1 Native Mobile Apps

- **Platforms**: iOS (Swift) and Android (Kotlin)
- **Features**:
  - Offline issue drafting
  - Push notifications for status updates
  - Camera integration for instant photo capture
  - GPS auto-detection
  - Voice input for descriptions

#### 9.5.2 Progressive Web App (PWA)

- **Advantages**: Single codebase, works offline, installable
- **Service Workers**: Cache API responses for offline access
- **Background Sync**: Queue issues when offline, sync when online

### 9.6 Gamification & Citizen Engagement

#### 9.6.1 Citizen Reputation System

- **Points**: Earn points for verified reports
- **Badges**: "Top Reporter", "Quick Responder", "Community Hero"
- **Leaderboards**: Ward-wise and city-wise rankings
- **Rewards**: Recognition certificates, priority support

#### 9.6.2 Officer Performance Metrics

- **KPIs**: Resolution time, citizen satisfaction, reopen rate
- **Public Dashboard**: Transparent officer performance
- **Incentives**: Performance-based recognition

### 9.7 Integration with Existing Government Systems

#### 9.7.1 API Integrations

- **Municipal Corporation Systems**: Sync with existing work order systems
- **311 Services**: Integration with national helpline
- **Smart City Platforms**: Data sharing with smart city initiatives
- **GIS Systems**: Integration with government GIS databases

#### 9.7.2 Data Export

- **Formats**: CSV, JSON, XML, GeoJSON
- **Scheduled Exports**: Daily/weekly data dumps
- **API Access**: RESTful API for third-party integrations

### 9.8 Advanced Analytics

#### 9.8.1 Sentiment Analysis

- Analyze citizen sentiment from issue descriptions
- Track satisfaction trends over time
- Identify areas of frustration

#### 9.8.2 Root Cause Analysis

- Identify underlying infrastructure problems
- Correlate multiple issue types
- Recommend preventive maintenance

#### 9.8.3 Budget Optimization

- Estimate costs for issue resolution
- Prioritize based on cost-benefit analysis
- Track spending vs. outcomes

---

## 10. Technical Stack Summary

### 10.1 Frontend

- **Framework**: React 18.2.0
- **Build Tool**: Vite 5.0.11
- **UI Library**: Material-UI 5.15 or Tailwind CSS 3.4
- **State Management**: Redux Toolkit 2.0 or Zustand 4.4
- **Maps**: Leaflet.js 1.9.4
- **Charts**: Chart.js 4.4 or Recharts 2.10
- **HTTP Client**: Axios 1.6.5
- **PWA**: Workbox 7.0.0
- **Offline Storage**: idb 8.0.0 (IndexedDB wrapper)

### 10.2 Backend

- **Framework**: FastAPI 0.109.0 (Python 3.11.7)
- **ASGI Server**: Uvicorn 0.27.0
- **ORM**: SQLAlchemy 2.0.25
- **Spatial ORM**: GeoAlchemy2 0.14.3
- **Validation**: Pydantic 2.5.3
- **Authentication**: python-jose 3.3.0 (JWT)
- **Password Hashing**: passlib 1.7.4 (bcrypt)
- **Async HTTP**: httpx 0.26.0
- **Image Processing**: Pillow 10.2.0
- **Task Queue**: Celery 5.3.4
- **Migrations**: Alembic 1.13.1
- **Testing**: pytest 7.4, pytest-asyncio 0.23
- **Monitoring**: prometheus-client 0.19.0

### 10.3 Database & Storage

- **Primary Database**: PostgreSQL 15.5
- **Spatial Extension**: PostGIS 3.4.1
- **Text Search**: pg_trgm (built-in)
- **NoSQL Database**: Appwrite Database 1.5
- **Object Storage**: Appwrite Storage
- **Cache**: Redis 7.2.4
- **Connection Pooling**: psycopg2-binary 2.9.9

### 10.4 AI & ML

- **Primary**: Amazon Bedrock (Claude/Titan)
- **Alternative**: OpenAI API, Anthropic API
- **Translation**: AWS Translate (fallback)
- **ML Training**: AWS SageMaker (future)

### 10.5 Infrastructure

- **Cloud Provider**: AWS
- **Compute**: EC2 or App Runner
- **Load Balancer**: Application Load Balancer
- **CDN**: CloudFront
- **DNS**: Route 53
- **Monitoring**: CloudWatch
- **Secrets**: AWS Secrets Manager
- **IaC**: Terraform or CloudFormation

### 10.6 DevOps

- **Version Control**: Git (GitHub/GitLab)
- **CI/CD**: GitHub Actions or GitLab CI
- **Containerization**: Docker
- **Container Registry**: AWS ECR
- **Orchestration**: ECS or Kubernetes (optional)

---

## 11. Development Roadmap

### Phase 1: MVP (Months 1-2)

- User authentication and registration
- Basic issue reporting with image upload
- Map-based location tagging
- Issue listing and filtering
- Officer resolution workflow
- Basic analytics dashboard

### Phase 2: AI Integration (Month 3)

- Multilingual translation
- Automatic classification
- AI-powered weekly reports
- Risk scoring algorithm

### Phase 3: Advanced Features (Month 4)

- Reopen control mechanism
- Resolution history tracking
- Advanced analytics and visualizations
- Performance optimization
- Security hardening

### Phase 4: Scale & Polish (Month 5-6)

- Load testing and optimization
- Mobile-responsive improvements
- Comprehensive documentation
- User training materials
- Production deployment
- Monitoring and alerting setup

### Phase 5: Future Enhancements (Month 7+)

- IoT integration
- Predictive ML models
- Mobile applications
- SMS integration
- Gamification features

---

## 12. Success Metrics

### 12.1 User Engagement

- **Target**: 100,000 registered users in first year
- **Active Users**: 30% monthly active user rate
- **Issue Reports**: 50,000+ issues reported monthly
- **Resolution Rate**: 75%+ issues resolved within 7 days

### 12.2 System Performance

- **Uptime**: 99.9% availability
- **Response Time**: < 200ms API response (p95)
- **Error Rate**: < 0.5% of requests
- **AI Processing**: 95%+ success rate

### 12.3 Governance Impact

- **Response Time**: 50% reduction in average resolution time
- **Citizen Satisfaction**: 80%+ satisfaction rate
- **Transparency**: 100% of resolutions with proof
- **Predictive Accuracy**: 70%+ accuracy in risk zone identification

### 12.4 Cost Efficiency

- **Infrastructure Cost**: < $0.05 per active user per month
- **AI Processing Cost**: < $0.01 per issue
- **Total Cost of Ownership**: 60% lower than traditional systems

---

## 13. Risk Mitigation

### 13.1 Technical Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| AI service downtime | High | Fallback to manual classification, queue for retry |
| Database failure | Critical | Multi-AZ deployment, automated backups, read replicas |
| S3 outage | Medium | Retry logic, temporary local storage |
| High traffic spikes | Medium | Auto-scaling, CDN caching, rate limiting |
| Security breach | Critical | Regular audits, penetration testing, encryption |

### 13.2 Operational Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Spam/fake reports | Medium | Captcha, rate limiting, user verification |
| Officer non-compliance | High | Performance tracking, escalation workflow |
| Data privacy violations | Critical | GDPR-like compliance, regular audits |
| Budget overruns | Medium | Cost monitoring, auto-scaling limits |
| User adoption challenges | High | User training, multilingual support, simple UX |

### 13.3 Business Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Government policy changes | High | Flexible architecture, modular design |
| Competing solutions | Medium | Continuous innovation, user feedback |
| Funding constraints | High | Phased rollout, cost optimization |
| Stakeholder resistance | Medium | Pilot programs, success metrics, transparency |

---

## 14. Compliance & Standards

### 14.1 Data Protection

- **Indian IT Act 2000**: Compliance with data protection provisions
- **Digital Personal Data Protection Act 2023**: User consent, data minimization
- **ISO 27001**: Information security management

### 14.2 Accessibility

- **WCAG 2.1 Level AA**: Web accessibility standards
- **Screen Reader Support**: ARIA labels, semantic HTML
- **Keyboard Navigation**: Full keyboard accessibility
- **Color Contrast**: Minimum 4.5:1 ratio

### 14.3 API Standards

- **REST**: RESTful API design principles
- **OpenAPI 3.0**: API documentation standard
- **JSON Schema**: Request/response validation
- **OAuth 2.0**: Industry-standard authorization (future)

---

## 15. Conclusion

The FixIt Hub platform represents a comprehensive, AI-powered solution for modernizing civic infrastructure management in India. By combining multilingual support, intelligent automation, predictive analytics, and transparent governance, the system addresses critical gaps in traditional complaint management systems.

### Key Innovations

1. **AI-Powered Intelligence**: Multilingual processing and automated classification reduce manual effort and enable inclusive participation across India's linguistic diversity.

2. **Predictive Risk Scoring**: Statistical modeling identifies emerging infrastructure stress zones before they become critical, enabling proactive governance.

3. **Controlled Transparency**: The reopen mechanism balances citizen accountability with spam prevention, ensuring genuine issues receive attention while maintaining system integrity.

4. **Executive-Ready Insights**: AI-generated weekly reports transform raw data into actionable intelligence for decision-makers.

5. **Cloud-Native Scalability**: AWS-based architecture ensures the platform can scale from pilot cities to nationwide deployment.

### Implementation Readiness

The design provides:

- **Complete technical specifications** for immediate development
- **Scalable architecture** supporting 10M+ users
- **Security-first approach** with comprehensive protection measures
- **Cost-effective deployment** with clear budget projections
- **Future-proof extensibility** with well-defined enhancement roadmap

### Expected Impact

- **50% reduction** in average issue resolution time
- **75%+ resolution rate** within 7 days
- **100% transparency** with proof-based resolution tracking
- **Inclusive participation** across 22+ Indian languages
- **Data-driven governance** with predictive analytics

This platform has the potential to transform civic engagement in India, creating a more responsive, transparent, and efficient infrastructure management ecosystem that serves all citizens regardless of language, location, or technical literacy.

---

## Appendix A: Glossary

- **AI**: Artificial Intelligence
- **API**: Application Programming Interface
- **AWS**: Amazon Web Services
- **CDN**: Content Delivery Network
- **CORS**: Cross-Origin Resource Sharing
- **CRUD**: Create, Read, Update, Delete
- **EC2**: Elastic Compute Cloud
- **JWT**: JSON Web Token
- **ML**: Machine Learning
- **ORM**: Object-Relational Mapping
- **RBAC**: Role-Based Access Control
- **RDS**: Relational Database Service
- **REST**: Representational State Transfer
- **S3**: Simple Storage Service
- **SQL**: Structured Query Language
- **SSL/TLS**: Secure Sockets Layer / Transport Layer Security
- **UUID**: Universally Unique Identifier

## Appendix B: API Quick Reference

### Authentication

- `POST /api/v1/auth/register` - Register new user
- `POST /api/v1/auth/login` - Login and get tokens
- `POST /api/v1/auth/refresh` - Refresh access token

### Issues

- `POST /api/v1/issues` - Create new issue
- `GET /api/v1/issues` - List issues (with filters)
- `GET /api/v1/issues/{id}` - Get issue details
- `PATCH /api/v1/issues/{id}/resolve` - Resolve issue (officer)
- `POST /api/v1/issues/{id}/reopen` - Reopen issue (citizen)

### Analytics

- `GET /api/v1/stats` - Get system statistics
- `GET /api/v1/stats/risk-zones` - Get high-risk zones

### Reports

- `POST /api/v1/reports/generate` - Generate weekly report (admin)
- `GET /api/v1/reports/latest` - Get latest report
- `GET /api/v1/reports` - List all reports
- `GET /api/v1/reports/{id}` - Get specific report

---

## Appendix C: Database ER Diagram

```
┌─────────────────┐
│     users       │
├─────────────────┤
│ id (PK)         │
│ email           │
│ password_hash   │
│ full_name       │
│ role            │
│ is_verified     │
│ department      │
│ ward_assigned   │
└────────┬────────┘
         │
         │ 1:N
         │
┌────────▼────────────────────┐
│        issues               │
├─────────────────────────────┤
│ id (PK)                     │
│ title                       │
│ description                 │
│ category                    │
│ status                      │
│ latitude                    │
│ longitude                   │
│ ward                        │
│ reported_by (FK → users)    │
│ assigned_to (FK → users)    │
│ reopen_count                │
│ last_reopened_at            │
└────────┬────────────────────┘
         │
         │ 1:N
         │
┌────────▼────────────────────┐
│   resolution_history        │
├─────────────────────────────┤
│ id (PK)                     │
│ issue_id (FK → issues)      │
│ resolved_by (FK → users)    │
│ resolved_at                 │
│ proof_url                   │
│ notes                       │
│ reopened                    │
│ reopen_reason               │
└─────────────────────────────┘

┌─────────────────────────────┐
│     weekly_reports          │
├─────────────────────────────┤
│ id (PK)                     │
│ week_start                  │
│ week_end                    │
│ total_issues                │
│ resolved_issues             │
│ executive_summary           │
│ detailed_statistics (JSON)  │
│ generated_by (FK → users)   │
└─────────────────────────────┘
```

---

## Appendix D: Sample AI Prompts

### Translation Prompt

```
You are a professional translator specializing in Indian languages.

Task: Translate the following civic complaint from {source_language} to English.

Requirements:
- Preserve technical terms (pothole, drainage, etc.)
- Keep location names unchanged
- Maintain the urgency and tone
- Output only the translated text

Original text:
{description}

Translated text:
```

### Classification Prompt

```
You are a civic infrastructure expert.

Task: Classify the following issue into one category.

Categories:
- pothole: Road surface damage, holes in roads
- garbage: Waste management, overflowing bins
- water: Water supply issues, leakage, contamination
- electrical: Streetlight problems, power issues
- road: Road damage, cracks, surface issues
- drainage: Sewage, blocked drains, flooding
- other: Issues not fitting above categories

Issue description:
{description}

Respond in JSON format:
{
  "category": "category_name",
  "confidence": 0.0-1.0,
  "reasoning": "brief explanation"
}
```

### Executive Summary Prompt

```
You are a senior civic analyst preparing a weekly report for government officials.

Task: Generate a concise executive summary (200-300 words) based on the following data.

Focus on:
1. Key trends and changes from previous week
2. High-risk areas requiring immediate attention
3. Performance metrics (resolution rate, response time)
4. Actionable recommendations

Data:
{json_statistics}

Generate a professional, action-oriented summary suitable for senior officials.
```

---

## Document Information

**Version**: 1.0
**Last Updated**: February 15, 2026
**Author**: Technical Architecture Team
**Status**: Final Design Document
**Target Audience**: Development Team, Stakeholders, Hackathon Judges

**Document History**:

- v1.0 (Feb 15, 2026): Initial comprehensive design document

---

**End of Design Document**
