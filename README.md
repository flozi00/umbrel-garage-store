# Garage S3 – Umbrel Community App Store

1. Push this folder to a public GitHub repo.
2. Umbrel → App Store → ⋯ → Community App Stores → paste the repo URL.
3. Install **Garage**. Web UI opens via Umbrel; S3 API is on port `3900` (region `garage`).

Data lives in `~/umbrel/app-data/florian-garage/data/`.

## Endpoints (with Tailscale)

| Endpoint | URL | Auth |
|---|---|---|
| S3 API (LAN, plain HTTP) | `http://umbrel.local:3900` | S3 keys |
| S3 API (tailnet, TLS) | `https://<umbrel>.<tailnet>.ts.net:10000` | S3 keys (+ tailnet membership) |
| Garage UI | App tile (port 8080) | Umbrel gateway |
| Inter-node RPC | `<tailscale-ip>:3901` (TCP) | shared `GARAGE_RPC_SECRET` |

The tailnet HTTPS endpoint and `rpc_public_addr` are managed by the app's
`hooks/pre-start` (same persistence pattern as florian-hermex-webui's 8443).

## Clustering multiple Umbrel Garage instances

Garage is a geo-distributed store: nodes cluster over the RPC port (3901, TCP),
encrypted/authenticated by the cluster-wide `rpc_secret`. On a tailnet this needs
no VPN extra — WireGuard is already the transport.

1. Generate a cluster secret: `openssl rand -hex 32`
2. On **every** Garage instance (Umbrel app page → ⚙ settings): set
   `GARAGE_RPC_SECRET` to that same value, restart the app once.
3. Note each instance's full node ID: `docker exec florian-garage_garage_1 /garage node id -q`
   (or read it in Garage UI → Cluster).
4. From one instance, connect a peer:
   `docker exec florian-garage_garage_1 /garage node connect <full-node-id>@<peer-tailscale-ip>:3901`
5. Assign roles & apply the layout (Garage UI → Cluster, or
   `garage layout apply`); pick a zone per node (e.g. `dc1`, `dc2`, …).
6. Verify: `garage status` on any node lists all nodes. Data is then replicated
   per the layout regardless of which node a client hits.

Notes
- Peers must be in the same tailnet (Tailscale ACLs permitting 3901) and use
  Garage >= v2.0 (TCP RPC; pre-2.0 used QUIC/UDP 3901 and cannot inter-operate).
- Replication factor is per-cluster layout: keep `replication_factor = 1` on all
  nodes unless you deliberately want N-way replication (set it BEFORE first
  start; changing later requires a layout migration).
- The Umbrel LAN hostname (`umbrel.local:3900`) is plain HTTP — fine inside the
  LAN, use the tailnet HTTPS endpoint for anything leaving it.