# Pitch Agent — managed-agent template

> Verification status (2026-11-25): this is a workflow specification/template. Manifest references passed structural checks; no business-output or hosted-agent acceptance run was performed. Linux deployment dry runs are blocked by the deployment script's GNU `mktemp` incompatibility. Worker schemas/permission descriptions below express intended requirements: `deploy-managed-agent.sh` does not currently wire `validate.py` into worker responses. See the [repository feature audit](../../README.md#feature-verification--2026-11-25) and [roadmap](../../ROADMAP.md).

## Overview

Comps, precedents, LBO → branded pitch deck, end to end. Same source as the [`pitch-agent`](../../plugins/agent-plugins/pitch-agent) Cowork plugin — this directory is the Managed Agent cookbook for `POST /v1/agents`.

## Deploy

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export CAPIQ_MCP_URL=... DALOOPA_MCP_URL=...
../../scripts/deploy-managed-agent.sh pitch-agent
```

## Steering events

See [`steering-examples.json`](./steering-examples.json).

## Security & handoffs

Task-decomposition split — less about untrusted inputs (data comes from CapIQ/Daloopa MCPs), more about parallelism and artifact isolation. Exactly one worker holds `Write`:

| Leaf | Tools | Connectors |
|---|---|---|
| `researcher` | `Read`, `Grep` | CapIQ, Daloopa (read-only) |
| `modeler` | `Read`, `Bash` (sandboxed) | CapIQ, Daloopa (read-only) |
| **`deck-writer`** (Write-holder) | `Read`, `Write`, `Edit` | None |

Artifacts land in `./out/pitch-<target>.pptx` and `./out/model.xlsx` via `pptx-author` / `xlsx-author`.

**Handoff:** to rebuild the model after a thesis change, the orchestrator emits a `handoff_request` for `model-builder`; `scripts/orchestrate.py` (or your workflow engine) routes it as a new steering event. See the script for the allowlist + payload-validation pattern.
