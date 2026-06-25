# A.R.M.O.R. - Advanced Remote Monitor & Operation Recon

A.R.M.O.R. is a proprietary, real-time tactical assistant system designed for autonomous and remote-controlled tracked platforms. The system integrates deep neural networks with computer vision algorithms for automated detection, tracking, targeting, and monocular distance estimation of ground targets.

The system architecture focuses on thermal signature mitigation, passive acquisition, and cryptographic transmission security, with dynamic target configuration capabilities.

## Core Design Assumptions

### Model-Based Detection & Identification

- Real-time classification and recognition of combat vehicles strictly based on operational factions (**NATO** vs. **Eastern Block**) using an optimized YOLOv8 model.
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

### Hybrid Tactical Architecture (Cellular Relay Grid)
To achieve maximum physical mobility and operational flexibility without sacrificing heavy computational power, the system utilizes a **Hybrid Tactical Architecture** with a mobile gateway relay. Instead of forcing the operator to transport a high-performance workstation to the field, the system bridges the local physical edge with a remote processing node via a secure, peer-to-peer VPN tunnel (Tailscale).

### Component Segmentation
1. **Kinematic Execution Node (Onboard Platform):** A low-power ESP32 microcontroller operating at the physical edge. It manages the vehicle's local Wi-Fi connection, telemetry ingestion, and low-level hardware execution (PWM servo controls for turret alignment and drive-train operation).
2. **Mobile Gateway & Tactical HUD Node (Operator Smartphone):** A portable relay carried by the operator. It interfaces locally with the vehicle via Wi-Fi to capture the raw video stream and handles real-time local HUD rendering. Concurrently, it acts as an internet gateway, forwarding the stream to the remote processing station over an encrypted 5G/LTE network.
3. **Remote Processing Server (Base Station):** A stationary, high-performance home computer equipped with an NVIDIA GeForce RTX 4070 (Tensor Cores). It hosts the YOLOv8 faction detection pipeline, ByteTrack tracking, and OpenCV-based IFF logic, operating continuously as an automated background daemon.

### Design Justification (Engineering Trade-offs)
- **Thermal Signature Reduction:** Offloading AI inference prevents high thermal emissions on the tracked vehicle, drastically reducing its infrared and thermal signature on the field.
- **Power Optimization:** Eliminating onboard GPU/NPU hardware minimizes power draw, dramatically increasing battery life and operational range of the tracked mobile platform.
- **Cost-to-Performance & Weight Ratio:** Allows the use of full-scale desktop architectures (Tensor Cores) to achieve massive FPS tracking speeds without payload weight penalties on a 1:16 scale chassis.
- **Transmission Security:** Utilizing a VPN (Tailscale) layer guarantees that all telemetry and video feeds are fully encrypted, protecting the system from unauthorized interception and spoofing attacks.

### Uplink (Video Pipeline)
- The vehicle's optical module captures a raw video stream and transmits it to the smartphone relay via low-latency local Wi-Fi.
- The smartphone compresses the stream using hardware-accelerated codecs (H.264/H.265) and pushes it via WebRTC/RTSP over the 5G VPN tunnel directly to the remote server.
- Real-time AI processing (faction detection, tracking, IFF) is performed entirely on the remote host's GPU.

### Downlink & Telemetry
- Processed targeting data, distance calculations, and dynamic HUD layers are generated at the remote server and transmitted back to the smartphone relay.
- The smartphone instantly bridges these incoming control commands, servo angles, and targeting coordinates back to the vehicle's ESP32 control board.
- **Protocol:** Communication across the entire network layer is performed using raw, lightweight **UDP datagrams**. This guarantees maximum freshness of control packages and eliminates latency spikes (*jitter*) inherent in retransmission-based protocols during real-time platform operation.

## Software Stack

- Python 3.11
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
