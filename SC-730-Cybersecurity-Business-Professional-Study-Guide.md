# SC-730: Cybersecurity Business Professional Study Guide

_Last updated: May 15, 2026_

This guide breaks down the SC-730 exam objectives into plain-English study areas, then points you to the relevant Microsoft Learn pages for deeper reading. Use the breakdown first to understand what the exam is testing, then use the linked Microsoft Learn content to fill in the details.

The audience is a business professional who uses email, Teams, Microsoft 365, cloud files, mobile devices, remote access, and AI tools. You are not expected to tune SIEM rules or administer Defender. You are expected to recognize risk, protect data, follow policy, and escalate early.

## Exam Snapshot

| Area | Weight | What to Focus On |
| --- | ---: | --- |
| Understand cybersecurity concepts | 25-30% | Shared responsibility, accountability, policies, MFA, password managers, ransomware impact, key terms, deepfakes |
| Understand cybersecurity risks and threats | 30-35% | Public Wi-Fi, phishing, pretexting, baiting, malware indicators, insider threat signals, suspicious emails and links |
| Apply basic security practices to protect the organization | 25-30% | Secure devices/accounts/workspaces, sensitivity labels, rights management, data handling, backups and recovery |
| Report and respond to security incidents | 10-15% | When to report, what details to include, reporting channels, breach actions, escalation triggers |

Passing score: 700 or greater.

## Study Breakdown With Microsoft Learn Links

Start with the official exam guide, then study by objective. The goal is not to memorize every product feature. The goal is to understand what a business user should recognize, avoid, protect, verify, report, or escalate.

Start here:

