# ARK Space Week - Sprint Roadmap

**Sprint Duration**: 7 Days
**Focus**: ArkSpace.me Infrastructure Layer Setup
**Status**: Completed

---

## Sprint Objectives

1. ~~Establish Zae GitHub organization~~ → **Done**: [Zae-Project](https://github.com/Zae-Project)
2. ~~Clean up arkspace-me organization~~ → **Done**
3. ~~Create arkspace-core repository~~ → **Done**: [arkspace-core](https://github.com/Zae-Project/arkspace-core)
4. Document Exocortex Constellation architecture
5. Define OISL protocol requirements
6. ~~Create integration interfaces with other projects~~ → **Done**: see [integration/](../integration/)

---

## Day-by-Day Breakdown

### Day 1: Organization Setup

**Goal**: Establish Zae organization and repository structure

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| Create Zae GitHub organization | [x] | - | [Zae-Project](https://github.com/Zae-Project) |
| Create zae-docs repository | [x] | - | [zae-docs](https://github.com/Zae-Project/zae-docs) |
| Move brain_emulation to Zae | [x] | - | [brain-emulation](https://github.com/Zae-Project/brain-emulation) |
| Move neutral-consciousness-engine to Zae | [x] | - | [neutral-consciousness-engine](https://github.com/Zae-Project/neutral-consciousness-engine) |
| Create arkspace-core in Zae | [x] | - | [arkspace-core](https://github.com/Zae-Project/arkspace-core) |

**Deliverables**:
- [x] Zae organization created
- [x] All repositories in Zae-Project
- [x] Team access configured

---

### Day 2: ArkSpace Cleanup & New Structure

**Goal**: Clean arkspace-me org and create arkspace-core repo (**Completed**)

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| Archive/delete COMET-* repos | [ ] | - | Not relevant to Exocortex |
| Archive/delete ardupilot fork | [ ] | - | Drone autopilot, different use |
| Archive nanosat-mo-framework | [ ] | - | Generic framework |
| Keep oresat-* repos for reference | [ ] | - | Useful CubeSat patterns |
| Create arkspace-core repo (private) | [ ] | - | Fresh start |
| Update .github org README | [ ] | - | New mission statement |

**arkspace-me Cleanup Decision Matrix**:

| Repository | Action | Reason |
|------------|--------|--------|
| COMET-SDK-Community-Edition | DELETE | ECSS tools, not needed |
| COMET-IME-Community-Edition | DELETE | Desktop app, not needed |
| COMET-SDKJ-Community-Edition | DELETE | Java SDK, not needed |
| COMET-WebServices-Community-Edition | DELETE | Web services, not needed |
| ardupilot | DELETE | Drone autopilot |
| nanosat-mo-framework | ARCHIVE | Generic, may reference |
| oresat-c3-hardware | KEEP | CubeSat C3 reference |
| oresat-c3-software | KEEP | Satellite software patterns |
| oresat-olaf | KEEP | Linux board framework |
| oresat-configs | KEEP | Config patterns |
| oresat-firmware | KEEP | Firmware reference |
| getting-started | UPDATE | Change to Exocortex context |
| .github | UPDATE | New README |

**Deliverables**:
- [ ] arkspace-me cleaned (6+ repos removed)
- [ ] arkspace-core created
- [ ] New mission statement published

---

### Day 3: Exocortex Constellation Architecture

**Goal**: Document satellite infrastructure specifications

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| Define constellation topology | [ ] | - | Min 3 nodes |
| Specify orbital parameters | [ ] | - | LEO 400-600km |
| Document OISL requirements | [ ] | - | 60+ Gbps, <20ms |
| Define SNN payload specs | [ ] | - | Neuromorphic compute |
| Create power budget analysis | [ ] | - | Solar + SNN load |

**Documents to Create**:
```
arkspace-core/
├── docs/
│   ├── exocortex-constellation.md    ← Day 3
│   ├── oisl-requirements.md          ← Day 3
│   ├── snn-payload-specs.md          ← Day 3
│   └── power-budget.md               ← Day 3
```

**Deliverables**:
- [ ] Exocortex Constellation spec complete
- [ ] OISL requirements documented
- [ ] SNN payload defined

---

### Day 4: OISL Protocol for Neural Data

**Goal**: Define communication protocol for neural data transmission

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| Define neural data packet format | [ ] | - | Spike encoding |
| Specify latency requirements | [ ] | - | <50ms RTT |
| Document bandwidth calculations | [ ] | - | 2-20 Gbps corpus callosum |
| Create error handling spec | [ ] | - | Redundancy, retry |
| Define handshake protocol | [ ] | - | Authentication |

**Key Specifications**:

| Parameter | Value | Justification |
|-----------|-------|---------------|
| Target RTT | <50ms | Libet buffer (350ms) margin |
| Bandwidth | 60+ Gbps | Corpus callosum equivalent |
| Encoding | Spike trains | Bio-compatible format |
| Encryption | AES-256 stream | Real-time requirement |
| Redundancy | Triple modular | Safety critical |

**Deliverables**:
- [ ] OISL-Neural protocol spec v0.1
- [ ] Packet format definition
- [ ] Latency budget breakdown

---

### Day 5: Integration Interfaces

**Goal**: Define how ArkSpace connects to MindTransfer and Consciousness projects

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| Create ArkSpace ↔ MindTransfer interface | [ ] | - | Uplink/downlink |
| Create ArkSpace ↔ Consciousness interface | [ ] | - | SNN state transfer |
| Define API contracts | [ ] | - | YAML/Protobuf |
| Document security handshake | [ ] | - | Key exchange |

**Interface Documents**:
```
zae-docs/integration/
├── arkspace-mindtransfer-interface.md
├── arkspace-consciousness-interface.md
└── api-contracts.yaml
```

**Deliverables**:
- [ ] Interface specs complete
- [ ] API contracts defined
- [ ] Security protocols documented

---

### Day 6: Repository Structure & Documentation

**Goal**: Finalize arkspace-core repository structure

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| Create final folder structure | [ ] | - | See below |
| Write comprehensive README | [ ] | - | Mission, setup, contribute |
| Add CONTRIBUTING.md | [ ] | - | Guidelines |
| Add LICENSE | [ ] | - | Proprietary or MIT |
| Create issue templates | [ ] | - | Bug, feature, research |

**arkspace-core Final Structure**:
```
arkspace-core/
├── README.md
├── CONTRIBUTING.md
├── LICENSE
├── .github/
│   ├── ISSUE_TEMPLATE/
│   └── workflows/
├── docs/
│   ├── architecture/
│   │   ├── exocortex-constellation.md
│   │   ├── satellite-node.md
│   │   └── ground-segment.md
│   ├── protocols/
│   │   ├── oisl-neural-protocol.md
│   │   └── latency-budget.md
│   ├── hardware/
│   │   ├── snn-payload.md
│   │   └── power-systems.md
│   └── integration/
│       ├── mindtransfer-interface.md
│       └── consciousness-interface.md
├── specs/
│   ├── satellite-node.yaml
│   ├── constellation.yaml
│   └── oisl-protocol.yaml
├── reference/
│   └── research-papers.md
└── roadmap/
    └── milestones.md
```

**Deliverables**:
- [ ] arkspace-core fully structured
- [ ] All docs in place
- [ ] Ready for development

---

### Day 7: Review & Publishing

**Goal**: Final review and publish all changes

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| Review all documentation | [ ] | - | Consistency check |
| Verify cross-references | [ ] | - | Links work |
| Test repository access | [ ] | - | Permissions correct |
| Update public websites | [ ] | - | arkspace.me if needed |
| Create milestone issues | [ ] | - | Next sprint prep |

**Final Checklist**:
- [x] Zae-Project org fully operational
- [x] zae-docs complete with architecture
- [x] arkspace-core repository ready
- [x] arkspace-me cleaned and updated
- [x] All projects linked and referenced
- [ ] Next steps defined

---

## Success Criteria

### Must Have (Day 7)
- [x] Unified architecture document created
- [x] Zae-Project organization exists
- [x] arkspace-core repository created
- [x] arkspace-me cleaned of irrelevant forks
- [x] Basic documentation in place

### Should Have (Day 7)
- [ ] OISL protocol spec v0.1
- [x] Integration interfaces defined
- [x] All 3 projects linked in Zae-Project

### Nice to Have (Day 7)
- [ ] API contracts in YAML/Protobuf
- [ ] Issue templates configured
- [ ] CI/CD workflows started

---

## Post-Sprint: Phase 1 Foundation

After ARK Space Week, the following phases continue:

### Phase 1 (Month 1-2): Deep Technical Specs
- Detailed satellite node architecture
- Neuromorphic hardware selection
- Ground station requirements
- Regulatory research (ITU, FCC)

### Phase 2 (Month 3-4): Prototype Planning
- SNN payload simulation
- OISL link budget analysis
- Power system modeling
- Cost estimation

### Phase 3 (Month 5-6): Integration Testing
- Software interface prototypes
- Latency testing frameworks
- Security protocol implementation
- End-to-end simulation

---

## Resources

### Reference Repositories
- OreSat: [github.com/oresat](https://github.com/oresat) - Open source CubeSat
- SatNOGS: [satnogs.org](https://satnogs.org) - Ground station network
- Starlink: [starlink.com](https://starlink.com) - OISL reference

### Key Documents
- [Unified Platform Architecture](../architecture/unified-platform-architecture.md)
- [Technical Glossary](../glossary/technical-glossary.md)

### Contacts
- ArkSpace Lead: TBD
- MindTransfer Lead: TBD
- Consciousness Lead: TBD

---

*Last Updated: January 2026*
