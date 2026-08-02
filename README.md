# Active Directory Home Lab: User Management & Group Policy

## Description

This project simulates a small company's IT environment to practice core Tier-1 help desk and system administration tasks. Using VirtualBox, I built a Windows Server 2022 domain controller and joined a Windows 11 client to the domain, then configured Organizational Units, user accounts, and security groups to mirror a small business (Sales, IT, HR departments). I performed common help desk tasks against this environment — password resets, account lockouts, disabling departed employees, and moving users between departments — and configured domain-wide Group Policy to enforce a modern password and account lockout policy aligned with current NIST SP 800-63B (Revision 4, July 2025) guidance, rather than relying on Windows' outdated default settings.

## Languages and Utilities Used

- VirtualBox (hypervisor)
- Active Directory Domain Services (AD DS)
- Active Directory Users and Computers (ADUC)
- Group Policy Management Console (GPMC) / Group Policy Management Editor
- Windows Command Prompt (`gpupdate`, `net accounts`)
- DNS (Windows Server role, required for domain join)

## Environments Used

- Windows Server 2022 Standard Evaluation (Desktop Experience) — Domain Controller
- Windows 11 Enterprise Evaluation, version 25H2 — Domain-joined client

---

## Program Walk-through

Screenshots are referenced by their uploaded filename so you can match them exactly when assembling the GitHub post.

### 1. Isolated lab network
**Screenshot: `Screenshot_1.png`**
Created a Host-only Network in VirtualBox (`VirtualBox Host-Only Ethernet Adapter #2`, subnet `192.168.50.0/24`) to isolate the lab from the home network while still allowing the Domain Controller and client to communicate.

### 2. Installing Windows Server 2022
**Screenshot: `Screenshot_2_3.png`**
Selected Windows Server 2022 Standard Evaluation (Desktop Experience) for the Domain Controller build.

### 3. Installing the Active Directory Domain Services role
**Screenshot: `Screenshot_3_1.png`**
Installed AD DS along with Group Policy Management and Remote Server Administration Tools (RSAT) via Server Manager's Add Roles and Features wizard.

### 4. Promoting the server to a Domain Controller
**Screenshot: `Screenshot_4_0.png`**
Configured a new forest with root domain `corp.local` using the Active Directory Domain Services Configuration Wizard.

### 5. Domain Controller promotion confirmed
**Screenshot: `Screenshot_5_0.png`**
Logged in as `CORP\Administrator` following the DC promotion and reboot — proof the domain is live.

### 6. Organizational Unit structure
**Screenshot: `Screenshot_6_0.png`**
Created OUs mirroring a small company's departments: IT, Sales, and HR.

### 7. Creating test user accounts
**Screenshot: `Screenshot_7_0.png`**
Created individual test users inside each OU using the New Object - User wizard (e.g., Test User1 in Sales).

### 8. Multiple users per department
**Screenshot: `Screenshot_8_0.png`**
Sales OU populated with three test users (Test User1, Test User2, Test User3).

### 9. Security group with members
**Screenshot: `Screenshot_9_0.png`**
Created the `Sales-Team` security group and added the department's users as members.

### 10. Help desk task: password reset
**Screenshot: `Screenshot_10_0.png`**
Performed a standard password reset via ADUC's Reset Password function.

### 11. Help desk task: account lockout status (baseline)
**Screenshot: `Screenshot_11_0.png`**
Reviewed the Account tab's lockout status field prior to any lockout event, as a baseline for later comparison.

### 12. Help desk task: disabling a departed employee's account
**Screenshot: `Screenshot_12_0.png`**
Disabled Test User3's account to simulate offboarding — note the account icon change in the OU listing.

### 13. Help desk task: moving a user between OUs
**Screenshot: `Screenshot_13_0.png`**
Moved Test User1 from the Sales OU to the IT OU to simulate an internal department transfer.