- [Study guide for Exam SC-730: Cybersecurity Business Professional](https://learn.microsoft.com/credentials/certifications/resources/study-guides/sc-730)
- [Describe security and compliance concepts](https://learn.microsoft.com/training/modules/describe-security-concepts-methodologies/)
- [Shared responsibility in the cloud](https://learn.microsoft.com/azure/security/fundamentals/shared-responsibility)

### Domain 1: Understand Cybersecurity Concepts (25-30%)

#### Roles, Responsibilities, and Shared Responsibility

Breakdown:

- Microsoft and cloud providers protect the cloud platform and services.
- The organization owns its data, identities, access decisions, device hygiene, and policies.
- Employees own their behavior: protecting credentials, using approved tools, following labels, and reporting suspicious activity.
- Managers and business owners reinforce accountability and approve access based on business need.

Microsoft Learn:

- [SC-730 skills measured](https://learn.microsoft.com/credentials/certifications/resources/study-guides/sc-730#skills-measured)
- [Shared responsibility in the cloud](https://learn.microsoft.com/azure/security/fundamentals/shared-responsibility)
- [Describe the shared responsibility model](https://learn.microsoft.com/training/modules/describe-security-concepts-methodologies/2-describe-shared-responsibility-model)
- [Microsoft Cybersecurity Defense Operations Center](https://learn.microsoft.com/security/engineering/fy18-strategy-brief)

#### Security Awareness, Accountability, and Policies

Breakdown:

- Know examples of security awareness: training, reporting phishing, following clean desk practices, using approved apps, and applying data labels.
- Accountability means actions are traceable and users follow policy even when shortcuts seem faster.
- Know the purpose of security, privacy, acceptable use, remote work, data classification, and incident reporting policies.

Microsoft Learn:

- [SC-730 cybersecurity concepts objectives](https://learn.microsoft.com/credentials/certifications/resources/study-guides/sc-730#understand-cybersecurity-concepts-2530)
- [Describe governance, risk, and compliance concepts](https://learn.microsoft.com/training/modules/describe-security-concepts-methodologies/6-describe-compliance-concepts)
- [Microsoft 365 for business security best practices](https://learn.microsoft.com/microsoft-365/admin/security-and-compliance/m365b-security-best-practices)

#### Passwords, Password Managers, and MFA

Breakdown:

- Password reuse is risky because one breached site can expose other accounts.
- Password managers help create strong, unique passwords and can reduce phishing risk by autofilling only on matching sites.
- MFA makes stolen passwords less useful.
- Phishing-resistant MFA is stronger than SMS, email OTP, or basic push approvals.
- Unexpected MFA prompts should be denied and reported.

Microsoft Learn:

- [Microsoft Edge password manager security](https://learn.microsoft.com/deployedge/microsoft-edge-security-password-manager-security)
- [Password policy recommendations for Microsoft 365 passwords](https://learn.microsoft.com/microsoft-365/admin/misc/password-policy-recommendations)
- [Microsoft Entra authentication overview](https://learn.microsoft.com/entra/identity/authentication/overview-authentication)
- [Phishing-resistant authentication methods](https://learn.microsoft.com/entra/identity/authentication/overview-authentication#phishing-resistant-authentication-methods)

#### AI Tools and Sensitive Data

Breakdown:

- Treat AI prompts like a data-sharing location.
- Do not paste credentials, customer data, personal data, HR data, legal material, source code, security findings, confidential files, or regulated information into unauthorized AI tools.
- Approved AI still requires policy-aware use, correct account context, and allowed data types.

Microsoft Learn:

- [Protect sensitive data from AI-related risks](https://learn.microsoft.com/training/modules/purview-ai-protect-sensitive-data/)
- [Microsoft Purview data security and compliance protections for generative AI apps](https://learn.microsoft.com/purview/ai-microsoft-purview)
- [Block sensitive data going to sanctioned AI apps](https://learn.microsoft.com/purview/deploymentmodels/depmod-data-leak-shadow-ai-step3)

#### Core Terms and Emerging Threats

Breakdown:

- Vulnerability: weakness.
- Threat: possible cause of harm.
- Risk: likelihood and impact of harm.
- Exploit: method used to take advantage of a weakness.
- Encryption: protects readable data by making it unreadable without a key.
- Deepfake: manipulated or synthetic media used to impersonate or mislead.

Microsoft Learn:

- [Describe security and compliance concepts](https://learn.microsoft.com/training/modules/describe-security-concepts-methodologies/)
- [Describe encryption and hashing](https://learn.microsoft.com/training/modules/describe-security-concepts-methodologies/5-describe-encryption-hashing)
- [Microsoft Cybersecurity Defense Operations Center](https://learn.microsoft.com/security/engineering/fy18-strategy-brief)

### Domain 2: Understand Cybersecurity Risks and Threats (30-35%)

#### Phishing, Spoofing, and Social Engineering

Breakdown:

- Phishing tries to steal credentials, money, or data by pretending to be legitimate.
- Spear phishing is targeted; whaling targets executives; business email compromise targets payments and trusted business workflows.
- Spoofing makes a message, sender, site, or caller appear legitimate.
- Pretexting uses a fake story; baiting uses a tempting file, reward, or device.
- Best response: do not click, open, reply, or approve. Verify through a known channel and report.

Microsoft Learn:

- [Protect users against phishing and other attacks in Microsoft 365 for business](https://learn.microsoft.com/microsoft-365/admin/security-and-compliance/m365b-users-phishing-spam-malware)
- [Anti-phishing protection in cloud organizations](https://learn.microsoft.com/defender-office-365/anti-phishing-protection-about)
- [How to protect against phishing attacks](https://learn.microsoft.com/defender-endpoint/malware/phishing)
- [Report suspicious email or files to Microsoft](https://learn.microsoft.com/defender-office-365/submissions-report-messages-files-to-microsoft)

#### Malware, Ransomware, and Abnormal System Behavior

Breakdown:

- Malware signs include sudden slowness, pop-ups, disabled security tools, unknown apps, browser redirects, locked files, or messages sent without the user.
- Ransomware can encrypt files, steal data, disrupt operations, and target backups.
- The user response is to stop interacting, preserve evidence, and report quickly.

Microsoft Learn:

- [Prevent malware infection](https://learn.microsoft.com/defender-endpoint/malware/prevent-malware-infection)
- [Human-operated ransomware](https://learn.microsoft.com/security/ransomware/human-operated-ransomware)
- [Microsoft Incident Response ransomware approach and best practices](https://learn.microsoft.com/security/ransomware/incident-response-playbook-dart-ransomware-approach)

#### Public Wi-Fi and Remote Work Risks

Breakdown:

- Public Wi-Fi can involve fake hotspots, malicious captive portals, interception, and credential theft.
- Remote work adds risks from unmanaged networks, personal devices, shared spaces, shoulder surfing, and local-only storage.
- Use approved secure access, managed devices, updates, screen locks, and approved storage.

Microsoft Learn:

- [Microsoft 365 VPN split tunneling overview](https://learn.microsoft.com/microsoft-365/enterprise/microsoft-365-vpn-split-tunnel)
- [Microsoft 365 for business security best practices](https://learn.microsoft.com/microsoft-365/admin/security-and-compliance/m365b-security-best-practices)
- [Prevent malware infection](https://learn.microsoft.com/defender-endpoint/malware/prevent-malware-infection)

#### Insider Risk Indicators

Breakdown:

- Insider risk can be malicious, careless, or accidental.
- Indicators include unusual access, large data downloads, personal email forwarding, unauthorized storage, bypass attempts, or access unrelated to job duties.
- Do not confront the person; report observable facts through the correct process.

Microsoft Learn:

- [Microsoft Purview data security solutions](https://learn.microsoft.com/purview/purview-security)
- [Microsoft Purview Insider Risk Management](https://learn.microsoft.com/purview/insider-risk-management)
- [Microsoft Cybersecurity Defense Operations Center](https://learn.microsoft.com/security/engineering/fy18-strategy-brief)

### Domain 3: Apply Basic Security Policies to Protect the Organization (25-30%)

#### Securing Devices, Accounts, and Workspaces

Breakdown:

- Secure accounts with unique passwords, password managers, MFA, and reporting of suspicious sign-ins.
- Secure devices with updates, endpoint protection, screen locks, approved apps, and immediate lost-device reporting.
- Secure workspaces by protecting screens, printed documents, badges, and sensitive conversations.

Microsoft Learn:

- [Microsoft 365 for business security best practices](https://learn.microsoft.com/microsoft-365/admin/security-and-compliance/m365b-security-best-practices)
- [Step 3: Protect your Microsoft 365 user accounts](https://learn.microsoft.com/microsoft-365/enterprise/microsoft-365-secure-sign-in)
- [Prevent malware infection](https://learn.microsoft.com/defender-endpoint/malware/prevent-malware-infection)

#### Sensitive Data, Sensitivity Labels, and Rights Management

Breakdown:

- Sensitive data includes personal, financial, health, HR, student, customer, legal, credential, source code, and confidential business information.
- Sensitivity labels classify and protect files and emails.
- Labels can apply encryption, access restrictions, visual markings, and sharing controls.
- Rights management controls what a recipient can do with protected content: view, edit, print, copy, forward, or access after expiration.

Microsoft Learn:

- [Microsoft Purview data security solutions](https://learn.microsoft.com/purview/purview-security)
- [Learn about sensitivity labels](https://learn.microsoft.com/purview/sensitivity-labels)
- [Get started with sensitivity labels](https://learn.microsoft.com/purview/get-started-with-sensitivity-labels)
- [Microsoft 365 Rights Management](https://learn.microsoft.com/azure/azure-sovereign-clouds/public/microsoft-365-rights-management)
- [Implement information protection and DLP by using Microsoft Purview](https://learn.microsoft.com/training/paths/purview-implement-information-protection-data-loss-prevention/)

#### Data Handling, DLP, Backup, and Recovery

Breakdown:

- Data handling covers collect, use, transfer, store, retain, and destroy.
- Use approved storage and sharing locations so access controls, labels, DLP, audit, retention, and backup can apply.
- DLP helps prevent sensitive data from being leaked through email, endpoints, browsers, cloud apps, or AI tools.
- Backups matter only if data is stored where it can be protected and restored.

Microsoft Learn:

- [Microsoft Purview Data Loss Prevention](https://learn.microsoft.com/purview/dlp-learn-about-dlp)
- [Microsoft Purview data security solutions](https://learn.microsoft.com/purview/purview-security)
- [Prevent data loss in Microsoft Purview](https://learn.microsoft.com/training/modules/m365-compliance-information-prevent-data-loss/)
- [Microsoft Incident Response ransomware approach and best practices](https://learn.microsoft.com/security/ransomware/incident-response-playbook-dart-ransomware-approach)

### Domain 4: Report and Respond to Security Incidents (10-15%)

#### What to Report and What to Include

Breakdown:

- Report phishing, lost devices, malware signs, ransomware, unexpected MFA prompts, unauthorized access, accidental data sharing, and suspicious payment or access changes.
- Include facts: date, time, affected account/device/file/app, sender, URL, attachment, what happened, what action was taken, and whether sensitive data was involved.
- Do not delete evidence, forward broadly, or investigate alone.

Microsoft Learn:

- [Incident response playbooks](https://learn.microsoft.com/security/operations/incident-response-playbooks)
- [Report suspicious email or files to Microsoft](https://learn.microsoft.com/defender-office-365/submissions-report-messages-files-to-microsoft)
- [Security incident management overview](https://learn.microsoft.com/compliance/assurance/assurance-incident-management)

#### Breach, Ransomware, and Escalation Actions

Breakdown:

- Stop unsafe activity.
- Report immediately.
- Preserve evidence.
- Disconnect or isolate only when policy or IT/security guidance says to, or when active malware/ransomware behavior is occurring.
- Escalate when sensitive data, credentials, ransomware, lost devices, money movement, or business impact is involved.

Microsoft Learn:

- [Microsoft Incident Response ransomware approach and best practices](https://learn.microsoft.com/security/ransomware/incident-response-playbook-dart-ransomware-approach)
- [Human-operated ransomware](https://learn.microsoft.com/security/ransomware/human-operated-ransomware)
- [Incident response playbooks](https://learn.microsoft.com/security/operations/incident-response-playbooks)

## What the Exam Is Really Testing

SC-730 is about security judgment for business users. Most questions can be reduced to one of five decisions:

| Decision | Best Default |
| --- | --- |
| Is this request legitimate? | Verify through a known trusted channel before acting. |
| Is this data sensitive? | Classify it, label it, and share only through approved tools. |
| Is this sign-in or device behavior suspicious? | Stop, report, and let IT/security investigate. |
| Is this an incident? | Report quickly with facts; do not investigate alone. |
| Is this a shortcut around policy? | Do not bypass policy for urgency, convenience, or authority pressure. |

When answer choices look similar, choose the one that is:

- Least risky for sensitive data.
- Most aligned to organizational policy.
- Fastest to report or escalate when impact is unclear.
- Based on verification through a known channel, not a link or contact detail in the suspicious message.
- Focused on preserving evidence rather than deleting or forwarding suspicious content.

## Exam Reasoning Models

### The SC-730 Business User Loop

Use this loop for scenario questions:

1. Recognize: What is abnormal, sensitive, urgent, or outside normal process?
2. Pause: Do not click, approve, forward, download, paste, pay, or share yet.
3. Verify: Use a trusted channel or official workflow.
4. Protect: Apply label, access control, MFA, approved storage, or device hygiene.
5. Report: Use the approved channel and include factual details.
6. Escalate: Involve security/privacy/IT when sensitive data, credentials, ransomware, lost devices, or business impact are involved.

### The Data Sensitivity Test

If a question includes data, ask:

- Does it identify a person?
- Does it involve money, health, legal, HR, customer, student, government, or regulated information?
- Would exposure harm a customer, employee, partner, or the organization?
- Is it labeled Confidential, Highly Confidential, Restricted, or internal-only?
- Is it being copied into an unapproved tool, personal email, personal cloud storage, or public AI system?

If any answer is yes, the safer answer is to classify, label, restrict, verify recipients, and use approved sharing. Do not paste it into unmanaged AI tools.

### The Suspicious Request Test

Treat a request as suspicious when it includes:

- Urgency: "right now," "before close of business," "I am in a meeting."
- Authority pressure: executive, vendor, legal, HR, finance, IT support.
- Secrecy: "do not tell anyone," "keep this between us."
- Process bypass: unusual payment method, direct link, personal email, new bank details.
- Sensitive ask: credentials, MFA code, payroll, customer data, confidential file, privileged access.
- Technical mismatch: lookalike domain, odd link, unexpected attachment, unusual sender or reply-to.

Best answer: pause and verify through a known channel or approved workflow.

### The Incident Threshold

Do not wait for certainty. Report when there is a credible possibility of:

- Credential compromise.
- Sensitive data exposure.
- Lost or stolen work device.
- Malware or ransomware.
- Unauthorized access.
- Suspicious payment, payroll, vendor, or account change.
- Policy violation involving data, AI tools, or unapproved storage.

Exam trap: "Investigate it yourself" is usually weaker than "report through the approved channel" for a business user.

## How to Study

Use the exam weights to prioritize your time. The largest domain is cybersecurity risks and threats, so be comfortable recognizing suspicious behavior, social engineering, malware signs, abnormal device behavior, and risky communication requests. The next two large areas are cybersecurity concepts and basic protection practices, so you should know how everyday employee actions support security, privacy, and compliance.

A practical study rhythm:

1. Read the exam objective list once end to end.
2. Study one domain at a time using the notes below.
3. For each objective, ask: What would a business user see? What should they do? What should they avoid? When should they report or escalate?
4. Practice scenario questions, especially emails, unexpected payment requests, lost devices, AI prompts, public Wi-Fi, ransomware, and sensitive data handling.

Do not memorize only product names. SC-730 is more likely to test whether you understand what the control is for. For example, know that sensitivity labels classify and protect content, DLP helps prevent inappropriate sharing, MFA reduces account takeover risk, and incident reporting gets suspicious activity to the right team.

## Domain 1: Understand Cybersecurity Concepts

### Shared Responsibility

Cybersecurity is shared across the organization. Security teams build policies, tools, monitoring, and response processes, but every employee has responsibilities that reduce risk.

Know these responsibilities:

- Employees follow policy, protect credentials, use approved tools, complete awareness training, and report suspicious activity.
- Managers reinforce accountable behavior, make sure teams know reporting channels, and support policy compliance.
- IT and security teams deploy controls, monitor risk, investigate incidents, and recover services.
- Cloud providers protect their infrastructure and services, while customers and users still protect accounts, data, devices, configuration, and usage.

Exam tip: If a question asks who owns security, avoid answers that put all responsibility on IT. Business users own their behavior, data handling, and reporting obligations.

How to answer shared responsibility questions:

| Question Pattern | Strong Answer |
| --- | --- |
| "Who should report phishing?" | The user who receives it, using the approved reporting channel. |
| "Who investigates?" | IT/security, not the business user. |
| "Who decides policy?" | The organization through security/privacy/compliance leadership. |
| "Who follows policy?" | Everyone, including business users and managers. |
| "Who protects cloud infrastructure?" | The cloud/service provider protects the platform; the customer protects users, access, data, and configuration. |

### Security Awareness and Accountability

Employee participation in security awareness includes:

- Completing required training.
- Reporting phishing and suspicious requests.
- Following clean desk and workspace rules.
- Using approved apps, storage locations, and collaboration channels.
- Applying labels or protections when handling sensitive content.
- Keeping devices updated and locked.
- Asking for verification when a request seems unusual.

Accountability means users can explain and follow the policy that applies to their work. It also means reporting mistakes quickly, such as sending sensitive data to the wrong person.

### Policies and Data-Handling Standards

You should understand how policies guide routine work:

- Security policy: rules for accounts, devices, software, remote work, and reporting.
- Privacy policy: rules for collecting, using, sharing, storing, retaining, and deleting personal data.
- Acceptable use policy: rules for approved devices, networks, apps, and internet use.
- Data classification policy: rules for labeling and protecting information based on sensitivity.
- Incident reporting policy: when, where, and how to report suspicious events or policy violations.

### What Not to Share with AI Tools

Do not paste or upload sensitive information into AI tools unless the organization has approved that use and the tool is governed by company policy. Examples include:

- Passwords, secrets, API keys, tokens, private keys, or credentials.
- Personal data such as names with IDs, health data, financial data, government IDs, or HR records.
- Customer data, contract terms, confidential pricing, source code, legal material, or regulated data.
- Internal-only business plans, incident details, security findings, or vulnerability information.
- Any data labeled Confidential, Highly Confidential, Restricted, or similar.

Microsoft Purview can help organizations block sensitive data in AI prompts through endpoint DLP, browser data security, network data security, sensitivity labels, and monitoring.

Think of AI tools as another sharing destination. If you would not email the data to an external party or paste it into a public website, you should not paste it into an unapproved AI prompt. Even approved AI tools still require policy-aware use: use the right account, approved tenant, allowed data types, and proper labels.

Common exam trap: "Remove the customer name and paste the contract" may still be wrong if the contract includes proprietary terms, pricing, legal language, or other identifiable details. Sanitization must be complete and policy-approved.

### Password Managers and Strong Passwords

Password managers help users create and store strong, unique passwords. This reduces password reuse and makes phishing harder because password managers typically autofill only on the legitimate site where the password belongs.

Know these password principles:

- Use unique passwords for work and nonwork accounts.
- Avoid easy-to-guess passwords like common words, names, birthdays, or keyboard patterns.
- Do not reuse organization passwords on external sites.
- Use MFA or passwordless authentication where available.
- Prefer approved password managers over writing passwords in notes, documents, or chats.

### Multifactor Authentication

MFA protects accounts by requiring more than one proof of identity. The common factors are:

- Something you know: password or PIN.
- Something you have: phone, security key, registered device.
- Something you are: biometric factor.

Microsoft recommends phishing-resistant authentication such as Windows Hello for Business, passkeys/FIDO2 security keys, passkeys in Microsoft Authenticator, and certificate-based authentication. Traditional MFA is better than password-only sign-in, but SMS, one-time codes, and push approvals can still be targeted by phishing or user fatigue attacks.

Exam tip: MFA reduces the chance that a stolen password alone gives an attacker access.

Unexpected MFA prompts are suspicious. The correct user behavior is to deny the prompt if possible and report it. Approving an MFA prompt you did not initiate can give an attacker access.

Phishing-resistant MFA is stronger because it is designed to resist fake sign-in pages and token theft. Passkeys/FIDO2, Windows Hello for Business, and certificate-based authentication are stronger than SMS codes or simple push approvals.

### Business Processes Attackers Target

Attackers target business processes that move money, data, access, or trust. Examples:

- Invoice payment and wire transfer approval.
- Vendor onboarding and procurement.
- Payroll and direct deposit changes.
- Executive communications.
- Customer support workflows.
- File sharing and document approval.
- Password reset and account recovery.
- Help desk identity verification.

If a request creates urgency, bypasses normal approval, asks for secrecy, changes payment details, or requests sensitive data, verify through a known trusted channel.

High-risk business workflows usually involve one of four things: money, identity, access, or sensitive data. If a scenario involves any of these, assume normal process matters more than speed.

### Remote Work Risks

Remote work increases exposure because users may work from home networks, public Wi-Fi, shared spaces, personal devices, or unmanaged locations.

Key controls:

- Use approved VPN, Zero Trust access, or secure access tools when required.
- Keep devices updated and protected by endpoint security.
- Lock screens when away.
- Protect screens from shoulder surfing.
- Avoid public charging stations and untrusted peripherals.
- Use trusted networks or hotspots instead of unknown public Wi-Fi.
- Store work files only in approved locations.

### Updates and Security Patches

Software updates fix vulnerabilities that attackers can exploit. Required updates should not be ignored because unpatched systems are easier targets for malware, ransomware, and unauthorized access.

### Ransomware Impact

Ransomware can encrypt data, disrupt operations, disable systems, steal information, damage reputation, and create legal or regulatory obligations. Modern ransomware actors often try to disable security tools, delete or corrupt backups, and steal data before encrypting systems.

Business user actions matter:

- Do not open suspicious files or links.
- Report early signs quickly.
- Disconnect from the network if instructed by policy or IT.
- Do not pay or negotiate with attackers yourself.
- Preserve evidence and follow incident response instructions.

### Key Terms

| Term | Meaning |
| --- | --- |
| Vulnerability | A weakness that could be exploited, such as unpatched software or poor configuration |
| Threat | A potential cause of harm, such as a criminal group, malware, or phishing campaign |
| Risk | The likelihood and impact of harm when a threat can exploit a vulnerability |
| Encryption | Converting readable data into protected ciphertext that requires a key to read |
| Exploit | A method or code used to take advantage of a vulnerability |
| Deepfake | Synthetic or manipulated audio, image, or video that impersonates a real person or event |

Term relationship to remember: a threat uses an exploit against a vulnerability, which creates risk. Encryption reduces the impact of exposure because unreadable data is harder to misuse without the key.

## Domain 2: Understand Cybersecurity Risks and Threats

This is the heaviest domain. Expect many "spot the red flag" questions. The best answers usually avoid interaction with the suspicious item, verify independently, and report through the approved channel.

### Public Wi-Fi Risks

Public Wi-Fi can expose users to network eavesdropping, fake hotspots, man-in-the-middle attacks, malicious captive portals, and credential theft.

Safer behavior:

- Avoid sensitive work on public Wi-Fi when possible.
- Use approved VPN or secure access tools if required.
- Verify the network name with the venue.
- Avoid entering credentials on suspicious captive portals.
- Use HTTPS sites and approved apps.
- Turn off auto-join for unknown networks.

### Social Engineering

Social engineering manipulates people into taking unsafe actions.

| Technique | What It Looks Like | Safer Response |
| --- | --- | --- |
| Phishing | Fake email, text, Teams message, or website asking for credentials, money, data, or clicks | Do not click; report it; verify through trusted channels |
| Pretexting | Attacker invents a believable story, such as IT support, a vendor, or an executive request | Verify identity and request legitimacy through known channels |
| Baiting | Attacker offers a reward, download, USB drive, file, or benefit to lure action | Do not interact with unknown media or files |
| Spoofing | Message appears to come from a trusted person or company but is forged | Check sender details, links, context, and request pattern |

### Suspicious Email and Communication Signs

Look for:

- Urgent or threatening language.
- Requests for credentials, MFA codes, payment, gift cards, or sensitive data.
- Unexpected attachments or links.
- Sender address mismatches or lookalike domains.
- External forwarding or unusual reply-to addresses.
- Poor grammar, strange formatting, or outdated logos.
- Requests to bypass process or keep the request secret.
- A login page that looks familiar but has inconsistencies.
- A link destination that does not match the displayed text.

Best answer pattern: Do not reply, click, or open attachments. Report using the approved channel, such as Outlook's Report button, help desk, or incident form.

Link-reading trick: the visible text is not enough. A link that says `microsoft.com` can point somewhere else. In exam wording, "hover over the link" usually means inspect the real destination. If the destination is unexpected, shortened, misspelled, or unrelated, do not use it.

Attachment trick: an expected business process can still be suspicious when the file is unexpected, password-protected, macro-enabled, or arrives from a lookalike sender. Verify first.

### Malware Indicators

Common signs of malware or infection:

- Device becomes slow, unstable, or crashes repeatedly.
- Unexpected pop-ups, browser redirects, or new toolbars.
- Antivirus or security tools are disabled.
- Unknown apps appear.
- Files are missing, encrypted, renamed, or inaccessible.
- Unusual network activity or battery drain.
- Messages are sent from your account without your knowledge.
- Frequent account lockouts or sign-in prompts.

### Insider Threat Indicators

Potential insider threat signals can include:

- Unusual access to sensitive files unrelated to job duties.
- Large downloads, copying, or transfers of data.
- Attempts to bypass controls.
- Use of unauthorized storage, personal email, or unmanaged AI tools.
- Access at unusual times or from unusual locations.
- Sudden changes in behavior combined with risky data activity.

Not every signal means malicious intent. Some insider risk is accidental or negligent. The correct response is to follow reporting and investigation processes, not confront the person directly.

Exam trap: Avoid answers that assume guilt. The safer framing is "potential insider risk indicator" and "report through the appropriate process."

### Verify Digital Requests

When asked for access, payment, sensitive data, or urgent action:

- Verify using a trusted phone number, known Teams chat, or official workflow.
- Do not use contact information provided only in the suspicious message.
- Check whether the request matches normal process.
- Confirm identity and authority.
- Escalate requests involving sensitive data, financial change, privileged access, or policy exceptions.

### Access Controls

Access controls limit who can access systems and data.

Know these examples:

- Least privilege: users get only the access they need.
- MFA: adds a second proof of identity.
- Role-based access: permissions are assigned by role.
- Conditional Access: sign-in decisions can consider user, device, location, risk, and app.
- Device compliance: access can require a healthy, managed device.
- Sensitivity labels and rights management: protect files and emails after sharing.
- Approval workflows: require manager or owner approval for access.

How to distinguish controls:

| Need | Best-Matching Control |
| --- | --- |
| Prove the user is really the user | MFA or phishing-resistant authentication |
| Give only needed permissions | Least privilege or role-based access |
| Decide access based on device/location/risk | Conditional Access |
| Protect a document after it is shared | Sensitivity label with encryption/rights management |
| Stop sensitive data from being uploaded or pasted somewhere risky | Data Loss Prevention |
| Remove access when business need ends | Access review or offboarding process |

## Domain 3: Apply Basic Security Practices to Protect the Organization

### Secure Devices, Accounts, and Workspaces

For accounts:

- Use strong unique passwords or passwordless sign-in.
- Use MFA and do not approve unexpected MFA prompts.
- Never share passwords or MFA codes.
- Report suspicious sign-in prompts or account activity.

For devices:

- Keep OS and apps updated.
- Use approved endpoint protection.
- Lock the screen when away.
- Do not install unauthorized software.
- Report lost or stolen devices immediately.

For workspaces:

- Protect screens and printed documents.
- Store sensitive papers securely.
- Use approved collaboration and storage locations.
- Avoid discussing sensitive information in public spaces.

### Recognize and Classify Sensitive Data

Sensitive data can include:

- Personal information and identifiers.
- Financial, health, legal, HR, or student records.
- Customer records and contracts.
- Credentials, secrets, access tokens, and keys.
- Intellectual property, source code, business plans, and confidential strategy.
- Security incidents, vulnerabilities, architecture, or investigation data.

### Sensitivity Labels

Microsoft Purview sensitivity labels classify and protect data across Microsoft 365 apps and services. Labels can apply:

- Encryption.
- Access restrictions.
- Visual markings such as headers, footers, and watermarks.
- Sharing restrictions.
- Protection that travels with the file or email.

Typical label examples:

| Label | Use Case |
| --- | --- |
| Public | Approved for anyone |
| General/Internal | Routine work content for employees |
| Confidential | Business-sensitive information that should be limited |
| Highly Confidential/Restricted | Regulated, high-impact, or need-to-know data |

Exam tip: Apply the label that matches the sensitivity and sharing need. A document containing personal data, financial records, or customer confidential content should not be left unlabeled or treated as public.

How labels differ from permissions: permissions decide who can access a location or item; labels describe and protect the sensitivity of the content itself. A file in SharePoint can still need a label because it may be downloaded, emailed, or shared.

### Rights Management

Rights management protects what people can do with content. It can restrict viewing, editing, printing, forwarding, copying, downloading, or access expiration. This helps protect data even after it leaves its original storage location.

### Safe Internet and Data Handling

Data handling covers the full lifecycle:

- Collect only what is needed.
- Use data only for approved purposes.
- Store data in approved systems.
- Share data only with authorized people.
- Transfer data using approved secure methods.
- Retain data only as long as required.
- Destroy or delete data according to policy.

Avoid:

- Sending sensitive data to personal email.
- Uploading work files to unapproved cloud storage.
- Sharing broad links when specific people should be granted access.
- Posting sensitive data into unmanaged AI tools.
- Keeping copies after the retention period.

Data handling is a lifecycle question. The exam may ask what to do at collection, use, transfer, storage, retention, or disposal. A strong answer minimizes data, uses approved storage, limits sharing, and follows retention policy.

### Backup and Recovery

Backups support recovery from accidental deletion, device loss, corruption, malware, and ransomware. Business users should know where files are supposed to be stored so organizational backup and retention protections apply.

Good practices:

- Save work in approved locations such as OneDrive, SharePoint, Teams, or other governed systems.
- Do not keep the only copy on a local desktop or removable drive.
- Follow recovery instructions from IT.
- Report missing, encrypted, or corrupted files quickly.

Backup exam logic: backups matter only if they are protected and restorable. For a business user, the practical action is to save work in approved locations and report data loss quickly rather than keeping unmanaged local-only copies.

## Domain 4: Report and Respond to Security Incidents

### Situations That Require Reporting

Report:

- Phishing attempts or suspicious messages.
- Lost or stolen devices.
- Unauthorized access or unexpected sign-in prompts.
- Accidental sharing of sensitive data.
- Malware symptoms or ransomware notes.
- Requests to bypass normal process.
- Suspicious payment, vendor, payroll, or access changes.
- Policy violations involving sensitive data, credentials, or unauthorized tools.

### What to Include in a Report

Include facts, not guesses:

- Date and time discovered.
- Type of incident.
- Affected device, account, file, app, or data.
- Sender, recipient, URLs, attachment names, or screenshots if policy allows.
- What action you took, such as clicked link, opened file, replied, approved MFA, or shared data.
- Whether sensitive data may have been exposed.
- Any business process affected.

### Reporting Channels

Use the channel your organization defines. Examples:

- Outlook Report button.
- Help desk ticket.
- Security incident form.
- Security operations mailbox.
- Manager escalation.
- Privacy or compliance reporting process.

Exam tip: Reporting quickly is usually more important than investigating on your own.

### Basic Breach Actions

If a breach or suspected breach occurs:

- Stop sharing the affected data.
- Report immediately through the correct channel.
- Preserve evidence such as the message, file, or screenshot if policy allows.
- Disconnect a device from the network if instructed or if ransomware/malware is suspected.
- Do not delete evidence unless instructed.
- Do not contact the attacker.
- Follow IT/security instructions for password reset, device isolation, or recovery.

Escalation is required when sensitive data is exposed, ransomware is suspected, privileged access may be compromised, a lost device contains work data, or business operations are disrupted.

## Scenario Practice

Use these scenarios to practice the business-user reasoning the exam is likely to test.

### Scenario 1: Unexpected MFA Prompt

You receive an MFA approval request, but you are not signing in.

Best response: Deny the prompt if possible, report it immediately, and follow account recovery or password reset instructions. Do not approve to make the prompt go away.

### Scenario 2: Vendor Payment Change

A vendor emails new bank details and says payment must be sent today.

Best response: Verify through an existing trusted vendor contact or approved procurement workflow. Do not rely on phone numbers or links in the email.

### Scenario 3: Public Wi-Fi

You need to review confidential customer files from an airport Wi-Fi network.

Best response: Avoid public Wi-Fi for sensitive work when possible. Use approved secure access/VPN if required, make sure the device is managed and updated, and do not download files to unmanaged storage.

### Scenario 4: AI Prompt

A coworker wants to paste a customer contract into a public AI chatbot to summarize it.

Best response: Do not paste customer confidential or regulated data into unmanaged AI tools. Use only approved AI tools and follow data classification, DLP, and privacy policy.

### Scenario 5: Possible Ransomware

Files are suddenly renamed and unreadable, and a ransom note appears.

Best response: Stop using the device, disconnect from the network if instructed by policy or IT, report immediately, preserve evidence, and wait for security guidance.

### Scenario 6: Suspicious Attachment

You receive an unexpected invoice attachment from a lookalike vendor domain.

Best response: Do not open it. Report the message and verify through a known vendor channel if there is a business need.

### Scenario 7: Lost Phone

You lose a phone that has work email and Teams access.

Best response: Report immediately so IT can locate, lock, wipe, or revoke access according to policy.

### Scenario 8: Executive Deepfake Call

You receive a voice call that sounds like a senior executive asking you to urgently send customer data to a personal email address.

Best response: Treat the request as suspicious because it involves urgency, authority pressure, sensitive data, and an unusual channel. Verify through a known trusted channel and report if suspicious.

### Scenario 9: Unapproved AI Summary

A team member copied HR performance comments into a public AI chatbot to summarize them.

Best response: Report the potential data exposure through the appropriate privacy/security channel. HR data is sensitive and should not be shared with unapproved AI tools.

### Scenario 10: Unexpected Password Reset Email

You receive a password reset email that you did not request.

Best response: Do not click the link. Go directly to the official site or contact IT through a trusted channel, then report the suspicious reset request.

### Scenario 11: Sensitive File Shared Too Broadly

You notice a payroll file is shared with "Anyone with the link."

Best response: Stop or correct the sharing if you are authorized, report the exposure, and apply the correct access restriction and sensitivity label.

### Scenario 12: Personal Cloud Storage

A user uploads a confidential project file to a personal cloud drive to work from home.

Best response: This is unsafe data handling. Work files should stay in approved organizational storage where access controls, labels, DLP, retention, and audit can apply.

### Scenario 13: Browser Warning on a Link

A security warning appears after clicking a link from an email that looked like a Microsoft sign-in page.

Best response: Do not proceed. Close the page, report the message, and change credentials only through an official trusted path if instructed.

### Scenario 14: Suspicious Inbox Rule

You find a rule forwarding all your email to an unknown external address.

Best response: Treat it as possible account compromise, report it immediately, and follow IT/security instructions.

### Scenario 15: Coworker Requests Your MFA Code

A coworker says they are locked out and asks you to approve a prompt or share your code so they can finish a task.

Best response: Never share MFA codes or approve another person's sign-in. Direct them to the help desk or official account recovery process.

## Answer Choice Red Flags

Be skeptical of answer choices that say to:

- Click the link to see whether it is legitimate.
- Forward the suspicious email to coworkers for awareness.
- Delete the evidence before reporting.
- Approve an MFA prompt to stop repeated notifications.
- Use personal email or personal storage because it is faster.
- Paste sensitive data into an unapproved AI tool after only minor edits.
- Contact the sender using the phone number in the suspicious message.
- Ignore the issue because there is no proof yet.
- Confront a suspected insider directly.
- Disable security software to complete a task.

Prefer answer choices that say to:

- Report using the approved channel.
- Verify through a known trusted path.
- Preserve evidence.
- Use approved storage and sharing.
- Apply appropriate labels and access restrictions.
- Escalate sensitive data exposure, ransomware, lost devices, or credential compromise.
- Follow organizational policy.

## Mini Practice Questions

1. A project manager receives an email from a vendor asking to change bank details before an invoice is paid. What should they do first?

Answer: Verify through a known trusted vendor contact or approved procurement workflow before making any change.

2. A user gets an MFA approval prompt while not signing in. What is the safest action?

Answer: Deny the prompt if possible and report it as suspicious.

3. A file contains customer names, contract terms, and pricing. Can it be pasted into a public AI tool if names are removed?

Answer: No, not unless policy and an approved tool allow it. The file can still contain confidential business and customer information.

4. A laptop with work email is stolen from a car. What matters most?

Answer: Report immediately so IT can revoke access, locate, lock, wipe, or investigate according to policy.

5. A user notices files renamed with strange extensions and a ransom note. What should they do?

Answer: Stop using the device, disconnect if instructed by policy or IT, preserve evidence, and report immediately.

6. A coworker is downloading a large amount of sensitive data unrelated to their role. What should you do?

Answer: Report observable behavior through the appropriate channel. Do not confront or accuse the coworker.

7. A user wants to send a confidential spreadsheet externally. What should happen first?

Answer: Confirm business need and recipient, apply the correct label/protection, and use approved sharing.

8. An email link displays a familiar brand but points to a misspelled domain. What should the user do?

Answer: Do not click. Report the message and access the service only through a known legitimate URL if needed.

9. A user accidentally sends sensitive data to the wrong external recipient. What should they do?

Answer: Report immediately with facts about what was sent, to whom, and when. Do not hide the mistake.

10. A browser warns that a site is malicious. What is the best response?

Answer: Do not bypass the warning. Close it and report if it came from work communication.

## Last-Minute Memory Hooks

- Money, identity, access, and sensitive data always require verification.
- Urgency plus secrecy is a social engineering smell.
- Unexpected MFA means possible credential compromise.
- Sensitive data should stay in approved tools with labels, access control, and retention.
- Business users report; security teams investigate.
- Preserve evidence; do not delete, forward broadly, or keep poking at the suspicious item.
- Public Wi-Fi, personal storage, and unmanaged AI tools are risky sharing paths.
- Ransomware response starts with stopping spread and reporting, not negotiating.

## Quick Review Checklist

Before exam day, make sure you can explain:

- Why cybersecurity is a shared responsibility.
- How MFA helps and why phishing-resistant MFA is stronger.
- Why password reuse is dangerous and how password managers help.
- What sensitive data should not be shared with AI tools.
- How to recognize phishing, pretexting, baiting, spoofing, and malicious links.
- Common malware and insider risk indicators.
- Why public Wi-Fi and remote work increase risk.
- What least privilege, Conditional Access, sensitivity labels, and rights management do.
- How data should be collected, used, shared, stored, retained, and destroyed.
- Why backups matter for recovery.
- When to report incidents and what details to include.
- What immediate steps to take during a suspected data breach or ransomware event.

## Official Microsoft References

These are the official references used to build the guide. Read them for source detail, but use the sections above to practice how the exam expects you to reason.

- [Study guide for Exam SC-730: Cybersecurity Business Professional](https://learn.microsoft.com/credentials/certifications/resources/study-guides/sc-730)
- [Microsoft Cybersecurity Defense Operations Center](https://learn.microsoft.com/security/engineering/fy18-strategy-brief)
- [Microsoft Entra authentication overview](https://learn.microsoft.com/entra/identity/authentication/overview-authentication)
- [Microsoft Purview data security solutions](https://learn.microsoft.com/purview/purview-security)
- [Learn about sensitivity labels](https://learn.microsoft.com/purview/sensitivity-labels)
- [Microsoft Edge password manager security](https://learn.microsoft.com/deployedge/microsoft-edge-security-password-manager-security)
- [Password policy recommendations for Microsoft 365 passwords](https://learn.microsoft.com/microsoft-365/admin/misc/password-policy-recommendations)
- [Protect users against phishing and other attacks in Microsoft 365 for business](https://learn.microsoft.com/microsoft-365/admin/security-and-compliance/m365b-users-phishing-spam-malware)
- [Prevent malware infection](https://learn.microsoft.com/defender-endpoint/malware/prevent-malware-infection)
- [How to protect against phishing attacks](https://learn.microsoft.com/defender-endpoint/malware/phishing)
- [Microsoft Purview data security and compliance protections for generative AI apps](https://learn.microsoft.com/purview/ai-microsoft-purview)
- [Incident response playbooks](https://learn.microsoft.com/security/operations/incident-response-playbooks)
- [Microsoft Incident Response ransomware approach and best practices](https://learn.microsoft.com/security/ransomware/incident-response-playbook-dart-ransomware-approach)
