# Compliance Dashboard

A subscription-owner-facing Azure Policy compliance dashboard, built as an Azure Monitor Workbook backed by Azure Resource Graph (ARG) and Policy Insights.

## What it does

- **Scopes automatically to each viewer's own subscriptions** via their own delegated Azure RBAC — no per-owner configuration needed.
- **Overview tab**: a top-line count of non-compliant resources, split by urgency (Critical/Warning), plus a bar chart breakdown — the most important number first, no pie/donut/gauge charts.
- **My Non-Compliant Resources tab**: every non-compliant finding, sorted most-urgent-first, with a plain-language **Guidance** column explaining what to do — not just the raw policy name.
- **Urgency classification**: findings are split into two tiers — **Critical** (`Deny` / `DeployIfNotExists` / `Modify` — blocking or requires a remediation task) and **Warning** (`Audit` / `AuditIfNotExists` — observational only) — rendered with the Workbook's native icon-and-color Thresholds renderer, so status is never conveyed by color alone.
- **Subscription filter**: a multi-select parameter for owners who hold more than one subscription.

## How it works

The Workbook queries `PolicyResources` (`microsoft.policyinsights/policystates`) for non-compliant findings, joined against the relevant policy assignments' `nonComplianceMessages` to attach human-authored guidance text instead of raw policy IDs. The assignment lookup runs at tenant-wide Resource Graph scope so that guidance resolves correctly regardless of whether a policy was assigned at the subscription or at a parent management group — Resource Graph's own access control still limits each viewer to what they're actually permitted to see, so this doesn't widen anyone's visibility.

## Repository structure

```
workbook/
  compliance-workbook.json      # Raw Workbook definition (paste into the Azure portal's Advanced Editor)
  compliance-workbook.arm.json  # ARM template wrapper for scripted/CI deployment
```

## Deploying

```bash
az deployment group create \
  --resource-group <your-resource-group> \
  --template-file workbook/compliance-workbook.arm.json \
  --parameters workbookDisplayName="Subscription Compliance Workbook"
```

To update an existing deployment in place rather than create a new one, also pass `--parameters workbookId=<existing-guid>`.

Alternatively, open `workbook/compliance-workbook.json` in the Azure portal (Azure Monitor → Workbooks → New → Advanced Editor → Gallery Template / paste) to preview or edit interactively before deploying.

## Prerequisites

- `nonComplianceMessages` authored on the relevant policy assignments — the Workbook will still function without them, but findings will show a generic fallback message instead of specific remediation guidance.
- Reader access (or above) on the target subscription(s) for anyone viewing the Workbook.

## Status

Early-stage / pilot. Validated against a single-subscription environment; behavior at full multi-subscription tenant scale, and true cross-subscription visibility isolation between different owners, has not yet been independently verified.