### 14. Domain-joining the client machine
**Screenshot: `Screenshot_14_0.png`**
Confirmed the Windows 11 client successfully joined `corp.local` via System Properties (Computer Name/Domain Changes).

### 15. Logging in as a domain user
**Screenshot: `Screenshot_15_0.png`**
Logged into the domain-joined client as a `CORP\` domain account rather than a local account, confirming end-to-end domain authentication.

### 16. Department-specific Group Policy Object
**Screenshot: `Screenshot_16_0.png`**
Created and linked `Sales-Desktop-Policy`, a GPO scoped to the Sales OU, to demonstrate a department-specific setting (as opposed to a domain-wide security policy — see next section).

### 17. Domain-wide Password & Account Lockout Policy
**Screenshots: `Screenshot_20_0.png` (Password Policy) and `Screenshot_21_0.png` (Account Lockout Policy)**
Configured Account Policies directly in **Default Domain Policy** — the correct location, since Password/Lockout Policy only takes effect domain-wide when linked at the domain root, not on an OU-linked GPO. Settings were chosen to align with current **NIST SP 800-63B Revision 4** guidance (finalized July 2025) rather than Windows' legacy defaults:

| Setting | Value | Why |
|---|---|---|
| Minimum password length | 14 characters | NIST Rev. 4 recommends 15+ for password-only auth; 14 is the maximum the standard Windows GPO UI supports without enabling an additional "Relax minimum password length limits" setting. Also the current DoD STIG value for Windows Server 2022. |
| Password complexity requirements | Disabled | Rev. 4 explicitly prohibits mandatory character-mixing rules, since they push users toward predictable patterns rather than real entropy. |
| Maximum password age | 0 (never expires) | Rev. 4: periodic rotation isn't required without evidence of compromise. |
| Minimum password age | 1 day | Prevents rapidly cycling through password history to loop back to a previous password. |
| Enforce password history | 24 remembered | Prevents reuse of recent passwords. |
| Account lockout threshold | 5 invalid attempts | Standard brute-force protection. |
| Account lockout duration | 15 minutes | |
| Reset lockout counter after | 15 minutes | |

*Note: Windows' native Group Policy doesn't screen new passwords against breached-password lists, which Rev. 4 also recommends — a production environment would pair this with a third-party tool (e.g., Specops Password Policy, Azure AD Password Protection).*

### 18. Applying the policy to the client
**Screenshot: `Screenshot_18_0.png`**
Ran `gpupdate /force` on the domain-joined client to pull the updated Group Policy.

### 19. Confirming the policy at the domain level
**Screenshot: `Screenshot_22_0.png`**
Ran `net accounts` on the Domain Controller to confirm the effective domain-wide policy matches configuration — minimum password length 14, lockout threshold 5, lockout duration 15 minutes, all correctly enforced.

### 20. Triggering a real account lockout
**Screenshot: `Screenshot_23_0.png`**
Deliberately entered an incorrect password 5+ times for an enabled test account, triggering AD's lockout threshold. The login screen returned: *"The referenced account is currently locked out and may not be logged on to."*

### 21. Confirming the lockout in Active Directory
**Screenshot: `Screenshot_24_0.png`**
Verified the lockout server-side — the Account tab explicitly states the account is currently locked out on the Domain Controller.

### 22. Resolving the lockout
**Screenshot: `Screenshot_25_0.png`**
Resolved the lockout the way a Tier-1 technician would: checked "Unlock account" and applied the change. Confirmed the user could log in again with the correct password.

---

## Notes

- Screenshots showing an earlier, corrected attempt (an initial password policy configured on the Sales-linked GPO, and a first lockout test that ran into a disabled account rather than a true lockout) were left out of this walkthrough in favor of the final, working sequence above, to keep the write-up focused on the correct method.
- All lab data (IP ranges, domain name, usernames) is fictional and scoped to an isolated VirtualBox network — nothing here touches a real production environment.
<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
