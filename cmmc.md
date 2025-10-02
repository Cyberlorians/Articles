# CMMC 2.0 → Zero Trust (CISA & DoD) → Microsoft Mapping

This document aligns **CMMC 2.0 practices** with **Zero Trust frameworks** (CISA ZTMM + DoD ZT Strategy) and shows **Microsoft Zero Trust guidance** references.  

---

## 📘 Level 1 — Foundational (17 Practices)

| CMMC Practice ID | NIST Ref (Rev.2/3) | CISA ZTMM Mapping | DoD ZT Pillar | Microsoft Zero Trust Guidance |
|------------------|--------------------|-------------------|---------------|-------------------------------|
| AC.L1-3.1.1 Limit system access to authorized users | NIST 800-171 Rev.2: 3.1.1 | Identity 1.1 (Basic auth), Identity 2.1 (MFA adoption) | Identity | [DoD ZT Strategy - Identity Pillar](https://learn.microsoft.com/en-us/security/zero-trust/dod-zero-trust-strategy-intro#identity) |
| AC.L1-3.1.2 Limit system access to processes acting on behalf of authorized users | NIST 800-171 Rev.2: 3.1.2 | Workloads 1.1 | Applications & Workloads | [MSFT Zero Trust Workload Protection](https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity-access) |
| AC.L1-3.1.20 Verify and control/limit connections to external systems | NIST 800-171 Rev.2: 3.1.20 | Network/Environment 1.1 | Networks/Environment | [Microsoft ZT - Network Segmentation](https://learn.microsoft.com/en-us/security/zero-trust/network-segmentation) |
| IA.L1-3.5.1 Identify users before granting access | NIST 800-171 Rev.2: 3.5.1 | Identity 1.2 (Strong auth), Identity 2.3 (CBA/MFA) | Identity | [Microsoft Entra ID Authentication](https://learn.microsoft.com/en-us/azure/active-directory/authentication/) |
| AU.L1-3.3.1 Create and retain system audit logs | NIST 800-171 Rev.2: 3.3.1 | Visibility & Analytics 1.1 | Visibility & Analytics | [Microsoft Sentinel + M-21-31](https://learn.microsoft.com/en-us/azure/sentinel/) |

---

## 📗 Level 2 — Advanced (110 Practices)

| CMMC Practice ID | NIST Ref (Rev.2/3) | CISA ZTMM Mapping | DoD ZT Pillar | Microsoft Zero Trust Guidance |
|------------------|--------------------|-------------------|---------------|-------------------------------|
| AC.L2-3.1.3 Control the flow of CUI in information systems | NIST 800-171 Rev.3: 3.1.3 | Data 1.1 | Data | [MS Purview Data Loss Prevention](https://learn.microsoft.com/en-us/purview/dlp-learn-about-dlp) |
| AC.L2-3.1.5 Employ least privilege | NIST 800-171 Rev.2: 3.1.5 | Identity 2.2 (Least privilege), Automation 2.1 | Identity / Automation | [Microsoft Entra PIM](https://learn.microsoft.com/en-us/azure/active-directory/privileged-identity-management/) |
| SC.L2-3.13.8 Implement cryptographic mechanisms to prevent unauthorized disclosure | NIST 800-171 Rev.3: 3.13.8 | Data 2.1 (Encryption) | Data | [Microsoft Defender for Cloud - Encryption Controls](https://learn.microsoft.com/en-us/azure/security/fundamentals/encryption-overview) |
| IR.L2-3.6.1 Establish an incident-handling capability | NIST 800-171 Rev.2: 3.6.1 | Visibility & Analytics 2.3 | Visibility & Analytics | [Microsoft Sentinel SOAR](https://learn.microsoft.com/en-us/azure/sentinel/automate-incident-handling-with-playbooks) |

---

## 📙 Level 3 — Expert (NIST 800-172, Advanced Practices)

| CMMC Practice ID | NIST Ref (Rev.172) | CISA ZTMM Mapping | DoD ZT Pillar | Microsoft Zero Trust Guidance |
|------------------|--------------------|-------------------|---------------|-------------------------------|
| AC.L3-172.1 Employ enhanced identity verification techniques | NIST 800-172: 3.5.x | Identity 3.1 (Adaptive auth, risk-based) | Identity | [Microsoft Entra Conditional Access + Risk-based Policies](https://learn.microsoft.com/en-us/azure/active-directory/conditional-access/) |
| IR.L3-172.2 Respond to advanced persistent threat events | NIST 800-172: 3.6.x | Visibility & Analytics 3.2 | Visibility & Analytics | [Microsoft Defender XDR + Threat Intelligence](https://learn.microsoft.com/en-us/microsoft-365/security/defender-xdr/) |
| SC.L3-172.3 Implement out-of-band management to isolate compromised systems | NIST 800-172: 3.13.x | Network/Environment 3.1 | Networks/Environment | [Microsoft ZT - Network Containment](https://learn.microsoft.com/en-us/security/zero-trust/network-segmentation) |

---

# 🔑 Key
- **CMMC Practice ID** → The official practice from CMMC 2.0  
- **NIST Ref** → NIST 800-171 Rev. 2 / Rev. 3 (for Level 2) or NIST 800-172 (for Level 3)  
- **CISA ZTMM Mapping** → Function + maturity stage reference (e.g., Identity 1.1)  
- **DoD ZT Pillar** → One of 7 DoD Zero Trust pillars  
- **Microsoft Guidance** → Direct MSFT documentation link for control implementation  

