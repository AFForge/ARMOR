# A.R.M.O.R. - Advanced Remote Monitor & Operation Recon

A.R.M.O.R. is a proprietary, real-time tactical assistant system designed for autonomous and remote-controlled tracked platforms. The system integrates deep neural networks with computer vision algorithms for automated detection, tracking, targeting, and monocular distance estimation of ground targets.

The system architecture is built on complete signature neutrality with dynamic target configuration capabilities.

## Core Design Assumptions

### Model-Based Detection & Identification

- Real-time recognition of specific combat vehicle types and modifications (e.g. T-72, T-90, Leopard, Abrams) using an optimized YOLOv8 model.
- Detection accounts for structural anomalies and atypical units (collective class: **Unidentified**).

### Dynamic ROE (Rules of Engagement) Configuration

- Implementation of a variable Threat Library.
- The system allows defining a target matrix prior to platform deployment.
- Specific vehicle profiles can be assigned to **Ally** or **Target** status through local configuration files.
- No neural network retraining is required.

### Persistent Target Tracking & Targeting

- Continuous object tracking using ByteTrack / BoT-SORT.
- Assignment of unique tactical identifiers (IDs).
- Constant computation of targeting and guidance vectors.

### Monocular Distance Estimation

- Distance estimation based on geometric analysis of YOLOv8 bounding box dynamics.
- Utilizes known vehicle dimensions and camera optical parameters.

### Two-Stage IFF (Identification Friend or Foe) Verification

- Independent OpenCV-based vision filter.
- HSV color-space analysis within the targeting vector.
- Verification of physical identification markers (e.g. tactical tape).
- Acts as a primary safety interlock against friendly fire.

## Architecture & Communication

### Standalone Tactical Edge Node & Hybrid Topology
The system eliminates the need for external cloud server infrastructure and WAN internet connectivity, ensuring complete RF-emission control and resilience to electronic warfare (EW). 

The architecture is divided into two distinct layers:
1. **Edge Execution Layer (Onboard):** A low-power microcontroller unit (ESP32) acts as the physical vehicle controller and local Access Point (AP). It directly interfaces with high-torque servos, drive-train motor controllers (via PWM), and manages raw sensor output.
2. **Tactical Processing Node (Operator Terminal):** A high-performance station equipped with a dedicated GPU accelerator (NVIDIA GeForce RTX 4070 utilizing Tensor Cores) responsible for heavy computer vision workloads and AI inference.

### Uplink
- The vehicle's optical module captures a raw video stream.
- The stream is transmitted directly to the operator terminal via low-latency Wi-Fi protocol.
- Real-time AI processing (detection, tracking, IFF) is performed entirely on the operator's GPU.

### Downlink & Telemetry
- Processed targeting data, distance calculations, and HUD information are generated at the operator station.
- Return control commands, servo angles, and targeting coordinates are transmitted back to the vehicle's ESP32 control board.
- **Protocol:** Communication is performed using raw, lightweight **UDP datagrams** instead of TCP/WebSockets. This guarantees maximum freshness of control packages and eliminates latency spikes (*jitter*) inherent in retransmission-based protocols during real-time combat platform operation.

## Software Stack

- Python 3.x
- Ultralytics YOLOv8
- OpenCV
- Roboflow
  
## Hardware Stack (Test & Target Platforms)
- **Primary Control Unit:** ESP32 (NodeMCU / custom board)
- **Actuators:** High-torque PWM Servos (Turret Azimuth/Elevation), ESC Motor Controllers
- **Inference Hardware:** NVIDIA GeForce RTX 4070 (TensorRT optimized)
- **Scale Prototyping Platform:** 1:16 Tracked Armor Chassis (Heng Long 7.0 system integration)

### Functions

- Object detection, tracking, and targeting
- HSV color analysis for the IFF subsystem
- HUD interface processing
- Dataset management
- Data augmentation
- Image clustering

## Implementation Status (Version 0.1.0)

- [x] Local network topology design and optimization (Direct Vehicle-to-Terminal pipeline)
- [x] Database structure preparation for specific combat vehicle profiles
- [ ] Data sourcing, labeling, and anomaly filtering in Roboflow
- [ ] Compilation and training of production weights for the YOLOv8 model
- [ ] Implementation of the monocular distance estimation module and IFF script in OpenCV
- [ ] Integration of physical guidance systems (servos) with the control logic
- [ ] Implement of a hardware-level Fail-Safe watchdog on ESP32 (auto-stop on telemetry timeout)
- [ ] Integration of real-time voltage and RSSI diagnostics into the UDP telemetry stream
- [ ] Predictive target interception logic (Lead Angle calculation based on target velocity vectors)
