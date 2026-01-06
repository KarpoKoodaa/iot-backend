# IoT Backend Platform
This project is a self-hosted IoT Backend platform designed to run on a Linux server.
It provides device communication, data collection, visualization, and firmware updates.

The system is fully containerized using Docker compose, allowing easy development and deployment,

---
## Feature 

- **MQTT Broker (Mosquitto)**
    Secure device-to-cloud communication using MQTT with TLS/mTLS.

- **Node-RED**
    Flow-based logic, device processing, automation, and routing between services.

- **InfluxDB**
    Time-series database for sensor data.

- **Grafana**
    Dashboards and visualization.

---
## Documentation

The documents below defines the architecture, runtime, data contracts, and security model.

### Core Documentation

- **Architecture Overview**
  - [`docs/architecture.md`](docs/architecture.md)
  - High-level system design, trust boundaries, and component responsibilities.

- **Runtime Environment**
  - [`docs/runtime.md`](docs/runtime.md)
  - How the platform runs in practice (Docker, persistence, startup, operations).

- **Telemetry Data Model & Topic Contract**
  - [`docs/telemetry_model.md`](docs/telemetry_model.md)
  - MQTT topic structure, payload formats, QoS rules, and lifecycle semantics.

- **IoT Gateway Identity & Authorization Model**
  - [`docs/device-identity.md`](docs/device-identity.md)
  - MQTT authentication, ACL rules, gateway trust model, and onboarding.

---
### Document Relationships

- Architecture defines *what exists* and *why*
- Telemetry Model defines *what data flows*
- Identity Model defines *who may publish/subscribe*
- Runtime defines *how everything is executed*

All documents are versioned and evolve independently.

---
## License
MIT License

Copyright (c) [2026] [Kariantti Laitala]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.