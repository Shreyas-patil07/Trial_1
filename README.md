# 🌍 FixIt Hub  
## AI-Powered Civic Intelligence Platform

FixIt Hub is a web-based civic intelligence platform that enables citizens to report public infrastructure issues and helps government authorities respond with transparency, accountability, and data-driven prioritization.

**FixIt Hub uses AI-powered classification and Appwrite Cloud infrastructure to enable scalable civic issue reporting and intelligent governance analytics.**

The system transforms scattered civic complaints into structured analytics and predictive risk indicators using AI.

---

## 🎯 Problem

Civic issue reporting in India is:

- Fragmented across platforms  
- Reactive instead of proactive  
- Limited by language barriers  
- Lacking structured analytics  
- Weak in accountability tracking  

Municipalities often respond only after issues escalate, without early warning systems or risk-based prioritization.

---

## 💡 Solution

FixIt Hub centralizes civic issue reporting and enhances it with AI-driven intelligence.

The platform:

- Accepts multilingual issue submissions  
- Automatically translates and classifies issues  
- Tracks resolution with proof-based verification  
- Allows controlled citizen reopen (max 2 times, 36-hour cooldown)  
- Calculates ward-level predictive risk scores  
- Generates AI-powered weekly executive summaries  

This enables proactive governance instead of reactive complaint handling.

---

## 🚀 MVP Features (Web-Only)

### 👤 User Roles
- **Public/Citizen** - Report issues, track status, reopen resolved issues
- **Officer** - Resolve issues, manage territory, customize reports by location and issue type  

### 📝 Issue Reporting
- Title and description  
- Image upload  
- GPS-based location tagging  
- Multilingual submission support  
- AI translation and classification
- Territory-aware automatic routing

### 🗺️ Territory-Based Issue Management

FixIt Hub solves the critical problem of manual complaint routing through intelligent geographic jurisdiction mapping.

**The Problem:**
- Officials waste time filtering irrelevant complaints
- Manual forwarding between departments causes delays
- Duplicate complaints from the same area
- No accountability for specific territories

**Our Solution:**
Every issue is automatically:
- Geotagged with precise GPS coordinates
- Mapped to administrative territories (ward/district/zone)
- Routed to the responsible authority
- Filtered so officials only see their jurisdiction

**Key Features:**

1. **Automatic Jurisdiction Detection**
   - Point-in-polygon spatial queries
   - Instant routing to correct authority
   - No manual forwarding required

2. **Smart Filtering for Citizens**
   - View nearby issues (within radius)
   - Sort by distance, severity, status
   - Prevent duplicate reporting

3. **Territory Dashboard for Officials**
   - See only issues in assigned jurisdiction
   - Filter by territory, department, priority
   - Geographic heatmaps and clusters
   - **Customize reports by location and issue type**
   - **Export filtered reports (PDF, CSV, Excel)**
   - **Generate custom analytics dashboards**

4. **Proximity Intelligence**
   - Identify issue clusters (e.g., 12 reports within 200m)
   - Solve multiple complaints in single operation
   - Data-driven resource allocation

**Benefits:**
- ⚡ Faster resolution (no manual routing)
- 🎯 Clear accountability per territory
- 📊 Data-driven governance insights
- 🚫 Reduced duplicate complaints
- 🔍 Transparent issue tracking by area  

### 🔄 Resolution Workflow
- Officer marks issue as resolved with proof photo  
- Resolution notes required  
- Transparent resolution history  
- Citizen reopen logic (max 2 attempts + cooldown)  

### 📊 Intelligence & Analytics
- Ward-level risk score calculation  
- Weekly statistics aggregation  
- AI-generated executive summary (200–300 words)
- **Officer-customizable reports**:
  - Filter by location (ward, district, zone)
  - Filter by issue type (pothole, water, electrical, etc.)
  - Custom date ranges
  - Export formats (PDF, CSV, Excel)
  - Scheduled report generation
- Dashboard displaying:
  - Total Issues  
  - Fixed Issues  
  - Pending Issues
  - Territory-specific metrics  

---

## 🧠 AI Capabilities

- Language detection and translation  
- Automatic issue categorization with confidence score  
- Risk hotspot identification  
- Executive summary generation for decision-makers  

