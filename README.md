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

### Standalone Tactical Edge Node

The system eliminates the need for external server infrastructure and WAN internet connectivity.

The onboard microprocessor unit acts as an independent base station (Access Point / Local Server).

### Uplink

- The vehicle's optical module captures a raw video stream.
- The stream is transmitted directly to the operator terminal.
- AI processing is performed on a dedicated GPU accelerator (NVIDIA GeForce RTX 4070) utilizing Tensor Cores.

### Downlink

- Processed targeting data, distance calculations, and HUD information are generated at the operator station.
- Return control commands and targeting coordinates are transmitted back to the control board.
- Communication is performed using lightweight telemetry packets (WebSockets / local protocol) to minimize latency.

## Software Stack

- Python 3.x
- Ultralytics YOLOv8
- OpenCV
- Roboflow

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
