# CMMC v2 ↔ Zero Trust Mapping (CISA ZTMM + DoD ZT)

| CMMC Domain | Representative Controls | Zero Trust Pillar | Microsoft / DoD Guidance | CISA Reference | DoD Reference |
|-------------|--------------------------|-------------------|--------------------------|----------------|---------------|
| **Access Control (AC)** | AC.L2-3.1.x — limit access, least privilege, session control | Identity / Governance | Conditional Access, PIM, JIT Access | Identity 1.1 (Access & AuthN) | User 1.1 (Authenticate & Authorize) |
| **Identification & Authentication (IA)** | IA.L2-3.5.x — MFA, unique IDs, credential mgmt | Identity / Device | MFA, CBA/PIV, device compliance | Identity 2.1 (MFA & Credential Hygiene) | User 2.1 (Authentication Enforcement) |
| **Audit & Accountability (AU)** | AU.L2-3.3.x — logging, reviews, protections | Visibility & Analytics | Sentinel logs, XDR telemetry, anomaly detection | Visibility 1.1 (Monitoring & Analysis) | Visibility & Analytics 1.2 (Audit & Monitor) |
| **Configuration Management (CM)** | CM.L2-3.4.x — baselines, change control | Device / Workload | Intune baselines, Azure Policy, Defender hardening | Devices 1.2 (Config Enforcement) | Device 1.3 (Baseline Enforcement) |
| **Incident Response (IR)** | IR.L2-3.6.x — handling, reporting, testing | Automation & Orchestration | Sentinel SOAR, auto-isolation, IR playbooks | Automation 1.1 (Orchestration) | Automation 2.1 (Incident Containment) |
| **System & Communications Protection (SC)** | SC.L2-3.13.x — boundaries, encryption, segmentation | Network / Data / Workload | Microsegmentation, TLS, Firewall, DLP | Networks 3.1 (Segmentation & Filtering) | Network 2.2 (Data Flow Control) |
| **Risk Assessment (RA)** | RA.L2-3.11.x — scans, remediation | Governance / Visibility | Defender VM, Secure Score, Compliance Manager | Visibility 2.1 (Risk Analytics) | Governance 1.1 (Risk Mgmt Activities) |
| **Security Assessment (CA)** | CA.L2-3.12.x — assessments, POA&M | Governance / Automation | Compliance Manager CMMC templates, attack simulation | Governance 1.1 (Assess & Validate) | Governance 2.1 (Assessment Activities) |
| **System Integrity (SI)** | SI.L2-3.14.x — malware defense, patching | Device / Workload | Defender AV/EDR, ASR, patch automation | Devices 2.1 (Integrity & Protection) | Device 2.2 (Integrity Verification) |
| **Media Protection (MP)** | MP.L2-3.8.x — removable media, sanitization | Data / Device | BitLocker, removable storage controls, Purview labels | Data 2.1 (Protection & Control) | Data 2.2 (Encryption & Control) |
| **Maintenance (MA)** | MA.L2-3.7.x — controlled maintenance | Device / Governance | Intune update enforcement, maintenance logs | Devices 1.3 (Lifecycle Mgmt) | Device 3.1 (Maintenance Activities) |
| **Personnel Security (PS)** | PS.L2-3.9.x — screening, offboarding | Identity / Governance | Lifecycle mgmt, Entra provisioning | Identity 3.1 (Lifecycle Mgmt) | User 3.2 (Identity Lifecycle) |
| **Physical Protection (PE)** | PE.L2-3.10.x — physical access | Governance / Policy | Facility access policies, integration with IT ZT | Governance 3.1 (Policy & Oversight) | Governance 4.1 (Physical Security) |
| **Awareness & Training (AT)** | AT.L2-3.2.x — training programs | Governance / Policy | Awareness & training programs, MS Learn | Governance 2.1 (Training & Awareness) | Governance 3.1 (Culture & Training) |
