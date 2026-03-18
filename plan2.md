Pothole Detection & Telemetry System Plan

1. System Overview

A hybrid solution using a micro:bit v1 as a "Remote Sensor Head" and a Smartphone as the "Logic & Mapping Engine."

2. Technical Stack

Hardware: BBC micro:bit v1.3b/1.5 (Sensor Node).

Firmware: MicroPython (Optimized for 16KB RAM).

Communication: BLE 4.0 (Custom GATT Service).

Client: Web Bluetooth API (Chrome/Edge on Mobile).

Visualization: HTML5 Canvas & CSS3 Transforms.

3. Data Flow & Payload Structure

To minimize RAM usage on v1, we use a single 8-byte notification packet:
| Bytes | Content | Range |
|-------|---------|-------|
| 0-1   | Accel X | -2048 to 2047 |
| 2-3   | Accel Y | -2048 to 2047 |
| 4-5   | Accel Z | -2048 to 2047 |
| 6     | Temp    | -128 to 127 |
| 7     | Light   | 0 to 255 |

4. Logical Phases

Phase A: Calibration

Stateless Start: micro:bit begins broadcasting.

Orientation Lock: Web app provides a "Level" button to set the zero-offset for the car's mounting angle.

Phase B: Real-time Telemetry

Visuals: Render G-G Diagram (Friction Circle) and Horizon (Pitch/Roll).

Smoothing: Apply a simple Moving Average Filter in JavaScript to the display data, while keeping raw data for pothole detection.

Phase C: Pothole Intelligence

Trigger: if (abs(z_current - z_previous) > Threshold).

Geotagging: Web app calls navigator.geolocation.getCurrentPosition().

Storage: Save to localStorage (as a JSON array of coordinates).

Monitoring: Loop checks current location vs. localStorage list.

5. Implementation Roadmap

Step 1: Write MicroPython "Binary Notification" script.

Step 2: Build Web App "Handshake" (Request Device -> Connect -> Subscribe).

Step 3: Implement Gauge/Canvas logic.

Step 4: Add Geolocation awareness and proximity alerts.