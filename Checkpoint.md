# FixIt Hub - Backend Development Checkpoint

## Overview

This document breaks down the backend implementation into small, manageable modules. Each module is a self-contained piece that integrates into the larger system architecture.

---

## Module Structure

The backend is organized into the following layers:

1. **Core Infrastructure** - Foundation services
2. **Authentication & Authorization** - User management and security
3. **Issue Management** - Core issue CRUD operations
4. **Territory Management** - Geographic and jurisdiction handling
5. **AI Processing** - Translation and classification
6. **Analytics & Risk Scoring** - Data aggregation and insights
7. **Report Generation** - Custom reports and exports
8. **Notification System** - Email and in-app notifications
9. **API Layer** - RESTful endpoints
10. **Background Jobs** - Async task processing

---

## 1. Core Infrastructure Modules

### 1.1 Configuration Manager
**File**: `server/src/core/config.py`
**Purpose**: Centralized configuration management
**Size**: ~50 lines
**Dependencies**: None
**Key Functions**:
- Load environment variables
- Validate required configs
- Provide config access interface

### 1.2 Database Connection Manager
**File**: `server/src/core/database.py`
**Purpose**: Appwrite Database connection and client initialization
**Size**: ~40 lines
**Dependencies**: Appwrite SDK
**Key Functions**:
- Initialize Appwrite client
- Create database connection
- Handle connection pooling

### 1.3 Storage Manager
**File**: `server/src/core/storage.py`
**Purpose**: Appwrite Storage operations
**Size**: ~60 lines

**Dependencies**: Appwrite SDK
**Key Functions**:
- Upload files to storage
- Generate file URLs
- Delete files
- Validate file types and sizes

### 1.4 Logger
**File**: `server/src/core/logger.py`
**Purpose**: Structured logging
**Size**: ~45 lines
**Dependencies**: Python logging
**Key Functions**:
- Configure JSON logging
- Log levels (INFO, ERROR, DEBUG)
- Request ID tracking

### 1.5 Error Handlers
**File**: `server/src/core/errors.py`
**Purpose**: Custom exception classes and error handling
**Size**: ~70 lines
**Dependencies**: FastAPI
**Key Functions**:
- Define custom exceptions
- HTTP error responses
- Validation error formatting

### 1.6 Middleware
**File**: `server/src/core/middleware.py`
**Purpose**: Request/response middleware
**Size**: ~80 lines
**Dependencies**: FastAPI
**Key Functions**:
- CORS handling
- Request logging
- Rate limiting
- Request ID generation

---

## 2. Authentication & Authorization Modules

### 2.1 Password Utilities
**File**: `server/src/auth/password.py`
**Purpose**: Password hashing and verification
**Size**: ~30 lines
**Dependencies**: bcrypt
**Key Functions**:
- Hash password (bcrypt cost 12)
- Verify password


### 2.2 JWT Token Manager
**File**: `server/src/auth/jwt.py`
**Purpose**: JWT token creation and validation
**Size**: ~60 lines
**Dependencies**: PyJWT
**Key Functions**:
- Create access token (15 min expiry)
- Create refresh token (7 day expiry)
- Verify token
- Decode token payload

### 2.3 User Registration Service
**File**: `server/src/auth/registration.py`
**Purpose**: User registration logic
**Size**: ~90 lines
**Dependencies**: Appwrite SDK, password utilities
**Key Functions**:
- Validate registration data
- Check email uniqueness
- Create user in Appwrite Auth
- Send verification email

### 2.4 User Login Service
**File**: `server/src/auth/login.py`
**Purpose**: User authentication
**Size**: ~70 lines
**Dependencies**: JWT manager, Appwrite SDK
**Key Functions**:
- Authenticate credentials
- Generate tokens
- Track failed login attempts
- Account lockout logic

### 2.5 Role-Based Access Control (RBAC)
**File**: `server/src/auth/rbac.py`
**Purpose**: Permission checking
**Size**: ~50 lines
**Dependencies**: None
**Key Functions**:
- Check user role
- Verify permissions
- Role decorators for routes

