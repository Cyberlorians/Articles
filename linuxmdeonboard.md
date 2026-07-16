# Onboard RHEL 8 to Microsoft Defender for Endpoint

This customer-ready runbook installs Microsoft Defender for Endpoint (MDE) on Red Hat Enterprise Linux (RHEL) 8, connects the server to the correct Microsoft Defender tenant, and validates antivirus and EDR functionality.

> **Validated:** The manual procedure in this article was tested end to end on RHEL 8.10 with MDE version `101.26042.0011`. The agent reported `licensed: true`, `healthy: true`, and `definitions_status: up_to_date`; the EICAR test file was quarantined.
>
> **Scope:** This guide uses the manual method so every step is visible. Microsoft currently recommends the [Defender deployment tool](https://learn.microsoft.com/defender-endpoint/linux-install-with-defender-deployment-tool) for simplified or larger deployments.

## What the onboarding download actually contains

The Defender portal currently presents three Linux choices:

| Portal choice | What it does | When to use it |
|---|---|---|
| **Defender deployment tool** | Installs MDE and onboards the server with one tenant-specific package | Preferred for a new deployment |
| **Onboarding script** | Downloads a ZIP containing `MicrosoftDefenderATPOnboardingLinuxServer.py` | Use when `mdatp` is already installed or when following this manual guide |
| **Onboarding file** | Provides tenant onboarding data for configuration-management tools | Use with Ansible, Chef, Puppet, or similar automation |

The Python onboarding script is also a configuration delivery mechanism. You do **not** edit it. When run as root, the tenant-specific script generates:

```text
/etc/opt/microsoft/mdatp/mdatp_onboard.json
```

That JSON associates the server with your Defender organization. Installing the `mdatp` RPM alone does not onboard or license the server.

## Prerequisites

Before starting, confirm the following:

- A supported server license: Defender for Servers Plan 1 or Plan 2, Defender for Endpoint for Servers, or Defender for Business Servers.
- RHEL 8.x with `systemd` and at least one CPU core, 1 GB RAM, and 2 GB free disk.
- Root or `sudo` access to the server.
- Permission to download an onboarding package from the Microsoft Defender portal.
- `python3`, `curl`, and `unzip` available on the server.
- Outbound HTTPS access to the required [MDE streamlined connectivity URLs](https://learn.microsoft.com/defender-endpoint/streamlined-device-connectivity-urls-commercial), including `*.endpoint.security.microsoft.com`, `packages.microsoft.com`, and `https://config.edge.skype.com/config/v1`.

> **Network warning:** MDE for Linux does not support PAC, WPAD, authenticated proxies, or TLS/SSL inspection. Use a transparent or static proxy and bypass TLS inspection for Defender destinations. See [MDE for Linux prerequisites](https://learn.microsoft.com/defender-endpoint/mde-linux-prerequisites) and [Linux connectivity troubleshooting](https://learn.microsoft.com/defender-endpoint/linux-support-connectivity).

## Step 1 — Confirm the operating system

```bash
cat /etc/redhat-release
uname -m
systemctl --version | head -1
```

Expected RHEL release:

```text
Red Hat Enterprise Linux release 8.x
```

Microsoft supports RHEL 7.2+, 8.x, 9.x, and 10.x. This runbook intentionally uses the RHEL 8 production repository.

## Step 2 — Install required utilities

```bash
sudo dnf install -y dnf-plugins-core python3 unzip curl
```

## Step 3 — Test outbound connectivity

First test the package repository:

```bash
curl -sS -o /dev/null -w 'packages.microsoft.com: %{http_code}\n' \
  https://packages.microsoft.com/
```

An HTTP response such as `200` or `301` confirms basic HTTPS reachability. This does not prove all Defender destinations are reachable; after installation, run the built-in MDE connectivity test.

If a static proxy is required, configure it after installation with:

```bash
sudo mdatp config proxy set --value http://proxy.example.com:8080
```

Do not place credentials in the command. Authenticated proxies are not supported.

## Step 4 — Configure the Microsoft production repository

```bash
sudo dnf config-manager --add-repo \
  https://packages.microsoft.com/config/rhel/8/prod.repo

sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc

sudo dnf makecache
```

Confirm the repository is available:

```bash
sudo dnf repolist | grep packages-microsoft-com-prod
```

The `prod` channel is recommended for production systems. Switching channels later requires uninstalling and reinstalling the product.

## Step 5 — Install Microsoft Defender for Endpoint

```bash
sudo dnf install -y mdatp
```

Confirm the package and service:

```bash
rpm -q mdatp
mdatp version
sudo systemctl status mdatp --no-pager
```

A reboot is normally not required. AuditD immutable mode is an exception documented by Microsoft.

### Expected state before onboarding

At this point, the application is installed but is not connected to a Defender tenant:

```bash
mdatp health --field healthy
mdatp health --field licensed
mdatp health --field org_id
```

Expected result:

```text
ATTENTION: No license found. Contact your administrator for help.
false
false
""
```

This is expected. Continue with the tenant onboarding script.

## Step 6 — Download the tenant onboarding script

1. Open the [Microsoft Defender portal](https://security.microsoft.com).
2. Go to **Settings** > **Endpoints** > **Device management** > **Onboarding**.
3. For **Step 1**, select **Linux**. Some Microsoft Learn pages or tenants may label this choice **Linux Server**.
4. Under **Deploy by downloading and applying packages or files**, find **Onboarding script**.
5. Select **Download script**.
6. Save the downloaded ZIP as `WindowsDefenderATPOnboardingPackage.zip`.

> **Protect this download:** It is tenant-specific. Do not commit it to source control, publish it, email it broadly, or reuse a package from another customer tenant.

Copy it to the server. For example, from an administrator workstation:

```bash
scp WindowsDefenderATPOnboardingPackage.zip \
  admin@rhel-server.example.com:/tmp/
```

## Step 7 — Extract and run the onboarding script

On the RHEL server:

```bash
cd /tmp
unzip WindowsDefenderATPOnboardingPackage.zip
ls -l MicrosoftDefenderATPOnboardingLinuxServer.py
sudo python3 MicrosoftDefenderATPOnboardingLinuxServer.py
```

Expected output includes:

```text
Generating /etc/opt/microsoft/mdatp/mdatp_onboard.json ...
```

RHEL 8 requires `python3`. Do not modify or repackage the onboarding script.

## Step 8 — Verify licensing and health

Run the following checks:

```bash
mdatp health --field org_id
mdatp health --field licensed
mdatp health --field healthy
mdatp health --field definitions_status
mdatp health --field release_ring
```

Expected results:

| Field | Expected value |
|---|---|
| `org_id` | A non-empty GUID for the customer organization |
| `licensed` | `true` |
| `healthy` | `true` |
| `definitions_status` | `up_to_date` |
| `release_ring` | `Production` when using the production channel |

The first definition download can take several minutes. During that time, `healthy` can temporarily remain `false` and `definitions_status` can report `updating`.

For a concise health summary:

```bash
for field in org_id licensed healthy definitions_status \
  real_time_protection_enabled passive_mode_enabled cloud_enabled; do
  printf '%-38s ' "$field"
  mdatp health --field "$field"
done
```

For detailed component health:

```bash
mdatp health --details edr
mdatp health --details definitions
```

See Microsoft's [`mdatp health` field reference](https://learn.microsoft.com/defender-endpoint/health-status).

## Step 9 — Run the built-in connectivity test

```bash
mdatp connectivity test
```

Each required destination should report `[OK]`. If checks fail:

1. Confirm outbound TCP 443 is allowed.
2. Confirm Defender URLs bypass TLS inspection.
3. Confirm the certificate issuer is Microsoft rather than the firewall or proxy CA.
4. Restart MDE and test again:

```bash
sudo systemctl restart mdatp
mdatp connectivity test
```

Certificate errors such as `curl error 60`, `CERTIFICATE_VERIFY_FAILED`, or unexpected HTTP 502 responses commonly indicate TLS interception.

## Step 10 — Confirm active antivirus protection

Check the protection state before running a test file:

```bash
mdatp health --field real_time_protection_enabled
mdatp health --field passive_mode_enabled
mdatp health --field cloud_enabled
```

For an active antivirus deployment, expect:

```text
true
false
true
```

If real-time protection is disabled and this server is intended to use MDE as the active antivirus product, enable it:

```bash
sudo mdatp config real-time-protection --value enabled
```

Then verify again:

```bash
mdatp health --field real_time_protection_enabled
mdatp health --field passive_mode_enabled
```

> **Coexistence warning:** Do not enable active mode blindly when another Fanotify-based security product is running in blocking mode. Review Microsoft's [side-by-side security solution guidance](https://learn.microsoft.com/defender-endpoint/mde-side-by-side). If MDE must coexist with another antivirus product, follow the customer's approved passive-mode design and configure mutual exclusions.

## Step 11 — Validate antivirus with EICAR

EICAR is a harmless industry-standard antivirus test string. Run this only during an approved change or validation window.

```bash
curl -o /tmp/eicar.com.txt https://secure.eicar.org/eicar.com.txt
```

Real-time protection should quarantine it. Confirm the detection:

```bash
mdatp threat list
```

Expected fields include:

```text
Name: Virus:DOS/EICAR_Test_File
Type: "virus"
Status: "quarantined"
Path: "/tmp/eicar.com.txt"
```

If the file is not detected immediately, confirm active protection and run an explicit custom scan:

```bash
sudo mdatp scan custom --path /tmp/eicar.com.txt
mdatp threat list
```

Also verify that `/tmp` uses a [filesystem supported by MDE real-time protection](https://learn.microsoft.com/defender-endpoint/mde-linux-prerequisites#supported-filesystems-for-real-time-protection-and-quick-full-and-custom-scans):

```bash
findmnt -T /tmp/eicar.com.txt
```

## Step 12 — Validate EDR reporting

First confirm the server appears in the [Defender device inventory](https://security.microsoft.com/machines). Initial appearance can take 5–20 minutes.

Then run Microsoft's Linux EDR simulation on a test server during an approved validation window:

```bash
cd /tmp
curl -L -o mde_linux_edr_diy.sh https://aka.ms/MDE-Linux-EDR-DIY
chmod +x mde_linux_edr_diy.sh
sudo ./mde_linux_edr_diy.sh
```

After several minutes:

1. Open **Microsoft Defender XDR**.
2. Review the generated alert.
3. Open the device timeline.
4. Confirm the activity is attributed to the expected test server.
5. Close or classify the alert according to the customer's test procedure.

## Final acceptance checklist

- [ ] `rpm -q mdatp` returns an installed version.
- [ ] `systemctl status mdatp` reports the service as active.
- [ ] `org_id` is a non-empty GUID for the correct tenant.
- [ ] `licensed` is `true`.
- [ ] `healthy` is `true`.
- [ ] `definitions_status` is `up_to_date`.
- [ ] `mdatp connectivity test` succeeds.
- [ ] The intended active or passive antivirus mode is confirmed.
- [ ] EICAR is quarantined when active antivirus protection is intended.
- [ ] The server appears in Defender device inventory.
- [ ] The EDR validation alert appears and is documented.

## Troubleshooting quick reference

| Symptom | Likely cause | Action |
|---|---|---|
| `No license found` | Onboarding script was not run, failed, or came from the wrong tenant | Download a fresh package from the correct tenant and rerun it with `sudo python3` |
| `org_id` is blank | Device is not onboarded | Check for `/etc/opt/microsoft/mdatp/mdatp_onboard.json` and rerun onboarding |
| `healthy` is `false` immediately after onboarding | Definitions are still downloading | Check `mdatp health --field definitions_status` and wait several minutes |
| Connectivity test fails | Firewall, proxy, DNS, or TLS inspection issue | Allow required URLs, bypass TLS inspection, and retest |
| `curl error 60` or `CERTIFICATE_VERIFY_FAILED` | TLS inspection replaced the Microsoft certificate | Configure a no-inspection exception for MDE destinations |
| EICAR remains on disk | Real-time protection disabled, passive mode, unsupported filesystem, or conflicting AV | Check protection fields, `findmnt`, and `conflicting_applications`; use an explicit scan |
| Service is not running | Failed install, dependency, filesystem, or service problem | Check `systemctl status mdatp` and the installation journal |
| Device not visible in portal | Initial reporting delay or EDR connectivity issue | Wait up to 20 minutes, run connectivity test, and check detailed EDR health |

Useful diagnostic commands:

```bash
sudo systemctl status mdatp --no-pager
sudo journalctl -u mdatp --since '30 minutes ago' --no-pager
mdatp health
mdatp health --details edr
mdatp connectivity test
sudo mdatp diagnostic create
```

The diagnostic archive can contain hostnames, usernames, IP addresses, and other sensitive data. Review and transfer it only through the customer's approved support channel.

## Uninstall

Uninstalling the package is not the same as intentionally offboarding a device. Follow the customer's retention and offboarding process first. Download a current offboarding script from **Settings** > **Endpoints** > **Device management** > **Offboarding** when required.

To remove the installed package from RHEL 8:

```bash
sudo dnf remove -y mdatp
```

## Microsoft documentation

This article summarizes and clarifies the following Microsoft documentation. Microsoft Learn remains authoritative for supported versions, prerequisites, URLs, and product behavior.

- [Prerequisites for Microsoft Defender for Endpoint on Linux](https://learn.microsoft.com/defender-endpoint/mde-linux-prerequisites)
- [Deploy Microsoft Defender for Endpoint on Linux manually](https://learn.microsoft.com/defender-endpoint/linux-install-manually)
- [Installer script-based deployment for MDE on Linux](https://learn.microsoft.com/defender-endpoint/linux-installer-script)
- [Defender deployment tool for Linux](https://learn.microsoft.com/defender-endpoint/linux-install-with-defender-deployment-tool)
- [MDE streamlined connectivity URLs — commercial](https://learn.microsoft.com/defender-endpoint/streamlined-device-connectivity-urls-commercial)
- [Troubleshoot MDE for Linux connectivity](https://learn.microsoft.com/defender-endpoint/linux-support-connectivity)
- [Troubleshoot MDE for Linux installation](https://learn.microsoft.com/defender-endpoint/linux-support-install)
- [Investigate MDE agent health issues](https://learn.microsoft.com/defender-endpoint/health-status)
- [MDE Client Analyzer overview](https://learn.microsoft.com/defender-endpoint/overview-client-analyzer)

---

*Last validated: July 16, 2026 — RHEL 8.10, MDE production channel.*
