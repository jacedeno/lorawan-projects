# ChirpStack on K3s

**Status:** Phase 1 — deploying. ChirpStack v4 (LoRaWAN Network Server) on a bare-metal K3s cluster.

## Architecture decisions

- **PostgreSQL via CloudNativePG** — 3-instance HA cluster with automatic failover, the platform's standard pattern for stateful services. CNPG generates the credentials Secret; ChirpStack reads the DSN from it. Bootstrap installs `pg_trgm` and `hstore` (required by ChirpStack migrations).
- **MQTT broker: existing HiveMQ CE instance** on the IoT stack host (LAN). Reusing the proven production broker means zero new network exposure and one less stateful service to operate. The cluster only makes *outbound* connections to it — consistent with the zero-inbound posture. An in-cluster broker remains a possible later refinement.
- **Gateway path: `chirpstack-gateway-bridge` runs on the gateway Pi itself**, converting the Semtech UDP packet-forwarder traffic (localhost-only on the Pi) to MQTT toward the broker. Consequence: **no UDP ingress into the cluster at all** — the cluster never opens a port for the radio side. See [`../gateway/`](../gateway/).
- **Web UI via Traefik Ingress** (`ClusterIP` + Ingress, TLS entrypoint) — LAN-reachable through the cluster's pinned Traefik entry; public exposure only if/when a Cloudflare Tunnel route is added deliberately.
- **Secrets are never in git.** `secrets.example.yaml` documents the shape; the real Secret is created imperatively (see below).

## Deploy

Applied from the orchestration host with `kubectl`:

```bash
kubectl apply -f namespace.yaml
kubectl apply -f postgres-cluster.yaml     # wait: kubectl -n lorawan get cluster
kubectl apply -f redis.yaml

# one-time, out-of-band — never committed:
kubectl -n lorawan create secret generic chirpstack-secrets \
  --from-literal=API_SECRET="$(openssl rand -base64 32)" \
  --from-literal=MQTT_USERNAME="" \
  --from-literal=MQTT_PASSWORD=""

kubectl apply -f chirpstack-config.yaml
kubectl apply -f chirpstack.yaml
```

Verify:

```bash
kubectl -n lorawan get pods                      # all Running
kubectl -n lorawan logs deploy/chirpstack | head # migrations + region us915_1 loaded
```

First login: `admin` / `admin` — **change it immediately**, then create the tenant, gateway, device profile, and application.

## Region

US915, **sub-band 2 (`us915_1`, channels 8–15)** — must match the gateway's channel plan in [`../gateway/`](../gateway/).

## Migration note (Docker Compose → K3s)

An earlier iteration of this deployment ran on Docker Compose on a standalone host. This K3s deployment replaces it: same ChirpStack v4 + Postgres + Redis shape, now with HA Postgres (CNPG), declarative version-controlled manifests, and the cluster's standard ingress/secret handling. Device and gateway definitions are re-created rather than migrated — at prototype fleet size, a clean re-join is simpler and safer than a datastore migration.
