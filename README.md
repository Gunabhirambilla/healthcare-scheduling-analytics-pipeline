# Healthcare Scheduling Session Reconstruction & Funnel Analytics

## Overview

Healthcare self-scheduling systems allow patients to book appointments online through scheduling widgets. However, scheduling logs often record **each scheduling attempt as a separate record**, including cases where users:

- open multiple browser tabs
- refresh the scheduling page
- restart the scheduling workflow
- attempt booking from multiple devices

This behavior inflates drop-off metrics and results in **misleading scheduling conversion rates**.

This project builds a **cloud-based analytics pipeline** that reconstructs true scheduling sessions from noisy tab-level logs, performs accurate funnel analysis, and identifies operational bottlenecks in patient self-scheduling workflows.

The pipeline uses **AWS S3 for data ingestion, Snowflake for warehousing and transformation, and a visualization layer for analytics dashboards**.

---

# Problem Statement

Raw scheduling logs capture **tab-level attempts**, not **user sessions**.

Example scenario:

A patient opens the scheduling link in multiple tabs and completes booking from one of them.

| Attempt | Outcome |
|--------|--------|
| Tab 1 | Drop-off |
| Tab 2 | Drop-off |
| Tab 3 | Drop-off |
| Tab 4 | Drop-off |
| Tab 5 | Booked |

Naive analytics result:

```
Drop-offs = 4  
Bookings = 1  
Conversion Rate = 20%
```

Actual behavior:

```
1 scheduling session  
1 successful booking  
Conversion Rate = 100%
```

Without correcting this data issue, healthcare organizations:

- overestimate scheduling drop-offs
- underestimate booking conversion rates
- misidentify operational problems in scheduling workflows

---

# Project Objectives

This project aims to:

1. Reconstruct **true scheduling sessions** from tab-level scheduling logs
2. Correct inflated drop-off metrics caused by duplicate attempts
3. Perform accurate **scheduling funnel analysis**
4. Build a **modern cloud data pipeline** using AWS and Snowflake
5. Generate insights that help healthcare providers improve scheduling conversion

---

# Dataset

The dataset represents scheduling attempts recorded by a healthcare scheduling system.

Key fields used in this project include:

| Column | Description |
|------|-------------|
| practice_date_time | Scheduling workflow start time |
| scheduled_datetime | Scheduling completion time |
| patient_id | Patient identifier (0 indicates not booked) |
| slot_id | Selected appointment slot |
| device_type | Device used to access scheduling |
| location_id | Clinic location |
| provider_id | Healthcare provider |
| appointment_type_id | Type of visit |
| IPv4 | User IP address |
| city/state/postal | Geographic information |

Important dataset characteristics:

- Each browser tab generates a **separate record**
- Only successful bookings produce a **valid patient_id**
- Multiple attempts may belong to the **same scheduling session**

---

# Project Architecture

The pipeline simulates a **modern analytics engineering workflow**.

```
CSV Dataset
     │
     ▼
AWS S3 (Raw Data Lake)
     │
     ▼
Snowflake External Stage
     │
     ▼
Snowflake Staging Tables
     │
     ▼
SQL Transformation Layer
(Session Reconstruction)
     │
     ▼
Fact & Dimension Tables
     │
     ▼
Semantic Views
     │
     ▼
Analytics Dashboard
```

---

# Data Pipeline Workflow

### 1. Raw Data Ingestion

The raw dataset is stored as a CSV file and uploaded to **AWS S3**.

```
Local CSV → AWS S3
```

S3 acts as the **raw data lake layer**.

---

### 2. Snowflake Data Loading

Snowflake reads the CSV file from S3 using an **external stage** and loads it into a raw table.

```
S3 → Snowflake Raw Table
```

Example:

```
raw_scheduling_logs
```

---

### 3. Staging Layer

The staging layer performs basic cleaning and standardization:

- timestamp formatting
- null handling
- removing invalid records
- standardizing fields

Example table:

```
stg_scheduling_logs
```

---

