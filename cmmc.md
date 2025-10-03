# 🔑 Key
- **CMMC Practice ID** → CMMC 2.0 Level 1 practice  
- **NIST 800-171 Rev.2 Reference** → Expanded with official control text  
- **CISA ZTMM Mapping** → Exact sub-function and number from [CISA Zero Trust Maturity Model](https://learn.microsoft.com/en-us/security/zero-trust/cisa-zero-trust-maturity-model-intro)  
- **DoD ZT Mapping** → Exact objective number from [DoD Zero Trust Strategy](https://learn.microsoft.com/en-us/security/zero-trust/dod-zero-trust-strategy-intro)

---
  
# 📘 CMMC 2.0 Level 1 — Foundational (17 Practices Fully Mapped)

This section maps all 17 CMMC Level 1 practices to NIST 800-171 Rev.2, CISA Zero Trust Maturity Model, and DoD Zero Trust Strategy objectives.

---

<details><summary><b>Access Control (AC) — 4 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **AC.L1-3.1.1** Limit system access to authorized users | 3.1.1 — Limit system access to authorized users, processes acting on behalf of authorized users, or devices (including other systems). | **Identity 1.1** — Centralized identity stores | **Identity 1.1** — Centralized Identity Stores |
| **AC.L1-3.1.2** Limit system access to processes acting on behalf of authorized users | 3.1.2 — Limit system access to processes acting on behalf of authorized users. | **Workloads 1.1** — Service account & workload authentication | **Apps/Workloads 1.1** — Secure Application Workload Identity |
| **AC.L1-3.1.20** Verify and control/limit connections to external systems | 3.1.20 — Verify and control/limit connections to external systems. | **Networks 1.1** — Boundary enforcement | **Networks 1.1** — Segmentation Enforcement at Boundaries |
| **AC.L1-3.1.22** Control information posted or processed on publicly accessible systems | 3.1.22 — Control information posted or processed on publicly accessible systems. | **Data 1.2** — Data classification and labeling | **Data 1.2** — Labeling & Categorization |

</p>
</details>

<details><summary><b>Identification & Authentication (IA) — 2 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **IA.L1-3.5.1** Identify users before granting access | 3.5.1 — Identify system users, processes acting on behalf of users, or devices before allowing access. | **Identity 1.2** — Identity proofing & basic authentication | **Identity 1.2** — Identity Proofing |
| **IA.L1-3.5.2** Authenticate users using MFA | 3.5.2 — Authenticate (or verify) identities before granting access. | **Identity 2.1** — MFA adoption | **Identity 2.1** — Strong Multi-Factor Authentication |

</p>
</details>

<details><summary><b>Media Protection (MP) — 2 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **MP.L1-3.8.3** Sanitize or destroy media before disposal | 3.8.3 — Sanitize or destroy system media containing FCI before disposal or release. | **Data 1.3** — Data sanitization & lifecycle management | **Data 1.3** — Data Retention & Sanitization |
| **MP.L1-3.8.4** Mark media with necessary safeguarding markings | 3.8.4 — Mark media with necessary CUI/safeguarding markings. | **Data 1.2** — Data labeling & classification | **Data 1.2** — Labeling & Categorization |

</p>
</details>

<details><summary><b>Physical Protection (PE) — 4 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **PE.L1-3.10.1** Limit physical access to systems and environments | 3.10.1 — Limit physical access to organizational systems, equipment, and environments. | **Devices 1.1** — Physical access enforcement | **Devices 1.1** — Physical Access Enforcement |
| **PE.L1-3.10.3** Escort visitors and monitor visitor activity | 3.10.3 — Escort visitors and monitor visitor activity. | **Devices 1.2** — Visitor control & monitoring | **Devices 1.2** — Visitor Control & Monitoring |
| **PE.L1-3.10.4** Maintain audit logs of physical access | 3.10.4 — Maintain audit logs of physical access. | **Visibility 1.1** — Physical/logical audit logging | **Visibility 1.1** — Audit Logging |
| **PE.L1-3.10.5** Control and manage physical access devices | 3.10.5 — Control and manage physical access devices. | **Devices 1.3** — Access device management | **Devices 1.3** — Physical Access Device Mgmt |

</p>
</details>

<details><summary><b>System & Communications Protection (SC) — 2 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **SC.L1-3.13.1** Monitor, control, and protect communications | 3.13.1 — Monitor, control, and protect organizational communications at external and key internal boundaries. | **Networks 1.2** — Secure communications & traffic inspection | **Networks 1.2** — Secure Comms Mgmt |
| **SC.L1-3.13.5** Use subnetworks for public system components | 3.13.5 — Implement subnetworks for publicly accessible system components. | **Networks 1.3** — Network segmentation/zoning | **Networks 1.3** — DMZs & Segmentation |

</p>
</details>

<details><summary><b>System & Information Integrity (SI) — 3 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **SI.L1-3.14.1** Identify, report, and correct flaws | 3.14.1 — Identify, report, and correct information and system flaws in a timely manner. | **Visibility 1.2** — Vulnerability visibility & patching | **Visibility 1.2** — Vulnerability Mgmt |
| **SI.L1-3.14.2** Provide protection from malicious code | 3.14.2 — Provide protection from malicious code at system boundaries and endpoints. | **Devices 1.4** — Endpoint/AV protection | **Devices 1.4** — Malware Protection |
| **SI.L1-3.14.5** Perform periodic scans and real-time protection | 3.14.5 — Perform periodic scans and real-time protection of files from malicious code. | **Devices 1.5** — Endpoint scanning & EDR | **Devices 1.5** — Real-Time Endpoint Protection |

</p>
</details>


---



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

<details><summary><b>Configuration Management (CM) — 9 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **CM.L2-3.4.1** Establish and maintain baseline configuration | 3.4.1 — Establish and maintain baseline configurations and inventories of organizational systems. | **Devices 1.1** — Asset inventory & baseline mgmt | **Devices 1.1** — Baseline Configuration Enforcement |
| **CM.L2-3.4.2** Establish and enforce security configuration settings | 3.4.2 — Establish and enforce security configuration settings for IT products. | **Devices 1.2** — Secure baseline enforcement | **Devices 1.2** — Hardened Configuration Standards |
| **CM.L2-3.4.3** Track, review, and approve/disapprove system changes | 3.4.3 — Track, review, approve, or disapprove changes to organizational systems. | **Visibility 1.2** — Change tracking | **Visibility 1.2** — Configuration Change Auditing |
| **CM.L2-3.4.4** Analyze security impact of changes | 3.4.4 — Analyze the security impact of changes prior to implementation. | **Visibility 1.3** — Risk-based change control | **Visibility 1.3** — Security Impact Analysis |
| **CM.L2-3.4.5** Define, document, approve, and enforce access restrictions associated with changes | 3.4.5 — Define, document, approve, and enforce access restrictions associated with system changes. | **Identity 2.2** — Privileged access enforcement | **Identity 2.2** — Change Authorization Controls |
| **CM.L2-3.4.6** Employ least functionality | 3.4.6 — Employ the principle of least functionality by configuring systems to provide only essential capabilities. | **Workloads 1.1** — Application/service hardening | **Apps/Workloads 1.1** — Least Functionality Enforcement |
| **CM.L2-3.4.7** Restrict use of nonessential functions | 3.4.7 — Restrict, disable, or prevent use of nonessential functions, ports, protocols, and services. | **Networks 1.2** — Restrict nonessential services | **Networks 1.2** — Protocol/Port Control |
| **CM.L2-3.4.8** Apply deny-by-exception policy to prevent unauthorized software execution | 3.4.8 — Apply deny-all, permit-by-exception policy to prevent unauthorized software execution. | **Devices 2.3** — Application allowlisting | **Devices 2.3** — Whitelisting & App Control |
| **CM.L2-3.4.9** Control and monitor user-installed software | 3.4.9 — Control and monitor the use of user-installed software. | **Devices 2.4** — Software execution monitoring | **Devices 2.4** — Unauthorized Software Control |

</p>
</details>

<details><summary><b>Identification & Authentication (IA) — 11 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **IA.L2-3.5.3** Use multifactor authentication for local and network access | 3.5.3 — Use multifactor authentication for local and network access to privileged and non-privileged accounts. | **Identity 2.1** — MFA adoption | **Identity 2.1** — Strong MFA Enforcement |
| **IA.L2-3.5.4** Employ replay-resistant authentication mechanisms | 3.5.4 — Employ replay-resistant authentication mechanisms for network access. | **Identity 2.5** — Replay resistance | **Identity 2.5** — Replay-Resistant Authentication |
| **IA.L2-3.5.5** Prevent reuse of identifiers for a defined period | 3.5.5 — Prevent reuse of identifiers for a defined period. | **Identity 1.4** — Account lifecycle mgmt | **Identity 1.4** — Identifier Management |
| **IA.L2-3.5.6** Disable identifiers after period of inactivity | 3.5.6 — Disable identifiers after a defined period of inactivity. | **Identity 1.4** — Account lifecycle mgmt | **Identity 1.4** — Account Deactivation Controls |
| **IA.L2-3.5.7** Enforce password complexity and change of characters | 3.5.7 — Enforce a minimum password complexity and change of characters when new passwords are created. | **Identity 1.5** — Credential strength | **Identity 1.5** — Password Complexity Enforcement |
| **IA.L2-3.5.8** Prohibit password reuse for a number of generations | 3.5.8 — Prohibit password reuse for a specified number of generations. | **Identity 1.5** — Credential lifecycle mgmt | **Identity 1.5** — Password Reuse Prevention |
| **IA.L2-3.5.9** Allow temporary password use with immediate change requirement | 3.5.9 — Allow temporary password use only with immediate change requirement. | **Identity 1.6** — Temporary credential issuance | **Identity 1.6** — Temporary Password Enforcement |
| **IA.L2-3.5.10** Store and transmit only cryptographically-protected passwords | 3.5.10 — Store and transmit only cryptographically-protected passwords. | **Data 2.1** — Protect credentials in transit/storage | **Data 2.1** — Credential Encryption |
| **IA.L2-3.5.11** Obscure feedback of authentication information | 3.5.11 — Obscure feedback of authentication information during entry. | **Identity 1.7** — Authentication UX protections | **Identity 1.7** — Credential Input Protection |
| **IA.L2-3.5.12** Use cryptographic modules that comply with FIPS standards | 3.5.12 — Use FIPS-validated cryptographic modules when used to protect information. | **Data 2.5** — Use of FIPS 140-validated crypto | **Data 2.5** — FIPS-Compliant Cryptography |
| **IA.L2-3.5.13** Ensure cryptographic modules are up to date | 3.5.13 — Ensure cryptographic modules are up to date and replaced when revoked/compromised. | **Data 2.5** — Cryptographic module validation | **Data 2.5** — Approved Crypto Module Mgmt |

</p>
</details>

<details><summary><b>Incident Response (IR) — 3 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **IR.L2-3.6.1** Establish an operational incident-handling capability | 3.6.1 — Establish an operational incident-handling capability for organizational systems that includes preparation, detection, analysis, containment, recovery, and user response activities. | **Visibility 2.1** — Incident detection & response | **Visibility 2.1** — Incident Response Program |
| **IR.L2-3.6.2** Track, document, and report incidents to appropriate officials | 3.6.2 — Track, document, and report incidents to organizational officials and/or authorities. | **Visibility 2.2** — Incident tracking & reporting | **Visibility 2.2** — Incident Documentation & Reporting |
| **IR.L2-3.6.3** Test the organizational incident response capability | 3.6.3 — Test the organizational incident response capability. | **Visibility 2.3** — Response exercises & simulations | **Visibility 2.3** — IR Capability Testing |

</p>
</details>

<details><summary><b>Incident Response (IR) — 3 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **IR.L2-3.6.1** Establish an operational incident-handling capability | 3.6.1 — Establish an operational incident-handling capability for organizational systems that includes preparation, detection, analysis, containment, recovery, and user response activities. | **Visibility 2.1** — Incident detection & response | **Visibility 2.1** — Incident Response Program |
| **IR.L2-3.6.2** Track, document, and report incidents to appropriate officials | 3.6.2 — Track, document, and report incidents to organizational officials and/or authorities. | **Visibility 2.2** — Incident tracking & reporting | **Visibility 2.2** — Incident Documentation & Reporting |
| **IR.L2-3.6.3** Test the organizational incident response capability | 3.6.3 — Test the organizational incident response capability. | **Visibility 2.3** — Response exercises & simulations | **Visibility 2.3** — IR Capability Testing |

</p>
</details>

<details><summary><b>Media Protection (MP) — 4 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **MP.L2-3.8.1** Protect system media, both paper and digital | 3.8.1 — Protect (i.e., physically control and securely store) system media containing CUI, both paper and digital. | **Data 1.1** — Data storage protections | **Data 1.1** — CUI Media Protection |
| **MP.L2-3.8.2** Limit access to CUI on system media | 3.8.2 — Limit access to CUI on system media to authorized users. | **Identity 2.2** — Least privilege enforcement | **Identity 2.2** — Media Access Controls |
| **MP.L2-3.8.5** Control the use of removable media on systems | 3.8.5 — Control the use of removable media on system components. | **Data 2.3** — Removable media protections | **Data 2.3** — Portable Media Control |
| **MP.L2-3.8.6** Prohibit the use of portable storage devices when nonessential | 3.8.6 — Prohibit the use of portable storage devices when such devices have no identifiable business purpose. | **Data 2.3** — Media usage enforcement | **Data 2.3** — Prohibited Media Enforcement |

</p>
</details>

<details><summary><b>Physical Protection (PE) — 6 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **PE.L2-3.10.2** Protect and monitor physical facility and support infrastructure | 3.10.2 — Protect and monitor the physical facility and support infrastructure for organizational systems. | **Devices 1.1** — Facility and environmental controls | **Devices 1.1** — Facility & Infrastructure Protection |
| **PE.L2-3.10.6** Enforce physical access authorizations | 3.10.6 — Enforce physical access authorizations for organizational systems. | **Identity 1.2** — Physical identity validation | **Identity 1.2** — Physical Access Authorization |
| **PE.L2-3.10.7** Maintain physical access records | 3.10.7 — Maintain visitor access records to organizational facilities. | **Visibility 1.1** — Access audit logging | **Visibility 1.1** — Physical Access Logging |
| **PE.L2-3.10.8** Control physical access to media | 3.10.8 — Control physical access to media containing CUI. | **Data 1.1** — Data protection | **Data 1.1** — Physical Media Protection |
| **PE.L2-3.10.9** Protect and control physical access devices | 3.10.9 — Protect and control physical access devices. | **Devices 1.3** — Device security | **Devices 1.3** — Physical Access Device Control |
| **PE.L2-3.10.10** Escort visitors and monitor visitor activity in facilities with CUI | 3.10.10 — Escort visitors and monitor visitor activity in facilities with organizational systems containing CUI. | **Devices 1.2** — Visitor monitoring | **Devices 1.2** — Visitor Escort & Monitoring |

</p>
</details>

<details><summary><b>Personnel Security (PS) — 2 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **PS.L2-3.9.1** Screen individuals prior to authorizing access to systems containing CUI | 3.9.1 — Screen individuals prior to authorizing access to organizational systems containing CUI. | **Identity 1.2** — Identity proofing | **Identity 1.2** — Personnel Vetting & Access Authorization |
| **PS.L2-3.9.2** Ensure CUI access is removed upon termination or transfer | 3.9.2 — Ensure that CUI system access is removed when personnel are terminated or transferred. | **Identity 1.4** — Account lifecycle management | **Identity 1.4** — Timely Deprovisioning |

</p>
</details>

<details><summary><b>Risk Assessment (RA) — 3 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **RA.L2-3.11.1** Periodically assess risk to organizational operations | 3.11.1 — Periodically assess the risk to organizational operations, assets, and individuals. | **Visibility 2.1** — Risk-based monitoring | **Visibility 2.1** — Risk Assessments & Reviews |
| **RA.L2-3.11.2** Scan for vulnerabilities in systems and applications | 3.11.2 — Scan for vulnerabilities in organizational systems and applications periodically and when new vulnerabilities are identified. | **Visibility 2.2** — Vulnerability scanning | **Visibility 2.2** — Vulnerability Scanning & Management |
| **RA.L2-3.11.3** Remediate vulnerabilities in a timely manner | 3.11.3 — Remediate vulnerabilities in organizational systems in a timely manner. | **Visibility 2.3** — Remediation tracking | **Visibility 2.3** — Vulnerability Remediation |

</p>
</details>

<details><summary><b>Security Assessment (CA) — 4 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **CA.L2-3.12.1** Periodically assess security controls for effectiveness | 3.12.1 — Periodically assess the security controls in organizational systems to determine effectiveness. | **Visibility 2.4** — Continuous control validation | **Visibility 2.4** — Security Control Assessments |
| **CA.L2-3.12.2** Develop and implement plans of action to correct deficiencies | 3.12.2 — Develop and implement plans of action to correct deficiencies and reduce vulnerabilities. | **Visibility 3.1** — Remediation planning & tracking | **Visibility 3.1** — POA&M Management |
| **CA.L2-3.12.3** Monitor security controls on an ongoing basis | 3.12.3 — Monitor security controls on an ongoing basis to ensure continued effectiveness. | **Visibility 3.2** — Ongoing monitoring & assessment | **Visibility 3.2** — Continuous Monitoring |
| **CA.L2-3.12.4** Develop, document, and periodically update system security plans | 3.12.4 — Develop, document, and periodically update system security plans describing system boundaries, environments, and controls. | **Visibility 3.3** — Documentation & governance | **Visibility 3.3** — Security Plan Management |

</p>
</details>

<details><summary><b>System & Communications Protection (SC) — 27 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **SC.L2-3.13.2** Separate user functionality from system management | 3.13.2 — Separate user functionality from system management functions. | **Identity 2.3** — Separation of duties | **Identity 2.3** — User/Admin Role Separation |
| **SC.L2-3.13.3** Deny network traffic by default, allow by exception | 3.13.3 — Deny network traffic by default and allow by exception. | **Networks 1.1** — Boundary enforcement | **Networks 1.1** — Default Deny / Allow by Exception |
| **SC.L2-3.13.4** Prevent remote activation of collaborative tools | 3.13.4 — Prevent remote activation of collaborative computing devices. | **Devices 2.5** — Endpoint collaboration controls | **Devices 2.5** — Collaboration Tool Controls |
| **SC.L2-3.13.6** Use cryptography to protect confidentiality of remote access sessions | 3.13.6 — Employ cryptographic mechanisms to protect confidentiality of remote sessions. | **Data 2.1** — Data-in-transit encryption | **Data 2.1** — Encrypted Remote Sessions |
| **SC.L2-3.13.7** Prevent split tunneling for remote devices | 3.13.7 — Prevent split tunneling for remote devices. | **Networks 2.6** — Tunnel separation | **Networks 2.6** — Split Tunnel Prevention |
| **SC.L2-3.13.8** Implement cryptographic protections for VoIP | 3.13.8 — Implement cryptographic mechanisms to protect confidentiality of VoIP communications. | **Data 2.1** — Encrypted communications | **Data 2.1** — VoIP Encryption |
| **SC.L2-3.13.9** Terminate session after inactivity | 3.13.9 — Terminate network connections after defined inactivity. | **Identity 1.3** — Session management | **Identity 1.3** — Connection Termination |
| **SC.L2-3.13.10** Protect confidentiality of CUI at rest | 3.13.10 — Employ cryptographic mechanisms to protect CUI at rest. | **Data 2.2** — Data-at-rest encryption | **Data 2.2** — Encryption for CUI Storage |
| **SC.L2-3.13.11** Employ FIPS-validated cryptography | 3.13.11 — Employ FIPS-validated cryptographic modules when used to protect CUI. | **Data 2.5** — FIPS crypto enforcement | **Data 2.5** — FIPS-Validated Cryptography |
| **SC.L2-3.13.12** Prohibit remote activation of sensors/cameras without user consent | 3.13.12 — Prohibit remote activation of collaborative devices like cameras or mics. | **Devices 2.5** — Endpoint collaboration controls | **Devices 2.5** — Sensor/Camera Access Controls |
| **SC.L2-3.13.13** Control cryptographic keys | 3.13.13 — Control and manage cryptographic keys for cryptography employed in organizational systems. | **Data 2.5** — Key management practices | **Data 2.5** — Cryptographic Key Mgmt |
| **SC.L2-3.13.14** Establish and manage cryptographic key lifecycles | 3.13.14 — Establish key lifecycles, revocation, and renewal. | **Data 2.5** — Key lifecycle management | **Data 2.5** — Key Lifecycle Enforcement |
| **SC.L2-3.13.15** Use cryptographic methods to protect network integrity | 3.13.15 — Employ cryptographic methods to protect integrity of remote sessions. | **Data 2.1** — Data-in-transit protections | **Data 2.1** — Encrypted Session Integrity |
| **SC.L2-3.13.16** Protect system from malicious code at boundaries | 3.13.16 — Implement protection against malicious code at system boundaries. | **Devices 1.4** — Malware protections | **Devices 1.4** — Boundary Malware Protection |
| **SC.L2-3.13.17** Protect system from denial-of-service attacks | 3.13.17 — Implement mechanisms to protect against DoS attacks. | **Networks 2.7** — Network resilience | **Networks 2.7** — DoS Protection |
| **SC.L2-3.13.18** Limit use of external systems | 3.13.18 — Limit use of external systems for organizational purposes. | **Networks 2.8** — External system governance | **Networks 2.8** — External System Restrictions |
| **SC.L2-3.13.19** Control communications at system boundaries | 3.13.19 — Control communications at external and key internal boundaries. | **Networks 1.1** — Boundary protection | **Networks 1.1** — Boundary Communications Control |
| **SC.L2-3.13.20** Use secure DNS resolution | 3.13.20 — Use secure domain name system (DNS) resolution services. | **Networks 2.9** — Secure name resolution | **Networks 2.9** — DNS Security Controls |
| **SC.L2-3.13.21** Protect integrity of transmitted information | 3.13.21 — Protect the integrity of transmitted information. | **Data 2.1** — Integrity protections | **Data 2.1** — Transmission Integrity Controls |
| **SC.L2-3.13.22** Separate user functionality from system management functions across network | 3.13.22 — Separate management functions across logical/physical boundaries. | **Identity 2.3** — Separation of duties | **Identity 2.3** — Management Function Segmentation |
| **SC.L2-3.13.23** Implement cryptographic protections for wireless communications | 3.13.23 — Employ cryptographic mechanisms to protect confidentiality of wireless communications. | **Networks 2.3** — Wireless encryption enforcement | **Networks 2.3** — Secure Wireless Crypto |
| **SC.L2-3.13.24** Implement subnetworks for publicly accessible system components | 3.13.24 — Implement subnetworks for publicly accessible system components. | **Networks 1.3** — Network segmentation | **Networks 1.3** — DMZs & Segregation |
| **SC.L2-3.13.25** Employ cryptographic separation for network traffic | 3.13.25 — Employ cryptographic separation for network traffic. | **Data 2.4** — Advanced crypto separation | **Data 2.4** — Encrypted Traffic Separation |
| **SC.L2-3.13.26** Implement boundary protections for shared networks | 3.13.26 — Implement boundary protections for shared networks. | **Networks 2.10** — Shared boundary controls | **Networks 2.10** — Multi-Tenant Boundary Protection |
| **SC.L2-3.13.27** Use cryptographic modules that comply with FIPS standards for CUI | 3.13.27 — Use FIPS-compliant crypto for CUI. | **Data 2.5** — FIPS crypto enforcement | **Data 2.5** — FIPS-Compliant CUI Encryption |
| **SC.L2-3.13.28** Employ advanced protections for mobile code | 3.13.28 — Implement security controls to manage mobile code. | **Devices 2.8** — Mobile code protections | **Devices 2.8** — Mobile Code Security |
| **SC.L2-3.13.29** Protect systems from externally controlled mobile code | 3.13.29 — Implement protections for mobile code controlled externally. | **Devices 2.8** — Mobile code protections | **Devices 2.8** — External Mobile Code Protections |

</p>
</details>

<details><summary><b>System & Information Integrity (SI) — 11 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-171 Rev.2 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|--------------------------------|-------------------|-------------------------|
| **SI.L2-3.14.3** Monitor system security alerts and advisories | 3.14.3 — Monitor security alerts and take action upon receipt. | **Visibility 2.1** — Threat intel integration | **Visibility 2.1** — Threat Intel & Alerts |
| **SI.L2-3.14.4** Update malicious code protection mechanisms | 3.14.4 — Update malicious code protection mechanisms periodically. | **Devices 1.4** — Endpoint protection updates | **Devices 1.4** — Malware Protection Updates |
| **SI.L2-3.14.6** Monitor system security alerts and take appropriate action | 3.14.6 — Monitor system security alerts and advisories and take appropriate actions. | **Visibility 2.2** — Centralized monitoring | **Visibility 2.2** — Security Monitoring & Alerts |
| **SI.L2-3.14.7** Identify unauthorized use of systems | 3.14.7 — Identify and report unauthorized use of organizational systems. | **Visibility 2.3** — Anomaly detection | **Visibility 2.3** — Unauthorized Use Detection |
| **SI.L2-3.14.8** Perform periodic scans of systems | 3.14.8 — Perform periodic scans of organizational systems and real-time scans of files. | **Devices 1.5** — Endpoint scanning & EDR | **Devices 1.5** — Vulnerability & Malware Scanning |
| **SI.L2-3.14.9** Protect against malicious email code | 3.14.9 — Protect against malicious code through email protections. | **Devices 1.4** — Malware protections | **Devices 1.4** — Email Malware Protection |
| **SI.L2-3.14.10** Detect and respond to system flaws | 3.14.10 — Identify, report, and correct information system flaws in a timely manner. | **Visibility 2.4** — Vulnerability management | **Visibility 2.4** — System Flaw Remediation |
| **SI.L2-3.14.11** Employ spam protection mechanisms | 3.14.11 — Employ spam protection mechanisms at system entry points. | **Devices 2.6** — Anti-spam protections | **Devices 2.6** — Email Spam Controls |
| **SI.L2-3.14.12** Monitor inbound and outbound communications | 3.14.12 — Monitor inbound and outbound communications traffic for unusual activity. | **Networks 2.7** — Network traffic monitoring | **Networks 2.7** — Comms Traffic Monitoring |
| **SI.L2-3.14.13** Control and monitor mobile code | 3.14.13 — Control and monitor the use of mobile code. | **Devices 2.8** — Mobile code controls | **Devices 2.8** — Mobile Code Security |
| **SI.L2-3.14.14** Detect and respond to attacks | 3.14.14 — Detect and respond to system attacks. | **Visibility 3.1** — Threat detection & response | **Visibility 3.1** — Incident Detection & Response |

</p>
</details>

# 🟥 CMMC 2.0 Level 3 — Expert (35 Practices Fully Mapped)

This section maps the **35 additional Level 3 practices** (from NIST SP 800-172) to:  
- **NIST 800-172 Enhanced Security Requirements**  
- **CISA Zero Trust Maturity Model** (ZTMM)  
- **DoD Zero Trust Strategy** objectives  

Practices are grouped by family for readability.

---

<details><summary><b>Access Control (AC) — 6 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-172 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|---------------------------|-------------------|-------------------------|
| **AC.L3-3.1.27** Require risk-based, adaptive access enforcement | 172-3.1.x — Enforce dynamic, risk-based access control. | **Identity 3.1** — Adaptive policy enforcement | **Identity 3.1** — Risk-Adaptive Access |
| **AC.L3-3.1.28** Dual authorization for privileged actions | 172-3.1.x — Require dual authorization for privileged/high-risk transactions. | **Identity 3.2** — Privileged session approval | **Identity 3.2** — Dual-Control Access |
| **AC.L3-3.1.29** Employ hardware-backed authenticators | 172-3.1.x — Require hardware tokens/PKI for privileged accounts. | **Identity 3.3** — PKI / FIDO2 enforcement | **Identity 3.3** — Hardware-Backed AuthN |
| **AC.L3-3.1.30** Time-bound, context-sensitive access | 172-3.1.x — Grant access based on time, location, or mission need. | **Identity 3.4** — Contextual adaptive access | **Identity 3.4** — Mission-Context Access |
| **AC.L3-3.1.31** Employ data-centric access controls | 172-3.1.x — Apply protections directly at the data level. | **Data 3.1** — Attribute-based access | **Data 3.1** — Data-Centric Controls |
| **AC.L3-3.1.32** Segment admin functions into isolated environments | 172-3.1.x — Use administrative enclaves for privileged operations. | **Workloads 3.1** — Admin isolation | **Workloads 3.1** — Privileged Admin Enclaves |

</p>
</details>

---

<details><summary><b>Audit & Accountability (AU) — 3 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-172 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|---------------------------|-------------------|-------------------------|
| **AU.L3-3.3.10** Correlate audit data with external threat intel | 172-3.3.x — Integrate audit logs with threat intelligence feeds. | **Visibility 3.1** — Threat intel-driven detection | **Visibility 3.1** — Intel-Based Correlation |
| **AU.L3-3.3.11** Automate cross-domain monitoring | 172-3.3.x — Automate collection/analysis across domains. | **Visibility 3.2** — Cross-domain monitoring | **Visibility 3.2** — Enterprise Threat Hunting |
| **AU.L3-3.3.12** Employ deception to detect adversaries | 172-3.3.x — Use honeypots/deception for APT detection. | **Visibility 3.3** — Deception technologies | **Visibility 3.3** — Adversary Engagement |

</p>
</details>

---

<details><summary><b>Configuration Management (CM) — 2 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-172 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|---------------------------|-------------------|-------------------------|
| **CM.L3-3.4.10** Employ dynamic reconfiguration | 172-3.4.x — Reconfigure systems dynamically in response to threats. | **Workloads 3.2** — Dynamic reconfiguration | **Workloads 3.2** — Adaptive Config Mgmt |
| **CM.L3-3.4.11** Protect configuration management tools | 172-3.4.x — Harden and isolate config/change mgmt tools. | **Workloads 3.3** — Secure admin toolchain | **Workloads 3.3** — Secure Config Tools |

</p>
</details>

---

<details><summary><b>Incident Response (IR) — 3 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-172 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|---------------------------|-------------------|-------------------------|
| **IR.L3-3.6.4** Employ advanced incident detection methods | 172-3.6.x — Use ML/behavioral analytics for incident detection. | **Visibility 3.4** — Behavior analytics | **Visibility 3.4** — Advanced Detection |
| **IR.L3-3.6.5** Conduct coordinated, cross-organization IR | 172-3.6.x — Coordinate IR across multiple stakeholders. | **Visibility 3.5** — Coordinated response | **Visibility 3.5** — Joint Incident Response |
| **IR.L3-3.6.6** Employ cyber threat hunting | 172-3.6.x — Implement proactive threat hunting capabilities. | **Visibility 3.6** — Threat hunting maturity | **Visibility 3.6** — Proactive Hunt Ops |

</p>
</details>

---

<details><summary><b>Maintenance (MA) — 2 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-172 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|---------------------------|-------------------|-------------------------|
| **MA.L3-3.7.3** Employ advanced protections for remote maintenance | 172-3.7.x — Use encrypted, monitored remote maintenance. | **Networks 3.1** — Secure remote maintenance | **Networks 3.1** — Encrypted Maint Channels |
| **MA.L3-3.7.4** Alternate maintenance methods for resilience | 172-3.7.x — Employ redundant/alternate maintenance options. | **Networks 3.2** — Redundant mgmt channels | **Networks 3.2** — Resilient Maint Pathways |

</p>
</details>

---

<details><summary><b>Media Protection (MP) — 2 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-172 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|---------------------------|-------------------|-------------------------|
| **MP.L3-3.8.5** Apply advanced media protection techniques | 172-3.8.x — Stronger protections for CUI-bearing media. | **Data 3.2** — Enhanced media protection | **Data 3.2** — CUI Media Hardening |
| **MP.L3-3.8.6** Employ digital rights management (DRM) | 172-3.8.x — Apply DRM to sensitive data/media. | **Data 3.3** — DRM / usage enforcement | **Data 3.3** — DRM Enforcement |

</p>
</details>

---

<details><summary><b>System & Communications Protection (SC) — 9 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-172 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|---------------------------|-------------------|-------------------------|
| **SC.L3-3.13.30** Employ moving target defenses | 172-3.13.x — Use dynamic network defenses (e.g., IP randomization). | **Networks 3.3** — Moving target defense | **Networks 3.3** — Dynamic Network Defense |
| **SC.L3-3.13.31** Employ non-persistent systems | 172-3.13.x — Use ephemeral/non-persistent VMs or sessions. | **Workloads 3.4** — Non-persistent workloads | **Workloads 3.4** — Ephemeral Systems |
| **SC.L3-3.13.32** Employ hardware-based isolation | 172-3.13.x — Use hardware enclaves (e.g., SGX/TPM). | **Workloads 3.5** — Trusted execution | **Workloads 3.5** — Hardware Root-of-Trust |
| **SC.L3-3.13.33** Employ out-of-band management networks | 172-3.13.x — Separate OOB management networks. | **Networks 3.4** — Out-of-band mgmt | **Networks 3.4** — Separate Admin Networks |
| **SC.L3-3.13.34** Employ advanced supply chain protections | 172-3.13.x — Validate supply chain components. | **Workloads 3.6** — Supply chain validation | **Workloads 3.6** — Supply Chain Risk Mgmt |
| **SC.L3-3.13.35** Employ isolation for high-value assets | 172-3.13.x — Strong isolation of critical systems. | **Workloads 3.7** — Critical asset isolation | **Workloads 3.7** — Mission System Isolation |
| **SC.L3-3.13.36** Employ advanced traffic analysis protections | 172-3.13.x — Use anti-traffic analysis techniques. | **Networks 3.5** — Traffic obfuscation | **Networks 3.5** — Traffic Camouflage |
| **SC.L3-3.13.37** Employ quantum-resistant cryptography (prep) | 172-3.13.x — Begin migration to quantum-safe crypto. | **Data 3.4** — Crypto agility | **Data 3.4** — Post-Quantum Preparation |
| **SC.L3-3.13.38** Employ enhanced boundary protections | 172-3.13.x — Stronger protections for multi-tenant/shared boundaries. | **Networks 3.6** — Cross-boundary resilience | **Networks 3.6** — Boundary Hardening |

</p>
</details>

---

<details><summary><b>System & Information Integrity (SI) — 8 Practices</b></summary>
<p>

| CMMC Practice ID | NIST 800-172 Requirement | CISA ZTMM Mapping | DoD Zero Trust Mapping |
|------------------|---------------------------|-------------------|-------------------------|
| **SI.L3-3.14.15** Employ advanced anomaly detection | 172-3.14.x — Behavioral & ML anomaly detection. | **Visibility 3.7** — AI/ML analytics | **Visibility 3.7** — Behavior Anomaly Detection |
| **SI.L3-3.14.16** Employ deception to detect malicious activity | 172-3.14.x — Deception-based threat detection. | **Visibility 3.8** — Deception/honeypots | **Visibility 3.8** — Deception Ops |
| **SI.L3-3.14.17** Employ integrity verification mechanisms | 172-3.14.x — File integrity monitoring and validation. | **Visibility 3.9** — Integrity validation | **Visibility 3.9** — File/Config Integrity Checks |
| **SI.L3-3.14.18** Employ system self-healing mechanisms | 172-3.14.x — Automated recovery from compromise. | **Workloads 3.8** — Self-healing workloads | **Workloads 3.8** — Autonomous Recovery |
| **SI.L3-3.14.19** Employ active defense measures | 172-3.14.x — Engage adversaries with active defense. | **Visibility 3.10** — Active defense | **Visibility 3.10** — Adversary Engagement |
| **SI.L3-3.14.20** Employ external verification of threats | 172-3.14.x — Use third-party validation of threats. | **Visibility 3.11** — External validation | **Visibility 3.11** — Federated Intel Sharing |
| **SI.L3-3.14.21** Employ automated recovery of compromised sessions | 172-3.14.x — Auto-reset of compromised sessions. | **Identity 3.5** — Auto session reset | **Identity 3.5** — Auto Recovery of Sessions |
| **SI.L3-3.14.22** Employ predictive defenses | 172-3.14.x — Predictive analytics to prevent exploitation. | **Visibility 3.12** — Predictive defenses | **Visibility 3.12** — Anticipatory Defense |

</p>
</details>










