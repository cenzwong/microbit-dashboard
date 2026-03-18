Modular Telemetry System Plan

1. System Overview

A two-stage architecture that separates hardware-level data acquisition from application-level logic.

2. Technical Stack

Hardware: BBC micro:bit v1.3b/1.5.

Firmware: MicroPython (Binary Data Server).

Protocol: BLE 4.0 (GATT Notifications).

Client Layer: Web Bluetooth API (JavaScript/HTML5).

3. Stage 1: The Backbone (Data Server Layer)

The goal of Stage 1 is to create a "Universal Sampler" firmware that does not need to be reflashed for different use cases.

3.1 Firmware Responsibility

Sensor Polling: Constant 20Hz-40Hz sampling of Accelerometer (3-axis), Compass, Temperature, and Light.

Binary Serialization: Packing raw sensor values into an efficient 8-byte buffer to fit within a single BLE packet.

GATT Service: Maintaining a stable Bluetooth connection and pushing "Notifications" whenever the buffer updates.

3.2 Automation Hooks (Future Proofing)

Since the raw data is always streaming, Stage 2 applications can trigger automations based on:

Luminosity: if (light < 50) -> Toggle Phone UI Dark Mode or trigger Home Automation via Webhook.

Thermal: if (temp > 40) -> Send alert for cabin overheating.

Motion: if (accel_z > threshold) -> Log impact/pothole event.

4. Stage 2: The Application Layer (Car Dashboard)

This layer sits on top of the backbone and interprets the raw data for a specific use case.

4.1 Data Processing Engine

Packet Parser: Converts the 8-byte stream back into human-readable integers.

Calibration Engine: Handles "Zeroing" the sensor to account for the car's mounting angle without changing the micro:bit code.

Physics Layer: Converts raw values into G-force ($G = \text{val}/1024$) and calculates Pitch/Roll angles.

4.2 Visualization (The "Cockpit")

G-G Diagram: Real-time friction circle rendering using HTML5 Canvas.

Artificial Horizon: CSS-based pitch/roll indicator.

Environmental Widgets: Real-time displays for temperature and ambient light.

4.3 Pothole Intelligence

Event Detection: Monitoring the $Z$-axis for sudden delta spikes.

Spatial Mapping: Correlating $Z$-spikes with Phone GPS coordinates.

Proximity Alerts: Comparing current location against a localStorage database of previously detected potholes.

5. Implementation Roadmap

Phase 1: The Backbone (Firmware)

[ ] Implement u-bluetooth notification loop.

[ ] Optimize struct.pack for the 8-byte payload.

[ ] Finalize the "Set-and-Forget" firmware image.

Phase 2: The Client (Web App)

[ ] Create the Web Bluetooth connection handshake.

[ ] Build the real-time telemetry parser.

[ ] Develop the Car Dashboard UI and Pothole alert logic.