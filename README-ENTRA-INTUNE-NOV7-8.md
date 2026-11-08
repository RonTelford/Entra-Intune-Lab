# Entra ID + Intune Lab — Rebuilt Nov 7–8, 2026

What I did: rebuilt the Entra ID / Intune lab from scratch on a new Windows 11 VM,
after the old lab VM was deleted. Joined it to Entra ID, enrolled it in Intune,
then worked through Conditional Access, a compliance policy, and an app deployment.

Lab tenant: `<tenant>.onmicrosoft.com` (test tenant — do not use real tenant names here)
Test user: `<test-user>@<tenant>.onmicrosoft.com` (standard user, no admin rights)
Admin account: `<admin>@<tenant>.onmicrosoft.com` (used only in the browser, never on the VM)

Trial note: the Business Premium trial for this tenant ends November 23, 2026.
Recurring billing is off, so it just expires. Do this lab before that date.

## Setup

- Host: desktop VMware Workstation
- New VM: `WIN11-Intune`, Windows 11 Pro
  - Windows 11 Pro — Home cannot Entra join. That is why Pro matters.
  - 8 GB RAM, 60 GB disk, NAT
  - Encryption: choose **"Encrypt only the files needed to support a TPM"**
    — that is all Windows 11 needs for the TPM. "Encrypt all files" is slower
    and not needed here. Remember the encryption password — you cannot open
    the VM without it.
  - Boot tip: when the VM starts, click inside the VM window right away and
    press any key as soon as you see "Press any key to boot from CD or DVD".
    You only get a few seconds. Miss it and it tries to boot from the empty
    hard disk and times out.

## Step 1 — Entra join (verified)

On the Win11 VM:

Settings > Accounts > Access work or school > Connect, then sign in with
the test user.

Check it in PowerShell:

```powershell
dsregcmd /status
```

Result:

| Field | Result |
|---|---|
| AzureAdJoined | YES |
| EnterpriseJoined | NO |
| DomainJoined | NO |

`dsregcmd` = Directory Services Registration Command. It is the tool that
checks if a device is joined to Entra ID and to on-prem AD.
EnterpriseJoined: NO is normal here — that is for old on-prem stuff, not
what we are doing.

## Step 2 — Intune / MDM enrollment (verified, after a fix)

After the Entra join, the VM said "connected to Entra, not MDM". Entra join
and MDM enrollment are two separate steps. Joining does not enroll you
by itself.

What was wrong: the MDM user scope was blank. Automatic enrollment was not
turned on for anyone.

Fix, as the admin in the browser:

1. Intune admin center > Devices > Enroll devices > Automatic enrollment
   (also shown as the Entra ID Mobility / MDM page)
2. Set **MDM user scope** to **All**
3. Save

Two traps I hit here:

- I was signed in as the **test user** when I tried to change the scope.
  The test user is a standard user and cannot change it. The scope stayed
  blank until I signed in as the **admin** and set it there.
- `gpupdate /force` does **not** fix group membership or enrollment.
  It is for Group Policy, not for MDM enrollment. Do not use it for this.

Then, on the VM, enroll by hand:

Settings > Accounts > Access work or school > Connect >
**"Enroll only in device management"** > sign in as the test user.

Result: the account now shows **MDM and Entra**, both connected.

Check the device in the admin center: Intune admin center >
Devices > All devices. The VM shows up there and reports **Compliant**.

## Step 3 — MFA

The test user is enrolled in Microsoft Authenticator for MFA.
Do not disable Authenticator from the phone — that is the second factor.
Keep it, especially for the admin account.

## Step 4 — Conditional Access, report-only first (verified)

Policy: **"Test - Require MFA for Testuser"**

- Users: the test user only
- Target resources: All resources (formerly "All cloud apps")
- Grant: Require multifactor authentication
- Enable policy: **Report-only** — never turn it On first

Before this policy will work, **security defaults must be disabled first**.
The portal warns you about this at the bottom of the New policy page.
Click "disable security defaults", set it to Disabled, save, then come back
and create the policy.

Lessons from the Users picker:

