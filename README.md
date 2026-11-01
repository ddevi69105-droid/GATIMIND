# GATIMIND
“Intelligence in Every Mile.

---

## 1. Executive Summary & The Problem

### Traditional Tracking vs. Reality
Standard bus-tracking solutions merely draw a dot on a map. In reality:
* Students still waste 20–30 minutes waiting at stops in bad weather because static schedules ignore live traffic jams.
* Students wave their hands to flag down buses only to be rejected because the bus is already 100% full.
* Drivers waste time and fuel taking detours to empty stops where zero students are waiting.
* In case of a breakdown or unexpected route change, chaos ensues with panicked phone calls to drivers while driving.

### The Innovation
**CommuteSync** transforms reactive transit ("wait and wave") into a **proactive, demand-driven ecosystem**:
1. **Dynamic Walk-Sync Alarm:** Triggers an alarm only when `Bus ETA == Student Walking Time + Buffer`, guaranteeing zero roadside wait time.
2. **Predictive Occupancy Engine:** Informs riders stops in advance whether seats or standing room are available before they ever step outside.
3. **Demand-Driven Dynamic Routing (Smart Skip):** Lets drivers bypass empty peripheral stops automatically, saving fuel and transit time.
4. **Safety & Route Deviation Intelligence:** Automatically detects unauthorized path departures and triggers security alerts.

---

## 2. Core Modules & Innovative Features

### A. Student Experience (Mobile App)
* **Smart Synchronized Alarm:**
  * Uses the student's live geolocation to calculate their pedestrian walking time to the stop.
  * If the bus is 12 minutes away and the student is a 6-minute walk away, the alarm fires precisely at 7 minutes prior (giving a 1-minute buffer to grab their bag).
* **Live Bus Occupancy Meter:**
  * Displays color-coded capacity indicators:
    * 🟢 Green: Seats available (> 25% free)
    * 🟡 Amber: Standing room only (< 25% seated)
    * 🔴 Red: At maximum capacity (suggests alternate backup bus)
* **Single-Tap "I Am Boarding Today":**
  * Students check in with one tap 15–30 minutes before arrival to register demand for their designated stop.
* **Safety Escort & Silent SOS:**
  * Displays emergency SOS button for late evening shifts.
  * Real-time trip status sharing with parents or emergency contacts.

### B. Driver Console (Low-Distraction Dashboard)
* **Zero-Touch Audio Navigation Cues:**
  * Rather than demanding screen interaction, voice prompts announce: *"Next stop: Sector 14. 4 students waiting. Proceed."* or *"Stop 6 has 0 passengers. Bypass via Expressway."*
* **Single-Tap Incident Broadcast:**
  * Dedicated buttons for `Flat Tyre`, `Traffic Jam`, `Breakdown`, or `Route Closed`. Instantly notifies all assigned riders and dispatch admins.
* **Driver Smartphone as GPS Beacon:**
  * No expensive onboard hardware required. The driver's standard Android phone acts as the high-accuracy location publisher.

### C. Campus Transport Admin Portal (Web Dashboard)
* **Fleet Telematics & Live Map:**
  * Birds-eye view of all buses, current speeds, delay indexes, and fuel consumption metrics.
* **Dynamic Route Analytics & Optimization:**
  * Identifies consistently low-demand stops to recommend permanent route shortening.
* **Green Campus & Carbon Offset Reports:**
  * Tracks total bus rides taken vs. equivalent private two-wheeler/cab rides to provide verified data for NAAC/NIRF sustainability metrics.

---

## 3. System Architecture & Tech Stack

```
+-------------------------------------------------------------+
|                     Client Tier                             |
|  [Student Flutter App]             [Driver Flutter App]     |
+-------------------------------------------------------------+
            │ (WebSockets / REST)               │ (MQTT / WS)
            ▼                                   ▼
+-------------------------------------------------------------+
|                      API Gateway                            |
|             (Reverse Proxy / Rate Limiter)                  |
+-------------------------------------------------------------+
            │                                   │
            ▼                                   ▼
+───────────────────────────+       +─────────────────────────+
|   Core Backend (Node/Go)  |       | Telemetry Ingestion     |
|   • Auth & Student Intent |       | • GPS Coordinate Stream |
|   • Geofence Walk Alarms  |       | • Adaptive Dead Reckon  |
+───────────────────────────+       +─────────────────────────+
            │                                   │
            ▼                                   ▼
+─────────────────────────────────────────────────────────────+
|                     Data & Messaging Tier                   |
|  • PostgreSQL / PostGIS (Relational & Spatial Geometries)   |
|  • Redis (Pub/Sub & Real-time Location Caching)             |
|  • RabbitMQ / MQTT Broker (High-throughput driver telemetry)|
+─────────────────────────────────────────────────────────────+
            │
            ▼
+─────────────────────────────────────────────────────────────+
|                    External Services                        |
|  • Mapbox / OSRM (Matrix API for Live ETA & Walking Times)  |
|  • Firebase Cloud Messaging (FCM for Priority Push Alarms)  |
+─────────────────────────────────────────────────────────────+
```

