# dsregcmd check

`dsregcmd` = Directory Services Registration Command. It checks if a
device is joined to Entra ID and to on-prem Active Directory.

Run in PowerShell or Command Prompt on the Windows device:

```powershell
dsregcmd /status
```

## What to look for

| Field | What it means |
|---|---|
| `AzureAdJoined: YES` | The device is Entra joined. This is what you want for the Entra lab. |
| `AzureAdJoined: NO` | Not Entra joined yet. |
| `EnterpriseJoined: NO` | Normal in this lab. That is for old on-prem enterprise join, not Entra join. |
| `DomainJoined: NO` | Not joined to an on-prem AD domain. Correct for the Win11 Entra lab VM — it is not a domain member. |
| `DomainJoined: YES` + `AzureAdJoined: YES` | Hybrid joined — joined to both on-prem AD and Entra ID. That is the answer to "how do you check if a device is hybrid-joined?" |

## Results from this lab (Nov 7–8, 2026)

Win11 VM `WIN11-Intune`, after Entra join with the test user:

- AzureAdJoined: **YES**
- EnterpriseJoined: **NO**
- DomainJoined: **NO**

That is Entra joined, not hybrid joined, and not enterprise joined.
