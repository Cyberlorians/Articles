# Microsoft Defender for Identity — Sensor v2.x Data Flows

> **Scope:** MDI standalone sensor v2.x installed directly on Domain Controllers  
> **Reference:** [Microsoft Learn — MDI Prerequisites](https://learn.microsoft.com/en-us/defender-for-identity/prerequisites)  
> **Last Updated:** February 2026

---

## Architecture Diagram

```mermaid
flowchart TB
    subgraph cloud["☁️ Microsoft Defender XDR Cloud"]
        mdi_cloud["MDI Cloud Service<br/>*.atp.azure.com"]
    end

    subgraph perimeter["🔒 Network Perimeter"]
        fw["Proxy / Firewall"]
    end

    subgraph dc["🖥️ Domain Controller — Sensor Host"]
        sensor["MDI Sensor v2.x"]
        updater["Sensor Updater<br/>(localhost)"]
        etw["AD DS<br/>ETW Provider"]
        dns_local["DNS Service"]
    end

    subgraph nnr["🔍 NNR Targets — DCs, ADFS, ADCS, Entra Connect"]
        nnr_ep["Network Name<br/>Resolution Endpoints"]
    end

    subgraph ldap["📁 LDAP / GC Targets — All Forest Domain Controllers"]
        ldap_ep["LDAP & Global Catalog<br/>Endpoints"]
    end

    nps["📡 RADIUS / NPS Server"]

    sensor -->|"TCP 443<br/>TLS 1.2+ HTTPS"| fw
    fw -->|"outbound to<br/>*.atp.azure.com"| mdi_cloud
    sensor -.->|"TCP 444<br/>SSL (localhost)"| updater
    sensor -.->|"ETW Tracing +<br/>Event Logs (local)"| etw
    sensor -->|"TCP/UDP 53<br/>DNS + rDNS"| dns_local
    sensor -->|"TCP 135 RPC/NTLM<br/>UDP 137 NetBIOS<br/>TCP 3389 RDP ClientHello<br/>TCP/UDP 445 SMB/Netlogon"| nnr_ep
    sensor -->|"TCP/UDP 389 LDAP<br/>TCP 636 LDAPS (TLS)<br/>TCP 3268 Global Catalog<br/>TCP 3269 GC-S (TLS)"| ldap_ep
    nps -->|"UDP 1813<br/>RADIUS Accounting<br/>(inbound)"| sensor

    style cloud fill:#e6f2ff,stroke:#0078D4,stroke-width:2px,color:#333
    style perimeter fill:#fff3e0,stroke:#FF8C00,stroke-width:2px,color:#333
    style dc fill:#e8f5e9,stroke:#107C10,stroke-width:2px,color:#333
    style nnr fill:#fce4ec,stroke:#D83B01,stroke-width:2px,color:#333
    style ldap fill:#e0f2f1,stroke:#008575,stroke-width:2px,color:#333
    style sensor fill:#0078D4,stroke:#005a9e,color:#fff
    style mdi_cloud fill:#0078D4,stroke:#005a9e,color:#fff
    style fw fill:#FF8C00,stroke:#cc7000,color:#fff
    style nps fill:#C239B3,stroke:#9c2d8f,color:#fff
```

---

## Complete Port Reference

### 1. Cloud Connectivity (Outbound)

| Port | Protocol | Transport | Direction | Destination | TLS | Purpose |
|------|----------|-----------|-----------|-------------|-----|---------|
| **443** | HTTPS | TCP | Outbound | `*.atp.azure.com` | TLS 1.2+ required | Sensor ↔ MDI cloud service communication, telemetry upload, detection signals, entity data sync |

- **Proxy support:** HTTP proxy with CONNECT tunneling; transparent proxy with certificate inspection
- **Alternative:** Azure ExpressRoute with Microsoft peering, BGP community `12076:5220`
- **Cert pinning:** None — standard TLS CA validation

### 2. Sensor Internal Services (localhost)

| Port | Protocol | Transport | Direction | Destination | TLS | Purpose |
|------|----------|-----------|-----------|-------------|-----|---------|
| **444** | SSL | TCP | Localhost | `127.0.0.1` | SSL (self-signed) | Sensor updater service — downloads and installs sensor updates |

### 3. Local Data Collection

| Source | Transport | Direction | Mechanism | Purpose |
|--------|-----------|-----------|-----------|---------|
| **AD DS** | Local IPC | Local | ETW tracing + Windows Event Logs | Captures authentication events (4624, 4625, 4768, 4769, 4776), directory changes, Kerberos/NTLM traffic |
| **DNS Server** | TCP/UDP 53 | Local/Network | DNS query interception + rDNS lookups | DNS-based threat detection, domain resolution for NNR |

### 4. Network Name Resolution (NNR)

| Port | Protocol | Transport | Direction | Targets | Purpose |
|------|----------|-----------|-----------|---------|---------|
| **135** | RPC/NTLM | TCP | Outbound | DCs, ADFS, ADCS, Entra Connect | RPC endpoint mapper — NTLM-based name resolution |
| **137** | NetBIOS | UDP | Outbound | DCs, ADFS, ADCS, Entra Connect | NetBIOS name resolution (legacy fallback) |
| **3389** | RDP | TCP | Outbound | DCs, ADFS, ADCS, Entra Connect | RDP ClientHello packet parsing — extracts machine identity without full RDP session |
| **445** | SMB | TCP/UDP | Outbound | DCs, ADFS, ADCS, Entra Connect | SMB session enumeration, Netlogon, SAM-R queries for lateral movement detection |

> **NNR priority order:** RPC (135) → NetBIOS (137) → RDP (3389) → DNS rDNS (53)

### 5. LDAP / Global Catalog Queries

| Port | Protocol | Transport | Direction | Targets | TLS | Purpose |
|------|----------|-----------|-----------|---------|-----|---------|
| **389** | LDAP | TCP/UDP | Outbound | All forest DCs | Optional (LDAP signing) | Standard LDAP queries — user/group/computer enumeration, entity enrichment |
| **636** | LDAPS | TCP | Outbound | All forest DCs | TLS required | Secure LDAP queries — encrypted directory lookups |
| **3268** | LDAP GC | TCP | Outbound | Global Catalog servers | Optional | Global Catalog read queries — cross-domain entity resolution |
| **3269** | LDAPS GC | TCP | Outbound | Global Catalog servers | TLS required | Secure Global Catalog queries — encrypted cross-domain lookups |

### 6. RADIUS Accounting (Inbound)

| Port | Protocol | Transport | Direction | Source | Purpose |
|------|----------|-----------|-----------|--------|---------|
| **1813** | RADIUS | UDP | **Inbound** | NPS / RADIUS servers | VPN and Wi-Fi authentication event collection — captures RADIUS accounting packets for identity correlation |

> **Note:** This is the only **inbound** port. The sensor listens on UDP 1813 for RADIUS accounting messages from NPS.

---

## Firewall Rule Summary

### Required Outbound Rules

```
ALLOW TCP 443   → *.atp.azure.com                  # MDI cloud (REQUIRED)
ALLOW TCP 444   → 127.0.0.1 (localhost only)        # Sensor updater
ALLOW TCP/UDP 53 → DNS servers                       # DNS resolution
ALLOW TCP 135   → DCs, ADFS, ADCS, Entra Connect    # NNR - RPC
ALLOW UDP 137   → DCs, ADFS, ADCS, Entra Connect    # NNR - NetBIOS
ALLOW TCP 3389  → DCs, ADFS, ADCS, Entra Connect    # NNR - RDP
ALLOW TCP/UDP 445 → DCs, ADFS, ADCS, Entra Connect  # SMB/Netlogon
ALLOW TCP/UDP 389 → All forest DCs                   # LDAP
ALLOW TCP 636   → All forest DCs                     # LDAPS
ALLOW TCP 3268  → Global Catalog servers              # LDAP GC
ALLOW TCP 3269  → Global Catalog servers              # LDAPS GC
```

### Required Inbound Rules

```
ALLOW UDP 1813  ← NPS/RADIUS servers                 # RADIUS accounting
```

---

## ExpressRoute Alternative

For environments using Azure ExpressRoute instead of public internet:

| Setting | Value |
|---------|-------|
| Peering type | Microsoft peering |
| BGP community | `12076:5220` |
| Service | Microsoft Defender for Identity |
| Protocol | HTTPS (TCP 443) |

> ExpressRoute replaces the TCP 443 outbound path only. All other ports remain internal network flows.

---

## Detection Categories by Port

| Port(s) | Detections Enabled |
|---------|-------------------|
| 443 | All cloud-side detections, entity behavioral analytics, UEBA |
| ETW/Event Logs | Pass-the-Hash, Pass-the-Ticket, Kerberoasting, Golden Ticket, Brute Force |
| 53 | DNS tunneling, DNS reconnaissance, suspicious DNS queries |
| 135, 137, 3389 | Network Name Resolution for lateral movement path mapping |
| 445 | Lateral movement (SMB), SAM-R reconnaissance, remote execution |
| 389, 636, 3268, 3269 | Directory enumeration, DCShadow, suspicious replication, LDAP reconnaissance |
| 1813 | VPN/Wi-Fi identity correlation, impossible travel via RADIUS |
