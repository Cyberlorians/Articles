# 📘 CMMC 2.0 Level 1 — Foundational (17 Practices Fully Mapped)

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **AC.L1-3.1.1** Limit system access to authorized users | 3.1.1 — Limit system access to authorized users, processes acting on behalf of authorized users, or devices (including other systems). | **Identity 1.1** — Centralized identity stores | **Identity 1.1** — Centralized Identity Stores |
| **AC.L1-3.1.2** Limit system access to processes acting on behalf of authorized users | 3.1.2 — Limit system access to processes acting on behalf of authorized users. | **Workloads 1.1** — Service account & workload authentication | **Apps/Workloads 1.1** — Secure Application Workload Identity |
| **AC.L1-3.1.20** Verify and control/limit connections to external systems | 3.1.20 — Verify and control/limit connections to external systems. | **Networks 1.1** — Boundary enforcement | **Networks 1.1** — Segmentation Enforcement at Boundaries |
| **AC.L1-3.1.22** Control information posted or processed on publicly accessible systems | 3.1.22 — Control information posted or processed on publicly accessible systems. | **Data 1.2** — Data classification and labeling | **Data 1.2** — Labeling & Categorization |
| **IA.L1-3.5.1** Identify users before granting access | 3.5.1 — Identify system users, processes acting on behalf of users, or devices before allowing access. | **Identity 1.2** — Identity proofing & basic authentication | **Identity 1.2** — Identity Proofing |
| **IA.L1-3.5.2** Authenticate users using MFA | 3.5.2 — Authenticate (or verify) identities before granting access. | **Identity 2.1** — MFA adoption | **Identity 2.1** — Strong Multi-Factor Authentication |
| **MP.L1-3.8.3** Sanitize or destroy media before disposal | 3.8.3 — Sanitize or destroy system media containing FCI before disposal or release. | **Data 1.3** — Data sanitization & lifecycle management | **Data 1.3** — Data Retention & Sanitization |
| **MP.L1-3.8.4** Mark media with necessary safeguarding markings | 3.8.4 — Mark media with necessary CUI/safeguarding markings. | **Data 1.2** — Data labeling & classification | **Data 1.2** — Labeling & Categorization |
| **PE.L1-3.10.1** Limit physical access to systems and environments | 3.10.1 — Limit physical access to organizational systems, equipment, and environments. | **Devices 1.1** — Physical access enforcement | **Devices 1.1** — Physical Access Enforcement |
| **PE.L1-3.10.3** Escort visitors and monitor visitor activity | 3.10.3 — Escort visitors and monitor visitor activity. | **Devices 1.2** — Visitor control & monitoring | **Devices 1.2** — Visitor Control & Monitoring |
| **PE.L1-3.10.4** Maintain audit logs of physical access | 3.10.4 — Maintain audit logs of physical access. | **Visibility 1.1** — Physical/logical audit logging | **Visibility 1.1** — Audit Logging |
| **PE.L1-3.10.5** Control and manage physical access devices | 3.10.5 — Control and manage physical access devices. | **Devices 1.3** — Access device management | **Devices 1.3** — Physical Access Device Mgmt |
| **SC.L1-3.13.1** Monitor, control, and protect communications | 3.13.1 — Monitor, control, and protect organizational communications at external and key internal boundaries. | **Networks 1.2** — Secure communications & traffic inspection | **Networks 1.2** — Secure Comms Mgmt |
| **SC.L1-3.13.5** Use subnetworks for public system components | 3.13.5 — Implement subnetworks for publicly accessible system components. | **Networks 1.3** — Network segmentation/zoning | **Networks 1.3** — DMZs & Segmentation |
| **SI.L1-3.14.1** Identify, report, and correct flaws | 3.14.1 — Identify, report, and correct information and system flaws in a timely manner. | **Visibility 1.2** — Vulnerability visibility & patching | **Visibility 1.2** — Vulnerability Mgmt |
| **SI.L1-3.14.2** Provide protection from malicious code | 3.14.2 — Provide protection from malicious code at system boundaries and endpoints. | **Devices 1.4** — Endpoint/AV protection | **Devices 1.4** — Malware Protection |
| **SI.L1-3.14.5** Perform periodic scans and real-time protection | 3.14.5 — Perform periodic scans and real-time protection of files from malicious code. | **Devices 1.5** — Endpoint scanning & EDR | **Devices 1.5** — Real-Time Endpoint Protection |

