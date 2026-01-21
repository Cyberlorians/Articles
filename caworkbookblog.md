# Gain Complete Visibility into Conditional Access: Introducing CA-SISM

Conditional Access is the **Zero Trust policy engine** at the heart of Microsoft Entra ID. It's the gatekeeper evaluating real-time signals—user identity 👤, device compliance 💻, location 🌍, and sign-in risk 🔎—to enforce adaptive access decisions.

But here's the uncomfortable truth: **most organizations are flying blind**.

---

## 🚨 The Problem

Security teams face a familiar set of frustrations when managing Conditional Access:

**Fragmented Visibility**  
CA policy results are scattered across sign-in logs. There's no unified view showing success, failure, or report-only outcomes at a glance. Want to know how many sign-ins were blocked by a specific policy last week? Time to write KQL.

**Troubleshooting is Painful**  
When users report access issues, admins spend 30+ minutes correlating logs to understand which policy blocked them and why. The authentication flow details are buried in JSON blobs, and "previously satisfied" results obscure what actually happened.

**Workload Identities Are Invisible**  
Service principals and app registrations power your automation—but they can't use MFA and often rely on secrets or certificates. They're prime attack targets, yet they operate with minimal CA oversight. Most organizations have no idea which CA policies are evaluating their workload identities.

**Emergency Accounts Go Unmonitored**  
Break-glass accounts are your last line of defense. They're also high-value targets. Without dedicated monitoring, you won't know if they're being used—legitimately or not—until it's too late.

**Change Tracking Chaos**  
Someone modified a CA policy. Who? When? What changed? Good luck piecing that together from raw audit logs. And if it was an automated process like Microsoft's Managed Policy Manager, the attribution is even harder to track.

**Exclusion Group Drift**  
Users and groups get added to CA exclusions for "temporary" reasons that become permanent. Before you know it, your carefully designed Zero Trust policies have swiss cheese holes.

---

## 💡 Why I Built CA-SISM

After countless customer engagements where the same questions kept surfacing—*"Why was this user blocked?"*, *"Are our break-glass accounts being used?"*, *"Who modified that CA policy?"*—I realized the community needed something better.

**CA-SISM (Conditional Access – Summaries, Insights, Security & Monitoring)** is a workbook designed to answer these questions in seconds, not hours.

### What Makes It Different?

| Traditional Approach | CA-SISM Approach |
|---------------------|------------------|
| Hunt through raw SigninLogs | Pre-built summaries with visual tiles |
| Write custom KQL per investigation | Click-to-drill from summary to detail |
| No workload identity visibility | Dedicated tab with risk correlation |
| Emergency accounts monitored ad-hoc | Purpose-built break-glass dashboard |
| Policy changes discovered by accident | Side-by-side change comparison |

---

## 🎯 How It Helps

### Dynamic Reporting
Real-time insights into tenant-defined Conditional Access policies using `AuditLogs` and `SigninLogs`. See policy outcomes—success, failure, report-only success, report-only failure, not applied—summarized in tiles you can click to drill down.

### What-If Analysis Support
Every CA troubleshooting session eventually leads to Microsoft's What-If tool. CA-SISM bridges the gap by presenting authentication flow details—including the **actual authentication method enforced** (not just "previously satisfied")—so you can validate What-If results against real-world behavior.

### Workload Identity Coverage
Service principals are often overlooked in CA monitoring. CA-SISM includes a dedicated tab that correlates `AADServicePrincipalSignInLogs` with `AADRiskyServicePrincipals` to surface:
- Workload identity sign-in outcomes
- CA policy evaluation results
- Associated risk levels from Entra ID Protection

### Emergency Account Tracking
Break-glass accounts get their own dashboard:
- Enter up to two UPNs or UserIds
- See sign-in events with full authentication method details
- Track first-factor and second-factor separately
- Monitor IP addresses and geographic locations
- Filter by authentication method

### Security & Compliance Auditing
- **Exclusion Group Changes**: Track who's being added to or removed from CA exclusion groups
- **RAU Membership**: Monitor Restricted Administrative Unit changes
- **Dynamic Group Rules**: Detect modifications to dynamic membership rules used in CA exclusions
- **Policy Change Comparison**: Side-by-side diff showing exactly what changed in a CA policy

---

## 📊 Workbook Structure

CA-SISM is organized into four tabs:

### Overview
- ✅ Data ingestion validation (checks if your required tables are populating)
- 📘 Quick-start guidance
- 🔗 Links to Zero Trust resources including Microsoft's CA templates, CISA SCuBA, Maester, and EIDSCA

### User Tab
Deep-dive into user Conditional Access flows with filters for:
- **UserPrincipalName** – Partial match supported
- **Application** – Target app filter
- **Policy** – Specific CA policy focus
- **Report** – Outcome filter (success, failure, report-only variants)
- **Region/Location** – Geographic filtering
- **Sign-in Risk & User Risk** – Risk-based filtering