### 2.6 Auth Middleware
**File**: `server/src/auth/middleware.py`
**Purpose**: Request authentication
**Size**: ~55 lines

**Dependencies**: JWT manager
**Key Functions**:
- Extract token from header
- Validate token
- Attach user to request
- Handle token expiry

---

## 3. Issue Management Modules

### 3.1 Issue Model
**File**: `server/src/models/issue.py`
**Purpose**: Issue data model and validation
**Size**: ~80 lines
**Dependencies**: Pydantic
**Key Functions**:
- Define issue schema
- Validation rules
- Status enum
- Category enum

### 3.2 Issue Repository
**File**: `server/src/repositories/issue_repository.py`
**Purpose**: Database operations for issues
**Size**: ~120 lines
**Dependencies**: Appwrite SDK
**Key Functions**:
- Create issue
- Get issue by ID
- List issues with filters
- Update issue
- Delete issue
- Pagination support

### 3.3 Issue Creation Service
**File**: `server/src/services/issue_creation.py`
**Purpose**: Business logic for creating issues
**Size**: ~100 lines
**Dependencies**: Issue repository, storage manager
**Key Functions**:
- Validate issue data
- Upload images
- Detect language
- Assign initial status
- Generate UUID
- Queue AI processing

### 3.4 Issue Update Service
**File**: `server/src/services/issue_update.py`
**Purpose**: Update issue status and details
**Size**: ~80 lines

**Dependencies**: Issue repository
**Key Functions**:
- Update status
- Add notes
- Validate state transitions
- Record audit trail

### 3.5 Issue Resolution Service
**File**: `server/src/services/issue_resolution.py`
**Purpose**: Handle issue resolution
**Size**: ~95 lines
**Dependencies**: Issue repository, storage manager
**Key Functions**:
- Validate resolution data
- Upload proof photo
- Update status to RESOLVED
- Record resolution timestamp
- Create resolution history entry

### 3.6 Issue Reopen Service
**File**: `server/src/services/issue_reopen.py`
**Purpose**: Handle issue reopening
**Size**: ~110 lines
**Dependencies**: Issue repository
**Key Functions**:
- Check reopen eligibility
- Validate cooldown period
- Verify original reporter
- Increment reopen count
- Update status to REOPENED
- Record reopen reason

### 3.7 Issue Search Service
**File**: `server/src/services/issue_search.py`
**Purpose**: Search and filter issues
**Size**: ~85 lines
**Dependencies**: Issue repository
**Key Functions**:
- Full-text search
- Filter by status, category, ward
- Sort by date, status
- Pagination

### 3.8 Duplicate Detection Service
**File**: `server/src/services/duplicate_detection.py`
**Purpose**: Detect duplicate issues
**Size**: ~75 lines

**Dependencies**: Issue repository, geospatial utilities
**Key Functions**:
- Check radius for existing issues
- Calculate distance between coordinates
- Find similar issues
- Return nearby issues

---

## 4. Territory Management Modules

### 4.1 Territory Model
**File**: `server/src/models/territory.py`
**Purpose**: Territory data model
**Size**: ~60 lines
**Dependencies**: Pydantic
**Key Functions**:
- Define territory schema
- GeoJSON polygon validation
- Territory type enum

### 4.2 Territory Repository
**File**: `server/src/repositories/territory_repository.py`
**Purpose**: Database operations for territories
**Size**: ~100 lines
**Dependencies**: Appwrite SDK
**Key Functions**:
- Create territory
- Get territory by ID
- List all territories
- Update territory
- Delete territory
- Get territories by officer

### 4.3 Geospatial Utilities
**File**: `server/src/utils/geospatial.py`
**Purpose**: Geographic calculations
**Size**: ~90 lines
**Dependencies**: shapely, geopy
**Key Functions**:
- Point-in-polygon check
- Calculate distance between coordinates
- Reverse geocoding
- Validate GeoJSON