AI integration is handled via Amazon Bedrock or external AI APIs.

---

## 🏗️ Technology Stack

### Frontend
- **React 18.2** - Modern, fast development experience with HMR
- **Vite 5.0** - Next-generation frontend tooling
- **Progressive Web App (PWA)** - Offline-first architecture with service workers
- **IndexedDB** - Local storage for offline issue reporting
- **Responsive Design** - Mobile-first approach with adaptive layouts

### Backend
- **FastAPI 0.109** - High-performance async API framework
- **Python 3.11** - Stable Python version with performance improvements
- **Celery 5.3** - Distributed task queue
- **Redis 7.2** - In-memory data store for caching and queuing
- **RESTful Architecture** - Clean, scalable API design with OpenAPI docs

### Infrastructure
- **Vercel** - Frontend hosting with edge CDN and automatic deployments
- **Render** - Backend hosting with auto-scaling and health checks
- **Appwrite Cloud 1.5** - BaaS for Authentication, Database, and Storage
- **Redis Cloud** - Managed Redis for task queue and caching layer

### Backend Services

FixIt Hub uses **Appwrite Cloud 1.5** for:

- **User Authentication** - Secure role-based access control with JWT
- **Database** - NoSQL document database with real-time sync
- **File Storage** - Issue images and proof photos with CDN delivery
- **Role-based Access Control** - Citizen, Officer, and Admin permissions

### Geospatial Architecture
- **PostgreSQL 15.5** - Stable production-ready database
- **PostGIS 3.4** - Advanced spatial queries and indexing
- **GiST Indexes** - Efficient point-in-polygon operations
- **Spatial Clustering** - DBSCAN algorithm for issue clustering
- **Territory Boundaries** - GeoJSON polygon storage for jurisdiction mapping

### Database
- **Appwrite Database** - Primary NoSQL document database for issues, users, and real-time data
- **PostgreSQL 15.5 + PostGIS 3.4** - Dedicated geospatial database for territory boundaries and spatial queries
- **Redis 7.2** - Caching layer (15-min TTL for analytics, 5-min for listings)
- **Data Distribution Strategy**:
  - Appwrite: User profiles, issue documents, resolution history
  - PostgreSQL: Territory polygons, spatial indexes, geospatial analytics
  - Redis: Session cache, task queue, computed analytics
- Real-time data synchronization and automatic backups

### Storage & CDN
- **Appwrite Storage** - Secure file storage with automatic encryption
- **Pre-signed Upload URLs** - Direct client-to-storage uploads
- **CDN Integration** - Global edge caching for fast image delivery
- **Image Optimization** - Automatic compression and format conversion

### AI Integration
- **Amazon Bedrock** - Claude 3 Sonnet / Titan models
- **OpenAI GPT-4** - Fallback AI provider
- **Asynchronous Processing** - Non-blocking AI pipeline via Celery workers
- **Predictive Analytics** - Risk scoring and hotspot identification
- **Executive Summaries** - Weekly AI-generated governance reports

### Monitoring & Observability
- **Prometheus 2.48** - Metrics collection and time-series database
- **Grafana 10.2** - Real-time dashboards and alerting
- **Render Logs** - Centralized logging with structured JSON
- **Key Metrics**:
  - API latency (p50, p95, p99)
  - Queue backlog depth
  - Error rates by endpoint
  - AI processing time
  - Database query performance

---

## 🔧 Why Appwrite?

Appwrite simplifies backend development by providing:

- **Built-in Authentication** - No need to build auth from scratch
- **Secure File Storage** - Automatic encryption and access control
- **Scalable Database** - Handles growing civic data efficiently
- **Role-based Permissions** - Fine-grained access control out of the box

Using Appwrite allows FixIt Hub to focus on civic intelligence features instead of infrastructure management.

---

## 🚀 Advanced Features

### 1. Duplicate Issue Detection

Prevents redundant reporting using multi-factor similarity:

- **Spatial Distance**: Issues within 100m radius
- **Time Window**: Reported within 24 hours
- **Category Match**: Same issue category
- **Perceptual Image Hashing**: pHash algorithm for visual similarity
- **Confidence Score**: Weighted combination of factors

