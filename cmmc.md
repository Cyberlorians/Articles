# CMMC v2 ↔ Zero Trust Mapping (CISA ZTMM + DoD ZT)
| Practice ID   | Practice Text (NIST 800-171 Rev.2) | Zero Trust Pillar | Microsoft / DoD Guidance | CISA ZTMM Ref | DoD ZT Ref |
|---------------|------------------------------------|-------------------|--------------------------|---------------|------------|
| AC.L2-3.1.1   | Limit system access to authorized users | Identity / Governance | Entra ID accounts only, Conditional Access enforcement | Identity 1.1 – Access & Authorization | User 1.1 – Authenticate & Authorize |
| AC.L2-3.1.2   | Limit access to authorized transactions/functions | Identity | RBAC, Entra PIM roles, separation of duties | Identity 1.2 – Authorization Granularity | User 1.2 – Role-based Access |
| AC.L2-3.1.3   | Control flow of CUI in systems | Data / Network | Microsoft Purview DLP, Sensitivity Labels, Azure Firewall | Data 2.1 – Data Access Control | Data 2.1 – Flow Enforcement |
| AC.L2-3.1.4   | Separate duties of individuals | Identity / Governance | Entra role assignments, PIM approvals | Identity 1.2 – Privilege Separation | User 1.3 – Segregation of Duties |
| AC.L2-3.1.5   | Employ least privilege, including for security functions | Identity | RBAC, Just-In-Time (PIM), Just-Enough-Access | Identity 1.1 – Least Privilege | User 1.4 – Least Privilege Enforcement |
| AC.L2-3.1.6   | Use non-privileged accounts for non-privileged functions | Identity | Tiered accounts, Conditional Access policy requiring standard accounts | Identity 1.1 – Account Management | User 1.5 – Privileged Account Use |
| AC.L2-3.1.7   | Prevent non-privileged users from executing admin functions | Identity | Entra ID role restrictions, Azure AD built-in roles | Identity 1.1 – Authorization Control | User 1.6 – Admin Function Restriction |
| AC.L2-3.1.8   | Limit unsuccessful login attempts | Identity / Device | Conditional Access lockout, Smart Lockout in Entra ID | Identity 2.2 – Authentication Controls | User 2.1 – Login Attempt Restriction |
| AC.L2-3.1.9   | Provide privacy & security notice at login | Governance | Windows GPO banners, Entra sign-in banner | Governance 2.1 – Policy Awareness | Governance 1.1 – Policy Display |
| AC.L2-3.1.10  | Use session lock after inactivity | Device / Identity | Intune timeout policies, GPO screenlock | Devices 1.1 – Session Enforcement | Device 1.2 – Idle Session Timeout |
| AC.L2-3.1.11  | Terminate sessions after defined time limit | Device / Identity | Conditional Access session lifetime policies | Devices 1.1 – Session Mgmt | Device 1.3 – Session Termination |
| AC.L2-3.1.12  | Monitor and control remote access sessions | Network / Identity | Defender for Endpoint monitoring, Conditional Access | Networks 2.1 – Remote Access Mgmt | Network 2.2 – Remote Access Control |
| AC.L2-3.1.13  | Authorize remote access prior to connection | Identity / Network | VPN with Entra ID auth, Conditional Access | Identity 1.1 – Access Verification | Network 2.3 – Remote Access Authorization |
| AC.L2-3.1.14  | Control use of remote access methods | Network | Intune restrict remote protocols, Entra restrictions | Networks 2.1 – Remote Method Control | Network 2.3 – Method Restriction |
| AC.L2-3.1.15  | Route remote access via managed access point | Network | Azure VPN Gateway, Azure Firewall, Bastion | Networks 3.1 – Segmentation & Access Points | Network 2.4 – Access Point Routing |
| AC.L2-3.1.16  | Authorize wireless access prior to connection | Network / Device | WPA3-Enterprise, Entra cert-based Wi-Fi auth | Networks 2.1 – Wireless AuthN | Network 2.5 – Wireless Authorization |
| AC.L2-3.1.17  | Protect wireless access using authentication & encryption | Network / Device | WPA3, EAP-TLS with certs, Intune Wi-Fi config | Networks 2.2 – Wireless Protection | Network 2.5 – Wireless Encryption |
| AC.L2-3.1.18  | Control connection of mobile devices | Device | Intune device compliance, MDM controls | Devices 2.1 – Mobile Device Mgmt | Device 2.6 – Mobile Device Control |
| AC.L2-3.1.19  | Encrypt CUI on mobile devices | Data / Device | BitLocker, iOS/Android encryption, Purview | Data 2.1 – Data Encryption | Data 2.1 – Mobile Encryption |
| AC.L2-3.1.20  | Verify and control external connections | Network / Governance | Azure Firewall, VPN, Conditional Access | Networks 3.1 – External Connection Control | Network 2.6 – External Connection Mgmt |
| AC.L2-3.1.21  | Limit use of external systems | Governance / Device | BYOD restrictions, Conditional Access, Intune compliance | Governance 1.2 – External System Policy | Governance 1.3 – External Use Control |
| AC.L2-3.1.22  | Control use of portable storage devices on systems | Device / Data | Intune removable storage restrictions, Defender policy | Data 2.1 – Media Control | Device 2.7 – Removable Media Control |