### 4.4 Territory Assignment Service
**File**: `server/src/services/territory_assignment.py`
**Purpose**: Automatic territory detection
**Size**: ~80 lines

**Dependencies**: Territory repository, geospatial utilities
**Key Functions**:
- Detect territory from coordinates
- Assign issue to territory
- Get assigned officer
- Handle no-match cases

### 4.5 Proximity Search Service
**File**: `server/src/services/proximity_search.py`
**Purpose**: Find nearby issues
**Size**: ~70 lines
**Dependencies**: Issue repository, geospatial utilities
**Key Functions**:
- Search issues within radius
- Calculate distances
- Sort by proximity
- Filter by category

### 4.6 Clustering Service
**File**: `server/src/services/clustering.py`
**Purpose**: Detect issue clusters
**Size**: ~95 lines
**Dependencies**: Issue repository, geospatial utilities
**Key Functions**:
- Identify clusters (3+ issues within 200m)
- Calculate cluster center
- Get dominant category
- Suggest batch resolution

### 4.7 Territory Analytics Service
**File**: `server/src/services/territory_analytics.py`
**Purpose**: Territory-level statistics
**Size**: ~100 lines
**Dependencies**: Issue repository, territory repository
**Key Functions**:
- Compute issue density
- Calculate resolution rates
- Identify hotspots
- Generate comparison reports

---

## 5. AI Processing Modules

### 5.1 AI Client
**File**: `server/src/ai/client.py`
**Purpose**: AI service integration
**Size**: ~70 lines

**Dependencies**: boto3 (for Bedrock) or OpenAI SDK
**Key Functions**:
- Initialize AI client
- Send prompt to AI
- Handle API errors
- Retry logic

### 5.2 Translation Service
**File**: `server/src/ai/translation.py`
**Purpose**: Multilingual translation
**Size**: ~85 lines
**Dependencies**: AI client
**Key Functions**:
- Detect language
- Translate to English
- Preserve technical terms
- Store original and translated text
- Handle translation failures

### 5.3 Classification Service
**File**: `server/src/ai/classification.py`
**Purpose**: Automatic issue categorization
**Size**: ~90 lines
**Dependencies**: AI client
**Key Functions**:
- Classify issue into category
- Calculate confidence score
- Auto-assign if confidence > 0.8
- Suggest category if 0.5-0.8
- Handle low confidence cases

### 5.4 AI Queue Manager
**File**: `server/src/ai/queue.py`
**Purpose**: Async AI task processing
**Size**: ~75 lines
**Dependencies**: Redis or in-memory queue
**Key Functions**:
- Queue translation tasks
- Queue classification tasks
- Process queue
- Retry failed tasks
- Track processing status

### 5.5 Executive Summary Generator
**File**: `server/src/ai/summary.py`
**Purpose**: Generate AI summaries for reports
**Size**: ~80 lines

**Dependencies**: AI client
**Key Functions**:
- Format data for AI
- Generate executive summary
- Validate summary length
- Fallback to template
- Retry on failure

---

## 6. Analytics & Risk Scoring Modules

### 6.1 Statistics Aggregator
**File**: `server/src/analytics/statistics.py`
**Purpose**: Compute system statistics
**Size**: ~110 lines
**Dependencies**: Issue repository
**Key Functions**:
- Count total issues
- Count by status
- Calculate resolution rate
- Compute average resolution time
- Week-over-week growth
- Category breakdown
- Ward breakdown

### 6.2 Risk Score Calculator
**File**: `server/src/analytics/risk_scoring.py`
**Purpose**: Calculate ward risk scores
**Size**: ~130 lines
**Dependencies**: Issue repository, territory repository
**Key Functions**:
- Calculate Repeated Reports Score (R)
- Calculate Time Unresolved Score (T)
- Calculate Growth Rate Score (G)
- Calculate Severity Score (S)
- Compute weighted risk score
- Normalize scores to 0-100

