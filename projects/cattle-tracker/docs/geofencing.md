# Geofencing — the "Virtual Fence" Layer

## Where this sits

Commercial virtual-fence systems (collar cues that steer animals) are three layers:

1. **Tracking** — know where every animal is
2. **Fence logic** — know when one crosses a boundary
3. **Control** — cue the animal back (audio/electric)

This project implements layers 1 and 2 — the data foundation the control layer depends on. Control is deliberately out of scope: it is an animal-welfare and regulatory domain of its own, and it changes nothing about the telemetry architecture underneath.

## Fence model

- A fence is a **named polygon** (list of lat/lon vertices) stored server-side, with an animal/group assignment and an active flag.
- Multiple fences can be active per animal (paddock + hazard exclusion zones).
- Fences are data, not firmware: moving a paddock boundary is an edit in the backend, touching zero devices — the whole point of *virtual* fencing for rotational grazing.

## Evaluation

Runs in the ingestion service, on every position uplink:

1. Discard/flag stale positions (device sets a staleness bit when reporting last-known-fix).
2. Point-in-polygon test (ray casting) against each active fence for that animal.
3. Compare with the animal's previous state → emit `fence_exit` / `fence_entry` events on transitions only.
4. Persist the event and current state; forward alert-worthy events to notifications.

### Honest limits of point-in-time evaluation

With a 10-minute cadence, an animal can be outside for up to 10 minutes before the system can possibly know. Three mitigations, all already in the design:

- **Motion-triggered cadence:** sustained movement (the thing that precedes most escapes) already promotes reporting to the faster tier.
- **Hysteresis / debounce:** a single fix a few meters over the line (GNSS noise) does not alert; two consecutive outside fixes, or one fix beyond a buffer distance, does. The buffer width is a config knob derived from measured GNSS scatter (Phase 2 data).
- **Silent-device alert:** a collar that stops reporting is itself an event — "no data" is never silently treated as "all good."

## Alert semantics

| Event | Condition | Delivery |
|---|---|---|
| Fence exit | debounced outside transition | immediate push (ntfy/Telegram) |
| Fence re-entry | debounced inside transition | informational |
| Heat stress | body temp + ambient/humidity over threshold profile | push |
| Low battery | battery below threshold | daily digest |
| Silent device | no uplink for 3× expected cadence | push |

Alert thresholds live in configuration with the same philosophy as the cadence policy: they are product decisions made visible, not magic numbers.

## Validation (Phase 3)

Drive tests with the vehicle-mounted station crossing a real fence polygon at walking, trotting, and vehicle speeds — the exit criterion is a notification within one uplink interval of the crossing.
