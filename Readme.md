# Hardware Switzerland: Passive Hardware Behavior Attestation

## The Core Principle

> **"Monitor hardware conversations without reading the mail."**

Hardware Switzerland is a neutral observer architecture that validates hardware behavior without participating in it. It sits between components (CPU↔Storage, CPU↔GPU, etc.) and logs only **metadata**: who talked to whom, when, and about what addresses—**never** the actual data.

## The Problem

Modern systems have layered security:
- **Data encrypted** (TPM, drive encryption)
- **Software verified** (secure boot, code signing)
- **Networks firewalled**

But we lack **hardware behavior verification**. We cannot answer:
- Did a malicious USB device DMA into memory?
- Did a compromised NIC whisper to the SSD?
- Did firmware get touched during maintenance?

We encrypt the letters but don't watch who's handling the envelopes.

## The Solution

A passive hardware monitor that:
1. **Sits physically between components** (PCIe interposer, SATA tap)
2. **Logs only metadata**: source, destination, timestamp, address ranges
3. **Never decrypts or interprets data**
4. **Creates admissible evidence** of hardware-level anomalies

### What It Sees

[2024-01-15T14:30:00.123456Z]
Source: PCIe BDF 02:00.0 (Network Card)
Destination: NVMe Controller 03:00.0
Operation: READ
Address: 0xE0003000-0xE0003008 (SMART data)
Anomaly: Network card shouldn't read drive health data



### What It Never Sees
- File contents
- Encryption keys  
- User data
- Application information

## Technical Implementation

### Basic Version (Proof of Concept)

[SSD] ← [FPGA Monitor] ← [Motherboard]
↓
[USB Serial Output → Logs]


**FPGA Logic:**
- PCIe/SATA physical layer snooping
- Command header extraction (source, destination, LBA/address)
- Timestamp with nanosecond precision
- Simple rule engine: "Alert if NIC accesses SSD"

### Production Version
- Multi-lane PCIe 4.0/5.0 interposer
- Onboard secure element for log integrity
- Network reporting (IPMI, Redfish)
- Correlation across multiple monitors

## Why "Switzerland"?

### Neutrality
- Not in the TPM chain
- Not in the encryption path  
- Not in the trust boundary
- Just observes electrons flow

### Universal Value
- **Users:** Privacy preserved (no data access)
- **IT Teams:** Attack evidence collected
- **Vendors:** Security claims verified
- **Auditors:** Compliance proven

## Detection Capabilities

1. **Supply Chain Attacks**
   - Unexpected device communication
   - Firmware region accesses
   - Hidden backdoor activation

2. **DMA Attacks**
   - Thunderbolt/USB4 device memory access
   - Peer-to-peer suspicious communication

3. **Covert Channels**
   - Abnormal timing patterns
   - "Health check" encoded signaling
   - Steganography in legitimate protocols

4. **Insider Threats**
   - Unauthorized hardware access
   - After-hours physical tampering
   - Rogue device installation

## Privacy & Legal Advantages

### Not a Wiretap
- Only observes envelope, not letter
- Analogous to logging phone call timestamps vs recording audio
- Already legal precedent for metadata collection

### Compliance Friendly
- **GDPR:** Doesn't process personal data
- **HIPAA:** Doesn't access PHI  
- **PCI DSS:** Additional intrusion detection layer
- **SOX:** Hardware access audit trail

## Use Cases

### Enterprise Data Centers
- Rack-level hardware integrity monitoring
- Multi-tenant isolation verification
- Forensic readiness for hardware attacks

### High-Security Facilities
- Air-gap monitoring (watches the actual gap)
- Secure room entry/exit correlation
- Supply chain verification

### Cloud Providers
- Customer hardware isolation proof
- Hypervisor escape detection
- Physical security compliance

### Security Research
- Hardware vulnerability study
- Attack pattern capture
- Defense evaluation

## Comparison with Existing Solutions

| Solution | What It Does | Hardware Switzerland |
|----------|--------------|---------------------|
| **TPM/Encryption** | Protects data at rest | Proves protection wasn't bypassed |
| **Secure Boot** | Verifies software | Watches hardware during boot |
| **Antivirus** | Scans files | Logs hardware access attempts |
| **Network IDS** | Monitors packets | Monitors PCIe/SATA bus |
| **DRM/Watermarking** | Tracks content | Tracks hardware conversations |

## Getting Involved

### For Hardware Engineers
- FPGA implementation (Verilog/VHDL)
- PCIe/SATA PHY integration
- Timing analysis optimization

### For Security Researchers
- Attack pattern signatures
- Anomaly detection algorithms
- Threat model development

### For Legal Experts
- Wiretap law analysis
- Compliance documentation
- International deployment guidance

### For System Integrators
- Deployment methodologies
- Management console development
- Enterprise integration patterns

## Repository Structure

/hardware-switzerland/
├── README.md # This document
├── ABOUT-THIS-PROJECT.md # Project origin story
├── specs/ # Technical specifications
│ ├── pcie-monitor.md
│ ├── sata-monitor.md
│ └── protocol-decoder.md
├── hardware/ # FPGA designs
│ ├── verilog/
│ └── constraints/
├── firmware/ # Monitor firmware
├── software/ # Analysis tools
│ ├── log-parser/
│ └── anomaly-detector/
├── legal/ # Compliance analysis
└── research/ # Academic papers, attack patterns


## Next Steps

1. **Proof of Concept**: Simple FPGA between NVMe drive and motherboard
2. **Legal Review**: Confirm wiretap law compliance in key jurisdictions
3. **Standardization Proposal**: Submit to OCP, NIST, TCG
4. **Reference Implementation**: Open source FPGA design + software

## FAQ

**Q: Isn't this a hardware backdoor?**  
A: It's the opposite—it detects backdoors. It has no access to data, no encryption keys, and can be openly inspected (FPGA code is public).

**Q: Will this slow down my system?**  
A: Passive tap adds nanoseconds of signal propagation delay—imperceptible. No processing in the data path.

**Q: Can attackers disable it?**  
A: Physical removal would be detected (gap in logs). Tampering would require physical access equivalent to already having won.

**Q: Why not just use existing logs?**  
A: OS logs can be tampered with. Hardware logs survive OS wipe, persist in pre-boot, and see DMA attacks that bypass OS entirely.

**Q: How is this different from BMC monitoring?**  
A: BMCs are complex, often proprietary, and themselves attack targets. This is simple, open, and outside the trust boundary.

## License

This concept and associated designs are released under **CERN Open Hardware License Version 2 - Permissive** to encourage broad adoption and collaboration.

## Contact & Discussion

- **GitHub Issues**: For technical discussion
- **Twitter**: [#HardwareSwitzerland](https://twitter.com/search?q=%23HardwareSwitzerland)

---

*"In a world of encrypted data and verified code, we watch the hardware conversations that shouldn't be happening."*