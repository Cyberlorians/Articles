# 📘 CMMC 2.0 Level 1 — Foundational (17 Practices)

| CMMC Practice ID | NIST Ref | CISA ZTMM Mapping | DoD ZT Pillar | Microsoft Zero Trust Guidance |
|------------------|----------|-------------------|---------------|-------------------------------|
| AC.L1-3.1.1 Limit system access to authorized users | 3.1.1 | Identity 1.1 (Basic auth) | Identity | [DoD ZT Strategy - Identity](https://learn.microsoft.com/en-us/security/zero-trust/dod-zero-trust-strategy-intro#identity) |
| AC.L1-3.1.2 Limit system access to processes acting on behalf of authorized users | 3.1.2 | Workloads 1.1 (App/service auth) | Applications & Workloads | [Zero Trust Workload Protection](https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity-access) |
| AC.L1-3.1.20 Verify and control/limit connections to external systems | 3.1.20 | Network 1.1 (Boundary enforcement) | Networks/Environment | [ZT - Network Segmentation](https://learn.microsoft.com/en-us/security/zero-trust/network-segmentation) |
| AC.L1-3.1.22 Control information posted or processed on publicly accessible systems | 3.1.22 | Data 1.1 (Basic protections for external data) | Data | [Microsoft Purview DLP](https://learn.microsoft.com/en-us/purview/dlp-learn-about-dlp) |
| IA.L1-3.5.1 Identify users before granting access | 3.5.1 | Identity 1.2 (Authentication) | Identity | [Microsoft Entra Authentication](https://learn.microsoft.com/en-us/azure/active-directory/authentication/) |
| IA.L1-3.5.2 Authenticate users using multifactor where possible | 3.5.2 | Identity 2.1 (MFA adoption) | Identity | [Microsoft Entra MFA](https://learn.microsoft.com/en-us/azure/active-directory/authentication/concept-mfa-howitworks) |
| MP.L1-3.8.3 Sanitize or destroy media before disposal | 3.8.3 | Data 1.3 (Data lifecycle) | Data | [Microsoft Purview - Information Protection](https://learn.microsoft.com/en-us/microsoft-365/compliance/information-protection?view=o365-worldwide) |
| MP.L1-3.8.4 Mark media with necessary CUI/safeguarding markings | 3.8.4 | Data 1.2 (Data classification) | Data | [Microsoft Purview - Sensitivity Labels](https://learn.microsoft.com/en-us/microsoft-365/compliance/sensitivity-labels?view=o365-worldwide) |
| PE.L1-3.10.1 Limit physical access to organizational systems, equipment, and environments | 3.10.1 | Devices 1.1 (Access enforcement) | Devices | [ZT Devices Guidance](https://learn.microsoft.com/en-us/security/zero-trust/devices-overview) |
| PE.L1-3.10.3 Escort visitors and monitor visitor activity | 3.10.3 | Devices 1.2 (Physical safeguards) | Devices | [Physical Security in ZT](https://csrc.nist.gov/publications/detail/sp/800-171/rev-2/final) |
| PE.L1-3.10.4 Maintain audit logs of physical access | 3.10.4 | Visibility & Analytics 1.1 | Visibility & Analytics | [Microsoft Sentinel - Physical/Logical Logs](https://learn.microsoft.com/en-us/azure/sentinel/) |
| PE.L1-3.10.5 Control and manage physical access devices | 3.10.5 | Devices 1.3 (Access control mechanisms) | Devices | [Microsoft Endpoint Manager - Access Control](https://learn.microsoft.com/en-us/mem/endpoint-manager-overview) |
| SC.L1-3.13.1 Monitor, control, and protect organizational communications (external and internal) | 3.13.1 | Network 1.2 (Traffic inspection, control) | Networks/Environment | [Microsoft Defender for Cloud - Network Security](https://learn.microsoft.com/en-us/azure/defender-for-cloud/) |
| SC.L1-3.13.5 Implement subnetworks for publicly accessible system components | 3.13.5 | Network 1.3 (Segmentation, zoning) | Networks/Environment | [Azure Virtual Network Segmentation](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview) |
| SI.L1-3.14.1 Identify, report, and correct information and system flaws in a timely manner | 3.14.1 | Visibility & Analytics 1.2 (Vulnerability visibility) | Visibility & Analytics | [Defender Vulnerability Management](https://learn.microsoft.com/en-us/microsoft-365/security/defender-vulnerability-management) |
| SI.L1-3.14.2 Provide protection from malicious code at system boundaries and endpoints | 3.14.2 | Devices 1.4 (Malware protection) | Devices | [Microsoft Defender Antivirus](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/microsoft-defender-antivirus) |
| SI.L1-3.14.5 Perform periodic scans and real-time protection of files | 3.14.5 | Devices 1.5 (Endpoint scanning) | Devices | [Microsoft Defender for Endpoint](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint) |

---

# 🔑 Key
- **CMMC Practice ID** → Practice from CMMC 2.0 Level 1  
- **NIST Ref** → 800-171 Rev.2 source requirement  
- **CISA ZTMM Mapping** → Zero Trust pillar + maturity reference  
- **DoD ZT Pillar** → One of the 7 DoD Zero Trust Strategy pillars  
- **Microsoft Guidance** → Implementation reference in MSFT Zero Trust docs  
