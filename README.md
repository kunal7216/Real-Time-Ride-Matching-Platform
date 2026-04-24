# Real-Time-Ride-Matching-Platform



## Overview

This project implements a real-time backend system for matching drivers and passengers based on proximity, availability, and system load.

It simulates systems used in ride-sharing platforms (Uber/Ola) and focuses on:
- Low-latency matching
- Real-time location processing
- Scalable event-driven architecture
- Efficient geospatial querying

---

## Problem Statement

Ride-sharing systems must solve:

- Matching nearest driver quickly
- Handling high request volumes
- Avoiding duplicate driver assignments
- Adapting to real-time location changes

This system addresses:
- Real-time matching under high concurrency
- Efficient geospatial indexing
- Scalable backend architecture

---

## High-Level Architecture

```

Client → API Gateway → Matching Service → Redis (Location Cache)
↓
Kafka
↓
Driver Service

````id="ride_arch"

---

## Core Components

### 1. API Service (Spring Boot)
- Handles ride requests
- Exposes endpoints

---

### 2. Matching Service
- Core logic for assigning drivers
- Uses geospatial queries

---

### 3. Redis (Location Store)
- Stores driver locations
- Uses geo-indexing

---

### 4. Kafka (Event Streaming)
- Handles async communication
- Updates driver state

---

### 5. Driver Service
- Manages driver availability
- Updates location in real time

---

## System Design Patterns Used

### 1. Event-Driven Architecture
- Kafka used for async updates
- Decouples services

---

### 2. Geospatial Indexing
- Redis GEO commands:
  - GEOADD
  - GEORADIUS

---

### 3. Caching Pattern
- Driver locations cached in Redis
- Fast lookup

---

### 4. Load-Aware Matching
- Assign drivers based on:
  - proximity
  - system load

---

### 5. Dynamic Pricing (Surge)
- Price increases during high demand

---

## Detailed Workflow

### Ride Request Flow

1. User sends request:
``` id="ride_api"
POST /ride/request
````

---

### Step-by-Step Matching

1. API receives request
2. Extract pickup location
3. Query Redis:

```id="ride_geo"
GEORADIUS drivers {lat} {lon} radius
```

4. Get nearby drivers

5. Apply filters:

* Availability
* Distance
* Load

6. Select best driver

7. Send assignment event via Kafka

---

### Driver Assignment Flow

1. Driver receives request
2. Accepts / rejects
3. System updates state

---

### Real-Time Location Updates

Drivers continuously send location:

```id="ride_location"
POST /driver/location
```

Redis update:

```id="ride_geoadd"
GEOADD drivers longitude latitude driver_id
```

---

## API Endpoints

### 1. Request Ride

```id="ride_req"
POST /ride/request
```

Request:

```json id="ride_req_json"
{
  "userId": "123",
  "pickup": [28.6, 77.2]
}
```

---

### 2. Update Driver Location

```id="driver_loc"
POST /driver/location
```

---

### 3. Driver Availability

```id="driver_status"
POST /driver/status
```

---

### 4. Ride Status

```id="ride_status"
GET /ride/{rideId}
```

---

## Redis Data Model

```id="ride_redis"
drivers → geo index
driver:{id} → status
ride:{id} → details
```

---

## Kafka Topics

```id="ride_kafka"
ride-request-topic
driver-update-topic
ride-assignment-topic
```

---

## Matching Algorithm

### Step 1:

Find nearby drivers

### Step 2:

Sort by:

```id="ride_sort"
distance + availability + load factor
```

### Step 3:

Assign best driver

---

## Dynamic Pricing Logic

```id="ride_price"
price = base_fare * demand/supply ratio
```

---

## Scalability Strategy

* Stateless API → horizontal scaling
* Redis → fast location queries
* Kafka → async processing
* Partition drivers by region

---

## Performance Optimization

* Redis GEO indexing → O(log N)
* Caching driver state
* Async event handling via Kafka

---

## Failure Handling

| Scenario             | Handling                    |
| -------------------- | --------------------------- |
| Driver rejects       | Reassign next driver        |
| No drivers           | expand radius               |
| High load            | throttle requests           |
| Duplicate assignment | controlled via state checks |

---

## Deployment

### Docker

```bash id="ride_docker"
docker-compose up
```

---

### AWS

* Deploy services on EC2
* Use load balancer
* Scale based on traffic

---

## Observability

* Logs:

  * ride requests
  * assignments
* Metrics:

  * latency
  * match success rate

---

## How to Run Locally

```bash id="ride_run"
docker-compose up
```

---

## Future Enhancements

* ML-based driver prediction
* Route optimization
* ETA estimation
* Multi-city scaling

---

## Key Learnings

* Real-time system design
* Geospatial querying
* Event-driven architecture
* Performance optimization under load

