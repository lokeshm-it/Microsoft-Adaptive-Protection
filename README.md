# Project 18 — Microsoft Purview Adaptive Protection

![Microsoft Purview](https://img.shields.io/badge/Microsoft-Purview-0078D4?style=flat-square&logo=microsoft&logoColor=white)
![Insider Risk Management](https://img.shields.io/badge/Insider%20Risk-Management-0078D4?style=flat-square)
![Adaptive Protection](https://img.shields.io/badge/Adaptive-Protection-0078D4?style=flat-square)
![Microsoft Entra ID](https://img.shields.io/badge/Microsoft-Entra%20ID-0078D4?style=flat-square&logo=microsoftazure&logoColor=white)
![Conditional Access](https://img.shields.io/badge/Conditional-Access-0078D4?style=flat-square)
![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?style=flat-square&logo=powershell&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

Part of the **Microsoft Purview Zero Trust Lab Series**. This project documents the configuration and verification of **Microsoft Purview Adaptive Protection**, which connects Insider Risk Management risk levels to Data Loss Prevention (DLP) and Conditional Access enforcement.

> **Note**
> This documentation is generated exclusively from the screenshots in [`images/`](images/). No setting, value, or configuration is described unless it is directly visible in a screenshot.

---

## Project Overview

Microsoft Purview Adaptive Protection dynamically adjusts data protection controls based on a user's assigned **insider risk level** (Elevated, Moderate, Minor). Once a user matches a defined risk level, that risk level can be used as a condition inside a Data Loss Prevention policy or a Conditional Access policy, allowing the DLP or Conditional Access policy to automatically enforce stricter controls against that specific user's activity — without an administrator having to manually adjust scope.

This project walks through the full configuration lifecycle observed in the lab: the initial disabled/broken state, re-enabling Adaptive Protection, defining insider risk levels, reviewing the Conditional Access and DLP policies that consume those risk levels, and validating the deployment with PowerShell.

## Objectives

- Identify and resolve the initial Adaptive Protection outage state (feature disabled, source insider risk policy deleted).
- Re-enable Adaptive Protection at the tenant level.
- Configure insider risk level thresholds (Elevated, Moderate, Minor) sourced from an Insider Risk Management policy.
- Review the Conditional Access policy that consumes Adaptive Protection insider risk levels.
- Review the Data Loss Prevention policies that consume Adaptive Protection insider risk levels.
- Verify the Adaptive Protection policy and all Insider Risk Management policies using PowerShell.

## Architecture

See [`architecture.md`](architecture.md) for the full architecture diagram and component breakdown.

Adaptive Protection sits between Insider Risk Management (which computes a per-user risk level) and two enforcement surfaces: Microsoft Purview DLP and Microsoft Entra Conditional Access. Both enforcement policies reference the insider risk level as a condition rather than duplicating risk logic themselves.

## Prerequisites

| # | Prerequisite | Status Observed in This Lab |
|---|---|---|
| 1 | Microsoft Purview compliance portal access | Confirmed |
| 2 | An active Insider Risk Management policy to source risk levels from | Confirmed — `Data Leaks Insider Risk Policy` |
| 3 | Microsoft Entra Conditional Access licensing/access | Confirmed — one Conditional Access policy referencing insider risk is present |
| 4 | Microsoft Purview DLP policies configured | Confirmed — two DLP policies reference Adaptive Protection |
| 5 | Exchange Online / Security & Compliance PowerShell for verification | Confirmed — used in Steps 6–7 |

> **Warning**
> The lab's initial state (Step 1) shows Adaptive Protection **turned off** and a message that the insider risk policy previously used for Adaptive Protection had been **deleted**. This is a real, observed failure state and is documented as the starting point of this project, not a hypothetical.

## Licensing Requirements

> Licensing tiers were not visible in the screenshots. The table below reflects Microsoft's published guidance and should be independently verified before production planning.

| License | Requirement |
|---|---|
| Microsoft 365 E5 / A5 / G5, or the Insider Risk Management / Compliance add-on on E3 | Required for Insider Risk Management and Adaptive Protection |
| Microsoft Entra ID P1 or P2 | Required for Conditional Access policy enforcement |
| Microsoft Purview DLP (included in relevant E5/compliance licensing) | Required for DLP policy enforcement |

## Required Roles

| Role | Purpose |
|---|---|
| Insider Risk Management Admin | Configure Adaptive Protection settings and insider risk levels |
| Compliance Administrator | Administer Purview DLP policies |
| Conditional Access Administrator / Security Administrator | Create and manage Conditional Access policies referencing insider risk |
| Global Reader (minimum, for verification) | Run read-only PowerShell verification cmdlets |

## Lab Topology

| Component | Value Observed in This Lab |
|---|---|
| Tenant | Microsoft Purview compliance portal (tenant domain not shown in these screenshots) |
| Source Insider Risk Policy | `Data Leaks Insider Risk Policy` |
| Adaptive Protection Policy (system-generated) | `Adaptive Protection policy for Insider Risk Management` |
| Conditional Access Policy | `Block access to Office Apps for users with Insider Risk (Preview)` — Report-only, Elevated |
| DLP Policies | `Adaptive Protection policy for Teams and Exchange DLP` (Exchange, Teams); `Adaptive Protection policy for Endpoint DLP` (Devices) |

## Implementation Steps

Full step-by-step detail with screenshots is documented in [`implementation.md`](implementation.md).

| Step | Title | Image |
|---|---|---|
| 1 | Review the Adaptive Protection dashboard (initial disabled state) | `1-Adaptive-Protection-Dashboard.png` |
| 2 | Enable Adaptive Protection in settings | `2-Adaptive-Protection-Settings.png` |
| 3 | Review Conditional Access policies using insider risk | `3-Adaptive-Protection-Conditional-Access-Policies.png` |
| 4 | Review Data Loss Prevention policies using insider risk | `4-Adaptive-Protection-DLP-Policies.png` |
| 5 | Configure insider risk levels (Elevated, Moderate, Minor) | `5-Adaptive-Protection-Insider-Risk-Levels.png` |
| 6 | Verify the Adaptive Protection policy using PowerShell | `6-Verify-Adaptive-Protection-Policy-PowerShell.png` |
| 7 | Verify all Insider Risk Management policies using PowerShell | `7-Verify-Insider-Risk-Policies-PowerShell.png` |

## Verification

Full verification detail is documented in [`verification.md`](verification.md). Verification was performed at two layers:

1. **Portal verification** — Conditional Access and DLP policy lists confirmed to reference Adaptive Protection insider risk levels (Steps 3–4).
2. **PowerShell verification** — `Get-InsiderRiskPolicy` confirmed the Adaptive Protection policy is `Enabled: True` with `HealthStatus: Healthy`, and that all three related Insider Risk Management policies (`Adaptive Protection policy for Insider Risk Management`, `IRM_Tenant_Setting_...`, `Data Leaks Insider Risk Policy`) are in `Mode: Enable` / `Enabled: True`.

## PowerShell Verification

```powershell
# Confirm the Adaptive Protection policy is enabled and healthy
Get-InsiderRiskPolicy | Where-Object {$_.Name -like "*Adaptive*"} | fl Name,Enabled,PolicyHealth

# Confirm the mode, state, and enabled status of every Insider Risk Management policy
Get-InsiderRiskPolicy | fl Name,Mode,State,Enabled
```

See [`verification.md`](verification.md) for expected output and full explanation of each field.

## Troubleshooting

See [`troubleshooting.md`](troubleshooting.md) for the full table of common issues, causes, and resolutions, including the exact failure state observed in Step 1 of this lab (Adaptive Protection disabled due to a deleted source insider risk policy).

## Security Best Practices

- Apply the principle of least privilege when assigning Insider Risk Management, Compliance Administrator, and Conditional Access Administrator roles.
- Deploy new Conditional Access policies that reference insider risk in **Report-only** mode first, as observed in this lab (`Block access to Office Apps for users with Insider Risk (Preview)` is currently Report-only), before switching to enforced.
- Pilot Adaptive Protection against a dedicated test user group before expanding scope.
- Regularly review the Microsoft Purview audit log for changes to Adaptive Protection settings, insider risk levels, and the policies that consume them.
- Monitor the health of the Adaptive Protection policy using `PolicyHealth` output (`HealthStatus`, `UnhealthyCount`, `RecommendReviewCount`) as part of routine operations.
- Always validate policy configuration and health with PowerShell in addition to the portal UI, since PowerShell output can reveal system-generated policies (for example, `IRM_Tenant_Setting_...`) not otherwise surfaced in the portal.

## Cleanup

> **Note**
> No cleanup actions (policy deletion, disabling Adaptive Protection, or removing test users) were captured in the available screenshots. If this lab is torn down, the following should be considered, based on standard Microsoft Purview administration practice:

- Set the Conditional Access policy `Block access to Office Apps for users with Insider Risk (Preview)` back to **Report-only** or disable it if it was enforced during testing.
- Disable or remove the DLP policies `Adaptive Protection policy for Teams and Exchange DLP` and `Adaptive Protection policy for Endpoint DLP` if they were created solely for this lab.
- Turn off Adaptive Protection in **Adaptive Protection settings** if the lab environment is being decommissioned. Note that turning off Adaptive Protection can take up to 6 hours to fully stop assigning and reset risk levels, per the on-screen guidance captured in Step 2.
- Remove any dedicated test security groups used to scope the lab.

## Learning Outcomes

After completing this project, you will be able to:

- Explain how Microsoft Purview Adaptive Protection connects Insider Risk Management risk levels to DLP and Conditional Access enforcement.
- Diagnose and recover from an Adaptive Protection outage caused by a deleted source insider risk policy.
- Configure Elevated, Moderate, and Minor insider risk level thresholds.
- Identify which Conditional Access and DLP policies are consuming Adaptive Protection risk levels.
- Verify Adaptive Protection and Insider Risk Management policy health using PowerShell.

## References

See [`references.md`](references.md) for the full list of Microsoft Learn documentation links.

---

## Repository Structure

```
Project-18-Adaptive-Protection/
├── README.md
├── implementation.md
├── verification.md
├── troubleshooting.md
├── references.md
├── architecture.md
├── project.json
├── LICENSE
├── .gitignore
└── images/
    ├── 1-Adaptive-Protection-Dashboard.png
    ├── 2-Adaptive-Protection-Settings.png
    ├── 3-Adaptive-Protection-Conditional-Access-Policies.png
    ├── 4-Adaptive-Protection-DLP-Policies.png
    ├── 5-Adaptive-Protection-Insider-Risk-Levels.png
    ├── 6-Verify-Adaptive-Protection-Policy-PowerShell.png
    └── 7-Verify-Insider-Risk-Policies-PowerShell.png
```

---

*Documentation authored strictly from the seven screenshots in `images/`. No configuration is described that is not directly visible in the referenced image.*


## Disclaimer

This repository documents a Microsoft Purview Adaptive Protection lab configuration for portfolio and educational purposes. Screenshots reference a non-production sandbox tenant. No production customer data, credentials, or secrets are included in this repository.
