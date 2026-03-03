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

