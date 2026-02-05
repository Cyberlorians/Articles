# Admin Consent vs User Consent: Speaker Notes

**Read this as you go through each slide. Everything you need is here.**

---

## 🔑 THE BIG IDEA (Memorize This)

> **"Consent is a TRUST decision, not an ACCESS decision."**
> 
> - PIM & RBAC control *who can perform admin actions*
> - Consent controls *what an application is trusted to do on behalf of users*
> - Once consent is granted, it's **permanent** until revoked—PIM doesn't protect you anymore

**Executive One-Liner (if asked to summarize):**
> "Admin consent is a permanent trust decision. PIM controls who can approve trust—but it does NOT limit what happens after trust is granted. That's why consent must be governed like any other privileged tenant configuration."

---

# 📊 SLIDE-BY-SLIDE GUIDE

---

## Slide 1: Title / Intro

**SAY THIS:**
> "Before we dive in, I want to set context. Consent phishing is one of the top attack vectors CISA tracks against federal agencies. It bypasses MFA entirely—you're not stealing a password, you're getting legitimate access granted to a malicious app."

**IF THEY ASK "Why does this matter?"**
> "Because once a user clicks Accept on a malicious app, that app has API access to their mailbox, files, whatever permissions it requested. Password reset doesn't help. MFA doesn't help. The app has tokens, not credentials."

---

## Slide 2: Your Question Answered

**THE ANSWER:**
> Allow limited user consent for low-risk apps from verified publishers.  
> Require admin consent for everything else.  
> Enable Admin Consent Workflow so users can request access.

**SAY THIS:**
> "I know what you're thinking—'if we block all user consent, we'll drown in helpdesk tickets.' That's why Microsoft's recommendation is nuanced: LOW-RISK permissions from VERIFIED publishers only. Everything else goes through admin review."

**IF THEY PUSH BACK:**
> "This approach balances security with productivity. Users aren't blocked—they're redirected through a controlled process. You get visibility, they get access when appropriate."

---

## Slide 3: What Microsoft Recommends

**THE SETTING:**
- Policy: `microsoft-user-default-low`
- Users CAN consent to: verified publisher apps + low-risk permissions
- Users CANNOT consent to: unverified apps or sensitive permissions

**SAY THIS:**
> "This is Microsoft's official guidance—it's in their security baselines and Zero Trust deployment docs. It's not just a best practice, it's the recommended default."

**LINK TO DROP IF ASKED:**
- [Configure User Consent Settings](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/configure-user-consent)

---

## Slide 4-5: Where to Configure / Steps

**THE PATH:**
```
Entra Admin Center → Identity → Applications → Enterprise Applications → Consent and Permissions → User Consent Settings
```

**SAY THIS:**
> "This is a 5-minute configuration change with massive security impact. No agents, no infrastructure changes—just policy."

**THE STEPS:**
1. Set "Users can consent to apps" → **Allow for verified publishers, low-risk only**
2. Set "Users can request admin consent" → **Yes**
3. Add reviewers (use PIM-eligible roles)
4. Set notification email

---

## Slide 6: Permission Classifications

**WHAT TO SAY:**
> "You control what 'low risk' means for YOUR organization. Microsoft suggests these as low impact—but you decide."

**LOW RISK (safe for user consent):**
| Permission | What It Does |
|------------|--------------|
| `User.Read` | Read signed-in user's profile |
| `openid` | Sign users in |
| `profile` | View basic profile |
| `email` | View email address |

**HIGH RISK (require admin consent):**
| Permission | What It Does | Why It's Dangerous |
|------------|--------------|-------------------|
| `User.Read.All` | Read ALL user profiles | Directory enumeration |
| `Mail.Read` | Read user's mail | Data exfiltration |
| `Mail.Send` | Send mail as user | Phishing, BEC |
| `Mail.ReadWrite` | Full mailbox access | Complete compromise |
| `Files.ReadWrite.All` | Access all files | Data exfiltration |
| `Directory.Read.All` | Read entire directory | Reconnaissance |

**IF THEY ASK "What if we're really sensitive?"**
> "Then don't classify anything as low impact. Force everything through admin review. It's more friction but maximum control."

---

## Slide 7: Admin Consent Workflow

**THE FLOW:**
```
User tries to consent → Blocked → Submits request → Admin reviews → Approve/Deny
```

**SAY THIS:**
> "This is the key to making restrictive consent work. Users don't hit a wall—they submit a request. You get visibility, they get access when appropriate."

**WHERE TO CONFIGURE:**
```
Enterprise Applications → Consent and Permissions → Admin Consent Settings → Enable
```

---

## Slide 8: PIM Integration

