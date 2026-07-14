# Troubleshooting — Microsoft Purview Adaptive Protection

This guide covers common Adaptive Protection issues, including the specific failure state directly observed in this lab (Step 1).

---

## Adaptive Protection Disabled

**Symptom:** The Adaptive Protection dashboard displays *"Adaptive Protection is currently turned off. Insider risk levels won't be assigned to users until it's turned back on from settings."*

**Observed in this lab:** Yes — Step 1, `1-Adaptive-Protection-Dashboard.png`.

**Cause:** The **Adaptive Protection** toggle in Adaptive Protection settings is set to Off.

**Resolution:** Navigate to **Adaptive Protection settings** and set the toggle to **On**, then select **Save** (Step 2, `2-Adaptive-Protection-Settings.png`).

> **Warning**
> If Adaptive Protection was previously on and is turned off, it can take **up to 6 hours** for risk level assignment to fully stop and reset, per the on-screen guidance in Step 2. The reverse (turning on) should also be expected to take time to propagate before risk levels populate.

---

## Policy Not Applying

**Symptom:** A Conditional Access or DLP policy references an insider risk level, but no enforcement action is observed for a user who should match that level.

**Cause:** Common causes include: the policy is in Report-only mode (as observed for the Conditional Access policy in this lab, Step 3); Adaptive Protection was recently toggled on and has not finished propagating; or the user has not yet been assigned the referenced risk level.

**Resolution:** Confirm the policy's **Policy status** column (Report-only vs. On) in the Conditional Access (Step 3) or DLP (Step 4) list. Confirm the user currently holds the expected risk level under **Insider risk levels → Users assigned insider risk levels**. Allow sufficient propagation time after any recent Adaptive Protection or policy change.

---

## Insider Risk Levels Missing

**Symptom:** No insider risk levels are defined, or the **Insider risk policy** dropdown under Insider risk levels has no valid selection.

**Observed in this lab:** A related condition — Step 1 shows *"The insider risk policy that was being used for Adaptive Protection was deleted."*

**Cause:** The source Insider Risk Management policy referenced by Adaptive Protection was deleted or deactivated.

**Resolution:** Navigate to **Insider risk levels** and select a valid, active policy from the **Insider risk policy** dropdown (Step 5 shows `Data Leaks Insider Risk Policy` selected as the resolution). Re-define the Elevated, Moderate, and Minor conditions if they were reset.

---

## Conditional Access Not Visible

**Symptom:** No Conditional Access policies appear under Adaptive Protection → Conditional Access, or **Policies using insider risk levels** on the dashboard shows **Not started** for Conditional Access (as observed in Step 1 of this lab).

**Cause:** No Conditional Access policy has yet been created with the "Insider risk" condition, or the signed-in account lacks Conditional Access read permissions.

**Resolution:** Use the **Quick setup** action shown on the dashboard (Step 1) or **Create policy** on the Conditional Access page (Step 3) to create a new policy referencing insider risk. Confirm the account has Security Administrator or Conditional Access Administrator permissions in Microsoft Entra ID.

---

## DLP Policies Missing

**Symptom:** No DLP policies appear under Adaptive Protection → Data Loss Prevention.

**Cause:** No DLP policy has been created with the "Insider risk level for Adaptive Protection is" condition.

**Resolution:** Use **Create policy** on the Data Loss Prevention page (Step 4) to build a new DLP policy that includes this condition, scoped to the required locations (for example, Exchange, Teams, or Devices, as seen in this lab's two existing policies).

---

## Permissions

**Symptom:** Adaptive Protection settings, insider risk levels, or policy lists are not visible or are read-only for the signed-in account.

**Cause:** The signed-in account is not assigned to an Insider Risk Management role group with sufficient permission (for example, Insider Risk Management Admins), or lacks Compliance Administrator / Conditional Access Administrator rights for the respective policy surfaces.

**Resolution:** Assign the account to the appropriate Microsoft Purview role group and/or Microsoft Entra ID role, following least-privilege principles.

---

## Licensing

**Symptom:** Adaptive Protection, Insider Risk Management, or Conditional Access features are unavailable or greyed out.

**Cause:** The tenant or specific users lack the required Microsoft 365 E5 (or equivalent add-on) and Microsoft Entra ID P1/P2 licensing.

**Resolution:** Verify licensing assignment against the requirements listed in [`README.md`](README.md#licensing-requirements). Licensing was not directly visible in the screenshots for this lab and should be independently confirmed.

---

## Synchronization Delays

**Symptom:** A configuration change (enabling Adaptive Protection, updating insider risk levels, modifying a policy) does not appear to take effect immediately.

**Cause:** Adaptive Protection and Insider Risk Management changes are not instantaneous. Per Step 2, disabling Adaptive Protection can take up to 6 hours to fully propagate and reset risk levels.

**Resolution:** Allow adequate time after any configuration change before concluding a setting has failed to apply. Re-verify using both the portal (Steps 3–5) and PowerShell (Steps 6–7) after the expected propagation window.

---

## PowerShell Errors

**Symptom:** `Get-InsiderRiskPolicy` returns an authentication error, empty results, or "command not found."

**Cause:** No active Security & Compliance PowerShell session, insufficient permissions on the connected account, or the `ExchangeOnlineManagement` module is not installed/imported.

**Resolution:**

```powershell
# Install the module if missing
Install-Module -Name ExchangeOnlineManagement -Scope CurrentUser -Force

# Connect to Security & Compliance PowerShell
Connect-IPPSSession -UserPrincipalName <admin-upn>

# Re-run verification
Get-InsiderRiskPolicy | fl Name,Mode,State,Enabled
```

If the command succeeds but returns no policies, confirm Insider Risk Management is provisioned in the tenant and that at least one policy has been created (see [`implementation.md`](implementation.md)).

---

## Summary Table

| Issue | Likely Cause | Resolution Reference |
|---|---|---|
| Adaptive Protection disabled | Toggle set to Off | Step 2 |
| Policy not applying | Report-only mode / propagation delay / risk level not yet assigned | Steps 2–4 |
| Insider risk levels missing | Source policy deleted | Step 5 |
| Conditional Access not visible | No policy created yet / missing permissions | Step 3 |
| DLP policies missing | No policy created yet | Step 4 |
| Permissions | Missing role group / Entra ID role assignment | — |
| Licensing | Missing E5/add-on or Entra ID P1/P2 | README.md |
| Synchronization delays | Up to 6 hours for toggle changes to propagate | Step 2 |
| PowerShell errors | Module not installed, no active session, insufficient rights | Steps 6–7 |
