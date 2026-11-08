# Entra join + Intune / MDM enrollment — steps that worked

Test tenant: `<tenant>.onmicrosoft.com`
Test user: `<test-user>@<tenant>.onmicrosoft.com` (standard user)
Admin: `<admin>@<tenant>.onmicrosoft.com` (browser only, never on the VM)

## 1. Entra join the VM

On the Windows 11 Pro VM:

Settings > Accounts > Access work or school > Connect, then sign in
with the test user.

Verify:

```powershell
dsregcmd /status
```

Want: `AzureAdJoined: YES`.

## 2. If it says Entra but not MDM

Entra join and MDM enrollment are separate. Joining does not enroll
you by itself.

First check the MDM user scope, as the **admin** in the browser:

Intune admin center > Devices > Enroll devices > Automatic enrollment

- **MDM user scope** must be **All** (or a group that includes the test user).
- If it is blank, automatic enrollment is off for everyone, and the
  device will never enroll on its own.

Trap: if you are signed in as the test user, you cannot change this
setting. The test user is a standard user. Sign in as the admin,
set the scope to All, and Save. Check it saved — go back to the page
and confirm it still says All.

## 3. Enroll by hand

On the VM:

Settings > Accounts > Access work or school > Connect >
**"Enroll only in device management"** > sign in as the test user.

Result: the account shows **MDM and Entra**, both connected.

Check in the admin center: Intune admin center > Devices > All devices.
The device should be listed and report Compliant.

## 4. Sync the device

If a new policy or app does not show up yet, sync the device:

- Company Portal > your device > Check access / Sync, or
- Settings > Accounts > Access work or school > click the test user
  account > Info > Sync

Then wait a few minutes and refresh the admin center page. The
admin center lags behind the device — the device itself is the
source of truth. Company Portal saying "This device meets compliance
and security policies" means it is compliant, even if the admin
center still shows 0 devices for a while.

## 5. What does NOT fix enrollment

- `gpupdate /force` — that is for Group Policy, not for MDM
  enrollment or group membership. Do not use it for this.
- Re-signing in as the same user without changing the scope.
- Disabling Authenticator / MFA — keep Authenticator on the phone.
  That is the second factor, and you need it.