### 6.3 Risk Classifier
**File**: `server/src/analytics/risk_classifier.py`
**Purpose**: Classify risk levels
**Size**: ~50 lines
**Dependencies**: None
**Key Functions**:
- Classify into CRITICAL/HIGH/MEDIUM/LOW
- Assign color codes
- Identify high-risk zones


### 6.4 Trend Analyzer
**File**: `server/src/analytics/trends.py`
**Purpose**: Analyze trends over time
**Size**: ~95 lines
**Dependencies**: Issue repository
**Key Functions**:
- Track issue trends
- Identify patterns
- Seasonal analysis
- Growth predictions

### 6.5 Performance Metrics
**File**: `server/src/analytics/performance.py`
**Purpose**: Officer and system performance
**Size**: ~85 lines
**Dependencies**: Issue repository
**Key Functions**:
- Officer resolution metrics
- Average response time
- Resolution quality scores
- Workload distribution

---

## 7. Report Generation Modules

### 7.1 Report Template Model
**File**: `server/src/models/report_template.py`
**Purpose**: Report template data model
**Size**: ~65 lines
**Dependencies**: Pydantic
**Key Functions**:
- Define template schema
- Filter configuration
- Validation rules

### 7.2 Report Template Repository
**File**: `server/src/repositories/report_template_repository.py`
**Purpose**: Database operations for templates
**Size**: ~100 lines
**Dependencies**: Appwrite SDK
**Key Functions**:
- Create template
- Get template by ID
- List templates by officer
- Update template
- Delete template
- Get public templates

### 7.3 Report Data Collector
**File**: `server/src/reports/data_collector.py`
**Purpose**: Collect data for reports
**Size**: ~110 lines

**Dependencies**: Issue repository, statistics aggregator
**Key Functions**:
- Apply template filters
- Aggregate data
- Calculate metrics
- Format for export

### 7.4 PDF Report Generator
**File**: `server/src/reports/pdf_generator.py`
**Purpose**: Generate PDF reports
**Size**: ~140 lines
**Dependencies**: ReportLab or WeasyPrint
**Key Functions**:
- Create PDF document
- Add header and footer
- Generate charts (pie, bar, line)
- Format tables
- Add executive summary
- Page numbering

### 7.5 CSV Report Generator
**File**: `server/src/reports/csv_generator.py`
**Purpose**: Generate CSV reports
**Size**: ~60 lines
**Dependencies**: Python csv module
**Key Functions**:
- Format data as CSV
- Handle special characters
- UTF-8 encoding
- Column headers

### 7.6 Excel Report Generator
**File**: `server/src/reports/excel_generator.py`
**Purpose**: Generate Excel reports
**Size**: ~120 lines
**Dependencies**: openpyxl or xlsxwriter
**Key Functions**:
- Create workbook
- Add summary sheet
- Add detailed data sheet
- Create pivot tables
- Apply conditional formatting
- Add charts

### 7.7 Report Scheduler
**File**: `server/src/reports/scheduler.py`
**Purpose**: Schedule recurring reports
**Size**: ~100 lines

**Dependencies**: APScheduler or Celery
**Key Functions**:
- Schedule report generation
- Parse cron expressions
- Execute scheduled tasks
- Handle failures
- Send email notifications

### 7.8 Weekly Report Service
**File**: `server/src/reports/weekly_report.py`
**Purpose**: Generate automated weekly reports
**Size**: ~115 lines
**Dependencies**: Statistics aggregator, AI summary generator
**Key Functions**:
- Collect weekly data
- Generate AI summary
- Store report
- Trigger notifications

---

## 8. Notification System Modules

### 8.1 Email Service
**File**: `server/src/notifications/email.py`
**Purpose**: Send email notifications
**Size**: ~80 lines
**Dependencies**: SMTP library or SendGrid
**Key Functions**:
- Send email
- Format HTML templates
- Handle attachments
- Retry on failure

