# Entra ID + Intune Lab

Cloud identity and device management lab — Entra fundamentals,
device enrollment, and MDM connection, before any Conditional
Access or compliance work.

**Environment**
- Microsoft 365 Business Premium trial tenant (started October 24, 2026)
- Windows 11 Pro test VM in VMware Workstation on a desktop host
  (separate from the Active Directory lab, which runs on a laptop)
- Test user: standard user, no administrator access, licensed for
  Business Premium
- Admin work done from the host browser (Microsoft 365 admin center,
  Entra admin center, Intune admin center); the VM is the managed
  test device only — the host computer itself was not Entra-joined

**What was done and verified**
1. Created a test user in the Microsoft 365 admin center and assigned
   a Business Premium license. The user is a standard user with no
   administrator access.
2. Set up the Windows 11 Pro VM for work or school and signed in with
   the test user, Entra-joining the device to the trial tenant.
   Windows Hello PIN was NOT set up — sign-in is with the test user
   password. A PIN can be added later from Settings > Accounts >
   Sign-in options, but it is not part of this verified work.
3. Verified the join on the device:
   - `whoami` returns `azuread\testuser`
   - `hostname` returns `WIN11-Intune`
   - `dsregcmd /status` shows `AzureAdJoined: YES`,
     `EnterpriseJoined: NO`, `DomainJoined: NO`
4. Verified MDM enrollment in Settings > Accounts > Access work or
   school: the device shows as connected to the tenant's MDM and to
   the tenant's Entra ID, and the device is listed in the Intune
   admin center on the host.

**What is NOT claimed here yet**
Conditional Access policies, compliance policies, configuration
profiles, and app deployment are next steps and are
not claimed as completed until they are actually built and verified
in the tenant. The trial tenant is a lab tenant only, not a
production environment.

**Screenshots (redacted)**
- `screenshots/AdminCenterTestUser.png` — Test user in
  the Microsoft 365 admin center: standard user, no administrator
  access.
- `screenshots/AzureADjoined.png` — `whoami`,
  `hostname`, and `dsregcmd /status` on WIN11-Intune confirming
  Entra Join.
- `screenshots/AccessWorkOrSchoolMDMConnected.png` — Settings >
  Accounts > Access work or school showing the device connected to
  MDM and Entra ID.

**Redaction note**
Tenant domain, admin account addresses, device ID, and certificate
thumbprint are blacked out in all screenshots. No passwords, PINs,
or admin credentials are included anywhere in this folder.

**Next steps**
- Conditional Access policy in report-only mode first (never
  enforce-first), excluding the admin account
- Intune compliance policy and configuration profile for the test
  device
- Win32 app deployment in a phased rollout
