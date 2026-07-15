# Data Pipeline — MQTT → InfluxDB 3

**Status:** designed (Phase 0); implementation lands in Phases 2–3.

## Shape

```
ChirpStack MQTT integration (decoded uplinks, JSON)
        │
        ▼
HiveMQ CE (MQTT 5.0)
        │  subscribe
        ▼
Ingestion service (Python)
  ├── validate + flatten payload
  ├── geofence evaluation (Phase 3)
  ├── alert events → ntfy / Telegram (Phase 3)
  └── line protocol → InfluxDB 3
```

## Planned contents

- [ ] Ingestion service source + tests (replayable captured-uplink fixtures — no radio needed to test)
- [ ] InfluxDB measurement schema (`location`, `telemetry`, `device_health`)
- [ ] Geofence storage and evaluation module
- [ ] Grafana geomap dashboard JSON

Idempotency: writes are keyed by device EUI + frame counter; malformed payloads are logged and dropped, never crash the consumer.
