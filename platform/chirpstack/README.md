# ChirpStack on K3s

**Status:** Phase 1 (in progress). This directory will hold version-controlled manifests for the LoRaWAN Network Server deployment.

## Planned contents

- [ ] ChirpStack deployment manifests (K3s)
- [ ] PostgreSQL cluster via the CloudNativePG operator
- [ ] Redis (device sessions / dedup state)
- [ ] MQTT integration configuration → HiveMQ (credentials via K8s Secrets — never in git)
- [ ] Migration notes from the previous Docker Compose deployment (architecture decision record)

## Conventions

- All manifests applied from this directory are the source of truth — no console-only changes.
- Secrets are referenced, never committed. Example values are placeholders.
