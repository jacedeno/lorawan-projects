# LoRaWAN Projects

**Low-power, long-range tracking and sensing for assets that live far from WiFi and power outlets** — built on a private, self-hosted LoRaWAN network.

![Platform architecture](images/architecture.svg)

## What this is

Some things worth monitoring don't sit in a building: grazing cattle, hikers on a trail, water tanks at the far end of a property. They have no WiFi, no power outlet, and often no cellular coverage worth paying for. This repository is a working platform for exactly those cases — small battery-powered devices that report location and sensor readings over **LoRaWAN**, a radio technology that trades bandwidth for what field assets actually need: **miles of range and years of battery life**.

Everything here runs on infrastructure I build and operate myself, end to end: the radio gateway, the network server, the message broker, the time-series database, and the dashboards. No cloud IoT platform required.

## Why LoRaWAN

Choosing a radio is a business decision before it is a technical one. For assets that are numerous, dispersed, and battery-powered, the trade-offs look like this:

| | LoRaWAN | Cellular (LTE-M/NB-IoT) | WiFi | BLE |
|---|---|---|---|---|
| Range | **Miles (rural)** | Carrier coverage | ~50 m | ~10–100 m |
| Battery life | **Years** | Months | Days | Months |
| Cost per device | **Low ($10–30 radios)** | Higher + SIM | Low | Low |
| Monthly fees | **None (private network)** | Per-SIM subscription | None | None |
| Payload size | Small (bytes) | Large | Large | Small |
| Best for | Dispersed, low-rate telemetry | High-rate mobile data | Indoor, powered | Short-range, wearables |

A position fix, two temperatures, and an activity index fit in ~16 bytes. Sending 16 bytes a few times an hour is precisely the workload LoRaWAN was designed for — and where WiFi and cellular are the wrong tool at any price.

## Projects

| Project | Status | Description |
|---|---|---|
| **[Cattle Tracker](projects/cattle-tracker/)** | 🚧 In development — design complete | GPS collar prototype: location, body & ambient temperature, activity, and geofence ("virtual fence") alerts |
| Hiker / biker tracker | 💡 Planned | Off-grid position reporting for outdoor recreation |
| Environmental monitors | 💡 Planned | Water level, soil, and weather nodes on the same network |

One network, many projects: every node added later reuses the same gateway, network server, and data pipeline.

## The platform

| Layer | Component | Notes |
|---|---|---|
| Device radio | Semtech LR1110 / SX126x class | LoRaWAN + GNSS/WiFi positioning |
| Gateway | Seeed WM1302 (SX1302) on Raspberry Pi | US915, 8 channels — [platform/gateway](platform/gateway/) |
| Network server | ChirpStack on bare-metal K3s | PostgreSQL via CloudNativePG — [platform/chirpstack](platform/chirpstack/) |
| Messaging | HiveMQ CE (MQTT 5.0) | ChirpStack integration publishes decoded uplinks |
| Ingestion | Python service | Decode, geofence evaluation, alerting — [platform/pipeline](platform/pipeline/) |
| Storage | InfluxDB 3 | Time series, SQL/Flight queries |
| Dashboards | Grafana | Geomap: live positions, geofences, telemetry |
| Access | Cloudflare Tunnel | Zero inbound ports |

Full detail in [docs/architecture.md](docs/architecture.md). Current status and next steps in [docs/roadmap.md](docs/roadmap.md). Working notes — including what broke and how it was fixed — in the [development log](docs/devlog.md).

## Related work

This platform reuses patterns from other systems I run in production:

- **Industrial vibration monitoring** — wireless sensor fleets at an operating cement plant, streaming over 4G/VPN into the same MQTT → InfluxDB → dashboard pipeline shape used here.
- **[Firefly IMU Dashboard](https://github.com/jacedeno/firefly-imu-dashboard)** — 9-DOF sensor fusion (Madgwick) streamed over BLE to a live 3D attitude display; the same accelerometer craft behind this project's activity sensing.
- **[M5Tough Launch Display](https://github.com/jacedeno/m5tough-firefly-launch)** — self-contained ESP32 firmware consuming a public REST API on constrained hardware.

## License

MIT