### 8.2 Email Templates
**File**: `server/src/notifications/templates.py`
**Purpose**: Email template rendering
**Size**: ~90 lines
**Dependencies**: Jinja2
**Key Functions**:
- Issue created template
- Issue resolved template
- Issue reopened template
- Report ready template
- Welcome email template

### 8.3 Notification Queue
**File**: `server/src/notifications/queue.py`
**Purpose**: Queue notifications for async sending
**Size**: ~70 lines

**Dependencies**: Redis or in-memory queue
**Key Functions**:
- Queue email notifications
- Process queue
- Track delivery status
- Retry failed sends

### 8.4 In-App Notification Service
**File**: `server/src/notifications/in_app.py`
**Purpose**: Manage in-app notifications
**Size**: ~75 lines
**Dependencies**: Appwrite SDK
**Key Functions**:
- Create notification
- Mark as read
- Get unread count
- List user notifications

---

## 9. API Layer Modules

### 9.1 Auth Routes
**File**: `server/src/routes/auth.py`
**Purpose**: Authentication endpoints
**Size**: ~120 lines
**Dependencies**: Auth services
**Endpoints**:
- POST /api/v1/auth/register
- POST /api/v1/auth/login
- POST /api/v1/auth/refresh
- POST /api/v1/auth/logout
- POST /api/v1/auth/reset-password

### 9.2 Issue Routes
**File**: `server/src/routes/issues.py`
**Purpose**: Issue management endpoints
**Size**: ~150 lines
**Dependencies**: Issue services
**Endpoints**:
- POST /api/v1/issues
- GET /api/v1/issues
- GET /api/v1/issues/{id}
- PATCH /api/v1/issues/{id}
- DELETE /api/v1/issues/{id}
- PATCH /api/v1/issues/{id}/resolve
- POST /api/v1/issues/{id}/reopen

### 9.3 Territory Routes
**File**: `server/src/routes/territories.py`
**Purpose**: Territory management endpoints
**Size**: ~110 lines

**Dependencies**: Territory services
**Endpoints**:
- POST /api/v1/territories
- GET /api/v1/territories
- GET /api/v1/territories/{id}
- PUT /api/v1/territories/{id}
- DELETE /api/v1/territories/{id}

### 9.4 Analytics Routes
**File**: `server/src/routes/analytics.py`
**Purpose**: Analytics and statistics endpoints
**Size**: ~90 lines
**Dependencies**: Analytics services
**Endpoints**:
- GET /api/v1/stats
- GET /api/v1/stats/risk-zones
- GET /api/v1/stats/trends
- GET /api/v1/stats/performance

### 9.5 Report Routes
**File**: `server/src/routes/reports.py`
**Purpose**: Report generation endpoints
**Size**: ~130 lines
**Dependencies**: Report services
**Endpoints**:
- POST /api/v1/reports/generate
- GET /api/v1/reports/latest
- GET /api/v1/reports
- GET /api/v1/reports/{id}
- POST /api/v1/reports/export

### 9.6 Report Template Routes
**File**: `server/src/routes/report_templates.py`
**Purpose**: Report template management
**Size**: ~100 lines
**Dependencies**: Report template services
**Endpoints**:
- POST /api/v1/report-templates
- GET /api/v1/report-templates
- GET /api/v1/report-templates/{id}
- PUT /api/v1/report-templates/{id}
- DELETE /api/v1/report-templates/{id}

### 9.7 Report Schedule Routes
**File**: `server/src/routes/report_schedules.py`
**Purpose**: Scheduled report management
**Size**: ~95 lines

**Dependencies**: Report scheduler
**Endpoints**:
- POST /api/v1/report-schedules
- GET /api/v1/report-schedules
- GET /api/v1/report-schedules/{id}
- PUT /api/v1/report-schedules/{id}
- DELETE /api/v1/report-schedules/{id}

