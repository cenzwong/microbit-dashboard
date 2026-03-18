This plan outlines a high-performance, lean telemetry system using the **micro:bit v1** as a dedicated sensor node and a **Web Bluetooth (WebBLE) dashboard** as the processing and visualization engine.

### System Architecture: Hybrid Telemetry Model

| Layer | Responsibility | Internal Mechanism |
| :--- | :--- | :--- |
| **Data Source** | micro:bit v1 (Firmware) | High-speed interrupt-driven sampling (Accel, Temp, Light). |
| **Transport** | BLE 4.0 (GATT) | Pushing binary-packed notifications to the client. |
| **Ingestion** | Web Bluetooth API | Browser-level handshake and stream decoding. |
| **Intelligence** | Web App (Logic) | G-force calculation, noise filtering, and EdgeML (if applicable). |
| **Presentation** | Web App (UI) | Real-time Canvas/SVG rendering (G-G Diagram, Horizon). |

---

### Phase 1: Firmware Design (The Sampler)
Since we are on v1 (16KB RAM), the firmware will be a "Dumb Sampler" to maximize throughput and minimize the risk of Heap fragmentation.

* **Sampling Strategy:** We will target a 20Hz–40Hz update rate. While the accelerometer can go faster, BLE 4.0 on the v1 becomes unstable if the notification queue is saturated.
* **Data Packing (Critical):** Instead of sending ASCII strings (e.g., `"X:10,Y:20"`), we will pack raw integers into a **Byte Buffer**.
    * *Accel:* 6 bytes (2 per axis).
    * *Temp/Light:* 2 bytes.
    * *Total Payload:* 8 bytes per packet. This ensures we stay well within the standard 20-byte MTU (Maximum Transmission Unit) of BLE.
* **GATT Profile:** We will use a custom Service UUID to keep the connection overhead lower than the "Official" heavy micro:bit services.

### Phase 2: Connectivity (The BLE Bridge)
The micro:bit will act as a **BLE Peripheral (Server)**. 

* **Advertising:** The board will broadcast its presence.
* **Security:** We will use "No Bonding" for simplicity, allowing any Web Bluetooth-enabled device to connect.
* **Notification Loop:** The firmware will update the characteristic value. The phone (Client) subscribes to these updates, so data is "pushed" instantly without polling.

### Phase 3: Ingestion & Processing (The Web App)
The Web App (running in Chrome/Edge on your phone) handles the "Heavy Lifting" that would crash a v1.

* **Decoding:** The app will receive an `ArrayBuffer`. We will use a `DataView` in JavaScript to extract the signed 16-bit integers.
* **Calibration Offset:** The app will allow a "Zero-G" button. When pressed, it records the current X, Y, Z values as the "Stationary" baseline to account for the car's mounting angle.
* **Physics Engine:** * Convert raw values to Gs: $G = \text{RawValue} / 1024$ (assuming default range).
    * Calculate Pitch/Roll using the Trigonometric functions provided by the phone's browser engine.

### Phase 4: Visualization (The Dashboard)
The UI will be designed for "At-a-glance" readability during driving.

1.  **G-G Friction Circle:** A HTML5 Canvas drawing a circle with a high-contrast "persistence dot" (showing the last 1 second of movement to see trends).
2.  **Digital Horizon:** A CSS-transformed div that rotates based on the Roll/Pitch data.
3.  **Environmental Ticker:** Small digital readouts for Cabin Temperature and Ambient Light (useful for "Tunnel Detection" or automatic UI dark-mode switching).

---

### Technical Trade-offs & Risks
* **RAM Constraint:** In MicroPython, the Bluetooth stack and the script share the same small heap. If the buffer grows too large, the micro:bit will reset. We will mitigate this by using `bluetooth.gatts_notify()` directly without storing historical data on the board.
* **Jitter:** BLE 4.0 latency can vary. The Web App must use timestamps from the *arrival* of the packet to calculate "Smooth" transitions in the UI.
* **Thermal Drift:** As the v1 CPU works, the internal Temperature sensor will climb. The Web App will need a "Calibration Offset" to subtract the estimated CPU heat from the cabin reading.

**Next Step:** Once you are ready, I can generate the **MicroPython code** for the sampler and the **HTML/JavaScript template** for the Web Dashboard. Which would you like to see first?