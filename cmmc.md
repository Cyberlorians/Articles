# CMMC v2 ↔ Zero Trust Mapping (CISA ZTMM + DoD ZT)

| CMMC v2 / NIST 800-171 Rev.2 Practice | Domain Summary | Zero Trust Pillar | Microsoft / DoD Guidance | CISA Reference | DoD Reference |
|---------------------------------------|----------------|-------------------|--------------------------|----------------|---------------|
| AC.L2-3.1.x — limit access, least privilege, session control | Access Control (AC) | Identity / Governance | Conditional Access, PIM, JIT Access | Identity 1.1 (Access & Authorization) | User 1.1 (Authenticate & Authorize) |
| IA.L2-3.5.x — MFA, unique IDs, credential management | Identification & Authentication (IA) | Identity / Device | MFA, CBA/PIV, device compliance | Identity 2.1 (MFA & Credential Hygiene) | User 2.1 (Authentication Enforcement) |
| AU.L2-3.3.x — logging, reviews, protections | Audit & Accountability (AU) | Visibility & Analytics | Sentinel logs, XDR telemetry, anomaly detection | Visibility 1.1 (Monitoring & Analysis) | Visibility & Analytics 1.2 (Audit & Monitor) |
| CM.L2-3.4.x — baselines, change control | Configuration Management (CM) | Device / Workload | Intune baselines, Azure Policy, Defender hardening | Devices 1.2 (Config Enforcement) | Device 1.3 (Baseline Enforcement) |
| IR.L2-3.6.x — incident handling, reporting, testing | Incident Response (IR) | Automation & Orchestration | Sentinel SOAR, auto-isolation, IR playbooks | Automation 1.1 (Orchestration) | Automation 2.1 (Incident Containment) |
| SC.L2-3.13.x — boundaries, encryption, segmentation | System & Communications Protection (SC) | Network / Data / Workload | Microsegmentation, TLS, Firewall, DLP | Networks 3.1 (Segmentation & Filtering) | Network 2.2 (Data Flow Control) |
| RA.L2-3.11.x — scans, remediation | Risk Assessment (RA) | Governance / Visibility | Defender VM, Secure Score, Compliance Manager | Visibility 2.1 (Risk Analytics) | Governance 1.1 (Risk Mgmt Activities) |
| CA.L2-3.12.x — assessments, POA&M | Security Assessment (CA) | Governance / Automation | Compliance Manager templates, attack simulation | Governance 1.1 (Assess & Validate) | Governance 2.1 (Assessment Activities) |
| SI.L2-3.14.x — malware defense, patching | System Integrity (SI) | Device / Workload | Defender AV/EDR, ASR, patch automation | Devices 2.1 (Integrity & Protection) | Device 2.2 (Integrity Verification) |
| MP.L2-3.8.x — removable media, sanitization | Media Protection (MP) | Data / Device | BitLocker, removable storage controls, Purview labels | Data 2.1 (Protection & Control) | Data 2.2 (Encryption & Control) |
| MA.L2-3.7.x — controlled maintenance | Maintenance (MA) | Device / Governance | Intune update enforcement, maintenance logs | Devices 1.3 (Lifecycle Mgmt) | Device 3.1 (Maintenance Activities) |
| PS.L2-3.9.x — screening, offboarding | Personnel Security (PS) | Identity / Governance | Lifecycle mgmt, Entra provisioning | Identity 3.1 (Lifecycle Mgmt) | User 3.2 (Identity Lifecycle) |
| PE.L2-3.10.x — physical access | Physical Protection (PE) | Governance / Policy | Facility access policies, integration with IT ZT | Governance 3.1 (Policy & Oversight) | Governance 4.1 (Physical Security) |
| AT.L2-3.2.x — training programs | Awareness & Training (AT) | Governance / Policy | Awareness & training programs, MS Learn | Governance 2.1 (Training & Awareness) | Governance 3.1 (Culture & Training) |