**⚠️ CRITICAL NUANCE (don't skip this):**

> "Here's what most people get wrong: PIM only gates the *act of granting consent*. It does NOT gate the *use of the permission* afterward."

**WHAT THIS MEANS:**
- Admin activates PIM role → grants consent → role expires
- BUT: The consent is **permanent**
- The app can use that permission 24/7, on behalf of ANY user who signs in
- Until someone explicitly revokes it

**SAY THIS:**
> "PIM protects the approval process. But once consent is granted, it's a permanent trust artifact in your tenant. That's why we need ongoing governance—not just controlled approval."

**ROLES TO PUT BEHIND PIM:**
- Global Administrator
- Cloud Application Administrator  
- Application Administrator

---

## Slide 9: Summary / Action Items

**THE 5 ACTIONS:**
1. ✅ Configure user consent (verified + low-risk only)
2. ✅ Enable admin consent workflow
3. ✅ Classify permissions (define your low-risk list)
4. ✅ Put consent roles behind PIM
5. ✅ Review regularly (audit existing consents)

**CLOSE WITH:**
> "The biggest gap we see: admin consent is granted once and never looked at again. Strong tenants treat consent like any other privileged configuration—inventory, review, revoke when no longer needed."

---

# 🛡️ FEDERAL COMPLIANCE (If They Ask)

**Quick hits for each framework:**

| Framework | What to Say |
|-----------|-------------|
| **CISA ZTMM** | "Identity Pillar, Governance Function—Advanced maturity requires controlled app access with periodic review" |
| **CISA SCuBA** | "Azure AD baseline specifically requires controlling third-party application access" |
| **DoD Zero Trust** | "Never trust, always verify—at the application layer, that means admin consent for sensitive permissions" |
| **OMB M-22-09** | "Application-level access controls are required. If you can't control which apps access your data, you can't claim Zero Trust" |
| **FedRAMP** | "Third-party integrations must be authorized and monitored" |

**DOCUMENT LINKS:**
- [CISA ZTMM - Identity Pillar](https://learn.microsoft.com/en-us/security/zero-trust/cisa-zero-trust-maturity-model-identity)
- [CISA SCuBA Azure AD Baseline (PDF)](https://www.cisa.gov/sites/default/files/2023-12/Azure%20Active%20Directory%20SCB_12.20.2023.pdf)

---

# 💬 COMMON QUESTIONS (Have These Ready)

### "If we block user consent, won't we get overwhelmed?"
> "That's why we use permission classifications. Users self-consent to low-risk stuff. Only sensitive permissions require review. After the initial wave of pre-approving common apps, volume is manageable."

### "What if we already have a lot of apps with user consent?"
> "Audit first. Run the PowerShell script, identify apps with sensitive permissions, revoke what's not justified. Then implement controls going forward."

### "How do we know if an app is 'verified'?"
> "Blue checkmark in the consent prompt. Microsoft verified the publisher owns their domain. It's not a guarantee the app is safe—just that the publisher is who they claim to be."

### "What about Microsoft apps like Teams?"
> "First-party Microsoft apps are pre-consented. This is about third-party and custom apps."

### "What's the difference between delegated and application permissions?"
> "Delegated = app acts on behalf of a signed-in user. Application = app acts as itself, no user needed. For backend services with no UI, use application permissions + managed identity—not delegated."

### "Does PIM protect us after consent is granted?"
> "No. PIM controls WHO can approve consent. Once approved, the consent persists 24/7 until someone revokes it. That's why you need ongoing review, not just controlled approval."

---

# ⚠️ CONSENT PHISHING (If You Need to Explain the Threat)

**Attack Flow:**
1. Attacker registers app in Entra ID (anyone can do this)
2. App requests Mail.Read, Files.ReadWrite
3. Attacker sends phishing email: "Click to access shared document"
4. User clicks → sees legit Microsoft consent prompt → clicks Accept
5. Attacker now has API access to mailbox/files
6. **MFA doesn't help**—user authenticated legitimately

**Why This Bypasses Everything:**
| Your Defense | Why It Fails |
|--------------|--------------|
| Password reset | App has OAuth tokens, not password |
| MFA | User already passed MFA |
| Email filter | Link goes to real Microsoft domain |
| EDR | No malware—it's cloud API calls |

---

# 🔍 MONITORING (For Follow-Up Conversations)

**Sentinel KQL - Detect New Consent Grants:**
```kql
AuditLogs
| where OperationName == "Consent to application"
| extend AppName = tostring(TargetResources[0].displayName)
| extend User = tostring(InitiatedBy.user.userPrincipalName)
| project TimeGenerated, User, AppName
| order by TimeGenerated desc
```

**Defender for Cloud Apps:**
- OAuth app governance
- Auto-detect risky apps
- Alert on sensitive permission grants

**Audit Script:**
- [Get-AzureADPSPermissions.ps1](https://gist.github.com/psignoret/41793f8c6211d2df5051d77ca3728c09)

---

# ✅ IMPLEMENTATION CHECKLIST (Give to Customer)

- [ ] Audit existing application consents
- [ ] Revoke suspicious/unnecessary consents
- [ ] Configure: "Allow for verified publishers, low-risk only"
- [ ] Define permission classifications
- [ ] Enable admin consent workflow
- [ ] Designate reviewers (PIM-eligible)
- [ ] Pre-approve known business apps
- [ ] Set up monitoring (Sentinel/MDCA)
- [ ] Document for compliance audits

---

# 📚 REFERENCE LINKS

| Topic | Link |
|-------|------|
| Configure User Consent | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/configure-user-consent) |
| Admin Consent Workflow | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/admin-consent-workflow-overview) |
| Permission Classifications | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/configure-permission-classifications) |
| Illicit Consent Grants | [learn.microsoft.com](https://learn.microsoft.com/en-us/defender-office-365/detect-and-remediate-illicit-consent-grants) |
| Consent Investigation Playbook | [learn.microsoft.com](https://learn.microsoft.com/en-us/security/operations/incident-response-playbook-app-consent) |
| Protect Against Consent Phishing | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/protect-against-consent-phishing) |

---

*Last Updated: February 5, 2026*
