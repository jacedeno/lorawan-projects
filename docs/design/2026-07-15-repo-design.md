# LoRaWAN Projects — Repository Design

**Date:** 2026-07-15
**Status:** Approved

## Overview

This repository hosts a family of LoRaWAN projects built on a shared, self-hosted platform: a private LoRaWAN network (gateway + network server) feeding an MQTT → time-series → dashboard pipeline. The flagship project is a **cattle tracker** — a GPS collar prototype that reports animal location, body temperature, ambient conditions, and activity, with server-side geofencing ("virtual fence" monitoring).

The repository is public and documentation-first: every phase leaves it complete and presentable at its current level of depth. The root README is written in layers so that a non-technical reader gets the story in 30 seconds while an engineer can drill down to manifests, firmware, and design docs.

## Goals

- Demonstrate an end-to-end LoRaWAN capability: device firmware → radio network → network server → data pipeline → dashboards and alerts, all on self-managed infrastructure.
- Ship a working cattle-tracker prototype: GPS position, body temperature (contact probe), ambient temperature/humidity, activity index, adaptive uplink cadence, and geofence alerts.
- Document the engineering decisions that drive product economics: power budget, payload design, reporting cadence, network trade-offs (LoRaWAN vs cellular/WiFi/BLE).

## Non-Goals

- Animal-control stimulus (audio/electric cues). Commercial virtual-fence collars control the animal; this project implements the shared technology base — tracking, geofence evaluation, and alerting.
- Production collar hardware (enclosure, harness, ruggedization, battery pack engineering). The prototype uses a dev board and says so.
- Multi-region support. US915 only.

## Repository Structure

```
lorawan-projects/
├── README.md                    # layered: story → business case → architecture
├── docs/
│   ├── architecture.md          # platform architecture, end-to-end
│   ├── roadmap.md               # phases and current status, kept honest
│   └── design/                  # dated design documents (this file)
├── platform/                    # shared infrastructure for all projects
│   ├── gateway/                 # WM1302 SPI module + Raspberry Pi HAT (US915)
│   ├── chirpstack/              # ChirpStack on K3s: manifests + migration notes
│   └── pipeline/                # ChirpStack MQTT → ingestion → InfluxDB 3
└── projects/
    └── cattle-tracker/          # flagship project
        ├── README.md            # problem, solution, status
        ├── docs/                # sensing design, payload spec, power budget, geofencing
        ├── firmware/            # Wio Tracker 1110 (PlatformIO)
        └── dashboard/           # map + telemetry (later phase)
```

Future projects from the same platform (e.g., a hiker/biker tracker, environmental monitoring) land under `projects/` without restructuring.

## Platform Architecture

```
Wio Tracker 1110 ──LoRaWAN US915──> WM1302 gateway (Raspberry Pi)
(GNSS + temp ×2 + accelerometer)          │  Semtech UDP / Basics Station
                                          ▼
                                  ChirpStack (K3s)
                                          │  MQTT integration
                                          ▼
                            MQTT broker → ingestion → InfluxDB 3
                                                          │
                                                          ▼
                                        Dashboard (map, geofence, alerts)
```

- **Network server:** ChirpStack, self-hosted on a bare-metal K3s cluster. Postgres is provisioned through the CloudNativePG operator; Redis runs alongside. All manifests are version-controlled in `platform/chirpstack/`. A previous ChirpStack deployment ran on Docker Compose; the migration to K3s is documented as an architecture decision (prototype → production pattern).
- **Gateway:** Seeed WM1302 concentrator module (SX1302, SPI, US915) on the official Raspberry Pi HAT. Build, OS provisioning, packet-forwarder configuration, and antenna notes documented in `platform/gateway/`.
- **MQTT broker:** HiveMQ CE (MQTT 5.0) — the same broker class already proven in prior telemetry work. Deployment target and connection details are documented in `platform/pipeline/`.
- **Pipeline:** ChirpStack's MQTT integration publishes decoded uplinks; a small Python ingestion service (MQTT subscriber) writes them to InfluxDB 3. Geofence evaluation (Phase 3) lives in this service. The design follows proven patterns from prior ingestion work and is documented as shared infrastructure for all projects in this repo.
- **Alerting:** geofence and telemetry alerts ride existing notification infrastructure (ntfy/Telegram).

## Cattle Tracker Design

### Sensing

