---
layout: default
title: MQTT
---

# MQTT

## Example project

- Arduino MKR WIFI 1010 + ENV shield
- Embedded C/C++ Arduino code for the MQTT client
  - **Setup**
    - Connect to the sensor shield
    - Configure the UART
    - Connect to WiFi
    - Connect to the MQTT broker (the Raspberry Pi)
  - **Loop**
    - At regular intervals, send the serialized JSON data

## IOTstack on Raspberry Pi

Applications run in Docker containers:

- **Portainer-ce** — maps each application to a Pi port
- **Mosquitto** — the MQTT broker
- **Node-RED**
- **InfluxDB** — database
- **Grafana** — front-end dashboard

Workflow:

1. Create an InfluxDB database via the CLI.
2. In Node-RED, process the MQTT topic (JSON parsing, other functions).
3. Send the result to the InfluxDB *out* node.
4. Use the debug nodes — very handy!
5. Deploy.
6. Run the queries in Grafana and display them across the different graphs
   (remember to save!).

[Back to Connectivity](./)