```python
def detect_duplicate(new_issue):
    nearby = find_issues_within_radius(new_issue.location, 100)  # meters
    recent = filter_by_time_window(nearby, hours=24)
    same_category = filter_by_category(recent, new_issue.category)
    
    for existing in same_category:
        image_similarity = compare_phash(new_issue.image, existing.image)
        if image_similarity > 0.85:
            return existing, "High confidence duplicate"
    
    return None, "No duplicate found"
```

### 2. Risk-Based Issue Prioritization

Officers receive prioritized dashboards based on:

- **Issue Age**: Days since reporting (weight: 0.3)
- **Citizen Votes**: Community upvotes (weight: 0.25)
- **Category Severity**: Water > Electrical > Drainage (weight: 0.25)
- **Reopen Count**: Previous failed resolutions (weight: 0.2)

```python
priority_score = (
    0.30 * normalize(days_open, max=30) +
    0.25 * normalize(vote_count, max=100) +
    0.25 * severity_weights[category] +
    0.20 * normalize(reopen_count, max=3)
)
```

### 3. Fraud-Resistant Resolution Proof

Ensures authentic resolution verification:

- **GPS Metadata Validation**: Proof photo must be within 50m of issue location
- **Timestamp Verification**: Photo taken after issue assignment
- **Image Hash Storage**: SHA-256 hash prevents tampering
- **EXIF Data Extraction**: Camera metadata for authenticity
- **Blockchain Anchoring** (Future): Immutable proof records

```python
def validate_resolution_proof(proof_image, issue):
    exif = extract_exif(proof_image)
    
    # Validate GPS proximity
    if haversine_distance(exif.gps, issue.location) > 50:
        raise ValidationError("Proof location mismatch")
    
    # Validate timestamp
    if exif.timestamp < issue.assigned_at:
        raise ValidationError("Proof predates assignment")
    
    # Store hash for tamper detection
    proof_hash = hashlib.sha256(proof_image).hexdigest()
    store_proof_hash(issue.id, proof_hash)
```

### 4. Spam & Abuse Prevention

Multi-layered protection system:

- **Rate Limiting**: 10 issues per day per user
- **Phone Verification**: OTP-based verification for new accounts
- **Reporter Reputation Score**: Based on resolution success rate
- **Behavioral Analysis**: Flags suspicious patterns
- **Admin Review Queue**: Low-reputation submissions require approval

```python
def check_spam_indicators(user, issue):
    if user.reputation_score < 0.3:
        return "REQUIRES_REVIEW"
    
    if count_issues_today(user) >= 10:
        return "RATE_LIMIT_EXCEEDED"
    
    if detect_duplicate_content(issue):
        return "POTENTIAL_SPAM"
    
    return "APPROVED"
```

### 5. Public Transparency Dashboard

Real-time civic performance metrics:

- **Ward-Level Statistics**: Resolution rates, avg time, issue density
- **Category Breakdown**: Visual charts of issue distribution
- **Officer Performance**: Leaderboard (anonymized)
- **Historical Trends**: 30/60/90-day comparisons
- **Public API**: Open data for civic tech developers

Access at: `/public/dashboard`

### 6. Offline-First Mobile Experience

Progressive Web App capabilities:

- **Service Worker**: Caches app shell and static assets
- **IndexedDB Storage**: Stores draft issues locally
- **Background Sync**: Auto-uploads when connection restored
- **Optimistic UI**: Instant feedback, sync in background
- **Network Detection**: Graceful degradation on poor connectivity

```javascript
// Service Worker - Background Sync
self.addEventListener('sync', event => {
  if (event.tag === 'sync-issues') {
    event.waitUntil(syncPendingIssues());
  }
});

async function syncPendingIssues() {
  const db = await openDB('fixit-hub');
  const pending = await db.getAll('pending-issues');
  
  for (const issue of pending) {
    await fetch('/api/issues', {
      method: 'POST',
      body: JSON.stringify(issue)
    });
    await db.delete('pending-issues', issue.id);
  }
}
```

### 7. Smart Territory Routing

Automatic officer assignment with escalation:

- **Primary Assignment**: Based on territory boundaries
- **Workload Balancing**: Distributes to least-busy officer
- **Escalation Rules**: Auto-escalate if unresolved > 7 days
- **Department Routing**: Category-based department assignment
- **Holiday Coverage**: Automatic reassignment during leave