### Detailed Tech Stack

| Layer | Recommended Technology | Why Chosen? |
| :--- | :--- | :--- |
| **Mobile Frontends** | Flutter (Dart) or React Native | Single codebase for Android & iOS; fast UI rendering for maps. |
| **Admin Dashboard** | Next.js (React) + Tailwind CSS | Fast server-side rendering, responsive real-time data tables. |
| **Telemetry Server** | Go (Golang) or Node.js (NestJS) | Extremely low latency and high concurrency handling for continuous GPS feeds. |
| **Database (Spatial)** | PostgreSQL + PostGIS | Native support for spatial queries, geofencing polygons, and distance calculations. |
| **Real-time Cache** | Redis | In-memory storage for high-speed bus coordinate lookups (`GEOADD`, `GEORADIUS`). |
| **Routing & Distance Engine** | OSRM (Open Source Routing Machine) or Mapbox API | Accurate walking & driving matrices without extreme Google Maps API pricing. |
| **Push & Alerts** | Firebase Cloud Messaging (FCM) + APNs | Reliable wake-up signals for background devices even when apps are closed. |

---

## 4. How the "Dynamic Walk-Sync Alarm" Works (Algorithm)

The alarm logic relies on dynamic synchronization:

$$T_{\text{alarm}} \iff \text{ETA}_{\text{bus}}(\text{Stop}) \le \text{ETA}_{\text{walk}}(\text{Student} \to \text{Stop}) + T_{\text{buffer}}$$

Where:
* $\text{ETA}_{\text{bus}}(\text{Stop})$: Real-time driving duration from the bus's current location to the target stop, factoring in live traffic speed.
* $\text{ETA}_{\text{walk}}(\text{Student} \to \text{Stop})$: Pedestrian duration calculated using the student's live coordinates.
* $T_{\text{buffer}}$: Configurable buffer time (e.g., 60–120 seconds for grabbing personal belongings).

```
[Start Background Monitor]
        │
        ▼
Fetch Bus GPS (Redis) ──► Compute Bus ETA to Stop
        │
        ▼
Fetch Student GPS ─────► Compute Student Walk ETA to Stop
        │
        ▼
Is (Bus ETA <= Walk ETA + Buffer)?
     ├── NO  ──► Sleep 30 seconds & Re-check
     └── YES ──► Trigger Critical Foreground Alarm & Vibration
```

---

## 5. Step-by-Step Implementation Roadmap

### Phase 1: Prototype / MVP (Weeks 1–3)
- [ ] Set up driver app to transmit GPS coordinates every 5 seconds to a Node/Redis backend.
- [ ] Build student app with live map rendering using Mapbox/Leaflet.
- [ ] Implement fixed geofencing notifications (1 km radius alert).

### Phase 2: Intelligence & Innovation Layer (Weeks 4–6)
- [ ] Integrate OSRM/Mapbox Matrix API for walk-time vs. drive-time calculations.
- [ ] Implement the single-tap "Boarding Today" check-in system.
- [ ] Deploy dynamic skip logic: alert driver console when a stop has 0 check-ins.
- [ ] Add capacity estimator based on student boarding counts.

### Phase 3: Field Testing & Production Hardening (Weeks 7–8)
- [ ] Conduct field trials on 2–3 active campus buses.
- [ ] Optimize battery consumption on the driver app using adaptive GPS polling (slower polling when stationary, faster when moving).
- [ ] Finalize the campus admin portal and carbon savings dashboard.

---

## 6. How to Pitch This to Judges & Colleges

* **To College Management:** "This saves 10–15% in fleet fuel costs via automated skip-routing and provides institutional transit transparency for accreditation."
* **To Students:** "Never freeze or stand under the scorching sun for 30 minutes again. Step out of your room exactly when the bus is arriving."
* **To Hackathon Judges:** "This isn't a passive GPS wrapper. It's an intelligent edge-dispatch system combining spatial analytics, dynamic pedestrian synchronization, and predictive load balancing."
