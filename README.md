![Platform architecture](images/architecture.svg)

<h1 align="center">LoRaWAN Projects</h1>
<p align="center"><em>Low-power, long-range tracking and sensing for assets that live far from WiFi and power outlets.<br>Built on a private, self-hosted LoRaWAN network.</em></p>

<p align="center">
  <a href="projects/cattle-tracker/"><img src="https://img.shields.io/badge/cattle%20tracker-in%20development-2ea44f?style=flat-square" alt="Cattle tracker: in development"></a>
  <a href="#why-lorawan"><img src="https://img.shields.io/badge/range-miles%20(rural)-1f6feb?style=flat-square" alt="Range: miles (rural)"></a>
  <a href="#why-lorawan"><img src="https://img.shields.io/badge/battery%20life-years-1f6feb?style=flat-square" alt="Battery life: years"></a>
  <a href="#why-lorawan"><img src="https://img.shields.io/badge/monthly%20fees-none-2ea44f?style=flat-square" alt="Monthly fees: none"></a>
  <a href="platform/gateway/"><img src="https://img.shields.io/badge/band-US915-6e7781?style=flat-square" alt="Band: US915"></a>
</p>
<p align="center">
  <img src="https://img.shields.io/badge/LoRaWAN-private%20network-1c3c7a?style=flat-square" alt="LoRaWAN">
  <img src="https://img.shields.io/badge/ChirpStack-LNS-0b6e4f?style=flat-square" alt="ChirpStack">
  <img src="https://img.shields.io/badge/K3s-bare--metal-ffc61c?style=flat-square&logo=k3s&logoColor=white" alt="K3s bare-metal">
  <img src="https://img.shields.io/badge/HiveMQ%20CE-MQTT%205.0-ffc000?style=flat-square&logo=mqtt&logoColor=white" alt="HiveMQ CE MQTT 5.0">
  <img src="https://img.shields.io/badge/InfluxDB-3-22adf6?style=flat-square&logo=influxdb&logoColor=white" alt="InfluxDB 3">
  <img src="https://img.shields.io/badge/Grafana-Geomap-f46800?style=flat-square&logo=grafana&logoColor=white" alt="Grafana Geomap">
  <img src="https://img.shields.io/badge/Cloudflare-Tunnel-f38020?style=flat-square&logo=cloudflare&logoColor=white" alt="Cloudflare Tunnel">
  <a href="LICENSE"><img src="https://img.shields.io/github/license/jacedeno/lorawan-projects?style=flat-square" alt="License"></a>
  <img src="https://img.shields.io/github/last-commit/jacedeno/lorawan-projects?style=flat-square" alt="Last commit">
</p>

## Overview

**Low-power, long-range tracking and sensing for assets that live far from WiFi and power outlets** — built on a private, self-hosted LoRaWAN network.

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

## Architecture

The platform diagram is shown at the top of this page. Layer by layer:

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

MIT — see [LICENSE](LICENSE).

---

<p align="center">
  Built by <b>Jose Cedeno</b> ·
  <a href="https://geekendzone.com">geekendzone.com</a> ·
  <a href="https://www.linkedin.com/in/joseangelcedeno/">LinkedIn</a>
</p>
