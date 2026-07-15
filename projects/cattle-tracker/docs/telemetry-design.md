# Telemetry Design

## Sensing

| Signal | Source | Resolution | Notes |
|---|---|---|---|
| Position | LR1110 GNSS scan | ~meters (network-solved) | WiFi-scan fallback near structures |
| Body temperature | Contact probe via Grove (DS18B20 class) | 0.01 °C encoded | Prototype stand-in for ear tag / rumen bolus |
| Ambient temp + humidity | Grove SHT4x-class sensor | 0.01 °C / 0.5 %RH | Heat-stress detection paired with body temp |
| Activity index | Onboard 3-axis accelerometer | 0–255 | Movement energy aggregated on-device between uplinks |
| Battery | ADC | ~20 mV steps | Fleet health |

Exact part numbers are confirmed against hardware on hand during Phase 2.

### Activity index

The accelerometer is sampled at low rate continuously (it is also the wake source for motion-triggered reporting). Between uplinks the firmware accumulates a simple movement-energy metric (mean absolute deviation of acceleration magnitude), scaled to 0–255. The raw stream never leaves the device — only the index does. Rumination/gait classification is a later refinement; the index alone already separates grazing / resting / distressed-running.

## Adaptive uplink cadence

Battery life is bought with silence. The policy:

| State | Trigger | Cadence |
|---|---|---|
| Resting | activity index below threshold | every 30 min |
| Grazing / moving | sustained motion detected | every 10 min |
| Alert-worthy motion | high activity burst (running) | every 2 min while it lasts |

All three values are configuration, not constants — they are the knobs that trade battery life against freshness, and the [power budget](power-budget.md) quantifies each setting's cost.

A fix that fails (no sky view) reports the last known position with a **staleness flag** rather than nothing: for a geofence system, "old position" and "no position" are different facts.

## Payload layout (16 bytes, port 2)

Little-endian, packed:

| Bytes | Field | Type | Encoding | Range |
|---|---|---|---|---|
| 0–3 | Latitude | int32 | degrees × 1e7 | ±90° |
| 4–7 | Longitude | int32 | degrees × 1e7 | ±180° |
| 8–9 | Body temp | int16 | °C × 100 | −327 … +327 °C |
| 10–11 | Ambient temp | int16 | °C × 100 | −327 … +327 °C |
| 12 | Humidity | uint8 | %RH × 2 | 0–100 % in 0.5 steps |
| 13 | Activity index | uint8 | 0–255 | — |
| 14 | Battery | uint8 | (mV − 2000) / 10 | 2.0–4.55 V |
| 15 | Flags | uint8 | bitfield | see below |

Flags: bit 0 = GNSS fix valid, bit 1 = position stale (last known), bit 2 = body probe fault, bit 3 = WiFi-scan position (not GNSS), bits 4–7 reserved.

16 bytes fits every US915 data rate including DR0 (11-byte limit excepted — DR0 uplinks drop to a 11-byte reduced form: position + flags only). The codec handles both forms.

## Codec

A JavaScript codec (ChirpStack device profile) decodes this layout to named JSON fields. It is version-controlled next to the firmware with **unit-tested encode/decode round-trips** — known byte vectors in, expected values out — runnable without hardware, so payload regressions are caught in CI, not in the pasture.