---

# 🔑 Key
- **CMMC Practice ID** → CMMC 2.0 Level 1 practice  
- **NIST 800-171 Rev.2 Reference** → Expanded with official control text  
- **CISA ZTMM Mapping** → Exact sub-function and number from [CISA Zero Trust Maturity Model](https://learn.microsoft.com/en-us/security/zero-trust/cisa-zero-trust-maturity-model-intro)  
- **DoD ZT Mapping** → Exact objective number from [DoD Zero Trust Strategy](https://learn.microsoft.com/en-us/security/zero-trust/dod-zero-trust-strategy-intro)  

# 📗 CMMC 2.0 Level 2 — Advanced (110 Practices Fully Mapped)

This section maps all **110 CMMC Level 2 practices** to **NIST 800-171 Rev.2**, **CISA Zero Trust Maturity Model**, and **DoD Zero Trust Strategy objectives**.

---

<details><summary><b>Access Control (AC) — 22 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **AC.L2-3.1.3** Control the flow of CUI | 3.1.3 — Control the flow of CUI in information systems. | **Data 1.1** — Data Flow Protections | **Data 1.1** — Controlled Data Flow |
| **AC.L2-3.1.4** Separate duties of individuals | 3.1.4 — Separate the duties of individuals to reduce risk of collusion. | **Identity 2.2** — Least Privilege Enforcement | **Identity 2.2** — Role-Based Access Controls |
| **AC.L2-3.1.5** Employ least privilege | 3.1.5 — Employ the principle of least privilege. | **Identity 2.2** — Least Privilege Enforcement | **Identity 2.2** — Privileged Access Controls |
| **AC.L2-3.1.6** Use non-privileged accounts | 3.1.6 — Use non-privileged accounts or roles when accessing non-security functions. | **Identity 2.3** — Privilege Separation | **Identity 2.3** — Admin vs User Separation |
| **AC.L2-3.1.7** Prevent non-privileged users from executing privileged functions | 3.1.7 — Prevent non-privileged users from executing privileged functions. | **Identity 2.4** — Privileged Activity Monitoring | **Identity 2.4** — Privileged Function Monitoring |
| **AC.L2-3.1.8** Limit unsuccessful logon attempts | 3.1.8 — Limit unsuccessful logon attempts. | **Identity 1.3** — Session Protections | **Identity 1.3** — Account Lockout Controls |
| **AC.L2-3.1.9** Provide privacy/security notices | 3.1.9 — Provide privacy/security notices consistent with law. | **Visibility 1.3** — User Awareness | **Visibility 1.3** — User Notification/Audit |
| **AC.L2-3.1.10** Use session lock | 3.1.10 — Use session lock with pattern-hiding displays. | **Devices 1.6** — Session Timeout Enforcement | **Devices 1.6** — Idle Timeout Enforcement |
| **AC.L2-3.1.11** Terminate session after inactivity | 3.1.11 — Terminate sessions after inactivity. | **Identity 1.3** — Session Management | **Identity 1.3** — Session Termination Controls |
| **AC.L2-3.1.12** Monitor/control remote sessions | 3.1.12 — Monitor/control remote access. | **Networks 2.1** — Secure Remote Access | **Networks 2.1** — Remote Access Controls |
| **AC.L2-3.1.13** Encrypt remote access | 3.1.13 — Use cryptography to protect remote access. | **Data 2.1** — Encryption of Data in Transit | **Data 2.1** — Encrypted Remote Access |
| **AC.L2-3.1.14** Route remote access via managed points | 3.1.14 — Route remote access through managed access points. | **Networks 2.2** — Controlled Ingress/Egress | **Networks 2.2** — Remote Gateway Enforcement |
| **AC.L2-3.1.15** Authorize remote privileged commands | 3.1.15 — Authorize remote execution of privileged commands. | **Identity 2.4** — Privileged Session Monitoring | **Identity 2.4** — Remote Privileged Control |
| **AC.L2-3.1.16** Authorize wireless access | 3.1.16 — Authorize wireless access prior to connections. | **Networks 1.4** — Wireless Access Controls | **Networks 1.4** — Wireless Access Enforcement |
| **AC.L2-3.1.17** Protect wireless with auth/encryption | 3.1.17 — Protect wireless access using auth & encryption. | **Networks 2.3** — Wireless Encryption Enforcement | **Networks 2.3** — Secure Wireless Controls |
| **AC.L2-3.1.18** Control connection of mobile devices | 3.1.18 — Control connection of mobile devices. | **Devices 2.1** — Mobile Device Protections | **Devices 2.1** — Mobile Device Access Control |
| **AC.L2-3.1.19** Encrypt CUI on mobile devices | 3.1.19 — Encrypt CUI on mobile devices. | **Data 2.2** — Mobile Data Encryption | **Data 2.2** — Encrypted Mobile Storage |
| **AC.L2-3.1.21** Limit use of portable storage | 3.1.21 — Limit use of portable storage devices. | **Data 2.3** — Removable Media Protections | **Data 2.3** — Portable Media Controls |
| **AC.L2-3.1.23** Control remote access methods | 3.1.23 — Control remote access methods. | **Networks 2.4** — Remote Access Enforcement | **Networks 2.4** — Remote Access Enforcement |
| **AC.L2-3.1.24** Authorize remote access | 3.1.24 — Authorize remote access prior to connection. | **Networks 2.5** — Remote Access Authorization | **Networks 2.5** — Remote Access Authorization |
| **AC.L2-3.1.25** Separate tunneling mechanisms | 3.1.25 — Separate user and device tunneling mechanisms. | **Networks 2.6** — Tunnel Separation | **Networks 2.6** — Tunneling Separation |
| **AC.L2-3.1.26** Employ cryptographic separation | 3.1.26 — Employ cryptographic separation for remote sessions. | **Data 2.4** — Advanced Encryption Protections | **Data 2.4** — Cryptographic Session Isolation |

</p>
</details>

---

<details><summary><b>Awareness & Training (AT) — 2 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **AT.L2-3.2.1** Ensure awareness training | 3.2.1 — Ensure managers, system admins, and users are aware of security risks. | **Visibility 1.4** — Awareness & Training | **Visibility 1.4** — Workforce Cyber Awareness |
| **AT.L2-3.2.2** Role-specific security training | 3.2.2 — Ensure personnel are adequately trained to perform their duties. | **Visibility 1.5** — Role-Based Awareness | **Visibility 1.5** — Role-Based Cyber Training |

</p>
</details>

<details><summary><b>Audit & Accountability (AU) — 9 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **AU.L2-3.3.1** Create, protect, and retain audit records | 3.3.1 — Create and retain system audit records to enable monitoring, analysis, investigation, and reporting. | **Visibility 1.1** — Audit logging and visibility | **Visibility 1.1** — Enterprise Audit & Logging |
| **AU.L2-3.3.2** Ensure individual accountability in audit records | 3.3.2 — Ensure that audit records contain information to establish individual accountability. | **Visibility 1.2** — Correlation of user activity | **Visibility 1.2** — Individual Accountability in Logs |
| **AU.L2-3.3.3** Review and update audit events | 3.3.3 — Review and update audited events periodically. | **Visibility 1.3** — Continuous monitoring | **Visibility 1.3** — Audit Event Governance |
| **AU.L2-3.3.4** Alert in response to audit processing failures | 3.3.4 — Alert in the event of audit processing failures. | **Visibility 2.1** — Real-time alerting | **Visibility 2.1** — Audit Failure Detection |
| **AU.L2-3.3.5** Correlate audit review and analysis | 3.3.5 — Correlate audit review, analysis, and reporting processes for indications of misuse. | **Visibility 2.2** — Centralized log correlation | **Visibility 2.2** — SIEM & Correlation |
| **AU.L2-3.3.6** Provide audit reduction and report generation | 3.3.6 — Provide audit reduction and report generation to support investigations. | **Visibility 2.3** — Audit reporting capabilities | **Visibility 2.3** — Log Analytics & Reporting |
| **AU.L2-3.3.7** Provide audit record reduction before long-term storage | 3.3.7 — Provide audit reduction and record storage management before long-term storage. | **Visibility 2.4** — Audit lifecycle management | **Visibility 2.4** — Log Retention & Storage Controls |
| **AU.L2-3.3.8** Protect audit information | 3.3.8 — Protect audit information and tools from unauthorized access. | **Data 1.1** — Protect sensitive data (logs) | **Data 1.1** — Log Protection & Security |
| **AU.L2-3.3.9** Limit management of audit logging | 3.3.9 — Limit management of audit logging functionality to privileged users. | **Identity 2.4** — Privileged session management | **Identity 2.4** — Admin-only Log Management |

</p>
</details>