### 4. Session Reconstruction

Scheduling sessions are reconstructed using:

- IPv4
- device_type
- browser
- geographic attributes
- time-based session window

A new session is created when the gap between attempts exceeds **30 minutes**.

SQL window functions are used to identify session boundaries.

---

### 5. Data Warehouse Modeling

The transformed dataset is modeled using **fact and dimension tables**.

Example tables:

```
fact_scheduling_sessions
dim_location
dim_provider
dim_device
dim_geography
```

This structure enables scalable analytics.

---

### 6. Semantic Layer

Semantic views simplify analytics queries.

Example views:

```
vw_scheduling_funnel
vw_booking_conversion
vw_location_performance
vw_device_performance
```

Dashboards query these views instead of raw tables.

---

### 7. Visualization Layer

A visualization tool is used to create dashboards for:

- scheduling funnel
- conversion metrics
- provider performance
- device performance
- geographic insights

---

### 8. Monitoring

AWS monitoring services track pipeline activity.

Monitoring includes:

- S3 data ingestion
- Snowflake load status
- pipeline execution metrics

---

# Scheduling Funnel

After session reconstruction, the scheduling funnel can be analyzed accurately.

```
Scheduling Started
        ↓
Slot Selected
        ↓
Appointment Booked
```

Drop-offs are calculated using **reconstructed sessions rather than raw attempts**.

---

# Key Metrics

### Session Metrics

- Total scheduling sessions
- Successful booking sessions
- Session conversion rate
- Average attempts per session
- Average scheduling duration

---

### Data Quality Metrics

- Drop-off inflation rate
- Duplicate tab behavior
- Session reconstruction accuracy

---

### Funnel Metrics

- Drop-offs before slot selection
- Drop-offs after slot selection
- Booking conversion rate

---

### Operational Metrics

- Booking rate by device type
- Booking rate by location
- Booking rate by provider
- Geographic booking distribution

---

# Example Insights

Example insights produced by the analysis:

- Multi-tab behavior inflated drop-off metrics by **~38%**
- Mobile users required **2.1× more attempts before booking**
- Certain locations showed significantly lower scheduling conversion rates
- Some providers have high scheduling attempts but low booking completion

---

# Business Recommendations

### Persistent Session Tracking

Introduce a **session identifier across browser tabs and refreshes** to improve scheduling analytics accuracy.

### Improve Mobile Scheduling UX

Mobile users show higher repeated attempts, suggesting scheduling UI friction.

### Optimize Provider Slot Availability

Certain locations have higher drop-offs due to limited appointment availability.

### Intelligent Provider Routing

Direct patients to providers with shorter wait times to improve booking conversion.

---

# Technologies Used

| Layer | Technology |
|------|------------|
Data Source | CSV Dataset |
Development | VS Code |
Cloud Storage | AWS S3 |
Data Warehouse | Snowflake |
Transformation | SQL |
Semantic Layer | Snowflake Views |
Visualization | Dashboard Tool |
Monitoring | AWS Monitoring Services |

---

# Repository Structure

```
healthcare-scheduling-session-reconstruction-funnel-analytics/

data/
    raw/
    processed/

sql/
    staging/
    session_reconstruction/
    analytics/

python/
    data_loader.py
    session_builder.py

models/
    fact_scheduling_sessions.sql
    dim_location.sql
    dim_provider.sql
    dim_device.sql

docs/
    methodology.md
    insights.md
```

---

# Skills Demonstrated

This project demonstrates:

- Cloud data pipeline design
- AWS S3 data ingestion
- Snowflake data warehousing
- Session reconstruction using SQL window functions
- Data modeling with fact & dimension tables
- Funnel analytics
- Operational insight generation

---

# Future Improvements

Potential enhancements include:

- Cross-device session stitching
- automated pipeline orchestration
- machine learning models for booking prediction
- anomaly detection for scheduling performance
- provider capacity optimization models

---

# Author

Gunabhiram Billa  
Data Analyst | Data Engineering Enthusiast
