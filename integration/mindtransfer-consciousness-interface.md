# MindTransfer ↔ TheConsciousness.ai/core Interface Specification

**Version**: 1.0.0  
**Status**: Draft  
**Last Updated**: January 2026

---

## Overview

This document defines the interface between **MindTransfer.me** (Interface Layer) and **TheConsciousness.ai/core** (Software Layer). This interface handles the translation between biological neural signals and the SNN computational substrate, including encryption, encoding, and safety mechanisms.

> **Note**: [theconsiousness.ai](https://theconsiousness.ai) (main) covers generic Artificial Consciousness research.  
> **[theconsiousness.ai/core](https://theconsiousness.ai/core)** is the specific Consciousness Substrate Transfer project integrated with ArkSpace and MindTransfer.

---

## Interface Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                                                                  │
│   MindTransfer.me                              TheConsciousness.ai               │
│   (Interface Layer)                           (Software Layer)                   │
│                                                                                  │
│   ┌─────────────────────────────┐                 ┌─────────────────────────┐   │
│   │  CORPUS CALLOSUM INTERFACE  │                 │     SNN ENGINE          │   │
│   │  ┌───────────────────────┐  │                 │  ┌───────────────────┐  │   │
│   │  │  CMOS Micro-Electrode │  │                 │  │  Input Encoding   │  │   │
│   │  │  Array (Read/Write)   │  │                 │  │  (Spike → LIF)    │  │   │
│   │  └───────────┬───────────┘  │                 │  └─────────┬─────────┘  │   │
│   │              │              │                 │            │            │   │
│   │  ┌───────────▼───────────┐  │                 │  ┌─────────▼─────────┐  │   │
│   │  │  Spike Detection &    │  │   SPIKE        │  │  Hemisphere       │  │   │
│   │  │  Feature Extraction   │──┼───STREAM──────►│  │  Emulation        │  │   │
│   │  └───────────┬───────────┘  │                 │  │  (Synthetic)      │  │   │
│   │              │              │                 │  └─────────┬─────────┘  │   │
│   │  ┌───────────▼───────────┐  │                 │            │            │   │
│   │  │  Sub-cranial         │  │                 │  ┌─────────▼─────────┐  │   │
│   │  │  Transceiver (UWB)   │  │   STIM          │  │  Output Decoding  │  │   │
│   │  └───────────────────────┘◄─┼───COMMANDS─────┼──│  (LIF → Stim)     │  │   │
│   └─────────────────────────────┘                 │  └───────────────────┘  │   │
│                                                   └─────────────────────────┘   │
│                                                                                  │
│   ┌─────────────────────────────┐                 ┌─────────────────────────┐   │
│   │  SIGNAL PROCESSING          │                 │   GENERATIVE MODEL      │   │
│   │  ┌───────────────────────┐  │                 │  ┌───────────────────┐  │   │
│   │  │  Analog-to-Digital    │  │                 │  │  Predictive       │  │   │
│   │  │  Conversion (ADC)     │  │                 │  │  Coding Network   │  │   │
│   │  └───────────┬───────────┘  │                 │  └─────────┬─────────┘  │   │
│   │              │              │                 │            │            │   │
│   │  ┌───────────▼───────────┐  │                 │  ┌─────────▼─────────┐  │   │
│   │  │  Spike Sorting &      │  │   PREDICTION   │  │  Error Signal     │  │   │
│   │  │  Neuron Identification│◄─┼───FEEDBACK─────┼──│  Generation       │  │   │
│   │  └───────────────────────┘  │                 │  └───────────────────┘  │   │
│   └─────────────────────────────┘                 └─────────────────────────┘   │
│                                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │                        NEURAL FIREWALL                                    │   │
│   │                 (Shared Component - Runs at Edge Node)                    │   │
│   │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌───────────┐ │   │
│   │  │  Encrypt    │───►│  Validate   │───►│  Anomaly    │───►│  Safety   │ │   │
│   │  │  (AES-256)  │    │  (Schema)   │    │  Detect     │    │  Enforce  │ │   │
│   │  └─────────────┘    └─────────────┘    └─────────────┘    └───────────┘ │   │
│   └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                  │
└──────────────────────────────────────────────────────────────────────────────────┘
```

---

## Data Flows

### 1. Uplink: Biological Brain → SNN Engine

Reading neural activity from the corpus callosum and encoding for SNN processing.

| Step | Source | Destination | Data Type | Bandwidth | Latency |
|------|--------|-------------|-----------|-----------|---------|
| 1 | Axons | CMOS Electrodes | Action potentials | N/A | <0.1ms |
| 2 | CMOS Array | ADC | Analog voltages | N/A (on-chip) | <0.5ms |
| 3 | ADC | Spike Detector | Digital samples | 2-20 Gbps | <0.5ms |
| 4 | Spike Detector | Feature Encoder | Detected spikes | 100 Mbps | <1ms |
| 5 | Feature Encoder | Neural Firewall | Encoded stream | 100 Mbps | <1ms |
| 6 | Neural Firewall | SNN Engine | Encrypted stream | 100 Mbps | <1ms |

### 2. Downlink: SNN Engine → Biological Brain

Stimulating the biological hemisphere with motor/sensory commands from the synthetic hemisphere.

| Step | Source | Destination | Data Type | Bandwidth | Latency |
|------|--------|-------------|-----------|-----------|---------|
| 1 | SNN Engine | Neural Firewall | Motor commands | 10-100 Mbps | <1ms |
| 2 | Neural Firewall | Safety Check | Filtered commands | 10-100 Mbps | <1ms |
| 3 | Safety Check | Stim Encoder | Verified commands | 10-100 Mbps | <1ms |
| 4 | Stim Encoder | DAC | Stimulation patterns | N/A (on-chip) | <0.5ms |
| 5 | DAC | CMOS Electrodes | Charge injection | N/A | <0.5ms |
| 6 | CMOS Electrodes | Axons | Electrical stimulation | N/A | <0.1ms |

---

## Message Types

### 1. Biological Spike Stream

Raw neural activity from biological hemisphere to SNN.

```yaml
message_type: BIO_SPIKE_STREAM
direction: BRAIN_TO_SNN
priority: REALTIME

format:
  header:
    session_id: uint64
    timestamp_ns: uint64
    sequence: uint32
    electrode_config_hash: uint32
    
  payload:
    spike_events:
      - electrode_id: uint16      # Which electrode detected
        channel_id: uint8         # Electrode sub-channel
        neuron_id: uint32         # Sorted neuron identity
        timestamp_offset_us: uint16
        amplitude_uV: int16
        waveform_features: float16[8]  # Extracted spike shape
        confidence: float16       # Spike sorting confidence
        
    lfp_summary:                  # Local field potential context
      - region_id: uint8
        power_spectrum: float16[8]  # Frequency band powers
        phase: float16

encoding: protobuf
compression: none (real-time constraint)
encryption: AES-256-GCM
max_latency_ms: 5
packet_rate: 10,000-50,000 per second
```

### 2. SNN Stimulation Commands

Motor/sensory commands from SNN to biological hemisphere.

```yaml
message_type: SNN_STIM_COMMAND
direction: SNN_TO_BRAIN
priority: REALTIME

format:
  header:
    session_id: uint64
    timestamp_ns: uint64
    sequence: uint32
    safety_signature: bytes[32]  # Signed by Neural Firewall
    
  payload:
    stimulation_events:
      - electrode_id: uint16
        target_neuron_type: enum(EXCITATORY, INHIBITORY)
        amplitude_uA: int16       # Microamps
        pulse_width_us: uint16
        waveform: enum(
          MONOPHASIC_CATHODIC,
          MONOPHASIC_ANODIC,
          BIPHASIC_CATHODIC_FIRST,
          BIPHASIC_ANODIC_FIRST,
          CHARGE_BALANCED_SYMMETRIC
        )
        inter_pulse_interval_us: uint16
        train_duration_ms: uint8
        
    timing:
      execution_time_ns: uint64   # When to execute (future)
      max_delay_tolerance_ms: uint8

encoding: protobuf
compression: none
encryption: AES-256-GCM
max_latency_ms: 5

safety_limits:
  max_amplitude_uA: 100
  max_charge_per_phase_nC: 50
  max_frequency_hz: 200
  max_duty_cycle_percent: 50
  min_inter_train_interval_ms: 10
```

### 3. Prediction Feedback

Generative model predictions sent back to optimize encoding.

```yaml
message_type: PREDICTION_FEEDBACK
direction: SNN_TO_INTERFACE
priority: HIGH

format:
  header:
    session_id: uint64
    timestamp_ns: uint64
    prediction_frame_id: uint32
    
  payload:
    predicted_spikes:
      - neuron_id: uint32
        predicted_time_us: uint32
        confidence: float16
        
    error_corrections:
      - neuron_id: uint32
        prediction_error: float16
        suggested_gain: float16
        
    encoding_parameters:
      compression_hint: float16   # How much to compress
      priority_neurons: uint32[]  # Focus on these

purpose: |
  The SNN builds a model of expected biological activity.
  When predictions match, MindTransfer can compress/skip redundant data.
  Only prediction errors need full transmission (bandwidth optimization).
```

### 4. Electrode Mapping Update

Configuration for which electrodes map to which synthetic neurons.

```yaml
message_type: ELECTRODE_MAPPING
direction: BIDIRECTIONAL
priority: CONFIG

format:
  header:
    session_id: uint64
    timestamp_ns: uint64
    mapping_version: uint32
    
  payload:
    mappings:
      - biological_electrode_id: uint16
        biological_neuron_id: uint32
        biological_region: enum(
          MOTOR_CORTEX,
          SENSORY_CORTEX,
          VISUAL_CORTEX,
          AUDITORY_CORTEX,
          PREFRONTAL,
          LIMBIC,
          OTHER
        )
        synthetic_neuron_id: uint32
        synthetic_layer: uint8
        connection_strength: float16
        bidirectional: boolean
        
    calibration:
      last_calibration_ns: uint64
      signal_quality: float16
      recommended_gains: float16[]

update_frequency: On-demand or daily recalibration
```

---

## Neural Encoding/Decoding

### Biological → Synthetic Translation

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    BIOLOGICAL TO SYNTHETIC ENCODING                              │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│   BIOLOGICAL INPUT                                                               │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │  Action Potential Waveform (from CMOS electrode)                         │   │
│   │                                                                          │   │
│   │        ▲ Voltage (mV)                                                    │   │
│   │    +40 │      ╭───╮                                                      │   │
│   │        │     ╱     ╲                                                     │   │
│   │      0 │────╱───────╲────────────────                                    │   │
│   │        │  ╱           ╲              (threshold ~-55mV)                  │   │
│   │    -70 │─╱─────────────╲─────────────                                    │   │
│   │        └──────────────────────────────► Time (ms)                        │   │
│   │           0    1    2    3    4                                          │   │
│   └─────────────────────────────────────────────────────────────────────────┘   │
│                              │                                                   │
│                              ▼                                                   │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │  SPIKE DETECTION & FEATURE EXTRACTION                                    │   │
│   │                                                                          │   │
│   │  1. Bandpass filter (300Hz - 6kHz)                                       │   │
│   │  2. Threshold crossing detection                                         │   │
│   │  3. Waveform extraction (1-2ms window)                                   │   │
│   │  4. Feature extraction (PCA/wavelets)                                    │   │
│   │  5. Spike sorting (cluster assignment)                                   │   │
│   └─────────────────────────────────────────────────────────────────────────┘   │
│                              │                                                   │
│                              ▼                                                   │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │  SYNTHETIC LIF ENCODING                                                  │   │
│   │                                                                          │   │
│   │  For each detected biological spike:                                     │   │
│   │  1. Map electrode → synthetic neuron (via electrode_mapping)             │   │
│   │  2. Inject current: I_inj = amplitude × gain × dt                        │   │
│   │  3. Update membrane: dV/dt = (V_rest - V + I_inj) / tau_m                │   │
│   │  4. If V > threshold: emit spike, reset to V_reset                       │   │
│   │                                                                          │   │
│   │  Result: Biological spike train → Synthetic LIF spike train              │   │
│   └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                  │
└──────────────────────────────────────────────────────────────────────────────────┘
```

### Synthetic → Biological Translation

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    SYNTHETIC TO BIOLOGICAL DECODING                              │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│   SYNTHETIC OUTPUT (SNN Motor Command)                                          │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │  LIF Neuron Spike → Stimulation Pattern                                  │   │
│   │                                                                          │   │
│   │  Spike train: │ │  │ ││ │  │  │ │ │                                     │   │
│   │               ─┴─┴──┴─┴┴─┴──┴──┴─┴─┴──► Time                            │   │
│   └─────────────────────────────────────────────────────────────────────────┘   │
│                              │                                                   │
│                              ▼                                                   │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │  SAFETY VALIDATION (Neural Firewall)                                     │   │
│   │                                                                          │   │
│   │  ✓ Check amplitude < 100 µA                                              │   │
│   │  ✓ Check frequency < 200 Hz (no seizure risk)                            │   │
│   │  ✓ Check charge balance (cathodic = anodic)                              │   │
│   │  ✓ Check duty cycle < 50%                                                │   │
│   │  ✓ Verify cryptographic signature                                        │   │
│   │  ✓ Check against known malicious patterns                                │   │
│   └─────────────────────────────────────────────────────────────────────────┘   │
│                              │                                                   │
│                              ▼                                                   │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │  CHARGE-BALANCED WAVEFORM GENERATION                                     │   │
│   │                                                                          │   │
│   │        ▲ Current (µA)                                                    │   │
│   │    +50 │  ┌────┐                      ┌────┐                             │   │
│   │        │  │    │                      │    │                             │   │
│   │      0 │──┘    └──────────────────────┘    └────────                     │   │
│   │        │            ┌────┐                      ┌────┐                   │   │
│   │    -50 │            │    │                      │    │                   │   │
│   │        └────────────┴────┴──────────────────────┴────┴──► Time           │   │
│   │                                                                          │   │
│   │  Biphasic charge-balanced pulses prevent tissue damage                   │   │
│   └─────────────────────────────────────────────────────────────────────────┘   │
│                              │                                                   │
│                              ▼                                                   │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │  BIOLOGICAL EFFECT                                                       │   │
│   │                                                                          │   │
│   │  Electrical stimulation → Axon depolarization →                          │   │
│   │  Action potential generation in target biological neurons                │   │
│   │                                                                          │   │
│   │  Net effect: SNN motor intention → Biological motor execution            │   │
│   └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                  │
└──────────────────────────────────────────────────────────────────────────────────┘
```

---

## Safety Mechanisms

### Stimulation Safety Limits

```yaml
safety_limits:
  # Per-pulse limits
  max_amplitude_uA: 100
  max_pulse_width_us: 500
  max_charge_per_phase_nC: 50
  
  # Frequency limits (seizure prevention)
  max_frequency_hz: 200
  forbidden_frequencies_hz:
    - range: [3, 30]      # Theta/Alpha/Beta seizure risk
      max_amplitude_uA: 20  # Reduced amplitude allowed
    - range: [50, 60]     # Line noise interference
      forbidden: true
      
  # Duty cycle limits (tissue heating prevention)
  max_duty_cycle_percent: 50
  min_inter_train_interval_ms: 10
  max_continuous_stimulation_s: 60
  
  # Charge density limits (electrode damage prevention)
  max_charge_density_uC_per_cm2: 30
  
  # Total session limits
  max_total_charge_per_hour_uC: 10000
  mandatory_rest_period_after_hour_min: 10
```

### Kill Switch Integration

```yaml
kill_switch:
  hardware_layer:
    location: Sub-cranial transceiver
    mechanism: Physical relay disconnect
    response_time_ms: <1
    triggers:
      - voltage_out_of_range
      - current_overcurrent
      - temperature_exceeded
      - watchdog_timeout
      
  software_layer:
    location: Edge Node Neural Firewall
    response_time_ms: <10
    triggers:
      - safety_limit_violation
      - anomaly_detected
      - authentication_failure
      - heartbeat_timeout
      - manual_emergency_button
      
  recovery_procedure:
    requires:
      - manual_reset_confirmation
      - safety_check_passed
      - user_consent_reconfirmed
      - 30_second_cooldown
```

### Anomaly Detection

```yaml
anomaly_detection:
  models:
    - name: seizure_pattern_detector
      type: CNN
      input: spike_frequency_spectrum
      threshold: 0.95
      action: immediate_block
      
    - name: unauthorized_motor_detector
      type: RNN
      input: motor_command_sequence
      baseline: user_movement_profile
      deviation_threshold: 3_sigma
      action: block_and_alert
      
    - name: replay_attack_detector
      type: sequence_analysis
      input: packet_sequences
      window_ms: 1000
      similarity_threshold: 0.9
      action: reject_and_log
      
  baseline_learning:
    initial_period_hours: 24
    update_frequency: daily
    requires_user_confirmation: true
```

---

## API Contracts

### Session Management API

```yaml
openapi: 3.0.0
info:
  title: MindTransfer-Consciousness Session API
  version: 1.0.0
  
paths:
  /sessions:
    post:
      summary: Initialize neural session
      requestBody:
        content:
          application/json:
            schema:
              type: object
              required:
                - user_id
                - interface_id
                - snn_endpoint
              properties:
                user_id: string
                interface_id: string
                snn_endpoint: string
                electrode_mapping_version: string
                safety_profile: string
      responses:
        201:
          description: Session created
          content:
            application/json:
              schema:
                type: object
                properties:
                  session_id: string
                  encryption_keys:
                    uplink_key: string
                    downlink_key: string
                  calibration_required: boolean
                  estimated_latency_ms: number
                  
  /sessions/{session_id}/calibrate:
    post:
      summary: Run electrode calibration
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                calibration_type: string
                  enum: [full, quick, impedance_only]
                duration_seconds: integer
      responses:
        200:
          description: Calibration complete
          content:
            application/json:
              schema:
                type: object
                properties:
                  electrode_status: array
                  signal_quality_scores: array
                  recommended_mappings: array
                  
  /sessions/{session_id}/stream:
    get:
      summary: WebSocket endpoint for real-time streaming
      description: |
        Bidirectional WebSocket for spike streams and stimulation commands.
        Protocol: Binary frames with protobuf encoding.
```

### Safety API

```yaml
paths:
  /safety/limits:
    get:
      summary: Get current safety limits
      
    put:
      summary: Update safety limits (requires elevated auth)
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/SafetyLimits'
              
  /safety/emergency-stop:
    post:
      summary: Trigger emergency stop
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                session_id: string
                reason: string
      responses:
        200:
          description: Emergency stop executed
          
  /safety/anomalies:
    get:
      summary: Get detected anomalies
      parameters:
        - name: session_id
          in: query
          schema:
            type: string
        - name: since
          in: query
          schema:
            type: string
            format: date-time
```

---

## Performance Requirements

### Latency Budget

| Component | Target | Maximum | Notes |
|-----------|--------|---------|-------|
| Spike detection | 0.5ms | 1ms | On-chip processing |
| Feature extraction | 0.5ms | 1ms | On-chip processing |
| Encryption | 0.5ms | 1ms | AES-256-GCM |
| UWB transmission | 1ms | 2ms | To edge node |
| Firewall validation | 0.5ms | 1ms | Edge node |
| **Total (one direction)** | **3ms** | **6ms** | |
| **Round trip** | **6ms** | **12ms** | Interface only |

### Bandwidth Requirements

| Data Type | Minimum | Typical | Maximum |
|-----------|---------|---------|---------|
| Spike stream (uplink) | 10 Mbps | 100 Mbps | 500 Mbps |
| Stim commands (downlink) | 1 Mbps | 10 Mbps | 100 Mbps |
| Calibration data | 100 Kbps | 1 Mbps | 10 Mbps |
| Telemetry | 10 Kbps | 100 Kbps | 1 Mbps |

### Signal Quality Requirements

| Metric | Minimum | Target |
|--------|---------|--------|
| Spike detection SNR | 5:1 | 10:1 |
| Spike sorting accuracy | 80% | 95% |
| Electrode impedance | 100kΩ - 1MΩ | 200-500kΩ |
| Channel crosstalk | <-40dB | <-60dB |

---

## Implementation Responsibilities

### MindTransfer.me Provides

- CMOS electrode array hardware
- Spike detection and sorting algorithms
- Sub-cranial transceiver
- UWB communication to edge node
- Local safety enforcement (hardware kill switch)
- Electrode calibration routines

### TheConsciousness.ai Provides

- SNN input encoding algorithms
- Stimulation pattern generation
- Neural Firewall software
- Anomaly detection models
- Generative model for prediction feedback
- Safety limit definitions

### Shared Responsibilities

- Electrode mapping protocol
- Calibration procedures
- Safety testing and validation
- Interface protocol specification
- Encryption key management

---

## Testing Requirements

### Unit Tests

1. **Spike Detection**: Verify >95% detection rate on synthetic data
2. **Encoding Accuracy**: Verify biological → synthetic fidelity
3. **Decoding Safety**: Verify all outputs pass safety checks
4. **Encryption**: Verify correct encrypt/decrypt cycle

### Integration Tests

1. **End-to-End Latency**: Verify <12ms round trip
2. **Bandwidth Saturation**: Verify no packet loss at peak bandwidth
3. **Safety Enforcement**: Verify malicious patterns blocked
4. **Failover**: Verify graceful degradation on component failure

### Safety Tests

1. **Kill Switch Response**: Verify <10ms hardware disconnect
2. **Anomaly Detection**: Verify known attack patterns detected
3. **Charge Limits**: Verify stimulation limits enforced
4. **Seizure Patterns**: Verify forbidden frequencies blocked

### Biocompatibility Tests

(Per medical device regulatory requirements)

1. **Electrode Impedance**: Long-term stability
2. **Tissue Response**: Inflammation markers
3. **Signal Quality**: Degradation over time

---

*Interface specification maintained jointly by MindTransfer.me and TheConsciousness.ai teams*