| Signal | Source | Purpose |
|---|---|---|
| Position | LR1110 GNSS scan (GPS/BeiDou) | Location, movement, geofence evaluation |
| Body temperature | Contact probe via Grove (DS18B20 class) | Health indicator; prototype stand-in for a collar/ear-tag contact sensor |
| Ambient temp + humidity | Grove SHT4x-class sensor | Heat-stress detection when combined with body temperature |
| Activity index | Onboard 3-axis accelerometer | Movement/rest classification; drives adaptive reporting |
| Battery voltage | ADC | Fleet health, TCO evidence |

Exact sensor part numbers are confirmed against hardware on hand during Phase 2 and recorded in `projects/cattle-tracker/docs/`.

### Telemetry strategy

- **Adaptive uplink cadence:** slow baseline reporting at rest (e.g., every 15–30 min), faster when the accelerometer detects sustained movement. The cadence policy and its battery impact are documented, not hard-coded magic numbers.
- **Compact binary payload** (~15–20 bytes): packed lat/lon, two temperatures, activity index, battery. A matching JavaScript payload codec is installed in ChirpStack and version-controlled with tests/examples.
- **Power budget document:** per-event energy cost (GNSS scan, LoRa TX, sleep current) and battery-life projections per reporting scenario. This is the engineering artifact that connects directly to product economics: battery life drives cost of ownership per animal.

### Geofencing ("virtual fence" monitoring)

- Geofences are polygons stored server-side.
- Each incoming position is evaluated (point-in-polygon) in the pipeline; a fence exit raises an alert through the existing notification stack.
- The docs state plainly how this relates to commercial virtual fencing: same tracking/geofence foundation, without the animal-stimulus layer.

### Dashboard

- **Phase 3 initial version: Grafana geomap** — live positions, movement traces, geofence overlay, temperature and activity panels. Zero custom code; reuses the existing observability stack.
- A custom map application (Next.js + MapLibre) is an optional later phase, only if it adds something Grafana cannot show.

## Roadmap

| Phase | Deliverable | Repo state when done |
|---|---|---|
| 0 | Root README (layered), `docs/architecture.md`, cattle-tracker design docs (sensing, payload, power budget, geofencing), honest roadmap | Complete as a documented design; shareable |
| 1 | WM1302 gateway operational; ChirpStack live on K3s with public manifests and migration notes | Private LoRaWAN network running |
| 2 | Wio Tracker 1110 firmware: GNSS + both temperatures + activity, payload codec, uplinks landing in InfluxDB | Real telemetry end-to-end |
| 3 | Geofence evaluation + alerts + Grafana geomap dashboard | Full "virtual fence monitoring" demo |

`docs/roadmap.md` always reflects the true current phase — no aspirational claims presented as done.

## Conventions

- **Language:** English everywhere — docs, code, comments, commit messages.
- **Commits:** conventional commits (`feat:`, `docs:`, `fix:`). No attribution trailers of any kind.
- **Secrets:** none in git, ever. Device EUIs/keys in examples are placeholders; a gitleaks scan gates the repo going public and precedes any push after that.
- **License:** MIT.
- **Honesty:** prototype status, known limitations, and open questions are documented as such.

## Testing

- Payload codec: unit-tested encode/decode round-trips (known byte vectors ↔ decoded values), runnable without hardware.
- Firmware: bench validation checklist per phase (GNSS fix acquisition, sensor reads, uplink received by ChirpStack, decoded payload correctness).
- Field validation: a vehicle-mounted mobile test station (tracker + 12 V power + antennas) stands in for a grazing animal — drive tests validate range, GNSS tracking quality, and geofence transitions in real terrain. Documented with photos in `projects/cattle-tracker/docs/`.
- Pipeline: a replayable sample-uplink fixture (captured MQTT messages) exercises ingestion and geofence evaluation without radio hardware.
- End-to-end acceptance per phase is defined in `docs/roadmap.md` (e.g., Phase 2 exit: a mapped fix from a walk-test appears in InfluxDB with correct temperatures).

## Error Handling & Resilience

- Device: missed-join and failed-uplink behavior (retry/backoff), GNSS-fix timeout policy (report last-known-fix with a staleness flag rather than nothing).
- Pipeline: ingestion is idempotent per (device EUI, frame counter); malformed payloads are logged and dropped, never crash the consumer.
- Gateway/backhaul outages: LoRaWAN Class A devices simply keep transmitting; the docs call out what is lost during an outage and why that is acceptable for this use case (position telemetry, not safety control).