### 9.8 User Routes
**File**: `server/src/routes/users.py`
**Purpose**: User management endpoints
**Size**: ~85 lines
**Dependencies**: User services
**Endpoints**:
- GET /api/v1/users/me
- PUT /api/v1/users/me
- GET /api/v1/users/{id}
- GET /api/v1/users

### 9.9 Health Check Routes
**File**: `server/src/routes/health.py`
**Purpose**: System health monitoring
**Size**: ~40 lines
**Dependencies**: Database, storage
**Endpoints**:
- GET /api/v1/health
- GET /api/v1/health/db
- GET /api/v1/health/storage

---

## 10. Background Jobs Modules

### 10.1 AI Processing Worker
**File**: `server/src/workers/ai_worker.py`
**Purpose**: Process AI tasks asynchronously
**Size**: ~90 lines
**Dependencies**: AI services, queue manager
**Key Functions**:
- Process translation queue
- Process classification queue
- Handle failures
- Update issue status

### 10.2 Report Generation Worker
**File**: `server/src/workers/report_worker.py`
**Purpose**: Generate reports asynchronously
**Size**: ~85 lines

**Dependencies**: Report services
**Key Functions**:
- Process report generation queue
- Generate scheduled reports
- Send email notifications
- Clean up old reports

### 10.3 Notification Worker
**File**: `server/src/workers/notification_worker.py`
**Purpose**: Send notifications asynchronously
**Size**: ~75 lines
**Dependencies**: Notification services
**Key Functions**:
- Process notification queue
- Send emails
- Track delivery status
- Retry failures

### 10.4 Analytics Worker
**File**: `server/src/workers/analytics_worker.py`
**Purpose**: Update analytics and risk scores
**Size**: ~80 lines
**Dependencies**: Analytics services
**Key Functions**:
- Update risk scores weekly
- Refresh cached statistics
- Generate trend data
- Update performance metrics

### 10.5 Cleanup Worker
**File**: `server/src/workers/cleanup_worker.py`
**Purpose**: Periodic cleanup tasks
**Size**: ~60 lines
**Dependencies**: Storage manager
**Key Functions**:
- Delete old reports
- Clean up temporary files
- Archive old data
- Optimize storage

---

## 11. Utility Modules

### 11.1 Date/Time Utilities
**File**: `server/src/utils/datetime.py`
**Purpose**: Date and time helpers
**Size**: ~50 lines
**Dependencies**: Python datetime
**Key Functions**:
- Format timestamps
- Calculate time differences
- Timezone conversions
- Week/month calculations


### 11.2 Validation Utilities
**File**: `server/src/utils/validation.py`
**Purpose**: Input validation helpers
**Size**: ~70 lines
**Dependencies**: Pydantic
**Key Functions**:
- Validate email format
- Validate phone number
- Validate coordinates
- Validate file types
- Sanitize inputs

### 11.3 Pagination Utilities
**File**: `server/src/utils/pagination.py`
**Purpose**: Pagination helpers
**Size**: ~45 lines
**Dependencies**: None
**Key Functions**:
- Calculate offset
- Generate page metadata
- Build pagination response

### 11.4 File Utilities
**File**: `server/src/utils/files.py`
**Purpose**: File handling helpers
**Size**: ~60 lines
**Dependencies**: Python os, pathlib
**Key Functions**:
- Validate file size
- Check file extension
- Generate unique filenames
- Clean up temp files

### 11.5 String Utilities
**File**: `server/src/utils/strings.py`
**Purpose**: String manipulation helpers
**Size**: ~55 lines
**Dependencies**: None
**Key Functions**:
- Slugify text
- Truncate text
- Generate random strings
- Format names

---

## 12. Testing Modules

### 12.1 Test Fixtures
**File**: `server/tests/fixtures.py`
**Purpose**: Reusable test data
**Size**: ~100 lines

**Dependencies**: pytest
**Key Functions**:
- Mock user data
- Mock issue data
- Mock territory data
- Mock AI responses

