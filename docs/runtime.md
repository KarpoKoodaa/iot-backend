# Runtime Environment 
This documentation describes how the IoT Backend Platform runs in practice:
what services are running, how they are started, how they store data, and how they are accessed.

## 1. Scope and Assumptions

- This runtime is designed for homelab use.
- The platform runs on Raspberry Pi (Ubuntu Server).
- All services run as Docker containers.
- Docker Compose is used for orchestration.
- This is not a hardened production environment.

## 2. High-Level Runtime Architecture

**Overview**
The platform consists of the following services:
| Service | Purpose |
|---------|---------|
| Mosquitto | MQTT broker for device telemetry |
| Node-RED | Message routing and transformation |
| InfluxDB | Time-series database |
| Grafana | Visualization and dashboards |

All services run on a single host and communicate over a private Docker network.

## 3. Host Environment

**Operating System**
- Ubuntu Server (ARM64).
- Installed on Raspberry Pi

**Hostname**
- iot-hub

**User Model**
- Admin user for system management.
- Non-privileged user for day-to-day operations.

## 4. Container Runtime
**Docker**
- Docker Engine installed via official packages.

**Startup Behavior**
- Docker daemon starts on boot.
- Docker Compose stack is started automatically using:
    - systemd service
    - Docker restart policies

## 5. Services and Ports

**Exposed Ports**

| Service | Port | Protocol | Access Scope | Notes |
|---------|------|----------|--------------|-------|
| Mosquitto | 1883 | MQTT | Local LAN | No TLS (dev) |
| Node-RED | 1880 | HTTP | Local LAN | Admin UI |
| InfluxDB | 8086 | HTTP  Local host / LAN | API |
| Grafana | 3000 | HTTP | Local LAN | Dashboards |

Ports are restricted at the firewall level (UFW).

## 6. Data Persistence

**Storage Location**
All persistent data is stored on an external SSD mounted at:
```
/mnt/iot-data
```
**Volume Mapping**
| Service | Path in Container | Host Path |
|---------|-------------------|-----------|
| Mosquitto | /mosquitto/data | /mnt/iot-data/mqtt/data |
| Mosquitto | /mosquitto/log | /mnt/iot-data/mqtt/log |
| Node-RED | /data | /mnt/iot-data/nodered |
| InfluxDB | /var/lib/influxdb2 | /mnt/iot-data/influxdb |
| Grafana | /var/lib/grafana | /mnt/iot-data/grafana |

## 7. Configuration Management

**Configuration Sources**
- Docker Compose (docker-compose.yaml)
- Service-specific config files:
    - ```mosquitto.conf```
- Node-RED flows (currently managed via UI)

**Version Control Policy**
- Configuration files may be version-controlled.
- Runtime state and generated data must not be committed.


## 8. Secrets Management

**Current Approach**
- Secrets are injected via:
    - ```.env``` file.
    - Docker environment variables.

**Stored secrets**
| Secret | Used by | Storage |
|--------|---------|---------|
| InfluxDB token | Node-RED | env var |

## 9. Network Security
**Firewall (UFW)**
- UFW is enabled.
- Only required ports are allowed from local subnet.
- SSH is allowed from LAN.

**MQTT Access**
- Anonymous access: disabled.
- Authentication: user/pass.
- Credentials stored in a persistent password file.
- User management is performed using mosquitto_passwd via Docker.
- TLS: No

Security configuration reflects current development phase.

NOTE: MQTT users are managed while the Mosquitto container is stopped. The password file is treated as immutable at the runtime.

## 10. Startup and Restart Behavior

**Restart Policy**
- All containers use:
```restart: unless-stopped```

**Expected Behavior After Reboot**
- Docker daemon starts.
- All containers start automatically.
- No manual step required.

## 11. Operational Tasks

**Start the Platform**
```
docker compose up -d
```
**Stop the Plaform**
``` 
docker compose down
``` 
**View logs**
``` 
docker compose logs -f
``` 
**Check Container Status**
```
docker compose ps
``` 

## 12. Known Limitations
- No TLS for MQTT or HTTP services.
- No per-device authentication. 
- Node-RED flows not version-controlled.
- Single-node deployment.

## 13. Future Improvements
- TLS for MQTT and HTTP.
- Device-level credentials.
- Infrastructure-as-a-Code (IaC).
- Versioned Node-RED flows.
- Backup strategy.

## 14. Built in Public
This project is built in public - mistakes included.