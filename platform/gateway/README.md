# LoRaWAN Gateway — WM1302 on Raspberry Pi

**Status:** Phase 1 (in progress). This directory will hold the full gateway build: hardware assembly, OS provisioning, packet-forwarder configuration, and antenna notes.

## Hardware

- **Seeed WM1302** concentrator module — Semtech SX1302 baseband, SPI variant, **US915**
- **WM1302 Raspberry Pi HAT** (official Seeed HAT)
- Raspberry Pi host

The SX1302 demodulates 8 channels concurrently: a real multi-device LoRaWAN gateway, not a single-channel test forwarder.

## Planned contents

- [ ] Assembly and enable-SPI provisioning steps (reproducible, headless)
- [ ] Semtech UDP packet forwarder build and `global_conf.json` (US915 sub-band, ChirpStack target)
- [ ] systemd unit + watchdog
- [ ] Antenna placement and coverage notes from drive tests
