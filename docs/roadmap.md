# Roadmap

Honest status: what runs today, what is designed, what is planned. This file is updated as phases complete — nothing below is claimed as done unless its exit criteria passed.

**Lineage:** this platform is a rebuild of an earlier working prototype of the same system — device, network, and dashboard validated end to end on a Docker-based stack. The phases below track bringing it back online properly: updated software stacks, the K3s platform, and re-integrated hardware.

## Phase 0 — Design & documentation ✅ (complete 2026-07-15)

**Goal:** the platform and flagship project fully designed and documented before hardware is wired.

- [x] Repository design document ([docs/design/2026-07-15-repo-design.md](design/2026-07-15-repo-design.md))
- [x] Platform architecture ([docs/architecture.md](architecture.md))
- [x] Architecture diagram (SVG/PNG)
- [x] Cattle tracker: telemetry design, payload specification, power budget, geofencing design

## Phase 1 — Private LoRaWAN network 🚧 (current)

**Goal:** a device can join and its uplinks reach the platform.

- [ ] WM1302 gateway assembled and provisioned (Raspberry Pi + HAT, packet forwarder)
- [x] ChirpStack deployed on K3s (manifests in `platform/chirpstack/`) — v4.9.0, CNPG HA Postgres, web UI live behind Traefik
- [x] HiveMQ reachable by ChirpStack MQTT integration — connected and subscribed
- [x] Migration notes from the previous Docker Compose ChirpStack deployment — in `platform/chirpstack/README.md`

**Exit criteria:** a test device completes OTAA join; its uplink appears decoded on the MQTT broker.

## Phase 2 — Cattle tracker firmware 🔜

**Goal:** real telemetry end to end.

- [ ] Wio Tracker 1110 firmware: GNSS scan, body-temperature probe, ambient temp/humidity, activity index, battery
- [ ] Adaptive uplink cadence (motion-triggered)
- [ ] Payload codec installed in ChirpStack, with unit-tested encode/decode round-trips
- [ ] Ingestion service writing to InfluxDB 3
- [ ] Exact sensor part numbers confirmed and documented

**Exit criteria:** a walk/drive test produces mapped position fixes in InfluxDB with correct temperatures.

## Phase 3 — Virtual fence monitoring 🔜

**Goal:** the full demo — geofences, alerts, live map.

- [ ] Geofence storage and point-in-polygon evaluation in the ingestion service
- [ ] Alert delivery (geofence exit, heat stress, low battery, silent device)
- [ ] Grafana geomap dashboard: live positions, traces, fence overlays, telemetry panels
- [ ] Field validation with the vehicle-mounted mobile test station (drive tests across fence boundaries)

**Exit criteria:** crossing a fence boundary during a drive test raises a notification within one uplink interval.

## Later ideas (not committed)

- Custom map application (only if Grafana proves limiting)
- Additional projects on the same network: hiker/biker tracker, environmental monitors, water-level sensors
- LR1110 WiFi-scan positioning for indoor/barn coverage
