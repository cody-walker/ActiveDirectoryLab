<h1>Active Directory Home Lab: User Management & Group Policy</h1>

<h2>Description</h2>
This project simulates a small company's IT environment to practice core Tier-1 help desk and system administration tasks. Using VirtualBox, I built a Windows Server 2022 domain controller, joined a Windows 11 client to the domain, and configured Organizational Units, user accounts, and security groups to mirror a small business (Sales, IT, HR). I performed common help desk tasks against this environment: password resets, account lockouts, disabling departed employees, and moving users between departments, and configured domain-wide Group Policy to enforce a modern password and account lockout policy aligned with current NIST SP 800-63B (Revision 4, July 2025) guidance, rather than relying on Windows' outdated default settings. 
<br />


<h2>Languages and Utilities Used</h2>

- <b>VirtualBox (hypervisor)</b> 
- <b>Active Directory Domain Services (AD DS)</b>
- <b>Active Directory Users and Computers (ADUC)</b>
- <b>Group Policy Management Console (GPMC) / Group Policy Management Editor</b>
- <b>Windows Command Prompt (gpupdate, net accounts)</b>
- <b>DNS (Windows Server role, required for domain join)</b>


<h2>Environments Used </h2>

- <b>Windows Server 2022 Standard Evaluation (Desktop Experience) — Domain Controller
- <b>Windows 11 Enterprise Evaluation, version 25H2 — Domain-joined client

<h2>Program walk-through:</h2>

First, we will reformat the external disk to be compatible with the device.

  1. Isolated lab network
     
  8. Click Erase, then click Done. <br/>
<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
