# 📄 MDS Provider API Integration – Design Document

## 🧭 Overview

This document outlines the architecture, design decisions, and implementation strategy for integrating the [Mobility Data Specification (MDS)](https://github.com/openmobilityfoundation/mobility-data-specification) **Provider API** within our Kiwibot delivery robot platform. This API will enable regulatory partners and cities to query operational data in real-time, enhancing transparency and urban mobility coordination.

---

## 🎯 Goals

- Implement MDS-compliant Provider API (version: `2.0`) with the following core endpoints as a starting point:
  - `GET /vehicles`
  - `GET /status_changes`
  - `GET /trips`
- Expose data securely using OAuth2 with Client Credentials grant
- Ensure API responses conform to [OpenMobilityFoundation’s OpenAPI schema](https://github.com/openmobilityfoundation/mds-openapi)
- Provide city agencies a real-time and secure window into our active fleet and completed trips

---

## 🧩 System Architecture

### 🖼️ High-Level Diagram

```text
+---------------------------+
| Fleet Management System  |
|  (Vehicle + Trip Data)   |
+------------+-------------+
             |
             v
+---------------------------+
|  MDS Data Adapter Layer   | <--- Translates internal data to MDS format
+------------+--------------+
             |
             v
+---------------------------+           +----------------------------+
|     Provider API Server   | <-------->|   OAuth2 Auth Server       |
|  (/vehicles, /trips, etc) |           | (Client Credentials Grant) |
+------------+--------------+           +----------------------------+
             |
             v
    +---------------------+
    | City or Regulator   |
    | (Authorized Clients)|
    +---------------------+
```

---

## 🔐 Security Model

### Authentication
- **OAuth 2.0** Client Credentials flow
- City agencies will be issued client credentials
- Tokens used in `Authorization: Bearer <token>` headers

### Authorization
- Role-based access control per client (if required)
- Rate limiting and access logs for monitoring

---

## 📊 Data Mapping

### Vehicles (`GET /vehicles`)

| MDS Field         | Internal Field       | Notes                          |
|-------------------|----------------------|--------------------------------|
| `vehicle_id`      | `robot_id`           | Unique ID per device           |
| `vehicle_type`    | `"robot"`            | Static or mapped               |
| `current_status`  | `availability_state` | Mapped enum                    |
| `propulsion_type` | `["electric"]`       | Static                         |
| `last_updated`    | `status_timestamp`   | ISO 8601, 'last update'        |
| `current_location`| `telemetry.gps`      | lat/lng from telemetry/location|

---

### Status Changes (`GET /status_changes`)

| MDS Field         | Internal Field      |
|-------------------|---------------------|
| `event_type`      | `job tag from remi` |
| `event_time`      | `event_timestamp`   |
| `vehicle_id`      | `robot_id`          |
| `event_location`  | `location.gps`      |
| `associated_trip` | `step_id` (nullable)|

---

### Trips (`GET /trips`) 

Candidate table: kiwibot-atlas.mds.trips
Pipeline: Not deployed

| MDS Field         | Internal Field             |
|-------------------|----------------------------|
| `trip_id`         | `step_id`                  |
| `vehicle_id`      | `robot_id`                 |
| `start_time`      | `step.start_time`          |
| `end_time`        | `step.end_time`            |
| `start_location`  | `step.point.start.lat/lng` |
| `end_location`    | `trip.point.end.lat/lng`   |
| `trip_distance`   | `distance_meters`          |
| `trip_duration`   | Derived (`end - start`)    |

---

## 🚧 Development Plan

| Phase       | Deliverable               | Owner        | Timeline  |
|-------------|---------------------------|--------------|-----------|
| Phase 1     | Spec Review, Data Mapping | Data?        | Week 1    |
| Phase 2     | Stub API + OAuth Setup    | IT?          | Week 2    |
| Phase 3     | Endpoint Implementation   | Data or IT?  | Week 3–4  |
| Phase 4     | Validation & Testing      | Data         | Week 5    |
| Phase 5     | Pilot with City Partner   | Data         | Week 6    |

---

## ✅ OpenAPI & Compliance Strategy

- Use the [mds-openapi spec](https://github.com/openmobilityfoundation/mds-openapi) as source of truth
- Validate API using:
  - `Speccy`
  - `openapi-cli`
  - Custom test suite with schema assertions
- Schema validation run in CI pipeline

---

## 📈 Monitoring & Logging

- OAuth access logging
- Endpoint-level request/response logging
- Alerting on:
  - Token auth failures
  - Field validation issues
  - Latency thresholds

---

## 🧪 Testing Plan

- Unit tests for data transformers
- Integration tests with simulated city client
- OpenAPI schema validation tests
- Load tests for `/vehicles` endpoint

---

## 🚀 Future Considerations

- Support `/events`, `/telemetry` in future phases
- Add webhook notifications to cities
- Fine-grained access controls by geography or fleet subset
- Compliance dashboard for internal ops team

---

## 📎 References

- [Mobility Data Specification (Main Repo)](https://github.com/openmobilityfoundation/mobility-data-specification)
- [MDS OpenAPI Spec](https://github.com/openmobilityfoundation/mds-openapi)
- [Stoplight Docs](https://openmobilityfnd.stoplight.io/)
