# Microsoft Defender for Identity - Documentation Contradictions & Clarification Needed

**Date:** January 26, 2026  
**Purpose:** Internal review of MDI documentation inconsistencies impacting deployment guidance  
**Status:** Pending Product Group clarification

---

## Executive Summary

During MDI deployment planning, several documentation inconsistencies were identified across Microsoft Learn documentation. These contradictions create confusion for customers deploying MDI sensors (both v2.x and v3.x) and may result in unnecessary configuration, wasted effort, or broken functionality.

---

## Contradiction #1: Directory Service Account (DSA) Recommendations

### Document A: Directory Service Accounts
**URL:** https://learn.microsoft.com/en-us/defender-for-identity/deploy/directory-service-accounts

**States:**
> "While a DSA is optional in some scenarios, we recommend that you configure a DSA for Defender for Identity for full security coverage."

> "For optimal security, it is recommended to use a Group Managed Service Account (gMSA)."

### Document B: v3 Sensor Prerequisites  
**URL:** https://learn.microsoft.com/en-us/defender-for-identity/deploy/prerequisites-sensor-version-3

**States:**
> "The sensor doesn't require credentials to be provided in the portal. Even if credentials are entered, the sensor uses the Local System identity on the server to query Active Directory and perform response actions."

### The Contradiction
- Document A recommends configuring gMSA DSA for "full security coverage"
- Document B says v3 sensors ignore configured credentials and use LocalSystem anyway

### Impact
Customers deploying v3 sensors who follow DSA documentation will waste time configuring gMSA that won't be used.

### Clarification Needed
- Should DSA page include v2.x vs v3.x callouts?
- Is DSA configuration truly unnecessary for v3 DC-only deployments?

---

## Contradiction #2: gMSA for Action Accounts

### Document A: Configure Action Accounts
**URL:** https://learn.microsoft.com/en-us/defender-for-identity/deploy/manage-action-accounts

**States:**
> "If you need to change this behavior, set up a dedicated gMSA and scope the permissions that you need."

Provides full instructions for configuring gMSA action accounts.

### Document B: v3 Sensor Prerequisites
**URL:** https://learn.microsoft.com/en-us/defender-for-identity/deploy/prerequisites-sensor-version-3

**States:**
> "If a Group Managed Service Account (gMSA) is configured for response actions, the response actions are disabled."

### The Contradiction
- Document A encourages gMSA configuration for action accounts
- Document B says gMSA for actions **disables response actions** on v3 sensors

### Impact
Customers who configure gMSA action accounts per documentation will have **broken response actions** on v3 sensors.

### Clarification Needed
- Should Action Accounts page warn about v3 behavior?
- What's the migration path for customers with gMSA action accounts moving to v3?

---

## Contradiction #3: SAM-R Configuration (Same Page!)

### Document: Configure SAM-R
**URL:** https://learn.microsoft.com/en-us/defender-for-identity/deploy/remote-calls-sam

### Statement 1 (Top of Page - Important Box):
> "As of mid-May 2025, Microsoft Defender for Identity no longer collects local administrators group members from endpoints using SAM-R queries. This data is currently used to build potential lateral movement path maps, which are no longer being updated."

### Statement 2 (Tip Box - Same Page):
> "While this procedure is optional, we recommend that you configure a Directory Service account and configure SAM-R for lateral movement path detection to fully secure your environment with Defender for Identity."

### Statement 3 (Body - Same Page):
Full step-by-step configuration instructions for:
- GPO configuration
- Intune Device Profile
- SDDL strings
- etc.

### The Contradiction
The same page:
1. Says SAM-R collection is **disabled** as of May 2025
2. **Recommends** configuring SAM-R
3. Provides full configuration instructions

### Impact
Customers will configure SAM-R for a deprecated/disabled feature.

### Clarification Needed
- Should the Tip recommendation be removed?
- Should configuration instructions be archived?
- What is the "alternative method being explored" mentioned in What's New?

---

## Contradiction #4: Test-MdiReadiness Script vs Documentation

### Script: Test-MdiReadiness.ps1
**Source:** https://github.com/microsoft/Microsoft-Defender-for-Identity/tree/main/Test-MdiReadiness

**Behavior:** Checks `RemoteSAM` configuration and reports "Failed" if not configured.

### Documentation: What's New - May 2025
**URL:** https://learn.microsoft.com/en-us/defender-for-identity/whats-new#may-2025

**States:**
> "Local administrators collection (using SAM-R queries) feature is disabled"

### The Contradiction
- Script checks for SAM-R configuration and fails without it
- Documentation says SAM-R is disabled and not used

