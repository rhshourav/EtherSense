# EtherSense

**Coherence-Gated Multistatic WiFi Sensing for Real-Time Pose, Vital Signs, and Room Field Modeling**

---

## Overview

**EtherSense** transforms commodity 2.4 GHz WiFi infrastructure into a structured RF perception system. Using synchronized ESP32 mesh nodes for CSI acquisition across channels 1/6/11, the system performs multiband and multistatic fusion to reconstruct:

* 17 human body keypoints
* Respiratory waveform and breathing rate
* Ballistocardiographic (BVP-derived) heart rate
* Room RF fingerprint and topology drift alerts

EtherSense is architecturally inspired by **RuVector** by **ruvnet**, extending its structured AI and signal-line reasoning principles into the RF sensing domain.

This is not a signal-processing demo. It is a coherence-driven field modeling system.

---

# Architecture

```
WiFi Router → RF propagation → human scattering → multipath field
    ↓
ESP32 Mesh (4–6 nodes) → CSI capture via TDM (Ch 1/6/11)
    ↓
Multi-Band Fusion (3 × 56 = 168 subcarriers per link)
    ↓
Multistatic Fusion (N × (N − 1) cross-links)
    ↓
Coherence Gate (measurement validation + drift rejection)
    ↓
Signal Processing Stack
    ↓
RuVector-Inspired AI Backbone
    ↓
CRV Signal-Line Protocol
    ↓
Outputs: Pose + Vitals + Room Model + Drift Diagnostics
```

---

# Core Components

## 1. CSI Acquisition Layer

* ESP32 nodes with CSI-enabled firmware
* Time-Division Multiplexed (TDM) capture across channels 1/6/11
* Phase and amplitude extraction
* Cross-node synchronization discipline

With `N` nodes:

```
Total Links = N × (N − 1)
```

Each link yields 168 virtual subcarriers via multiband aggregation.

---

## 2. Multi-Band Fusion

* 3 WiFi channels
* 56 subcarriers per channel
* 168 effective subcarriers per link

This reduces frequency-selective fading bias and increases spatial diversity.

---

## 3. Multistatic Cross-View Fusion

EtherSense does not rely on a single transmitter-receiver pair.

It constructs a cross-link embedding space where:

* Each link is treated as a viewpoint
* Attention weighting reduces occlusion sensitivity
* Graph-structured embeddings encode topology

This enables spatial reconstruction beyond line-of-sight constraints.

---

## 4. Coherence Gate

The coherence gate enforces structural signal integrity.

It:

* Rejects decorrelated CSI windows
* Detects environmental drift
* Prevents silent model degradation
* Enables multi-day deployment stability

This is a stability architecture, not just a filter.

---

## 5. Signal Processing Stack

The preprocessing layer includes:

* Hampel filtering (impulse rejection)
* Phase sanitization (SpotFi-inspired)
* Fresnel zone modeling (motion localization)
* Band-pass filtering for respiration
* BVP micro-motion extraction for cardiac signals
* STFT / spectrogram for spectral robustness

The objective is feature stability before AI inference.

---

## 6. RuVector-Inspired AI Backbone

The inference engine integrates:

* Cross-link attention
* Graph-based embedding
* Field-consistency modeling
* Compression-aware representation
* Topological constraints

Outputs:

* 17 RF-derived skeletal keypoints
* Respiratory waveform + rate
* Heart rate estimation
* Room fingerprint vector
* Drift anomaly indicators

---

## 7. CRV Signal-Line Protocol

EtherSense enforces a structured 6-stage inference pipeline:

1. **Gestalt** – Raw field aggregation
2. **Sensory** – Feature isolation
3. **Topology** – Cross-link embedding
4. **Coherence** – Stability validation
5. **Search** – Pose and vital inference
6. **Model** – Field-consistent output synthesis

This prevents ad-hoc feature leakage and enforces architectural discipline.

---

# Hardware Requirements

* 4–6 ESP32 boards (CSI-enabled firmware)
* 2.4 GHz WiFi router
* Stable power supply for each node
* Controlled TDM scheduling logic

Optional:

* Tripod-mounted node placement
* Fixed geometric deployment grid
* Environmental baseline capture session

---

# Software Stack

* Embedded ESP32 firmware (CSI capture)
* Python / C++ signal processing layer
* AI inference engine (PyTorch / ONNX compatible)
* Real-time visualization interface
* Drift monitoring module

---

# Outputs

EtherSense produces:

* Real-time skeletal pose (17 keypoints)
* Breathing waveform + BPM
* Heart rate estimation
* Room RF fingerprint embedding
* Coherence and drift alerts
* Deployment stability metrics

---

# Intended Use Cases

* RF-based human pose estimation research
* Privacy-preserving motion monitoring
* Contactless vital sign tracking
* Environmental topology modeling
* RF field behavior research

---

# Limitations

* Requires static node geometry
* Sensitive to large environmental rearrangements
* Not medical-grade cardiac monitoring
* Performance depends on mesh synchronization discipline

---

# ESP32-S3 CSI Node Firmware


## Firmware Files

