# IoT Gateway Identity and Authorization Model

## Document Version
- **Version**: 1.1.0
- **Last Updated**: 9.5.2026
- **Status**: Released

## Version History
| Version | Date | Changes | Status |
|---------|------|---------|--------|
| 1.0.0-draft | 4.1.2026 | Initial version | Draft |
| 1.0.0-rc1 | 5.1.2026 | Updated Telemetry Data Model | RC |
| 1.0.0 | 5.1.2026 | Released | Released |
| 1.1.0 | 9.5.2026 | Added sensor onboarding via NVS registry MQTT command | Released |

## Breaking Changes Policy
- Major version increment (X.0.0): Breaking changes to topic structure or required fields data.
- Minor version increment (1.X.0): New optional fields or topics.
- Patch version increment (1.0.X): Documentation clarification only.

## 1. Purpose
This document defines how IoT Gateways are identified, authenticated, and authorized within the IoT Backend Platform.

The goal is to ensure: 
- Each IoT Gateway has a unique identity.
- IoT Gateway can only access their own data.
- Backend services have appropriate elevated accesses.
- The system follows least-privilege principles.

## 2. Related Documentation
- [Architecture Overview](./architecture.md)
- [Telemetry Data Model and Topic Contract](./telemetry_model.md)

## 3. Identity Model

### IoT Gateway Identity

Each IoT Gateway is identified by:

| Attribute | Description |
|-----------|-------------|
| Username | Primary identity used for MQTT authentication |
| Password | Shared secret stored in Mosquitto password file |
| Gateway ID | Must match the MQTT username |
| MQTT Client ID | Arbitrary, not trusted for identity |

**Rule:**

    MQTT username MUST equal the gateway_id used in MQTT topics.
Example:
```
Username: gw-01
Topic: iot/site/gw-01/location/device-001/temperature
```

## 4. Authentication

### Mechanism
- Protocol: MQTT
- Authentication: Username / Password
- Broker: Mosquitto
- Anonymous access: Disabled

Credentials are validated against a Mosquitto password file.

## 5. Authorization (ACL)
Authorization is enforced using Mosquitto ACL rules.

### Device Permissions

Each device: 
- Can publish telemetry to its own topics
- Can subscribe to its own command topics
- Cannot access other devices' topics

**Allowed topics**

```
iot/{site}/{gateway_id}/{location}/{device_id}/+
iot/{site}/{gateway_id}/{location}/{device_id}/commands/#
``` 

### Backend Permissions (Node-RED)
Node-RED uses dedicated MQTT user with elevated privileges.

Capabilities:
- Read all telemetry
- Publish commands
- Publish events
- Debug and test flows

## 6. ACL Configuration
Example ACL file:
```conf
# Node-RED backend
user nodered
topic readwrite #

# Device access
pattern write iot/+/%u/+/+/#
pattern read  iot/+/%u/+/#/commands/#
```

## 7. Device Onboarding (Gateway)

1. Generate a unique device ID.
2. Create MQTT credentials.
    ```
    mosquitto_passwd /mosquitto/passwd_file gw-01
    ```
3. Flash credentials into device firmware.
4. Device connects using:
    - username = gateway_id
    - password = generated secret
5. ACL automatically enforces topic isolation.

## 8. Adding a New Sensor Node

New BLE sensor nodes can be registered with a running gateway **without reflashing** the gateway firmware. The gateway stores its device registry in ESP32 NVS (Non-Volatile Storage), which persists across reboots.

### Step 1 — Build and flash the sensor firmware

Create or copy a sensor firmware project (e.g. from `greenhouse-node/`). The only required change is the BLE advertised device name. The BLE contract (service UUID `0x180A`, characteristic UUID `0x2A6E`, packed `sensor_data_t` struct) must remain identical.

### Step 2 — Register the device on the gateway via MQTT

Publish to the gateway's registry command topic:

```
Topic:   iot/{site}/{gateway_id}/_gateway/commands/registry
Payload: {"action": "add", "ble_name": "<BLE name>", "device_id": "<device-id>", "location": "<location>"}
```

Example (adding a barn sensor to gateway `gw-01` at site `cottage`):
```
Topic:   iot/cottage/gw-01/_gateway/commands/registry
Payload: {"action": "add", "ble_name": "BarnNode", "device_id": "barn-node-01", "location": "barn"}
```

The gateway responds on:
```
Topic: iot/cottage/gw-01/_gateway/events/registry
Payload: {"status": "ok"}
```

### Step 3 — Verify data is flowing

The gateway picks up the new device on its next BLE scan cycle. Once connected, sensor data appears on:
```
iot/cottage/gw-01/barn/barn-node-01/temperature
iot/cottage/gw-01/barn/barn-node-01/humidity
... (one topic per measurement)
```

No backend changes are needed — Grafana dashboards and email reports pick up new `device_id` values automatically.

### Removing a device

```
Topic:   iot/cottage/gw-01/_gateway/commands/registry
Payload: {"action": "remove", "ble_name": "BarnNode"}
```

### Listing all registered devices

```
Topic:   iot/cottage/gw-01/_gateway/commands/registry
Payload: {"action": "list"}
```

See `docs/telemetry_model.md` §6 for full registry topic and payload reference.

## 9. Security Considerations
- Credentials are static (no rotation yet).
- No TLS (development phase).
- Password file stored on persistent storage.
- ACL enforces strict topic boundaries.

## 10. Limitations
- No per-device TLS certificates.
- No dynamic provisioning.
- No credential rotation.
- Manual onboarding.

## 11. Future Improvements
- TLS with client certificates.
- Automated provisioning. 
- Device credential rotation.
- Integration with OTA services.