**Visualizations include:**
- Report outcome summary tiles
- Geographic heat map
- Device platform and trust state breakdowns
- Client app usage distribution
- Risk level distributions
- Detailed authentication flow analysis (excluding "previously satisfied")

### Workload Identities Tab
Dedicated monitoring for service principals:
- Filter by workload identity name, resource, policy, region, and risk level
- Automatic join with `AADRiskyServicePrincipals` for risk correlation
- Track CA outcomes for non-human identities

### Security & Monitoring Tab
The operational security hub featuring:
- **Emergency Account Section** – Dedicated break-glass monitoring
- **Exclusion Group Auditing** – Track CA exclusion membership changes
- **Dynamic Group Monitoring** – Rule change detection
- **CA Policy Change Tracking** – Timeline, top editors, and change comparison

---

## 🔍 Real-World Scenarios

### "Why Can't I Access This App?"
1. Navigate to User tab
2. Filter by UserPrincipalName
3. Expand the **failure** tree
4. Click to see authentication flow
5. Use CorrelationId for full detail

**Time to resolution: ~2 minutes** vs. 30+ manually.

### "Was Our Break-Glass Account Compromised?"
1. Go to Security & Monitoring tab
2. Enter emergency account UPN(s)
3. Review sign-in history with IP, location, auth method
4. Correlate with exclusion group changes

### "Our Service Principal Was Flagged as Risky"
1. Navigate to Workload Identities tab
2. Filter by service principal name
3. Review CA outcomes correlated with risk level
4. Identify anomalous patterns

### "Who Changed Our MFA Policy?"
1. Go to Security & Monitoring tab
2. Review Conditional Access Change Log
3. Click CorrelationId for side-by-side comparison
4. Identify the actor (user or automated app)

---

## 🛡️ Zero Trust Alignment

CA-SISM directly supports Zero Trust principles:

| Principle | How CA-SISM Helps |
|-----------|-------------------|
| **Verify Explicitly** | See which signals were evaluated for each sign-in |
| **Least Privilege Access** | Track exclusion group membership to prevent creep |
| **Assume Breach** | Monitor emergency accounts for unauthorized usage |

---

## 📋 Prerequisites

### Required Logs
Ensure these tables are ingesting to your Log Analytics workspace:
- `SigninLogs` – User sign-in events
- `AuditLogs` – Policy and group changes
- `AADServicePrincipalSignInLogs` – Workload identity sign-ins
- `AADRiskyServicePrincipals` – Service principal risk detections

### Licensing
- **Entra ID P1** – Basic CA logging
- **Entra ID P2** – Risk-based CA, Identity Protection
- **Azure Monitor or Microsoft Sentinel** – Log Analytics workspace

---

## 🚀 Get Started

1. **Enable Diagnostic Settings**  
   Entra ID → Monitoring → Diagnostic settings → Add setting → Stream to Log Analytics

2. **Import the Workbook**  
   Azure Monitor → Workbooks → New → Advanced Editor → Paste JSON → Apply

3. **Select Your Workspace**  
   Choose your Log Analytics workspace and time range

4. **Explore**  
   Use tab navigation to investigate Users, Workload Identities, or Security & Monitoring

---

## 📚 Additional Resources

| Tool | Description |
|------|-------------|
| 🤿 [CISA SCuBA](https://github.com/cisagov/ScubaGear) | Federal security guidance |
| ⚙️ [Maester](https://maester.dev/docs/intro) | PowerShell security testing |
| 📥 [EIDSCA](https://github.com/Cloud-Architekt/AzureAD-Attack-Defense/blob/main/AADSecurityConfigAnalyzer.md) | Config analysis with Sentinel alerting |
| 🛡️ [DoD Zero Trust Strategy](https://www.microsoft.com/en-us/security/blog/2024/04/16/new-microsoft-guidance-for-the-dod-zero-trust-strategy/) | Enterprise ZT roadmap |
| 📘 [CISA Zero Trust Guidance](https://www.microsoft.com/en-us/security/blog/2024/12/19/new-microsoft-guidance-for-the-cisa-zero-trust-maturity-model/) | ZTMM alignment |

---

## The Bottom Line

Conditional Access is only as effective as your ability to monitor it. Without visibility into policy outcomes, emergency account usage, and configuration changes, you're operating blind in a Zero Trust world.

**CA-SISM changes that.** It transforms scattered log data into actionable insights, giving security teams the clarity they need to:
- ✅ Troubleshoot access issues faster
- ✅ Detect unauthorized emergency account usage
- ✅ Track privilege creep in exclusion groups
- ✅ Audit policy changes with full context

**Stop flying blind. Start seeing your Conditional Access posture clearly.**

---

**Download the workbook:** [GitHub Repository]

---

**Author:** Michael Crane  
Microsoft Customer Success Architect

**Tags:** `Conditional Access` `Zero Trust` `Microsoft Entra` `Microsoft Sentinel` `Azure Monitor` `Workbooks` `Identity Security`
