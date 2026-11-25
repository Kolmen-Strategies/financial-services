# Model Builder — managed-agent template

> Verification status (2026-11-25): this is a workflow specification/template. Manifest references passed structural checks; no business-output or hosted-agent acceptance run was performed. Linux deployment dry runs are blocked by the deployment script's GNU `mktemp` incompatibility. Worker schemas/permission descriptions below express intended requirements: `deploy-managed-agent.sh` does not currently wire `validate.py` into worker responses. See the [repository feature audit](../../README.md#feature-verification--2026-11-25) and [roadmap](../../ROADMAP.md).

## Overview

DCF, LBO, 3-statement, comps — built as a file artifact. Same source as the [`model-builder`](../../plugins/agent-plugins/model-builder) Cowork plugin — this directory is the Managed Agent cookbook for `POST /v1/agents`.

## Deploy

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export CAPIQ_MCP_URL=... DALOOPA_MCP_URL=...
../../scripts/deploy-managed-agent.sh model-builder
```

## Steering events

See [`steering-examples.json`](./steering-examples.json).

## Security & handoffs

Task-decomposition split — inputs come from trusted MCPs, so the split is about artifact isolation and re-verification. Exactly one worker holds `Write`:

| Leaf | Tools | Connectors |
|---|---|---|
| `data-puller` | `Read`, `Grep` | CapIQ, Daloopa (read-only) |
| **`builder`** (Write-holder) | `Read`, `Write`, `Edit`, `Bash` (sandboxed) | None |
| `auditor` | `Read`, `Grep` | None |

`auditor` re-checks ties and balances after `builder` writes `./out/model.xlsx`.

**Handoff:** when invoked from `earnings-reviewer` or `pitch-agent`, the calling agent's `handoff_request` is routed here by `scripts/orchestrate.py`.
