# Device Identity and Authorization Model

## 1. Purpose
This document defines how to IoT devices are identified, authenticated, and authorized within the IoT Backend Platform.

The goal is to ensure: 
- Each device has a unique identity.
- Devices can only access their own data.
- Backend services have appropriate elevated accesses.
- The system follows least-privilege principles.

## 2. Identity Model

### Device Identity

Each IoT device is identified by:

| Attribute | Description |
|-----------|-------------|
| Username | Primary identity used for MQTT authentication |
| Password | Shared secret stored in Mosquitto password file |
| Device ID | Must match the MQTT username |
| MQTT Client ID | Arbitrary, not trusted for identity |

**Rule:**

    MQTT username MUST equal the device_id used in MQTT topics.
Example:
```
Username: device-0001
Topic: iot/site/location/device-001/temperature
```

## 3. Authentication

### Mechanism
- Protocol: MQTT
- Authentication: Username / Password
- Broker: Mosquitto
- Anonymous access: Disabled

Credentials are validated against a Mosquitto password file.

## 4. Authorization (ACL)
Authorization is enforced using Mosquitto ACL rules.

### Device Permissions

Each device: 
- Can publish telemetry to its own topics
- Can subscribe to its own command topics
- Cannot access other devices' topics

**Allowed topics**

```
iot/{site}/{location}/{device_id}/+
iot/{site}/{location}/{device_id}/commands/#
``` 

### Backend Permissions (Node-RED)
Node-RED uses dedicated MQTT user with elevated privileges.

Capabilities:
- Read all telemetry
- Publish commands
- Publish events
- Debug and test flows

## 5. ACL Configuration
Example ACL file:
```conf
# Node-RED backend
user nodered
topic readwrite #

# Device access
pattern write iot/+/+/%u/+
pattern read  iot/+/+/%u/commands/#
```

## 6. Device Onboarding Process

1. Generate a unique device ID.
2. Create MQTT credentials.
    ```
    mosquitto_passwd /mosquitto/passwd_file device-001
    ```
3. Flash credentials into device firmware.
4. Device connects using:
    - username = device_id
    - password = generated secret
5. ACL automatically enforces topic isolation.

## 7. Security considerations
- Credentials are static (no rotation yet).
- No TLS (development phase).
- Password file stored on persistent storage.
- ACL enforces strict topic boundaries.

## 8. Limitations
- No per-device TLS certificates.
- No dynamic provisioning.
- No credential rotation.
- Manual onboarding.

## 9. Future Improvements
- TLS with client certificates.
- Automated provisioning. 
- Device credential rotation.
- Integration with OTA services.