| File                  | Size   | Flash Address | Description                                                         |
| --------------------- | ------ | ------------- | ------------------------------------------------------------------- |
| `bootloader.bin`      | 21 KB  | `0x0`         | ESP32-S3 second-stage bootloader                                    |
| `partition-table.bin` | 3 KB   | `0x8000`      | Flash partition layout                                              |
| `esp32-csi-node.bin`  | 789 KB | `0x10000`     | CSI firmware with MAC filtering + TDM support                       |
| `provision.py`        | —      | —             | NVS provisioning utility (WiFi + network + MAC filter + TDM config) |

---

# Quick Start (No Build Required)

## 1. Install esptool

```bash
pip install esptool
```

---

## 2. Flash Firmware

```bash
python -m esptool --chip esp32s3 --port COM7 --baud 460800 \
  --before default-reset --after hard-reset \
  write-flash --flash-mode dio --flash-freq 80m --flash-size 4MB \
  0x0 bootloader.bin \
  0x8000 partition-table.bin \
  0x10000 esp32-csi-node.bin
```

Replace:

* `COM7` → your serial port

  * Linux: `/dev/ttyUSB0`
  * macOS: `/dev/cu.usbserial-*`

---

## 3. Provision WiFi Credentials (No Rebuild Required)

```bash
python provision.py --port COM7 \
  --ssid "YourWiFi" --password "YourPassword" \
  --target-ip 192.168.1.20 --target-port 5005
```

Provisioning is written to NVS. Reflashing is not required for configuration changes.

---

# Run the Aggregator

### Option 1 — Native (Rust)

From the WiFi DensePose sensing server:

```bash
cargo run -p wifi-densepose-sensing-server -- \
  --http-port 3000 --source esp32
```

### Option 2 — Docker

```bash
docker run -p 3000:3000 -p 5005:5005/udp \
  ruvnet/wifi-densepose:latest \
  --source esp32
```

The aggregator listens for UDP CSI packets on port `5005`.

---

# MAC Address Filtering

To prevent CSI mixing across multiple access points, filter to a single AP.

### Runtime (No Reflash)

```bash
python provision.py --port COM7 \
  --filter-mac "AA:BB:CC:DD:EE:FF"
```

### Compile-Time (Kconfig)

Set:

```
CONFIG_CSI_FILTER_MAC="AA:BB:CC:DD:EE:FF"
```

in `sdkconfig.defaults`.

---

# TDM Mesh Setup (Multistatic Mode)

For 3+ node deployments:

### Node 0

```bash
python provision.py --port COM7 \
  --ssid "WiFi" --password "pass" \
  --target-ip 192.168.1.20 \
  --tdm-slot 0 --tdm-total 3
```

### Node 1

```bash
python provision.py --port COM8 \
  --tdm-slot 1 --tdm-total 3
```

### Node 2

```bash
python provision.py --port COM9 \
  --tdm-slot 2 --tdm-total 3
```

Each node transmits CSI frames only during its assigned time slot.

This prevents RF contention and enables deterministic multistatic fusion.

---

# Hardware Compatibility

| Board              | Status           | Notes                       |
| ------------------ | ---------------- | --------------------------- |
| ESP32-S3-DevKitC-1 | Verified         | Tested with CP210x USB-UART |
| ESP32-S3-WROOM-1   | Expected         | Same SoC                    |
| ESP32-S3-MINI-1    | Expected         | Same SoC                    |
| ESP32 (Classic)    | Rebuild Required | `idf.py set-target esp32`   |

---

# Firmware Capabilities

* WiFi STA + promiscuous mode CSI capture
* ADR-018 binary serialization

  * 20-byte header
  * Raw I/Q payload
* UDP streaming to configurable aggregator
* MAC address filtering (runtime + compile-time)
* TDM mesh time-slot protocol (ADR-029 / ADR-031)
* Channel hopping with configurable dwell time
* NVS runtime configuration:

  * SSID
  * Password
  * Target IP/Port
  * Node ID
  * MAC filter
  * TDM parameters
* ~20 Hz CSI frame rate (LLTF + HT-LTF + STBC HT-LTF2)
* 64–128 subcarriers per frame (bandwidth dependent)
* Automatic WiFi reconnection (up to 10 retries)

---

# Deployment Notes

* Maintain fixed node geometry for stable field modeling.
* Use a dedicated 2.4 GHz channel where possible.
* Disable aggressive router band steering.
* Baseline capture after installation improves coherence gating stability.

---

If you want this tightened further, I can:

* Merge it cleanly into your full README structure.
* Separate firmware into `/firmware/README.md`.
* Or convert this into a production-grade installation guide.

# Ethical & Privacy Notice

EtherSense operates through passive RF reflection modeling.
Deployment must comply with local privacy and data regulations.

The system is intended for research and controlled environments.
Do not deploy in private spaces without informed consent.

---

# Citation

If you build upon EtherSense in academic or commercial work, cite:

```
EtherSense: Coherence-Gated Multistatic WiFi Field Modeling for Pose and Vital Sign Reconstruction.
```

