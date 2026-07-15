# Development Log

Working notes, newest first. Same philosophy as the roadmap: what actually happened, including the parts that didn't work on the first try.

## 2026-07-15 — Phase 0 complete; ChirpStack live on K3s (Phase 1 underway)

### Shipped

- **Phase 0 closed:** repository design document, platform architecture (+ SVG/PNG diagram), layered root README, and the full cattle-tracker design set — telemetry design with the 16-byte payload layout, power budget with battery-life scenarios, and the geofencing ("virtual fence") design.
- **Phase 1 — 3 of 4 items done in one session:**
  - ChirpStack v4.9.0 deployed to the K3s cluster: namespace `lorawan`, 3-instance HA PostgreSQL via the CloudNativePG operator (bootstrap installs `pg_trgm` + `hstore`), Redis with a small PVC, config via ConfigMap (subPath mount so the image's built-in region files remain in place).
  - MQTT integration connected to the existing HiveMQ CE broker and subscribed to command topics. Reusing the proven broker keeps the cluster outbound-only — no new network exposure.
  - Web UI live behind Traefik (ClusterIP + Ingress); verified end to end.
  - Migration notes from the earlier Docker Compose deployment recorded in `platform/chirpstack/README.md`.
- Remaining for Phase 1: assemble and provision the WM1302 gateway (Raspberry Pi + HAT, packet forwarder + `chirpstack-gateway-bridge` on the Pi).

### Issues hit and fixed

1. **ChirpStack ↔ CNPG TLS handshake failure.** First deployment crash-looped with `error performing TLS handshake: invalid peer certificate: UnknownIssuer`. Cause: the CNPG-generated connection URI leads ChirpStack's Postgres client to negotiate TLS, but CNPG's operator-issued CA is not in the client's trust store. Fix: build the DSN from the CNPG secret's component keys (`username`/`password`/`host`) with `sslmode=disable` — acceptable here because the hop never leaves the cluster network. Documented inline in the ConfigMap.
2. **`kubectl get cluster` returned nothing** even though the CNPG cluster existed. Known CRD-shadowing behavior when another `clusters` CRD is installed: the short name resolves to the wrong API group. Always use the fully qualified `clusters.postgresql.cnpg.io`.

### Decisions

- **Broker:** reuse the existing HiveMQ CE instance rather than deploying a second broker in-cluster. Rationale: zero new exposure, one less stateful service, and the cluster only makes outbound connections. Revisit if the platform ever needs broker isolation per domain.
- **No UDP into the cluster:** the Semtech UDP packet forwarder will talk to `chirpstack-gateway-bridge` locally on the gateway Pi; only MQTT (TCP, outbound from the Pi) crosses the network. The cluster's zero-inbound posture holds.
- **Region:** US915 sub-band 2 (`us915_1`, channels 8–15). The gateway channel plan must match.