- Click **"Select users and groups"** — do not click "All users".
  All users would hit everyone, including the admin, and the portal will
  warn you not to lock yourself out.
- In the picker, check **"Users and groups"** at the bottom.
  Do **not** check "Guest or external users" — the test user is inside
  this tenant, not a guest, so it lives under Users and groups, not Guest.
- Exclude the admin. The portal offers "Exclude current user" — use it.
  The admin is the only way back in if a policy goes wrong.

Report-only means: the policy runs, logs what it *would* have blocked,
and blocks nothing. Check the sign-in logs after a test sign-in to see
what it would have done, before you ever turn it On.

## Step 5 — Compliance policy (verified)

Policy: **"Test_Win11 Compliance"**

- Platform: Windows 10 and later
- Profile type: Windows 10/11 compliance policy
- Compliance settings > System Security:
  - Require Secure Boot: **Require**
  - Require TPM 2.0: **Require**
  - BitLocker and the rest: left as Not configured for this first test
- Actions for noncompliance: Mark device noncompliant, Immediately, 0 days
- Assignments: **All users**

Note: I tried to assign it to the lab group first, but the group did not
show up in the group search (0 results for the lab group name). Do not fight the
search — for a lab with only two users, All users is fine. Do not assign
it under Excluded groups by mistake — the picker for Include and Exclude
looks the same, check the title at the top: "Select groups to include".

Result: Company Portal on the VM says:

> "This device meets [tenant] compliance and security policies.
> You can access resources like company email with this device."

The admin center compliance page lagged behind and showed 0 devices for
a while. That is normal — the device has to check in first. Sync the
device in Company Portal, wait a few minutes, then refresh the page.
The device itself is the source of truth here.

## Step 6 — App deployment (verified)

App: **Windows Terminal**, via Intune, Microsoft Store app (new)

- Intune admin center > Apps > Windows > Add
- App type: Microsoft Store app (new)
- Search the Microsoft Store app (new) > Windows Terminal
- Name: Windows Terminal
- Publisher: Microsoft Corporation
- Package Identifier: `9N0DX20HK701`
- Installer Type: UWP
- Install behavior: User
- Assignments: All users, **Required**
- Create

One blocker: the Privacy URL field came pre-filled as
`go.microsoft.com/fwlink/?LinkID=521839` with no `https://`, and the
form rejected it with "Enter a valid url". Fix: add `https://` to the
front so it reads `https://go.microsoft.com/fwlink/?LinkID=521839`.
Then Next enables.

Result: the app is listed in Intune as Created, Assigned: Yes.
On the VM, Start shows it as **"Terminal"** — Windows shortens the name.
Click it, it opens. That is the install working.

Note: Company Portal Home said "Your IT administrator did not make any
apps available to you." That is normal for a **Required** app — Required
apps install in the background, they do not show up as "available to
install" in Company Portal. Check Start, not the Portal Home page.

## What I verified, in order

| # | Check | Result |
|---|---|---|
| 1 | `dsregcmd /status` | AzureAdJoined: YES, EnterpriseJoined: NO |
| 2 | Access work or school | MDM + Entra, both connected |
| 3 | Intune > Devices > All devices | Device listed, Compliant |
| 4 | Authenticator | Test user enrolled for MFA |
| 5 | Conditional Access policy | Created, Report-only |
| 6 | Company Portal > Device status | "This device meets compliance and security policies" |
| 7 | Windows Terminal on the VM | Opens from Start (shown as "Terminal") |

## Files in this repo

- This README
- `snippets/dsregcmd-check.md` — the join check
- `snippets/entra-mdm-enrollment-steps.md` — the enrollment fix, step by step

No screenshots are included with this write-up. The only photos from
the night were phone photos of a monitor — blurry, angled, with moiré —
and most of them show the tenant or admin address in the header.
Retake screenshots with Win + Shift + S on the VM/host instead, and
redact the tenant domain, admin address, and test user address before
posting. See `screenshots/SCREENSHOT-REVIEW.md` in the posting package
for the full list of what was rejected and why.