```python
def assign_officer(issue):
    territory = find_territory(issue.location)
    officers = get_available_officers(territory)
    
    # Load balancing
    officer = min(officers, key=lambda o: o.active_issue_count)
    
    # Set escalation timer
    schedule_escalation(issue.id, days=7, escalate_to=territory.supervisor)
    
    return officer
```

### 8. Officer Report Customization

Officers can generate custom reports tailored to their needs:

**Customization Options:**

1. **Location-Based Filtering**
   - Select specific wards, districts, or zones
   - Multi-territory selection for regional reports
   - Custom boundary drawing on map

2. **Issue Type Filtering**
   - Single or multiple categories
   - Severity level filtering
   - Status-based filtering (open, resolved, reopened)

3. **Time Range Selection**
   - Predefined ranges (today, week, month, quarter, year)
   - Custom date range picker
   - Comparison periods (current vs previous)

4. **Export Formats**
   - PDF with charts and maps
   - CSV for data analysis
   - Excel with multiple sheets
   - JSON for API integration

5. **Scheduled Reports**
   - Daily digest emails
   - Weekly performance summaries
   - Monthly trend analysis
   - Custom schedule (e.g., every Monday 9 AM)

**Implementation:**

```python
@router.post("/reports/custom")
async def generate_custom_report(
    officer_id: str,
    filters: ReportFilters
):
    """
    Generate customized report based on officer preferences
    
    filters: {
        "territories": ["WARD_K_EAST", "WARD_K_WEST"],
        "categories": ["pothole", "water"],
        "status": ["OPEN", "IN_PROGRESS"],
        "date_range": {
            "start": "2026-02-01",
            "end": "2026-02-28"
        },
        "format": "pdf",  # pdf, csv, excel, json
        "include_charts": true,
        "include_map": true
    }
    """
    # Validate officer permissions
    officer = get_officer(officer_id)
    validate_territory_access(officer, filters.territories)
    
    # Query issues with filters
    issues = db.issues.find({
        'ward': {'$in': filters.territories},
        'category': {'$in': filters.categories},
        'status': {'$in': filters.status},
        'created_at': {
            '$gte': filters.date_range.start,
            '$lte': filters.date_range.end
        }
    })
    
    # Generate statistics
    stats = calculate_statistics(issues)
    
    # Generate visualizations
    charts = generate_charts(stats) if filters.include_charts else None
    heatmap = generate_heatmap(issues) if filters.include_map else None
    
    # Export in requested format
    if filters.format == 'pdf':
        return generate_pdf_report(stats, charts, heatmap)
    elif filters.format == 'csv':
        return generate_csv_report(issues)
    elif filters.format == 'excel':
        return generate_excel_report(stats, issues)
    else:
        return {'statistics': stats, 'issues': issues}

@router.post("/reports/schedule")
async def schedule_custom_report(
    officer_id: str,
    schedule: ReportSchedule
):
    """
    Schedule recurring custom reports
    
    schedule: {
        "name": "Weekly Pothole Report",
        "filters": {...},
        "frequency": "weekly",  # daily, weekly, monthly
        "day_of_week": "monday",
        "time": "09:00",
        "timezone": "Asia/Kolkata",
        "delivery": {
            "email": true,
            "dashboard": true
        }
    }
    """
    # Create scheduled task
    celery_beat_schedule = {
        f"custom_report_{officer_id}_{schedule.name}": {
            'task': 'tasks.generate_and_send_report',
            'schedule': crontab(
                day_of_week=schedule.day_of_week,
                hour=9,
                minute=0
            ),
            'args': (officer_id, schedule.filters)
        }
    }
    
    # Save schedule to database
    db.report_schedules.create({
        'officer_id': officer_id,
        'name': schedule.name,
        'filters': schedule.filters,
        'frequency': schedule.frequency,
        'celery_schedule': celery_beat_schedule
    })
    
    return {'message': 'Report scheduled successfully'}
```

**Report Templates:**

Officers can save and reuse report configurations:

```python
# Save report template
@router.post("/reports/templates")
async def save_report_template(officer_id: str, template: ReportTemplate):
    db.report_templates.create({
        'officer_id': officer_id,
        'name': template.name,
        'filters': template.filters,
        'format': template.format,
        'created_at': datetime.now()
    })

# Use saved template
@router.get("/reports/templates/{template_id}/generate")
async def generate_from_template(template_id: str):
    template = db.report_templates.get(template_id)
    return await generate_custom_report(
        officer_id=template.officer_id,
        filters=template.filters
    )
```

---

## ⚙️ Setup & Installation

### Prerequisites
- Node.js 18.19 LTS and npm 10.x
- Python 3.11.7
- PostgreSQL 15.5 with PostGIS 3.4 extension
- Redis 7.2
- Appwrite Cloud account

### Appwrite Setup

1. Create an account on [Appwrite Cloud](https://cloud.appwrite.io)
2. Create a new project
3. Create a Database and note the Database ID
4. Create a Collection called `issues` and note the Collection ID
5. Create a Collection called `users` for user metadata
6. Create a Collection called `resolution_history` for audit trail
7. Create a Storage bucket called `issue-images` and note the Bucket ID
8. Create a Storage bucket called `resolution-proofs`
9. Copy your Project ID and API endpoint

### PostgreSQL + PostGIS Setup

```sql
-- Create territories table
CREATE TABLE territories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    territory_code VARCHAR(50) UNIQUE NOT NULL,
    territory_name VARCHAR(255) NOT NULL,
    territory_type VARCHAR(50) NOT NULL,
    boundary GEOMETRY(POLYGON, 4326) NOT NULL,
    assigned_officers UUID[],
    department VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Create spatial index for fast lookups
CREATE INDEX idx_territories_boundary ON territories USING GIST(boundary);

-- Create index on territory code
CREATE INDEX idx_territories_code ON territories(territory_code);

-- Example: Insert a sample territory
INSERT INTO territories (territory_code, territory_name, territory_type, boundary, department)
VALUES (
    'WARD_K_EAST',
    'Ward K/East - Andheri',
    'ward',
    ST_GeomFromGeoJSON('{"type":"Polygon","coordinates":[[[72.84,19.11],[72.85,19.11],[72.85,19.12],[72.84,19.12],[72.84,19.11]]]}'),
    'Public Works'
);
```

### Environment Variables

Create a `.env` file in the frontend directory:

```env
# Appwrite Configuration
VITE_APPWRITE_ENDPOINT=https://cloud.appwrite.io/v1
VITE_APPWRITE_PROJECT_ID=your_project_id
VITE_APPWRITE_DATABASE_ID=your_database_id
VITE_APPWRITE_ISSUES_COLLECTION_ID=your_collection_id
VITE_APPWRITE_BUCKET_ID=your_bucket_id
```

Create a `.env` file in the backend directory:

```env
# Appwrite Configuration
APPWRITE_ENDPOINT=https://cloud.appwrite.io/v1
APPWRITE_PROJECT_ID=your_project_id
APPWRITE_API_KEY=your_api_key

# PostgreSQL + PostGIS
DATABASE_URL=postgresql://user:password@localhost:5432/fixit_hub
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=fixit_hub
POSTGRES_USER=your_user
POSTGRES_PASSWORD=your_password

# Redis Configuration
REDIS_URL=redis://localhost:6379/0
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/1

# AI Services
AI_PROVIDER=bedrock  # or openai, anthropic
AI_API_KEY=your_ai_api_key
AWS_ACCESS_KEY_ID=your_aws_key  # if using Bedrock
AWS_SECRET_ACCESS_KEY=your_aws_secret
AWS_REGION=us-east-1

# Application Settings
ENVIRONMENT=development  # or production
SECRET_KEY=your_secret_key_here
JWT_SECRET=your_jwt_secret
JWT_ALGORITHM=HS256
JWT_EXPIRATION_MINUTES=15

# Rate Limiting
RATE_LIMIT_PER_MINUTE=100
DAILY_ISSUE_LIMIT=10

# Monitoring (Optional)
PROMETHEUS_PORT=9090
GRAFANA_PORT=3000
```

### Installation

```bash
# Clone the repository
git clone https://github.com/Rijuls-code/fixit-hub.git
cd fixit-hub

# Install frontend dependencies
cd frontend
npm install
# Key dependencies:
# - react@18.2.0
# - react-dom@18.2.0
# - vite@5.0.11
# - leaflet@1.9.4 (for maps)
# - workbox-precaching@7.0.0 (for PWA)
# - idb@8.0.0 (for IndexedDB)

# Install backend dependencies
cd ../backend
pip install -r requirements.txt
# Requirements should include:
# - fastapi[all]==0.109.0
# - uvicorn[standard]==0.27.0
# - celery[redis]==5.3.4
# - redis==5.0.1
# - psycopg2-binary==2.9.9
# - sqlalchemy==2.0.25
# - geoalchemy2==0.14.3
# - alembic==1.13.1
# - pillow==10.2.0
# - python-jose[cryptography]==3.3.0
# - passlib[bcrypt]==1.7.4
# - python-multipart==0.0.6
# - appwrite==4.1.0
# - boto3==1.34.34 (for AWS Bedrock)
# - prometheus-client==0.19.0
# - pydantic==2.5.3
# - httpx==0.26.0

# Setup PostgreSQL + PostGIS
# On Ubuntu/Debian:
sudo apt-get install postgresql-15 postgresql-15-postgis-3.4
sudo -u postgres createdb fixit_hub
sudo -u postgres psql -d fixit_hub -c "CREATE EXTENSION postgis;"

# On macOS (using Homebrew):
brew install postgresql@15 postgis
createdb fixit_hub
psql -d fixit_hub -c "CREATE EXTENSION postgis;"

# Install and start Redis 7.2
# On Ubuntu/Debian:
sudo apt-get install redis-server
sudo systemctl start redis
redis-cli --version  # Verify: 7.2.4 or higher

# On macOS:
brew install redis
brew services start redis

# Run database migrations
cd backend
alembic upgrade head

# Start services (development)

# Terminal 1: Frontend
cd frontend
npm run dev

# Terminal 2: Backend API
cd backend
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Terminal 3: Celery Worker (AI Processing)
cd backend
celery -A tasks.celery_app worker --loglevel=info --concurrency=4

# Terminal 4: Celery Beat (Scheduled Tasks)
cd backend
celery -A tasks.celery_app beat --loglevel=info

# Optional: Flower (Celery Monitoring)
# Terminal 5:
cd backend
celery -A tasks.celery_app flower --port=5555
```

**Access Points:**
- Frontend: http://localhost:5173
- Backend API: http://localhost:8000
- API Docs: http://localhost:8000/docs
- Flower Dashboard: http://localhost:5555

---

## 🏛️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     FRONTEND LAYER                               │
│  React PWA (Vite) → Service Worker → IndexedDB (Offline)       │
│                          ↓                                       │
│                    Vercel Edge CDN                              │
└─────────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────────┐
│                     API GATEWAY LAYER                            │
│  Render Load Balancer → Rate Limiting → CORS → SSL/TLS         │
└─────────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────────┐
│                   BACKEND SERVICE LAYER                          │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │ Auth Service │  │ Issue Mgmt   │  │ Analytics    │         │
│  │              │  │ Service      │  │ Service      │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │ AI Processing│  │ Notification │  │ Territory    │         │
│  │ Workers      │  │ Service      │  │ Service      │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│                                                                  │
│                FastAPI (Async) + Celery Workers                 │
└─────────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────────┐
│                    DATA & STORAGE LAYER                          │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐                   │
│  │ Appwrite Cloud   │  │ PostgreSQL +     │                   │
│  │ - Auth           │  │ PostGIS          │                   │
│  │ - NoSQL DB       │  │ - Territories    │                   │
│  │ - File Storage   │  │ - Spatial Index  │                   │
│  └──────────────────┘  └──────────────────┘                   │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐                   │
│  │ Redis            │  │ AI Services      │                   │
│  │ - Task Queue     │  │ - Bedrock/OpenAI │                   │
│  │ - Cache Layer    │  │ - Translation    │                   │
│  │ - Session Store  │  │ - Classification │                   │
│  └──────────────────┘  └──────────────────┘                   │
└─────────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────────┐
│                 MONITORING & OBSERVABILITY                       │
│  Prometheus → Grafana → Alerting → Render Logs                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Architectural Features

1. **Modular Service Architecture**
   - Loosely coupled microservices
   - Independent scaling per service
   - Clear separation of concerns

2. **Asynchronous AI Pipeline**
   - Non-blocking API responses
   - Celery workers for heavy processing
   - Redis-backed task queue
   - Automatic retry with exponential backoff

3. **Geospatial Intelligence**
   - PostGIS for spatial queries
   - GiST indexes for O(log n) lookups
   - Point-in-polygon territory matching
   - Spatial clustering (DBSCAN)

4. **Offline-First PWA**
   - Service worker caching
   - IndexedDB for local storage
   - Background sync when online
   - Optimistic UI updates

5. **Scalability & Performance**
   - Horizontal scaling via Render
   - Redis caching (15-min analytics, 5-min listings)
   - CDN for static assets and images
   - Database connection pooling
   - Read replicas for analytics

6. **Security & Fraud Prevention**
   - Rate limiting (100 API requests/min per user, 10 issue submissions/day)
   - Phone verification for reporters
   - GPS metadata in proof images
   - Image hash verification
   - Reputation scoring system

The architecture is cloud-ready and designed for national-scale deployment.

---

## 👥 Team

FixIt Hub is developed by a multidisciplinary team focused on scalable backend systems, AI integration, and civic technology innovation.

### Members :

- Repo Owner : [**Rijul Singh**](https://github.com//Rijuls-code)
- Member : [Shreyas Patil](https://github.com//Shreyas-patil07)
- Member : [Vedant Sawant](https://github.com//vedantsawant2803-cloud)
- Member : [Nidhi Nikam](https://github.com//Nidhi194)

---

## 📌 Project Status

- ✅ Planning & Documentation Phase Completed
- 🚀 MVP implementation in progress
- 🔧 Appwrite Cloud integration active

---

## 🚀 Deployment

### Production Infrastructure

- **Frontend:** Vercel (automatic deployments from main branch)
  - Edge CDN with 99.99% uptime SLA
  - Automatic SSL/TLS certificates
  - DDoS protection and WAF
  
- **Backend:** Render (containerized FastAPI deployment)
  - Auto-scaling based on CPU/memory
  - Health checks every 30 seconds
  - Zero-downtime deployments
  - Automatic rollback on failure
  
- **Database:** Appwrite Cloud + PostgreSQL
  - Automatic daily backups
  - Point-in-time recovery
  - Read replicas for analytics
  - Connection pooling (20-50 connections)
  
- **Storage:** Appwrite Storage
  - CDN-backed global delivery
  - Automatic image optimization
  - Pre-signed upload URLs
  - 99.9% durability guarantee
  
- **AI Services:** Amazon Bedrock / External API
  - Async processing via Celery
  - Automatic retry on failure
  - Fallback to template-based responses
  
- **Cache Layer:** Redis Cloud
  - 15-min TTL for analytics
  - 5-min TTL for issue listings
  - Session storage
  - Task queue backend

### Scaling Strategy

**Horizontal Scaling:**
- Backend: 2-10 instances based on load
- Celery Workers: 3-20 workers for AI processing
- Database: Read replicas for analytics queries

**Vertical Scaling:**
- Database: Scale up during peak hours
- Redis: Increase memory for larger cache

**Performance Targets:**
- API Response Time: p95 < 200ms, p99 < 500ms
- Page Load Time: < 2s on 4G connection
- AI Processing: < 5s for translation + classification
- Image Upload: < 3s for 5MB file

### Monitoring & Alerts

**Prometheus Metrics:**
```yaml
# API Performance
http_request_duration_seconds{endpoint="/api/issues"}
http_requests_total{status="500"}

# Queue Health
celery_queue_length{queue="ai_processing"}
celery_task_duration_seconds{task="translate_issue"}

# Database Performance
postgres_query_duration_seconds{query="find_nearby_issues"}
postgres_connection_pool_size

# Business Metrics
issues_created_total{category="pothole"}
issues_resolved_total{ward="K_EAST"}
```

**Alert Rules:**
- API error rate > 5% for 5 minutes
- Queue backlog > 1000 tasks
- Database connection pool > 80% utilization
- AI processing failure rate > 10%

---

## 📄 License

MIT License



