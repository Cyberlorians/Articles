# SC-730: Cybersecurity Business Professional Study Guide

_Last updated: May 15, 2026_

This guide is built from the official Microsoft Learn SC-730 study guide and supporting Microsoft documentation. It is written for the exam audience: business professionals who use digital tools, collaboration platforms, cloud services, and connected systems, but who are not expected to be security engineers.

## Exam Snapshot

| Area | Weight | What to Focus On |
| --- | ---: | --- |
| Understand cybersecurity concepts | 25-30% | Shared responsibility, accountability, policies, MFA, password managers, ransomware impact, key terms, deepfakes |
| Understand cybersecurity risks and threats | 30-35% | Public Wi-Fi, phishing, pretexting, baiting, malware indicators, insider threat signals, suspicious emails and links |
| Apply basic security practices to protect the organization | 25-30% | Secure devices/accounts/workspaces, sensitivity labels, rights management, data handling, backups and recovery |
| Report and respond to security incidents | 10-15% | When to report, what details to include, reporting channels, breach actions, escalation triggers |

Passing score: 700 or greater.

## How to Study

Use the exam weights to prioritize your time. The largest domain is cybersecurity risks and threats, so be comfortable recognizing suspicious behavior, social engineering, malware signs, abnormal device behavior, and risky communication requests. The next two large areas are cybersecurity concepts and basic protection practices, so you should know how everyday employee actions support security, privacy, and compliance.

A practical study rhythm:

1. Read the exam objective list once end to end.
2. Study one domain at a time using the notes below.
3. For each objective, ask: What would a business user see? What should they do? What should they avoid? When should they report or escalate?
4. Practice scenario questions, especially emails, unexpected payment requests, lost devices, AI prompts, public Wi-Fi, ransomware, and sensitive data handling.

## Domain 1: Understand Cybersecurity Concepts

### Shared Responsibility

Cybersecurity is shared across the organization. Security teams build policies, tools, monitoring, and response processes, but every employee has responsibilities that reduce risk.

Know these responsibilities:

- Employees follow policy, protect credentials, use approved tools, complete awareness training, and report suspicious activity.
- Managers reinforce accountable behavior, make sure teams know reporting channels, and support policy compliance.
- IT and security teams deploy controls, monitor risk, investigate incidents, and recover services.
- Cloud providers protect their infrastructure and services, while customers and users still protect accounts, data, devices, configuration, and usage.

Exam tip: If a question asks who owns security, avoid answers that put all responsibility on IT. Business users own their behavior, data handling, and reporting obligations.

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

## Domain 2: Understand Cybersecurity Risks and Threats

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

### Backup and Recovery

Backups support recovery from accidental deletion, device loss, corruption, malware, and ransomware. Business users should know where files are supposed to be stored so organizational backup and retention protections apply.

Good practices:

- Save work in approved locations such as OneDrive, SharePoint, Teams, or other governed systems.
- Do not keep the only copy on a local desktop or removable drive.
- Follow recovery instructions from IT.
- Report missing, encrypted, or corrupted files quickly.

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
