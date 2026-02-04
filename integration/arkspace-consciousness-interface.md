# ArkSpace ↔ TheConsciousness.ai/core Interface Specification

**Version**: 1.0.0  
**Status**: Draft  
**Last Updated**: January 2026

---

## Overview

This document defines the interface between **ArkSpace.me** (Infrastructure Layer) and **TheConsciousness.ai/core** (Software Layer). This interface governs how the SNN Engine and Neural Firewall software runs on the satellite neuromorphic hardware.

> **Note**: [theconsciousness.ai](https://theconsciousness.ai) (main) covers generic Artificial Consciousness research.  
> **[theconsciousness.ai/core](https://theconsciousness.ai/core)** is the specific Consciousness Substrate Transfer project integrated with ArkSpace and MindTransfer.

---

## Interface Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                                                                  │
│   TheConsciousness.ai                              ArkSpace.me                   │
│   (Software Layer)                                (Infrastructure Layer)         │
│                                                                                  │
│   ┌─────────────────────────────┐                 ┌─────────────────────────┐   │
│   │      SNN ENGINE             │                 │    SATELLITE NODE       │   │
│   │  ┌───────────────────────┐  │                 │  ┌───────────────────┐  │   │
│   │  │  Leaky Integrate-and- │  │                 │  │   Neuromorphic    │  │   │
│   │  │   Fire (LIF) Models   │  │                 │  │   Processor       │  │   │
│   │  └───────────┬───────────┘  │                 │  │   (Loihi-class)   │  │   │
│   │              │              │                 │  └─────────┬─────────┘  │   │
│   │  ┌───────────▼───────────┐  │                 │            │            │   │
│   │  │  Synaptic Weight      │  │   DEPLOYMENT   │  ┌─────────▼─────────┐  │   │
│   │  │  Matrices             │──┼────────────────►│  │  SNN Runtime      │  │   │
│   │  └───────────┬───────────┘  │                 │  │  Environment      │  │   │
│   │              │              │                 │  └─────────┬─────────┘  │   │
│   │  ┌───────────▼───────────┐  │                 │            │            │   │
│   │  │  STDP Learning Rules  │  │                 │  ┌─────────▼─────────┐  │   │
│   │  └───────────────────────┘  │                 │  │  OISL Router      │  │   │
│   └─────────────────────────────┘                 │  └───────────────────┘  │   │
│                                                   └─────────────────────────┘   │
│                                                                                  │
│   ┌─────────────────────────────┐                 ┌─────────────────────────┐   │
│   │      GENERATIVE MODEL       │                 │    GROUND SEGMENT       │   │
│   │  ┌───────────────────────┐  │                 │  ┌───────────────────┐  │   │
│   │  │  Predictive Coding    │  │                 │  │   Edge Node       │  │   │
│   │  │  Network              │  │   TELEMETRY    │  │   (Neural         │  │   │
│   │  └───────────┬───────────┘  │◄────────────────┼──│    Firewall)      │  │   │
│   │              │              │                 │  └───────────────────┘  │   │
│   │  ┌───────────▼───────────┐  │                 │                         │   │
│   │  │  Prediction Error     │  │                 │  ┌───────────────────┐  │   │
│   │  │  Minimization         │  │                 │  │   Ground Station  │  │   │
│   │  └───────────────────────┘  │                 │  │   (RF/Optical)    │  │   │
│   └─────────────────────────────┘                 │  └───────────────────┘  │   │
│                                                   └─────────────────────────┘   │
│                                                                                  │
│   ┌─────────────────────────────┐                                               │
│   │      NEURAL FIREWALL        │                                               │
│   │  ┌───────────────────────┐  │                                               │
│   │  │  Anomaly Detection    │  │                                               │
│   │  │  (ML-based)           │  │                                               │
│   │  └───────────┬───────────┘  │                                               │
│   │              │              │                                               │
│   │  ┌───────────▼───────────┐  │         SECURITY POLICIES                     │
│   │  │  Traffic Filtering    │──┼────────────────────────────────────────►      │
│   │  │  Rules                │  │         (Deployed to Edge & Satellite)        │
│   │  └───────────┬───────────┘  │                                               │
│   │              │              │                                               │
│   │  ┌───────────▼───────────┐  │                                               │
│   │  │  Kill Switch Logic    │  │                                               │
│   │  └───────────────────────┘  │                                               │
│   └─────────────────────────────┘                                               │
│                                                                                  │
└──────────────────────────────────────────────────────────────────────────────────┘
```

---

## Deployment Architecture

### SNN Engine Deployment

The SNN Engine from TheConsciousness.ai is compiled and deployed to ArkSpace satellite nodes.

```yaml
deployment:
  source: TheConsciousness.ai SNN Engine
  target: ArkSpace Satellite Neuromorphic Payload
  
  artifacts:
    - name: snn_model
      format: nengo_compiled
      description: Compiled LIF network weights and topology
      
    - name: learning_rules
      format: stdp_config
      description: Spike-timing-dependent plasticity parameters
      
    - name: generative_model
      format: predictive_coding_weights
      description: Hierarchical predictive model for bandwidth optimization
      
  deployment_method:
    primary: ground_station_upload
    backup: oisl_relay
    verification: checksum_sha256
```

### Neural Firewall Deployment

```yaml
firewall_deployment:
  locations:
    - edge_node: Primary enforcement
    - satellite: Secondary enforcement
    - ground_station: Logging and audit
    
  artifacts:
    - name: anomaly_detector
      format: onnx_model
      description: ML model for detecting malicious patterns
      
    - name: filter_rules
      format: yaml
      description: Traffic filtering policies
      
    - name: kill_switch_logic
      format: embedded_firmware
      description: Hardware-level emergency cutoff
```

---

## Data Interfaces

### 1. Neural State Interface

Transfer of SNN state between satellite nodes.

```yaml
interface: neural_state
direction: satellite_to_satellite
protocol: OISL-Neural-v1

message_format:
  header:
    source_node_id: uint8
    destination_node_id: uint8
    timestamp_ns: uint64
    state_version: uint32
    checksum: uint32
    
  payload:
    neuron_states:
      count: uint32
      data:
        - neuron_id: uint32
          membrane_potential_mV: float32
          refractory_remaining_us: uint16
          last_spike_time_ns: uint64
          
    synapse_states:
      count: uint64
      data:
        - synapse_id: uint64
          pre_neuron_id: uint32
          post_neuron_id: uint32
          weight: float32
          eligibility_trace: float32

bandwidth_requirement: 10-60 Gbps
max_latency_ms: 10
compression: LZ4
encryption: AES-256-GCM
```

### 2. Spike Stream Interface

Real-time spike events for distributed SNN computation.

```yaml
interface: spike_stream
direction: bidirectional
protocol: OISL-Neural-v1

message_format:
  header:
    session_id: uint64
    source_node_id: uint8
    timestamp_ns: uint64
    sequence: uint32
    
  payload:
    spike_events:
      - neuron_id: uint32
        timestamp_offset_us: uint16
      # Variable length array, typically 1000-10000 events per packet

packet_rate: 10,000-100,000 packets/second
bandwidth_requirement: 1-10 Gbps
max_latency_ms: 5
encryption: AES-256-GCM
```

### 3. Prediction Error Interface

Generative model output for bandwidth optimization.

```yaml
interface: prediction_error
direction: brain_to_satellite
protocol: OISL-Neural-v1

message_format:
  header:
    session_id: uint64
    timestamp_ns: uint64
    prediction_frame_id: uint32
    
  payload:
    # Only send prediction errors, not full state
    error_vector:
      dimension: uint32
      values: float16[]  # Sparse representation
    
    # Prediction confidence for adaptive coding
    confidence: float16

compression_ratio: 10:1 to 100:1 (vs raw spikes)
bandwidth_requirement: 10-100 Mbps
max_latency_ms: 10
```

### 4. Security Event Interface

Neural Firewall alerts and logging.

```yaml
interface: security_event
direction: edge_to_cloud
protocol: HTTPS/TLS 1.3

message_format:
  header:
    event_id: uuid
    timestamp_ns: uint64
    severity: enum(INFO, WARNING, CRITICAL, EMERGENCY)
    source: enum(EDGE_NODE, SATELLITE, GROUND_STATION)
    
  payload:
    event_type: enum(
      ANOMALY_DETECTED,
      PATTERN_BLOCKED,
      RATE_LIMIT_EXCEEDED,
      AUTH_FAILURE,
      KILL_SWITCH_ACTIVATED
    )
    
    details:
      description: string
      affected_session_id: uint64
      blocked_pattern: bytes  # If applicable
      action_taken: string
      
response_time_critical: <100ms
response_time_warning: <1s
storage: persistent_log
```

---

## API Contracts

### SNN Deployment API

```yaml
openapi: 3.0.0
info:
  title: ArkSpace SNN Deployment API
  version: 1.0.0
  
paths:
  /nodes/{node_id}/snn:
    post:
      summary: Deploy SNN model to satellite node
      requestBody:
        content:
          application/octet-stream:
            schema:
              type: string
              format: binary
              description: Compiled SNN model binary
      parameters:
        - name: node_id
          in: path
          required: true
          schema:
            type: string
        - name: model_version
          in: query
          required: true
          schema:
            type: string
        - name: checksum
          in: query
          required: true
          schema:
            type: string
      responses:
        202:
          description: Deployment initiated
          content:
            application/json:
              schema:
                type: object
                properties:
                  deployment_id: string
                  estimated_completion_seconds: integer
                  
    get:
      summary: Get current SNN model status
      responses:
        200:
          description: Model status
          content:
            application/json:
              schema:
                type: object
                properties:
                  model_version: string
                  deployed_at: string
                  neuron_count: integer
                  synapse_count: integer
                  utilization_percent: number
                  
  /nodes/{node_id}/snn/state:
    get:
      summary: Get current SNN state snapshot
      parameters:
        - name: format
          in: query
          schema:
            type: string
            enum: [full, summary, sparse]
      responses:
        200:
          description: State snapshot
          
    put:
      summary: Load SNN state (for migration/restore)
      requestBody:
        content:
          application/octet-stream:
            schema:
              type: string
              format: binary
```

### Telemetry API

```yaml
paths:
  /nodes/{node_id}/telemetry:
    get:
      summary: Get satellite node telemetry
      responses:
        200:
          description: Current telemetry
          content:
            application/json:
              schema:
                type: object
                properties:
                  timestamp: string
                  neuromorphic_processor:
                    temperature_celsius: number
                    power_watts: number
                    utilization_percent: number
                    neuron_active_count: integer
                    spikes_per_second: integer
                  oisl:
                    connected_nodes: array
                    bandwidth_gbps: number
                    latency_ms: number
                  power_system:
                    battery_percent: number
                    solar_input_watts: number
                  thermal:
                    payload_temperature_celsius: number
                    
  /nodes/{node_id}/telemetry/stream:
    get:
      summary: Subscribe to telemetry stream (WebSocket)
      description: Real-time telemetry updates via WebSocket
```

### Neural Firewall API

```yaml
paths:
  /firewall/rules:
    get:
      summary: Get current firewall rules
      responses:
        200:
          description: Active rules
          
    put:
      summary: Update firewall rules
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                rules:
                  type: array
                  items:
                    type: object
                    properties:
                      rule_id: string
                      pattern: string
                      action: string
                      priority: integer
                      
  /firewall/anomalies:
    get:
      summary: Get detected anomalies
      parameters:
        - name: since
          in: query
          schema:
            type: string
            format: date-time
        - name: severity
          in: query
          schema:
            type: string
            enum: [INFO, WARNING, CRITICAL]
      responses:
        200:
          description: Anomaly list
          
  /firewall/kill-switch:
    post:
      summary: Activate kill switch
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                session_id: string
                reason: string
                scope: string
                  enum: [SESSION, NODE, CONSTELLATION]
      responses:
        200:
          description: Kill switch activated
```

---

## Computational Requirements

### Neuromorphic Payload Specifications

| Parameter | Requirement | Notes |
|-----------|-------------|-------|
| Neuron Capacity | 100M - 1B neurons | Per satellite node |
| Synapse Capacity | 100B - 1T synapses | Per satellite node |
| Spike Processing | 10B spikes/second | Real-time constraint |
| Power Budget | 50-200W | Satellite power limited |
| Thermal | -20 to +60C operational | Space environment |

### SNN Engine Requirements

| Component | Requirement |
|-----------|-------------|
| LIF Neuron Model | 32-bit float precision minimum |
| Time Resolution | 1ms biological time per 1ms wall clock |
| STDP Updates | Per-spike or per-epoch configurable |
| State Checkpointing | Every 1 second |
| Migration Time | <100ms for full state transfer |

### Generative Model Requirements

| Parameter | Value |
|-----------|-------|
| Prediction Horizon | 50-200ms |
| Hierarchical Levels | 3-5 layers |
| Error Encoding | Sparse, float16 |
| Compression Target | 10:1 minimum |

---

## State Synchronization

### Multi-Node SNN Distribution

When consciousness spans multiple satellite nodes:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DISTRIBUTED SNN TOPOLOGY                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│     Node A (Primary)           Node B (Secondary)        Node C (Backup)│
│     ┌─────────────┐            ┌─────────────┐          ┌────────────┐  │
│     │  Cortical   │◄──OISL───►│  Cortical   │◄──OISL──►│  Cortical  │  │
│     │  Regions    │  60 Gbps  │  Regions    │  60 Gbps │  Mirror    │  │
│     │  1-4        │  <10ms    │  5-8        │          │  (Hot      │  │
│     └─────────────┘            └─────────────┘          │   Standby) │  │
│           │                          │                  └────────────┘  │
│           │                          │                                   │
│           ▼                          ▼                                   │
│     ┌───────────────────────────────────────────┐                       │
│     │         GLOBAL STATE MANAGER               │                       │
│     │  • Spike routing between nodes             │                       │
│     │  • State consistency verification          │                       │
│     │  • Failover coordination                   │                       │
│     │  • Latency monitoring                      │                       │
│     └───────────────────────────────────────────┘                       │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

### State Migration Protocol

```yaml
state_migration:
  trigger:
    - satellite_handoff
    - load_balancing
    - node_failure
    - scheduled_maintenance
    
  procedure:
    1_prepare:
      - freeze_learning: true
      - buffer_incoming_spikes: true
      - checkpoint_state: true
      
    2_transfer:
      - compress_state: LZ4
      - encrypt_state: AES-256-GCM
      - transmit_via: OISL
      - verify_checksum: SHA-256
      
    3_activate:
      - load_state: true
      - verify_integrity: true
      - replay_buffered_spikes: true
      - resume_learning: true
      - confirm_to_source: true
      
    4_cleanup:
      - release_source_resources: true
      - update_routing_tables: true
      
  timing:
    max_freeze_duration_ms: 100
    max_transfer_duration_ms: 50
    max_total_duration_ms: 200
    
  failure_handling:
    transfer_failed: rollback_to_source
    verification_failed: retry_transfer
    activation_failed: alert_and_manual_review
```

---

## Security Integration

### Encryption Pipeline

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ENCRYPTION PIPELINE                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   CONSCIOUSNESS DATA                                                     │
│        │                                                                 │
│        ▼                                                                 │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │  LAYER 1: AES-256-GCM (Real-Time)                                │   │
│   │  • All spike streams                                              │   │
│   │  • All state transfers                                            │   │
│   │  • Latency: <1ms                                                  │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│        │                                                                 │
│        ▼                                                                 │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │  LAYER 2: Homomorphic Encryption (Identity/Handshake)            │   │
│   │  • Session establishment                                          │   │
│   │  • Key exchange                                                   │   │
│   │  • Identity verification                                          │   │
│   │  • Latency: 100-500ms (acceptable for non-realtime)              │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│        │                                                                 │
│        ▼                                                                 │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │  LAYER 3: TEE Enclave (On-Satellite)                             │   │
│   │  • Key storage                                                    │   │
│   │  • Decryption operations                                          │   │
│   │  • Isolated from main processor                                   │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

### Threat Detection

| Threat | Detection Method | Response |
|--------|------------------|----------|
| Seizure-inducing patterns | Frequency analysis | Immediate block |
| Unauthorized motor commands | Pattern matching | Block + alert |
| Replay attacks | Sequence verification | Reject + log |
| Man-in-the-middle | Certificate verification | Terminate session |
| Denial of service | Rate limiting | Throttle + alert |
| Data exfiltration | Traffic analysis | Block + investigate |

---

## Performance Requirements

### Latency Budget (ArkSpace ↔ Consciousness)

| Component | Target | Maximum |
|-----------|--------|---------|
| SNN spike processing | 1ms | 5ms |
| OISL inter-node | 5ms | 10ms |
| State synchronization | 10ms | 20ms |
| Encryption/Decryption | 0.5ms | 1ms |
| **Total per-hop** | **16.5ms** | **36ms** |

### Throughput Requirements

| Data Type | Minimum | Typical | Peak |
|-----------|---------|---------|------|
| Inter-node spikes | 1 Gbps | 10 Gbps | 60 Gbps |
| State snapshots | 100 Mbps | 1 Gbps | 10 Gbps |
| Telemetry | 1 Mbps | 10 Mbps | 100 Mbps |
| Security events | 100 Kbps | 1 Mbps | 10 Mbps |

### Availability

| Metric | Target |
|--------|--------|
| Single node uptime | 99.9% |
| Constellation availability | 99.99% |
| Failover time | <500ms |
| State recovery time | <1 second |

---

## Implementation Responsibilities

### TheConsciousness.ai Provides

- SNN Engine compiled for neuromorphic hardware
- Generative Model for prediction error encoding
- Neural Firewall software and rule sets
- Anomaly detection ML models
- Kill switch logic specification

### ArkSpace.me Provides

- Neuromorphic processor hardware
- OISL communication infrastructure
- TEE secure enclave
- Ground station connectivity
- Telemetry and monitoring systems

### Shared Responsibilities

- Interface protocol definition
- State migration procedures
- Security testing and auditing
- Performance optimization
- Failure recovery procedures

---

## Testing Requirements

### Integration Tests

1. **SNN Deployment**: Verify model upload and activation
2. **State Migration**: Verify <200ms migration with no data loss
3. **Spike Routing**: Verify correct inter-node spike delivery
4. **Failover**: Verify automatic recovery on node failure
5. **Security**: Verify encryption and firewall enforcement

### Performance Tests

1. **Latency**: Verify <20ms per-hop under load
2. **Throughput**: Verify 60 Gbps sustained OISL
3. **Scalability**: Verify 3+ node constellation
4. **Endurance**: Verify 24-hour continuous operation

### Security Tests

1. **Penetration**: Attempt unauthorized access
2. **Anomaly Detection**: Verify malicious pattern blocking
3. **Kill Switch**: Verify <100ms emergency shutdown
4. **Encryption**: Verify data confidentiality

---

*Interface specification maintained jointly by ArkSpace.me and TheConsciousness.ai teams*
