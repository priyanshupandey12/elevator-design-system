# Smart Elevator Control System — ERD README

## Overview
This system models a **multi-building elevator control platform** used in large infrastructures (malls, hospitals, corporate towers, etc.). It supports **real-time request handling, elevator assignment, ride tracking, and maintenance monitoring**.

The design separates:
- **Static configuration** (buildings, floors, elevators)
- **Operational data** (requests, assignments, rides)
- **Temporal tracking** (status, maintenance logs)

---

## Core Design Principles

- **Separation of concerns**
  - Configuration ≠ runtime data
- **Many-to-many flexibility**
  - Elevators ↔ Floors
- **Event-based tracking**
  - Requests → Assignments → Rides
- **Historical integrity**
  - No overwriting (status + maintenance tracked separately)

---

## Entities and Purpose

### 1. Buildings
Represents physical infrastructures.

| Field | Description |
|------|------------|
| building_id | Unique identifier |
| name | Building name |
| city, state, address | Location details |
| created_at, updated_at | Audit fields |

**Relationships**
- One building → many floors  
- One building → many elevators  

---

### 2. Floors
Represents floors inside a building.

| Field | Description |
|------|------------|
| floor_id | Unique identifier |
| building_id | Belongs to building |
| floor_number | Logical floor index |

---

### 3. Elevators
Represents elevator units.

| Field | Description |
|------|------------|
| elevator_id | Unique identifier |
| building_id | Belongs to building |
| elevator_code | Unique code |
| capacity_persons | Max people |
| capacity_weight_kg | Max load |

---

### 4. Elevator-Floor Mapping (`elevator_floors`)
Defines **which floors an elevator serves**.

| Field | Description |
|------|------------|
| id | Primary key |
| elevator_id | FK |
| floor_id | FK |

**Purpose**
- Supports:
  - Multiple elevators serving the same floor
  - One elevator serving multiple floors

---

### 5. Floor Requests
Represents user-generated requests from floors.

| Field | Description |
|------|------------|
| request_id | Unique request |
| floor_id | Origin floor |
| direction | up / down |
| status | pending / assigned / completed |
| created_at | Timestamp |

---

### 6. Ride Assignments
Maps requests to elevators.

| Field | Description |
|------|------------|
| ride_assignments_id | Primary key |
| request_id | FK |
| elevator_id | FK |
| assigned_at | Timestamp |
| status | assigned / cancelled / completed |

**Purpose**
- Decouples request from execution
- Allows reassignment if needed

---

### 7. Rides (Trips)
Represents actual elevator trips.

| Field | Description |
|------|------------|
| ride_id | Primary key |
| request_id | FK |
| elevator_id | FK |
| start_floor_id | FK |
| end_floor_id | FK |
| start_time | Start time |
| end_time | End time |

**Notes**
- Stores execution-level data
- Used for analytics

---

### 8. Elevator Status Tracking
Tracks dynamic elevator state over time.

| Field | Description |
|------|------------|
| tracker_id | Primary key |
| elevator_id | FK |
| status | idle / moving / maintenance |
| start_time | Start |
| end_time | End |

**Why separate**
- Avoids overwriting current status
- Enables historical tracking

---

### 9. Maintenance Logs
Tracks repair and downtime events.

| Field | Description |
|------|------------|
| maintenance_id | Primary key |
| elevator_id | FK |
| issue | Description |
| start_time | Start |
| end_time | End |
| fixed_by | Technician |
| status | ongoing / completed |

---

## Relationships Summary
Building 1 ──── N Floors
Building 1 ──── N Elevators

Elevator N ──── N Floors (via elevator_floors)

Floor 1 ──── N Requests

Request 1 ──── 1 Assignment
Elevator 1 ──── N Assignments

Request 1 ──── 1 Ride
Elevator 1 ──── N Rides

Elevator 1 ──── N Status Records
Elevator 1 ──── N Maintenance Logs


---

## Key System Capabilities Supported

### Infrastructure Queries
- Total buildings → `buildings`
- Elevators per building → `elevators`
- Floors per building → `floors`

### Operational Queries
- Requests from a floor → `floor_requests`
- Pending requests → filter `status = 'pending'`
- Elevator handling requests → `ride_assignments`

### Allocation Logic
- Elevator serving floors → `elevator_floors`
- Multiple elevators per floor → supported
- Multi-floor elevator → supported

### Analytics
- Rides per elevator → `rides`
- Most active elevator → aggregate `rides`
- Ride duration → `end_time - start_time`

### Monitoring
- Current status → latest `elevator_status_tracking`
- Maintenance downtime → `maintenance_logs`

---

## Design Strengths

- Scalable across multiple buildings  
- Flexible elevator-floor mapping  
- Event-driven architecture  
- Historical tracking preserved  
- Clean separation of static vs dynamic data  

---

## Optional Improvements

- Add **zones** for large buildings  
- Add **priority levels** in requests  
- Add **failure logs / error events**  
- Add **real-time location tracking (current_floor)**  

---

## Conclusion

This ERD is structured for:
- High concurrency systems  
- Operational monitoring  
- Analytical workloads  

It avoids:
- Data duplication  
- Tight coupling  
- Loss of historical data  

The model is production-ready and aligns with real-world elevator dispatch systems.