### Impact
Customers see "Failed" status for a configuration that's no longer relevant, causing confusion and unnecessary remediation efforts.

### Clarification Needed
- Will the script be updated to remove RemoteSAM check?
- Should the check be changed to informational rather than failed?

---

## Contradiction #5: DSA Requirements Include Deprecated Feature

### Document: Directory Service Accounts
**URL:** https://learn.microsoft.com/en-us/defender-for-identity/deploy/directory-service-accounts

**Lists DSA Required For:**
> "Requesting member lists for local administrator groups from devices seen in network traffic, events and ETW activities via a SAM-R call made to the device. Collected data is used to calculate potential lateral movement paths."

### Document: What's New - May 2025
**URL:** https://learn.microsoft.com/en-us/defender-for-identity/whats-new#may-2025

**States:**
> "The remote collection of local administrators group members from endpoints using SAM-R queries in Microsoft Defender for Identity will be disabled by mid-May 2025."

### The Contradiction
DSA requirements page lists SAM-R/LMP as a reason to configure DSA, but that feature is deprecated.

### Impact
Customers may configure DSA specifically for SAM-R support that no longer exists.

### Clarification Needed
- Should SAM-R be removed from the DSA requirements list?
- What are the remaining valid reasons to configure DSA?

---

## Summary Table: Documentation Conflicts

| # | Topic | Doc A Says | Doc B Says | Impact |
|---|-------|------------|------------|--------|
| 1 | DSA | "Recommend gMSA for full coverage" | "v3 uses LocalSystem, ignores credentials" | Wasted configuration effort |
| 2 | gMSA Action Accounts | "Configure gMSA for least-privilege" | "gMSA disables response actions on v3" | **Broken functionality** |
| 3 | SAM-R | Same page: "Disabled" AND "Recommended" | N/A | Configuring deprecated feature |
| 4 | Readiness Script | Checks RemoteSAM | SAM-R is disabled | False failure alerts |
| 5 | DSA Requirements | "Needed for SAM-R" | SAM-R is disabled | Misleading requirements |

---

## Recommended Documentation Updates

### Immediate Priority

1. **SAM-R Page** - Remove recommendation tip and configuration instructions, or clearly mark as deprecated/archived

2. **v3 Prerequisites** - Add prominent callout that DSA and gMSA Action Account guidance from other pages doesn't apply to v3

3. **Action Accounts Page** - Add warning: "For v3 sensors, gMSA configuration disables response actions"

### Medium Priority

4. **DSA Page** - Add version-specific sections or callouts for v2.x vs v3.x

5. **Test-MdiReadiness Script** - Remove or modify RemoteSAM check

6. **DSA Requirements List** - Remove SAM-R from "required for" list

### Documentation Structure Suggestion

Consider creating:
- **v2.x Deployment Guide** - Current gMSA/DSA guidance
- **v3.x Deployment Guide** - Simplified, LocalSystem-based
- Clear migration guide between versions

---

## Open Questions for Product Group

1. **DSA on v3:** Is DSA truly unnecessary for v3 DC-only single-domain deployments?

2. **gMSA Action Accounts:** What's the recommended migration path for customers using gMSA action accounts who want to move to v3?

3. **SAM-R Alternative:** The May 2025 announcement mentioned "An alternative method is being explored" for LMP - any update?

4. **Script Update:** Will Test-MdiReadiness.ps1 be updated to reflect current requirements?

5. **Mixed Environments:** What's the guidance for environments running both v2.x and v3.x sensors during migration?

6. **DeletedObjects ACL:** Is this still required for v3 sensors, or does LocalSystem have implicit access on DCs?

---

## References

| Document | URL |
|----------|-----|
| v3 Prerequisites | https://learn.microsoft.com/en-us/defender-for-identity/deploy/prerequisites-sensor-version-3 |
| Directory Service Accounts | https://learn.microsoft.com/en-us/defender-for-identity/deploy/directory-service-accounts |
| Configure Action Accounts | https://learn.microsoft.com/en-us/defender-for-identity/deploy/manage-action-accounts |
| Configure SAM-R | https://learn.microsoft.com/en-us/defender-for-identity/deploy/remote-calls-sam |
| What's New - May 2025 | https://learn.microsoft.com/en-us/defender-for-identity/whats-new#may-2025 |
| Test-MdiReadiness Script | https://github.com/microsoft/Microsoft-Defender-for-Identity/tree/main/Test-MdiReadiness |

---

## Next Steps

- [ ] Review with colleague
- [ ] Submit documentation feedback via Learn page feedback buttons
- [ ] Open support case or engage PG for clarification
- [ ] Update internal deployment guides based on PG response

---

*Document prepared for internal discussion - not for external distribution*