### 12.2 Auth Tests
**File**: `server/tests/test_auth.py`
**Purpose**: Test authentication flows
**Size**: ~120 lines
**Dependencies**: pytest, fixtures
**Test Cases**:
- User registration
- User login
- Token refresh
- Password reset
- RBAC checks

### 12.3 Issue Tests
**File**: `server/tests/test_issues.py`
**Purpose**: Test issue management
**Size**: ~150 lines
**Dependencies**: pytest, fixtures
**Test Cases**:
- Create issue
- Update issue
- Resolve issue
- Reopen issue
- Duplicate detection

### 12.4 Territory Tests
**File**: `server/tests/test_territories.py`
**Purpose**: Test territory management
**Size**: ~100 lines
**Dependencies**: pytest, fixtures
**Test Cases**:
- Create territory
- Point-in-polygon detection
- Territory assignment
- Proximity search

### 12.5 Analytics Tests
**File**: `server/tests/test_analytics.py`
**Purpose**: Test analytics calculations
**Size**: ~110 lines
**Dependencies**: pytest, fixtures
**Test Cases**:
- Statistics aggregation
- Risk score calculation
- Trend analysis


### 12.6 Report Tests
**File**: `server/tests/test_reports.py`
**Purpose**: Test report generation
**Size**: ~95 lines
**Dependencies**: pytest, fixtures
**Test Cases**:
- Template creation
- Report generation
- PDF export
- CSV export
- Excel export
- Scheduled reports

---

## Implementation Order

### Phase 1: Foundation (Week 1)
1. Core Infrastructure (1.1 - 1.6)
2. Authentication & Authorization (2.1 - 2.6)
3. Basic Issue Management (3.1 - 3.4)

### Phase 2: Core Features (Week 2)
4. Issue Resolution & Reopen (3.5 - 3.6)
5. Territory Management (4.1 - 4.4)
6. Issue Search & Duplicate Detection (3.7 - 3.8)

### Phase 3: Intelligence (Week 3)
7. AI Processing (5.1 - 5.5)
8. Analytics & Risk Scoring (6.1 - 6.4)
9. Proximity & Clustering (4.5 - 4.7)

### Phase 4: Reporting (Week 4)
10. Report Generation (7.1 - 7.6)
11. Report Scheduling (7.7 - 7.8)
12. Notification System (8.1 - 8.4)

### Phase 5: API & Workers (Week 5)
13. API Routes (9.1 - 9.9)
14. Background Jobs (10.1 - 10.5)
15. Testing (12.1 - 12.6)

---

## Module Dependencies Graph

```
Core Infrastructure (1)
    ↓
Authentication (2) ← Core
    ↓
Issue Management (3) ← Auth, Core
    ↓
Territory Management (4) ← Issue, Core
    ↓
AI Processing (5) ← Issue, Core
    ↓
Analytics (6) ← Issue, Territory
    ↓
Reports (7) ← Analytics, Issue
    ↓
Notifications (8) ← Issue, Reports
    ↓
API Layer (9) ← All Services
    ↓
Background Jobs (10) ← All Services
```

---

## File Size Summary

- **Total Modules**: 75+
- **Average File Size**: 60-120 lines
- **Largest Modules**: PDF Generator (140 lines), Issue Repository (120 lines)
- **Smallest Modules**: Password Utilities (30 lines), Health Check (40 lines)
- **Total Estimated Lines**: ~6,500 lines (excluding tests)

---

## Key Principles

1. **Single Responsibility**: Each module does one thing well
2. **Loose Coupling**: Modules communicate through interfaces
3. **High Cohesion**: Related functionality grouped together
4. **Testability**: Each module can be tested independently
5. **Reusability**: Utilities and services are reusable
6. **Maintainability**: Small files are easier to understand and modify

---

## Next Steps

1. Set up project structure
2. Implement Phase 1 modules
3. Write unit tests for each module
4. Integrate modules progressively
5. Test integration points
6. Deploy and monitor

---

**End of Checkpoint Document**
