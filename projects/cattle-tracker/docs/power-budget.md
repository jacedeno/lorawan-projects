# Power Budget

Battery life is the product economics of a tracking collar: it sets the service interval, and the service interval sets the cost of ownership per animal. This budget makes the energy cost of every design knob explicit — first as datasheet-based estimates, then (Phase 2) as measured values.

## Energy model

Each reporting cycle spends energy on four things:

```
E_cycle = E_gnss_scan + E_sensor_read + E_lora_tx + E_rx_windows
E_day   = N_cycles × E_cycle + 24 h × I_sleep
```

## Assumed numbers (datasheet-class, to be measured)

| Item | Assumption | Basis |
|---|---|---|
| GNSS scan | 30 mA × 3 s | LR1110 scan capture (position solved server-side) |
| LoRa TX | 120 mA × 0.4 s | +20 dBm, worst case DR0 airtime for this payload |
| RX windows | 5 mA × 1 s | Class A RX1/RX2 |
| Sensor read + MCU wake | 8 mA × 1 s | probe conversion dominates |
| Sleep floor | 10 µA | nRF52840 + LR1110 sleep + regulator quiescent |
| Battery | 3000 mAh (18650 Li-ion class) | prototype pack |

Per cycle: ≈ 0.045 mAh. Sleep: ≈ 0.24 mAh/day.

## Scenarios

| Reporting cadence | Cycles/day | mAh/day | Raw estimate | With 50% derating |
|---|---|---|---|---|
| Every 30 min (resting baseline) | 48 | ≈ 2.4 | ~3.4 years | **~1.7 years** |
| Every 10 min (grazing) | 144 | ≈ 6.7 | ~1.2 years | **~7 months** |
| Every 2 min (alert bursts, sustained) | 720 | ≈ 32.6 | ~3 months | **~6 weeks** |

The **50% derating** covers what datasheets don't: failed scans that retry, join overhead, temperature effects on the cell, self-discharge, and real antennas in real collars. Until Phase 2 replaces assumptions with current-probe measurements, the derated column is the honest one.

Two design consequences follow directly:

1. **The adaptive cadence is the battery.** A herd resting 16 h/day at 30-min cadence and grazing 8 h at 10-min cadence lands around a year of life — acceptable for an annual service cycle. A fixed 2-minute cadence would be a 6-week collar, i.e., a product failure.
2. **GNSS scan offload is the enabling choice.** A conventional GPS receiver needing 30 s to first fix at ~25 mA would spend ~5× more energy per position than the LR1110's capture-and-forward scan. This is why the radio choice *is* the product choice.

## What Phase 2 measures

- Actual current profiles per cycle (current probe on the dev board): scan, TX at each data rate, sleep floor
- GNSS scan success rate and duration in open pasture vs. tree cover
- Real cadence mix from the activity-driven policy during drive tests

The table above then gets a "measured" column, and the derating factor gets replaced by data.
