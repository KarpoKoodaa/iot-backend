# IoT Backend Platform
This project is a self-hosted IoT Backend platform designed to run on a Linux server.
It provides device communication, data collection, visualization, and firmware updates.

The system is fully containerized using Docker compose, allowing easy development and deployment,

---
## Feature 

- **MQTT Broker (Mosquitto)**
    Secure device-to-cloud communication using MQTT with TLS/mTLS.

- **Node-RED**
    Flow-based logic, device processing, automation, and routin between services.

- **InfluxDB**
    Time-series database for sensor data.

- **Grafana**
    Dashboards and visualization.

---
## Documentation
See the docs/ folder for:
    - Architecture diagram
    - Device identity model
    - OTA update flow
    - Deployment notes.

---
## License
MIT License

Copyright (c) [2025] [Kariantti Laitala]

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