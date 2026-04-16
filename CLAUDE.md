# IoT Backend — CLAUDE.md

See root `../CLAUDE.md` for the full system overview, MQTT topic schema, identity model, and BLE contract.
Backend docs in `docs/` are the **source of truth** for all protocol-level contracts.

---

## What this is

Docker Compose stack running on a Raspberry Pi (Ubuntu Server). Persistent data lives on an external SSD mounted at `/mnt/iot-data/`.

| Service | Image | Role |
|---|---|---|
| Mosquitto | `eclipse-mosquitto` | MQTT broker (port 1883; TLS on 8883 planned) |
| Node-RED | `nodered/node-red` | Flow-based processing and InfluxDB writes |
| InfluxDB | `influxdb:2.x` | Time-series storage |
| Grafana | `grafana/grafana` | Dashboards and visualization |

All containers use `restart: unless-stopped`.

---

## Directory layout

```
iot-backend/
├── docker-compose.yaml
├── .env                   # secrets — never commit
├── mqtt/
│   ├── mosquitto.conf
│   ├── passwd_file        # managed via mosquitto_passwd inside container
│   └── acl
├── node-red/
├── influxdb/
├── grafana/
├── docs/
│   ├── architecture.md
│   ├── telemetry_model.md
│   ├── device-identity.md
│   └── runtime.md
└── README.md
```

Data volumes on SSD:
```
/mnt/iot-data/{mqtt,nodered,influxdb,grafana}/
```

---

## Common operations

```bash
# Start all services
docker compose up -d

# View logs for a specific service
docker compose logs -f mosquitto

# Add a gateway MQTT user
docker exec mqtt mosquitto_passwd /etc/mosquitto/passwd_file <gateway_id>

# Reload ACL without restart
docker exec mqtt kill -HUP 1
```

---

## MQTT auth and ACL

Authentication: username/password via `passwd_file` inside the container.
Gateway MQTT username must match `gateway_id` used in topic construction.

ACL enforces gateway isolation — see `mqtt/acl` and `docs/device-identity.md` for full rules.

Node-RED credentials are in `.env`. Node-RED has `readwrite #` access.

---

## Node-RED → InfluxDB data mapping

Function node transforms MQTT payload before writing to InfluxDB:

- `value` → **field** (the numeric measurement)
- `device_id`, `site`, `gateway_id`, `location`, `measurement` → **tags**

Topic segments are parsed to extract tags. Timestamp comes from the payload `timestamp` field.

---

## Security notes

- `.env` holds all secrets (InfluxDB token, MQTT passwords, Grafana admin). Never commit.
- ACL file uses `%u` pattern — each gateway can only write to its own subtree.
- **TLS not yet enabled.** MQTT traffic is currently plaintext on the local network. See open items in root CLAUDE.md.

---

## Documentation

All docs are versioned (`draft` → `rc` → `release`). Do not change topic structure or payload format without updating `docs/telemetry_model.md` first — the gateway firmware depends on it.

| Doc | Current state |
|---|---|
| `architecture.md` | release |
| `telemetry_model.md` | rc → release pending |
| `device-identity.md` | rc → release pending |
| `runtime.md` | release |
